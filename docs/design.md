# Five‑Node Raft Cluster Design

---

## PHASE 0 – Pre‑work (≤ 2 days)

1. **Pick exact Java & toolchain versions**

   * JDK 21 LTS (virtual threads) for performance and reduced context‑switch overhead.
   * Maven 3.9.x for build; SpotBugs + Checkstyle + ErrorProne in the default lifecycle.
   * JUnit 5 + AssertJ for tests.
   * SLF4J 2 + Logback (JSON encoder) for logs.

2. **Define “done”**

   * Passes Raft §5.4 safety proofs (Leader Append‑Only, Election Safety, Log Matching, Leader Completeness, State‑Machine Safety).
   * Sustains **> 95 % cluster availability** with 200 ms network partitions injected 2×/min over 24 h (see Phase 4).
   * Achieves **p 99 commit latency < 150 ms** for 1 KB entries at 1 000 ops/s.

---

## PHASE 1 – High‑level architecture (2 days)

| Area                  | Decision                                                                                                                                                                                                       |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Wire protocol**     | Custom length‑prefixed binary frames over TCP                                                                                                                                                                  |
| **Transport**         | Netty 4, one boss group & one worker group per node                                                                                                                                                            |
| **Concurrency model** | One dedicated VirtualThread per inbound connection, plus a single‑threaded **RaftCore** event loop that owns the replicated state machine; communication via Disruptor ring buffer (no locks in the hot path). |
| **Persistence**       | Memory‑mapped log segment files (64 MB), CRC32‑c checksums, fsync at commitIndex advances                                                                                                                      |
| **Snapshotting**      | Copy‑on‑write RocksDB column family; background snapshot builder thread; installSnapshot RPC per Raft §7                                                                                                       |
| **Configuration**     | Static 5‑node cluster; future reconfiguration via joint consensus but out‑of‑scope for v1                                                                                                                      |

Document this in a **living ADR (Architecture Decision Record) repo** under `/docs/adr`.

---

## PHASE 2 – Code skeleton & build (3 days)

1. **Create Maven modules**

   ```markdown
   raft-parent
     ├─ raft-core      (pure algorithm)
     ├─ raft-transport (Netty I/O, message codecs)
     ├─ raft-storage   (log & snapshot)
     ├─ raft-cli       (command‑line admin, stress tools)
     └─ raft-integtest (wire‑level monkey tests)
   ```

2. **Define API contracts (raft-core)**

   ```java
   interface RaftNode {
       void start();     // non‑blocking
       CompletableFuture<ClientResponse> apply(byte[] command);
       CompletableFuture<Void> shutdown();
   }
   ```

3. **Generate protobuf schema** for `RequestVote`, `AppendEntries`, `InstallSnapshot`, `ClientRequest`, `ClientResponse`.  Lock proto files under `/protocol`.

4. **Stub out states**—`Follower`, `Candidate`, `Leader`—implementing a shared `RaftState` interface, each with:

   * `onTick()` – driven by a `ScheduledExecutorService` every 10 ms.
   * `onMessage(RaftMessage msg)`

---

## PHASE 3 – feature work (≈ 20 working days)

### 3.1 Leader Election (4 days)

| Step                     | Key Points                                                            |
| ------------------------ | --------------------------------------------------------------------- |
| Implement election timer | Randomized 150–300 ms; reset on valid AppendEntries or RequestVote    |
| RequestVote RPC handler  | Reject if term < currentTerm; update `votedFor` atomically            |
| Transition logic         | Candidate→Leader on majority votes; Candidate/Follower on higher term |

### 3.2 Log Replication (6 days)

1. **AppendEntries sender (leader)** – batch entries up to 16 KB or 50 ms, whichever first.
2. **Leader back‑pressure** – keep per‑follower in‑flight byte window (64 KB) to avoid overwhelming slow peers.
3. **Commit rule** – update `commitIndex` when a log entry (with current term) is replicated on **M = ⌈N/2⌉** nodes.
4. **ClientRequest path**

   ```text
   Client → Leader       (raft-core)
         → AppendEntries (parallel)
         → commitIndex advanced
         → stateMachine.apply()
         ← ClientResponse
   ```

### 3.3 Persistence (5 days)

| Component          | Details                                                                                        |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| **LogSegment**     | Immutable after closed; entries: `[len][checksum][payload]`                                    |
| **Segment index**  | In‑memory array of entry offsets (relies on mmap for fast reads)                               |
| **RaftStorage**    | APIs: `append(entry)`, `truncateSuffix(idx)`, `entries(lo,hi,maxBytes)`                        |
| **Crash recovery** | Replay segments at startup, rebuild volatile indices, truncate partial last entry via checksum |

### 3.4 Snapshotting (3 days)

* Trigger when log size > ½ disk quota or `(lastApplied - lastSnapshotIndex) > 50 000` entries.
* Snapshot file format: `[metadata.json][compressed RocksDB SST]`.
* During snapshot send: pipeline chunks (64 KB) over InstallSnapshot RPC; follower writes to temp file then atomically renames.

### 3.5 Fault Tolerance hooks (2 days)

1. **Heartbeat monitor**

   * Leader sends empty AppendEntries every 50 ms.
   * Follower election timeout = `2.5 × heartbeatInterval`.
2. **Automatic leader step‑down**

   * If leader receives AppendEntries/RequestVote with higher term ⇒ convert to Follower immediately.

---

## PHASE 4 – Test & verification (10 days)

| Layer          | Tooling                    | Scenarios                                                     |
| -------------- | -------------------------- | ------------------------------------------------------------- |
| Unit           | JUnit 5                    | Term rules, log matching                                      |
| Property‑based | jqwik                      | Never violates Raft safety invariants on randomized histories |
| Integration    | TestContainers + Toxiproxy | Network partition, reorder, delay, drop, half‑open TCP        |
| Chaos          | Jepsen‑style nemesis       | Kill ‑9, disk full, clock skew                                |
| Benchmark      | JMH                        | Throughput & latency curves (1→128 KB commands)               |

Define **“green build”** as *all* above steps passing on CI (GitHub Actions) for PR merge.

---

## PHASE 5 – Observability & tooling (3 days)

1. **Metrics** – Micrometer registry → Prometheus: `raft_role`, `raft_term`, `commit_index`, `append_latency_seconds{quantile}`.
2. **Structured logs** – one JSON line/record, include `nodeId`, `term`, `logIndex`, `caller`.
3. **CLI** – `raft-cli status <node>` prints role, term, matchIndex by querying a gRPC admin port.

---

## PHASE 6 – Deployment & runbook (5 days)

1. **Packaging**

   * Jib to build tiny distroless container images (`java -XX:+UseZGC -jar raft-node.jar`).
2. **Kubernetes manifests**

   * 5 × StatefulSet replicas, each with a unique `RAFT_NODE_ID` & service port.
   * PodDisruptionBudget = `minAvailable: 3`.
3. **Runbook pages**

   * “How to add a voter,” “Forcing leader resignation,” “Disaster recovery from snapshot”.

---

## PHASE 7 – Stretch goals / excellence extras

| Feature                            | Benefit                                   | Effort |
| ---------------------------------- | ----------------------------------------- | ------ |
| **Joint consensus (Raft §6)**      | Dynamic cluster resize with zero downtime | 8 days |
| **TLS + mTLS**                     | On‑the‑wire encryption + identity         | 2 days |
| **Linearizable read‑only queries** | Read without touching disk                | 1 day  |
| **Adaptive timeout**               | Lower latency on low‑RTT networks         | 2 days |

---

## Engineering disciplines to enforce (continuous)

* **TDD & code review** – two reviewers, mandatory design doc for every public API.
* **Static analysis gate** – build fails on new SpotBugs High or ErrorProne `ERROR`.
* **Performance budgets** – every PR must show JMH delta ≤ 5 % regression.
* **“Silicon Valley finish”** – end‑to‑end demo script: kill leader, show automatic failover, keep `curl` loop printing committed counter increments with no visible hiccup.

---

### Suggested day‑by‑day Gantt (first 6 weeks)

```markdown
Week 1: PH0–PH1
Week 2: PH2 + Election
Week 3: Log Replication
Week 4: Persistence
Week 5: Snapshotting + Fault hooks
Week 6: Testing suite + Observability
```

(Reserve two extra weeks for bug burn‑down, stretch goals, doc polish.)

---

#### Final word

Follow the plan **exactly** or edit it deliberately—never drift silently.  Every line above is either a **design contract** or a **measurable exit criterion**.  Treat it with the same fanatic insistence on detail that Jobs demanded of the iPhone’s chamfered edges.
