# Project Issues

Below is a **ready‑to‑paste backlog** of 45 GitHub issues that map one‑for‑one to the blueprint you asked for.
*Every issue already contains:*

* **Title** – starts with a 🔭 prefix for epic‑level items, 🧩 for stories, 🛠 for tasks/bugs, 📜 for docs/tests.
* **Milestone** – matches the phase in the plan.
* **Labels** – suggested default GitHub labels (add your own colour scheme).
* **Issue body** – exact checklist, acceptance criteria, dependencies, and “definition of done”.
* **Story points** – Fibonacci scale (1, 2, 3, 5, 8).

Copy‑paste each block into *New Issue* and keep the numbering. Update the “Depends on” cross‑links once the issues exist.

---

## MILESTONE : PHASE 0 – PRE‑WORK

**Issue #1**
**Title:** 🔭 Project bootstrap & tooling baseline
**Labels:** `epic`, `infrastructure`, `phase:0`
**Story Points:** 5
**Body:**

* [ ] Decide and document final JDK (21 LTS) and Maven versions.
* [ ] Add SpotBugs, Checkstyle, ErrorProne, SLF4J to `pom.xml` parent.
* [ ] Commit `.editorconfig`, `.gitignore`, GitHub Actions skeleton (JDK 21 + `mvn verify`).
  **Acceptance Criteria:** A green CI build that compiles an empty multi‑module skeleton.
  **Definition of Done:** All reviewers confirm *zero* linter or test failures.

---

## MILESTONE : PHASE 1 – ARCHITECTURE & ADRs

**Issue #2**
**Title:** 🧩 Write ADR‑0001: High‑level architecture decisions
**Labels:** `architecture`, `docs`, `phase:1`
**Points:** 3
**Body:**

* [ ] Cover transport (Netty), concurrency model (virtual threads + Disruptor), persistence strategy (mmap).
* [ ] Commit under `/docs/adr/0001‑high‑level‑decisions.md`.
  **Acceptance:** Document merged, signed off by lead and at least one other senior‑eng.

**Issue #3**
**Title:** 🧩 Define protobuf schemas for Raft RPCs
**Labels:** `protocol`, `phase:1`
**Points:** 3
**Body:**

* [ ] Create `proto/raft.proto` with `RequestVote`, `AppendEntries`, `InstallSnapshot`, `ClientRequest`, `ClientResponse`.
* [ ] Generate Java bindings via `protobuf‑maven‑plugin`.
* [ ] Add generated sources to Git ignore list.
  **Done:** `mvn verify` passes and generated classes are importable from `raft‑core`.

---

## MILESTONE : PHASE 2 – CODE SKELETON

**Issue #4**
**Title:** 🔭 Create multi‑module Maven project layout
**Labels:** `infrastructure`, `phase:2`
**Points:** 5
**Body checklist:**

* [ ] `raft‑parent` POM packs common deps + plugin mgmt.
* [ ] Children: `raft‑core`, `raft‑transport`, `raft‑storage`, `raft‑cli`, `raft‑integtest`.
* [ ] Verify each module has unit‑test placeholder.
  **Done:** `mvn test` in root builds five empty jars.

**Issue #5**
**Title:** 🧩 RaftNode public interface & state machine contract
**Labels:** `api`, `phase:2`
**Points:** 2
**Body:**

* [ ] Add `RaftNode` interface to `raft‑core`.
* [ ] Provide Javadoc explaining threading expectations.
  **Acceptance:** Interface published, no impl yet.

**Issue #6**
**Title:** 🧩 State enum & skeletal state classes (Follower/Candidate/Leader)
**Labels:** `api`, `phase:2`
**Points:** 3
**Depends on:** #5

* [ ] Implement abstract `RaftState` with `onTick()` / `onMessage()` hooks.
* [ ] Add empty concrete subclasses in `raft‑core`.
* [ ] Wire simple state transitions in a test stub.
  **Done:** Compilation & a unit test that flips states manually.

---

## MILESTONE : PHASE 3 – CORE FEATURES

### 3.1 Leader election

**Issue #7**
**Title:** 🧩 Election timer with randomized timeout
**Labels:** `consensus`, `phase:3`, `leader‑election`
**Points:** 2
**Body:**

* [ ] Config range 150–300 ms via `RaftConfig`.
* [ ] Reset on valid AppendEntries or RequestVote.
  **Done:** Unit test asserts timer fires within range.

**Issue #8**
**Title:** 🧩 Implement RequestVote RPC handler
**Labels:** `consensus`, `phase:3`, `leader‑election`
**Points:** 3
**Depends on:** #3, #6

* [ ] Reject outdated term, grant vote once per term (`votedFor`).
* [ ] Persist `currentTerm` + `votedFor` atomically to disk.
  **Acceptance:** Property‑based test: never grants 2 votes in same term.

**Issue #9**
**Title:** 🧩 Candidate vote counting and transition to Leader
**Labels:** `consensus`, `phase:3`, `leader‑election`
**Points:** 2
**Depends on:** #8

* [ ] Track votes via `AtomicInteger votesReceived`.
* [ ] On majority → call `becomeLeader()`, else step down on higher term.
  **Done:** Integration test runs 5 nodes, deterministic seed elects single leader.

### 3.2 Log replication

**Issue #10**
**Title:** 🧩 AppendEntries batching logic
**Labels:** `consensus`, `phase:3`, `log‑replication`
**Points:** 3
**Depends on:** #9

* [ ] Batch up to 16 KB or 50 ms.
* [ ] Unit test ensures batching threshold observed.

**Issue #11**
**Title:** 🛠 Per‑follower in‑flight window & back‑pressure
**Labels:** `performance`, `phase:3`
**Points:** 3
**Body:**

* [ ] Sliding window default 64 KB.
* [ ] If window full, pause new entries for that follower.
  **Done:** Stress test without follower OOM.

**Issue #12**
**Title:** 🧩 Leader commit rule & commitIndex advancement
**Labels:** `consensus`, `phase:3`
**Points:** 3
**Depends on:** #10, #11

* [ ] Majority criterion *plus* leader term check.
* [ ] Notify state machine via Disruptor event.
  **Acceptance:** Jepsen linearizability harness runs 1 000 ops w/o violate.

**Issue #13**
**Title:** 🧩 Client request path (apply + response)
**Labels:** `api`, `phase:3`
**Points:** 3
**Depends on:** #12

* [ ] Implement `RaftNode.apply(byte[] cmd)`.
* [ ] Future completes after commit & state‑machine apply.
  **Done:** Functional test increments counter 10 000 times at p99 < 150 ms.

### 3.3 Persistence

**Issue #14**
**Title:** 🧩 Log segment file format & append API
**Labels:** `storage`, `phase:3`
**Points:** 5
**Body:**

* [ ] Fixed 64 MB segments, `[len][crc][payload]`.
* [ ] Use `FileChannel.map` + CRC32c.
* [ ] Write unit test corrupting last byte; verify truncate.
  **Done:** `LogSegmentTest` green.

**Issue #15**
**Title:** 🧩 Segment index rebuild on startup
**Labels:** `storage`, `phase:3`
**Points:** 3
**Depends on:** #14

* [ ] On open, scan segment, populate offset array.
* [ ] Benchmark < 100 ms for 1 M entries.
  **Done:** JMH baseline recorded.

**Issue #16**
**Title:** 🛠 Crash‑recovery integration test
**Labels:** `test`, `phase:3`
**Points:** 2
**Depends on:** #15

* [ ] Run node, append 100 K entries, kill ‑9, restart, assert log intact.
  **Done:** Part of CI.

### 3.4 Snapshotting

**Issue #17**
**Title:** 🧩 Trigger & build local snapshot
**Labels:** `snapshot`, `phase:3`
**Points:** 3
**Body:**

* [ ] Trigger after 50 K applied or ½ disk quota.
* [ ] Write `[meta.json]+[rocksdb sst]` to temp then move.
  **Done:** Micro‑benchmark: < 1 s for 1 GB state.

**Issue #18**
**Title:** 🧩 InstallSnapshot RPC streaming (leader→follower)
**Labels:** `snapshot`, `phase:3`
**Points:** 3
**Depends on:** #17

* [ ] 64 KB chunked streaming.
* [ ] Follower writes temp file, rename on completion.
  **Done:** Integration test partitions until log diverges, verify catch‑up via snapshot.

### 3.5 Fault‑tolerance hooks

**Issue #19**
**Title:** 🧩 Heartbeat sender & follower election timeout
**Labels:** `fault‑tolerance`, `phase:3`
**Points:** 2
**Body:**

* [ ] Leader empty AppendEntries every 50 ms.
* [ ] Election timeout = 2.5× heartbeat interval.
  **Done:** Unit test ensures no spurious election when leader alive.

**Issue #20**
**Title:** 🛠 Leader step‑down on higher term detection
**Labels:** `fault‑tolerance`, `phase:3`
**Points:** 1
**Depends on:** #19

* [ ] Immediate transition to Follower and flush volatile state.
  **Done:** Chaos test increasing term triggers resign.

---

## MILESTONE : PHASE 4 – TEST & VERIFICATION

**Issue #21**
**Title:** 📜 Property‑based tests for Raft safety invariants
**Labels:** `test`, `phase:4`
**Points:** 5
**Body:**

* [ ] Use jqwik to generate random command sequences, crashes, elections.
* [ ] Assertions: Leader Append‑Only, Log Matching, Election Safety, etc.
  **Done:** 10 000 iterations pass in CI.

**Issue #22**
**Title:** 📜 Integration tests with Toxiproxy network chaos
**Labels:** `test`, `phase:4`
**Points:** 3
**Depends on:** #21

* [ ] Inject 200 ms partitions 2×/min for 20 min.
* [ ] Cluster availability > 95 %.
  **Done:** Gate on PR merge.

**Issue #23**
**Title:** 📜 Jepsen‑style linearizability test harness
**Labels:** `test`, `phase:4`
**Points:** 5
**Body:**

* [ ] Leverage Knossos + Clojure client; counter workload.
* [ ] Scenario scripts under `/jepsen`.
  **Done:** Report uploaded to CI artifacts.

**Issue #24**
**Title:** 🛠 Benchmark via JMH: throughput & latency curves
**Labels:** `performance`, `phase:4`
**Points:** 3
**Body:**

* [ ] 1 KB → 128 KB entries at 1 k→10 k ops/s.
* [ ] Produce flamegraph captures.
  **Done:** README links to HTML charts.

---

## MILESTONE : PHASE 5 – OBSERVABILITY & TOOLING

**Issue #25**
**Title:** 🧩 Micrometer metrics exposition
**Labels:** `observability`, `phase:5`
**Points:** 2
**Body:**

* [ ] Counters: `raft_role`, `raft_term`.
* [ ] Timers: `append_latency_seconds`.
* [ ] Export via Prometheus HTTP.
  **Done:** Curl `/actuator/prometheus` shows metrics.

**Issue #26**
**Title:** 🛠 Structured JSON logging
**Labels:** `observability`, `phase:5`
**Points:** 2
**Body:**

* [ ] Logback JSON encoder with line‑delimited JSON.
* [ ] Fields: `timestamp`, `level`, `nodeId`, `term`, `index`, `msg`.
  **Done:** Kibana dashboard demo.

**Issue #27**
**Title:** 🧩 raft‑cli admin tool
**Labels:** `tooling`, `phase:5`
**Points:** 3
**Body:**

* [ ] Subcommands: `status`, `leader`, `force‑stepdown`.
* [ ] gRPC on admin port 9090.
  **Done:** Manual QA: `raft-cli status node‑1` returns JSON.

---

## MILESTONE : PHASE 6 – DEPLOYMENT & RUNBOOK

**Issue #28**
**Title:** 🧩 Jib container build & distroless image
**Labels:** `deployment`, `phase:6`
**Points:** 3
**Body:**

* [ ] Multiplatform (linux/amd64, linux/arm64).
* [ ] Image size < 150 MB compressed.
  **Done:** Pushed to GHCR.

**Issue #29**
**Title:** 🧩 Kubernetes manifests for 5‑node StatefulSet
**Labels:** `deployment`, `k8s`, `phase:6`
**Points:** 3
**Depends on:** #28

* [ ] `StatefulSet`, `HeadlessService`, `ConfigMap` for JVM opts.
* [ ] PodDisruptionBudget `minAvailable:3`.
  **Done:** `kubectl apply` spins up healthy cluster in KIND.

**Issue #30**
**Title:** 📜 Runbook: common ops procedures
**Labels:** `docs`, `phase:6`
**Points:** 2
**Body:**

* [ ] Adding a node (manual joint‑consensus later).
* [ ] Leader resignation, disaster recovery, snapshot restore.
  **Done:** Markdown under `/docs/runbook.md`.

---

## MILESTONE : PHASE 7 – STRETCH GOALS

**Issue #31**
**Title:** 🔭 Joint consensus (dynamic reconfiguration)
**Labels:** `stretch`, `consensus`, `phase:7`
**Points:** 8
**Body:**

* [ ] Implement Raft §6 jointConsensus.
* [ ] Test adding + removing voters under load.
  **Done:** Jepsen membership churn pass.

**Issue #32**
**Title:** 🧩 TLS + mTLS support
**Labels:** `security`, `phase:7`
**Points:** 2
**Body:**

* [ ] Use Netty `SslContextBuilder`.
* [ ] CLI flag `--tls-ca`, `--cert`, `--key`.
  **Done:** All integ tests pass in both plain & TLS modes.

**Issue #33**
**Title:** 🧩 Linearizable read‑only queries (LeaseRead)
**Labels:** `feature`, `phase:7`
**Points:** 1
**Depends on:** #12

* [ ] Leader verifies still leader via last heartbeat (< electionTimeout/2).
* [ ] Responds without appending log entry.
  **Done:** Latency < 5 ms for read path.

**Issue #34**
**Title:** 🧩 Adaptive timeout based on RTT histogram
**Labels:** `performance`, `phase:7`
**Points:** 2
**Body:**

* [ ] Maintain EWMA of AppendEntries RTT.
* [ ] Election timeout = max(150 ms, 2× RTT p95).
  **Done:** Benchmark shows 20 % lower failover latency on LAN.

---

## CONTINUOUS ENGINEERING PRACTICES (always‑open issues)

**Issue #35**
**Title:** ⚙️ CI gate: block on SpotBugs High & ErrorProne ERROR
**Labels:** `ci`, `quality`
**Points:** 1
**Body:** Self‑explanatory, never close; update build as new rules appear.

**Issue #36**
**Title:** ⚙️ Performance budget guardrail (< 5 % regression)
**Labels:** `ci`, `performance`
**Points:** 1
**Body:**

* [ ] GitHub Action compares JMH baseline vs PR; fails if Δ > 5 %.
  **Done:** Baseline JSON stored as artifact.

**Issue #37**
**Title:** ⚙️ Design doc checklist for every public API change
**Labels:** `process`, `docs`
**Points:** 1
**Body:** Template in `.github/PULL_REQUEST_TEMPLATE.md`.

---

## BUG & TECH‑DEBT PLACEHOLDERS (create as needed)

Create empty shells now so numbers exist; flesh out when items discovered.

* **Issue #38–#45** – 🛠 **Reserved** for bugs/tech‑debt; assign sequentially.

---

### HOW TO IMPORT

1. **Create milestones** `Phase‑0` … `Phase‑7`.
2. **Create labels** exactly as referenced.
3. **Open each issue in order**, paste the block, and save.
4. Once all issues exist, edit the “Depends on” lines to actual `#N` cross‑links.
5. Use **Projects → Table** view; drag epics (#1, #2, #31) to “group by.”

Execute with the same **uncompromising clarity** Steve Jobs demanded from the iPhone team—no shortcuts, no hidden work.
