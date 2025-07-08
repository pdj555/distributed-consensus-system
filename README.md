# 🏛️ Distributed Consensus System

A **production-grade, fault-tolerant Raft consensus implementation** built with Java 21 LTS and designed for excellence.

## 🎯 Project Mission

Build a distributed consensus system that achieves:

- **> 95% cluster availability** under network partitions
- **< 150ms p99 commit latency** for 1KB entries at 1,000 ops/s  
- **Zero-downtime leader transitions**
- **Military-grade reliability** with comprehensive testing

## 🛠️ Technology Stack

### Runtime Requirements

| Component | Version | Rationale |
|-----------|---------|-----------|
| **JDK** | `21 LTS` | Virtual threads for performance, ZGC for low-latency GC |
| **Maven** | `3.9.x` | Modern build toolchain with improved dependency resolution |

### Core Dependencies

- **Transport**: Netty 4 (boss/worker thread groups)
- **Persistence**: Memory-mapped files + RocksDB snapshots  
- **Concurrency**: Single-threaded RaftCore + Disruptor ring buffer
- **Protocol**: Custom binary frames over TCP + Protobuf serialization
- **Testing**: JUnit 5 + AssertJ + jqwik + TestContainers
- **Observability**: SLF4J 2 + Logback JSON + Micrometer

## 🏗️ Architecture

```text
raft-parent/
├── raft-core/      # Pure Raft algorithm (no I/O)
├── raft-transport/ # Netty networking + message codecs  
├── raft-storage/   # Log segments + snapshot management
├── raft-cli/       # Admin tools + stress testing
└── raft-integtest/ # End-to-end wire-level tests
```

## 🚀 Quick Start

### Prerequisites

- JDK 21 LTS installed
- Maven 3.9.x installed

### Build

```bash
# Clone the repository
git clone https://github.com/your-org/distributed-consensus-system.git
cd distributed-consensus-system

# Build all modules
mvn clean verify

# Run tests
mvn test

# Start a 5-node cluster (coming soon)
java -jar raft-cli/target/raft-cli.jar cluster start
```

## 📋 Development

### Code Quality Standards

This project maintains **zero-tolerance** for quality regressions:

```bash
mvn spotbugs:check     # Zero HIGH findings
mvn checkstyle:check   # Zero violations  
mvn compile            # Zero ErrorProne errors
```

### Performance Requirements

- **Throughput**: 1,000+ operations/second sustained
- **Latency**: p99 < 150ms for 1KB entries
- **Availability**: > 95% during network partitions
- **Recovery**: < 100ms segment index rebuild for 1M entries

## 📚 Documentation

- **🏛️ [Constitution](docs/constitution.md)** - Coding standards and principles
- **📐 [Design](docs/design.md)** - Complete 7-phase implementation plan  
- **🎯 [Issues](docs/issues.md)** - 45+ ready-to-implement GitHub issues
- **📁 [ADRs](docs/adr/)** - Architecture Decision Records

## 🤝 Contributing

1. Read the [Constitution](docs/constitution.md) - our immutable principles
2. Follow the [Design Blueprint](docs/design.md) - exact implementation phases
3. Every PR requires 2 reviewer approvals
4. All quality gates must pass (static analysis + tests)

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

*"Simplicity is the ultimate sophistication. Details are not details. They make the design."* — Steve Jobs
