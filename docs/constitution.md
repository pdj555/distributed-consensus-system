# 🏛️ CODEBASE CONSTITUTION

## The DNA of Distributed Consensus Excellence

---

> *"Simplicity is the ultimate sophistication. Details are not details. They make the design."*  
> *— Steve Jobs*

This constitution defines the immutable principles, standards, and practices that govern every line of code, every decision, and every contribution to this distributed consensus system. It is the definitive guide for AI agents and human developers working within this codebase.

---

## 📋 PROJECT DOCUMENTATION REGISTRY

### Core Documentation

- **📐 Design Blueprint**: [`docs/design.md`](design.md) - Complete 7-phase implementation plan with measurable criteria
- **🎯 Issue Backlog**: [`docs/issues.md`](issues.md) - 45+ ready-to-implement GitHub issues mapped to design phases
- **🏛️ Constitution**: [`docs/constitution.md`](constitution.md) - This document - the DNA of the codebase
- **📖 README**: [`README.md`](../README.md) - Project overview and quick start

### Architecture Decision Records (ADRs)

- **📁 Location**: `/docs/adr/` - Living architecture decisions with rationale
- **🎯 Scope**: High-level architecture, tooling, protocol decisions
- **📝 Format**: Standard ADR template with context, decision, status, consequences

---

## 🎯 PROJECT MISSION & PRINCIPLES

### Core Mission

Build a **production-grade, fault-tolerant Raft consensus implementation** that achieves:

- **> 95% cluster availability** under network partitions
- **< 150ms p99 commit latency** for 1KB entries at 1,000 ops/s
- **Zero-downtime leader transitions**
- **Military-grade reliability** with comprehensive testing

### Fundamental Principles

#### 1. 🏆 **EXCELLENCE IS NON-NEGOTIABLE**

- Every component must meet or exceed the performance targets defined in [`docs/design.md`](design.md)
- Code quality gates are **mandatory** - build fails on any static analysis warnings
- Two-reviewer approval required for all changes
- "Silicon Valley finish" - demo-ready at all times

#### 2. 🔧 **SIMPLICITY THROUGH SOPHISTICATION**

- Choose the simplest solution that meets requirements
- Prefer composition over inheritance
- Single responsibility principle at all levels
- Clear separation of concerns across modules

#### 3. 📐 **ORGANIZATION IS FOUNDATIONAL**

- Follow the exact module structure: `raft-core`, `raft-transport`, `raft-storage`, `raft-cli`, `raft-integtest`
- Consistent naming conventions across all components
- API contracts defined first, implementation second
- Documentation-driven development

#### 4. 🚀 **FUTURE-FORWARD TECHNOLOGY**

- JDK 21 LTS with virtual threads for performance
- Modern build toolchain (Maven 3.9.x)
- Comprehensive observability (Prometheus metrics, structured JSON logs)
- Container-native deployment (distroless images, Kubernetes manifests)

#### 5. 🛡️ **SAFETY ABOVE ALL**

- Raft safety properties are **inviolable** (§5.4 guarantees)
- Comprehensive test coverage: unit, property-based, integration, chaos
- Crash-recovery scenarios tested continuously
- Performance budgets enforced via JMH

---

## 🛠️ TECHNICAL ARCHITECTURE

### Technology Stack

```yaml
Runtime: JDK 21 LTS (virtual threads, ZGC)
Build: Maven 3.9.x
Network: Netty 4 (boss/worker groups)
Persistence: Memory-mapped files, RocksDB snapshots
Concurrency: Single-threaded RaftCore + Disruptor ring buffer
Protocol: Custom binary frames over TCP, Protobuf serialization
Quality: SpotBugs + Checkstyle + ErrorProne
Testing: JUnit 5 + AssertJ + jqwik + TestContainers
Observability: SLF4J 2 + Logback JSON + Micrometer
```

### Module Architecture

```markdown
raft-parent/
├── raft-core/      # Pure Raft algorithm (no I/O)
├── raft-transport/ # Netty networking + message codecs  
├── raft-storage/   # Log segments + snapshot management
├── raft-cli/       # Admin tools + stress testing
└── raft-integtest/ # End-to-end wire-level tests
```

### Performance Requirements

- **Throughput**: 1,000+ operations/second sustained
- **Latency**: p99 < 150ms for 1KB entries
- **Availability**: > 95% during network partitions
- **Recovery**: < 100ms segment index rebuild for 1M entries
- **Memory**: JVM heap stays < 2GB under normal load

---

## 🤖 AI AGENT GUIDELINES

### What AI Agents CAN Do

#### ✅ **ENCOURAGED ACTIONS**

- **Implement features** following the exact phase plan in [`docs/design.md`](design.md)
- **Write comprehensive tests** for every new component
- **Add observability** (metrics, structured logs) to new code
- **Optimize performance** while maintaining correctness
- **Fix bugs** with root-cause analysis and prevention
- **Improve documentation** with concrete examples
- **Refactor code** to improve clarity and maintainability
- **Add safety checks** and error handling
- **Create integration tests** for failure scenarios

#### ✅ **REQUIRED PRACTICES**

- **Follow TDD**: Tests first, implementation second
- **Maintain API contracts**: Never break public interfaces without migration
- **Add comprehensive Javadoc** for all public APIs
- **Use structured logging** with node ID, term, and context
- **Implement metrics** for all user-facing operations
- **Add proper exception handling** with specific error types
- **Write atomic commits** with clear, descriptive messages

### What AI Agents CANNOT Do

#### ❌ **FORBIDDEN ACTIONS**

- **Break Raft safety properties** - Election Safety, Log Matching, Leader Completeness, State Machine Safety
- **Skip testing requirements** - All code must have unit + integration tests
- **Bypass code quality gates** - SpotBugs, Checkstyle, ErrorProne must pass
- **Introduce performance regressions** > 5% without explicit approval
- **Add new dependencies** without updating the architecture decision record
- **Implement features** not in the approved design document
- **Remove or weaken error handling** without strong justification
- **Use deprecated APIs** or outdated patterns

#### ⚠️ **REQUIRES EXPLICIT APPROVAL**

- **Change module boundaries** or major architectural decisions
- **Modify wire protocol** or serialization format
- **Alter performance targets** or acceptance criteria
- **Add new technology dependencies** not in the approved stack
- **Change build configuration** or CI/CD pipeline
- **Modify deployment patterns** or operational procedures

### AI Agent Decision Framework

When implementing features, AI agents must follow this decision hierarchy:

1. **🔍 Check Design Document**: Is this feature planned in [`docs/design.md`](design.md)?
2. **📋 Review Issues**: Is there a corresponding GitHub issue with acceptance criteria?
3. **🏗️ Architecture**: Does this fit the established module structure?
4. **🧪 Testing**: What tests are needed for safety and performance?
5. **📊 Observability**: What metrics and logs should be added?
6. **📚 Documentation**: How should this be documented for users?

---

## 🎯 QUALITY STANDARDS

### Code Quality Gates

#### Static Analysis (MANDATORY)

```bash
mvn spotbugs:check     # Zero HIGH findings
mvn checkstyle:check   # Zero violations
mvn compile            # Zero ErrorProne errors
```

#### Test Coverage (MINIMUM REQUIREMENTS)

- **Unit tests**: > 90% line coverage for `raft-core`
- **Integration tests**: All RPC interactions tested
- **Property-based tests**: Raft invariants verified with jqwik
- **Chaos tests**: Network partitions, node failures, clock skew

#### Performance Benchmarks (ENFORCED)

```bash
mvn jmh:run            # All benchmarks within 5% of baseline
```

### Documentation Standards

#### API Documentation

- **Public interfaces**: Complete Javadoc with examples
- **Thread safety**: Explicitly documented for all public APIs
- **Error conditions**: All possible exceptions documented
- **Usage patterns**: Code examples for complex APIs

#### Architecture Documentation

- **ADRs**: All significant decisions recorded with rationale
- **Runbooks**: Operational procedures for production scenarios
- **Design docs**: Living documents updated with implementation learnings

---

## 🔄 DEVELOPMENT WORKFLOW

### Commit Standards

```markdown
type(scope): description

- feat(core): implement leader election timeout randomization
- fix(storage): prevent corruption during concurrent segment writes  
- test(transport): add chaos test for network partitions
- docs(adr): document persistence strategy decision
- perf(core): optimize AppendEntries batching algorithm
```

### Pull Request Requirements

1. **📝 Linked Issue**: Every PR must reference a GitHub issue
2. **🧪 Tests Pass**: All quality gates must be green
3. **📊 Performance**: JMH benchmarks show no regression
4. **👥 Reviews**: Two approved reviews required
5. **📚 Documentation**: Updated for user-facing changes

### Branch Protection Rules

- **`main` branch**: Protected, requires PR + reviews
- **Feature branches**: Named `feature/issue-123-description`
- **Hotfix branches**: Named `hotfix/critical-issue-description`

---

## 🚨 EMERGENCY PROCEDURES

### Critical Bug Response

1. **🔴 Immediate**: Stop all non-critical development
2. **🔍 Root Cause**: Mandatory RCA document within 24h
3. **🛡️ Prevention**: Add tests to prevent recurrence
4. **📢 Communication**: Update all stakeholders immediately

### Performance Degradation

1. **📊 Measure**: Quantify the regression with JMH
2. **🎯 Isolate**: Identify the commit introducing the issue
3. **⏪ Revert**: Roll back if fix cannot be delivered in 4 hours
4. **🔧 Fix Forward**: Implement proper solution with tests

---

## 📈 CONTINUOUS IMPROVEMENT

### Regular Reviews

- **📅 Weekly**: Performance metrics review
- **📅 Monthly**: Architecture decision review
- **📅 Quarterly**: Technology stack evaluation

### Success Metrics

- **🎯 Availability**: Track cluster uptime > 95%
- **⚡ Performance**: Monitor p99 latency < 150ms  
- **🐛 Quality**: Zero critical bugs in production
- **🚀 Velocity**: Features delivered per sprint

---

## 🏆 DEFINITION OF DONE

A feature is **DONE** when:

- [ ] **Functionality**: Meets all acceptance criteria from GitHub issue
- [ ] **Safety**: Passes all Raft safety property tests
- [ ] **Performance**: Meets latency and throughput targets
- [ ] **Quality**: Passes all static analysis and test gates
- [ ] **Documentation**: API docs and runbooks updated
- [ ] **Observability**: Appropriate metrics and logging added
- [ ] **Deployment**: Ready for production deployment
- [ ] **Demo**: Can be demonstrated end-to-end

---

## 📜 CONSTITUTION AMENDMENTS

This constitution is a **living document** that evolves with the project. Changes require:

1. **📝 RFC Process**: Proposed changes documented with rationale
2. **👥 Team Consensus**: All core maintainers must approve
3. **📊 Impact Analysis**: Performance and compatibility impact assessed
4. **📚 Documentation**: Constitution updated with change history

---

*This constitution embodies our commitment to building not just working software, but exceptional software that stands the test of time. Every line of code, every design decision, every test case reflects these principles.*

**Remember**: *We are not just building a consensus system - we are crafting a masterpiece of distributed systems engineering.*

---

**Constitution Version**: 1.0  
**Last Updated**: $(date)  
**Next Review**: Quarterly  
**Authority**: Core Maintainers  
**Scope**: All code, documentation, and processes
