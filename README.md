# Gradle Java Version Upgrade Agent

> A discovery-first AI agent prompt for automating Java version upgrades in any Gradle-based Java service.  
> Reads **your** project and determines what needs changing — nothing is assumed.

---

## 🚀 What It Does

This agent executes a complete, production-safe Java version upgrade (e.g., Java 17 → 21) by working through **11 structured phases**:

| Phase | Type | Description |
|---|---|---|
| 0 | WRITE | Create isolated feature branch |
| 1 | READ | Discover build files, dependencies, CI/CD, container config |
| 2 | READ | Compatibility analysis — every pinned dependency checked |
| 3 | WRITE | Upgrade Gradle wrapper, Java toolchain, JaCoCo, Jib |
| 4 | WRITE | Dependency updates based on Phase 2 findings only |
| 5 | WRITE | Source code fixes (imports, package renames, test mocks) |
| 6 | WRITE | Container base image and JVM flag updates |
| 7 | WRITE | CI/CD pipeline Java version update |
| 8 | VERIFY | Full build validation + PR creation |
| 9 | — | Post-deploy verification checklist |
| 10 | ANALYSIS | Optional Java modernization opportunities |
| 11 | WRITE | Generate `JAVA_UPGRADE_REPORT.md` audit report |

---

## ✅ Key Features

- **Zero assumptions** — reads your actual project files before making any change
- **Business logic untouched** — only build, dependency, and infrastructure changes
- **Discovery-first** — Phase 1 fully reports findings before any writes begin
- **Compatibility verified** — every pinned dependency checked against the target Java version
- **BOM split-version detection** — catches the most common cause of post-upgrade `ClassNotFoundException`
- **Container-aware** — handles Jib, Dockerfile, and JRE path updates
- **CI/CD integration** — updates GitHub Actions workflows
- **Corporate network safe** — detects local JDK first, avoids Foojay auto-download
- **Full audit trail** — generates `JAVA_UPGRADE_REPORT.md` with every change documented
- **Rollback plan included** — documents how to revert if issues arise post-deploy

---

## 📋 Prerequisites

Before running the agent, ensure:

- [ ] A Gradle-based Java project with `build.gradle` or `build.gradle.kts`
- [ ] Gradle wrapper at `gradle/wrapper/gradle-wrapper.properties`
- [ ] Git repository initialized with a remote (`origin`)
- [ ] An LLM with file read/write capability (Claude Sonnet 4.5+, GPT-4o, etc.)
- [ ] JDK `TO_VERSION` installed locally *(optional — CI validates if not available)*

---

## 🔧 Usage

### Step 1 — Provide Input Parameters

When starting the agent, confirm these three values:

| Parameter | Example | Description |
|---|---|---|
| `FROM_VERSION` | `17` | Current Java version |
| `TO_VERSION` | `21` | Target Java version |
| `NEW_BASE_IMAGE` | `registry/java21:tag` | New container base image *(skip if no Jib/Dockerfile)* |

### Step 2 — Run the Agent

Copy the contents of `Gradle-Java-Version-Upgrade-Agent.md` and provide it to your LLM along with access to your repository files.

The agent will:
1. Ask you to confirm the input parameters above
2. Run precondition checks
3. Execute all 11 phases sequentially, pausing at key decision points
4. Create a pull request and generate an upgrade report

### Step 3 — Review the PR

The agent creates a `Java-Version-Upgrade` feature branch and opens a PR targeting your default branch. Review the changes, verify CI passes, and merge.

---

## 📁 File Structure

```
gradle-java-version-upgrade-agent/
├── Gradle-Java-Version-Upgrade-Agent.md   # The agent prompt (primary artifact)
├── README.md                              # This file
└── examples/                             # (Optional) Example upgrade reports
    └── JAVA_UPGRADE_REPORT_example.md
```

---

## 🎯 Supported Project Types

| Feature | Supported |
|---|---|
| Single-module Gradle projects | ✅ |
| Multi-module Gradle projects | ✅ |
| Spring Boot applications | ✅ |
| Kafka Streams services | ✅ |
| AWS SDK v2 projects | ✅ |
| Micrometer / Prometheus metrics | ✅ |
| Jib containerization | ✅ |
| Dockerfile containerization | ✅ |
| GitHub Actions CI/CD | ✅ |
| JUnit 4 / JUnit 5 / Mockito | ✅ |

---

## ⚠️ Known Pitfalls Handled

The agent proactively checks and resolves these common failure modes:

| Pitfall | Description |
|---|---|
| **Gradle cache corruption** | First-run cache lock failure on Windows |
| **Foojay SSL failure** | Corporate network blocks JDK auto-download — uses local JDK instead |
| **BOM version split** | Mixed AWS SDK / Spring / Micrometer sub-module versions causing `ClassNotFoundException` |
| **Missed test imports** | Package renames fixed in `src/main` but not `src/test` |
| **Unmocked real clients** | Test methods that call real external client builders without `MockedStatic` |
| **Hardcoded JRE paths** | Old Java version path in Jib JVM flags not updated for new base image |
| **`-Xdebug` flag** | Deprecated no-op flag removed (logs startup warning in Java 21) |
| **`mockito-inline` artifact** | Removed in Mockito 5 — functionality merged into `mockito-core` |
| **Jackson Afterburner** | Uses `sun.misc.Unsafe` — replaced with Blackbird for Java 9+ |
| **Micrometer package rename** | `io.micrometer.prometheus` → `io.micrometer.prometheusmetrics` in 1.13 |

---

## 📊 Example Output

After a successful run, the agent produces:

- **Feature branch**: `Java-Version-Upgrade`
- **Pull request**: With detailed description of all changes
- **Upgrade report**: `JAVA_UPGRADE_REPORT.md` committed to the branch

### Example Summary Table from a Java 17 → 21 upgrade:

| File | Phase | Change |
|---|---|---|
| `build.gradle` | 3 | Java toolchain → 21; Jib → 3.4.3; JaCoCo → 0.8.12 |
| `gradle/wrapper/gradle-wrapper.properties` | 3 | Gradle 8.2.1 → 8.10.2 |
| `gradle/scripts/dependencies.gradle` | 4 | Afterburner → Blackbird; Micrometer → 1.15.4; AWS SDK BOM → 2.31.78 |
| `gradle/scripts/jib.gradle` | 6 | Removed `-Xdebug` |
| `gradle.properties` | 6 | Base image → `java21:al2023-v0.0` |
| `.github/workflows/test.yml` | 7 | `code-version: "21"` |
| `.github/workflows/dev.yml` | 7 | `code-version: "21"` |
| `src/.../CTIKStreamsConfig.java` | 5 | Micrometer import fix |
| `src/.../BaseIVRIntegrationTest.java` | 5 | Micrometer import fix (test file) |

---

## 🔄 Rollback Plan

If issues arise after deploying:

**Container deployments (< 5 minutes):**
1. Identify the last known-good image tag from your deployment system
2. Revert the deployed image tag to that value

**Build pipeline rollback:**
1. Revert `TO_VERSION` → `FROM_VERSION` in the build workflow file
2. Push to trigger a new `FROM_VERSION` build

---

## 🧪 Post-Deploy Verification

After deploying to dev/test, confirm:

- [ ] Startup log shows correct Java version: `Starting App using Java 21.x.x`
- [ ] No `InaccessibleObjectException` in logs
- [ ] No `ClassNotFoundException` or `NoClassDefFoundError` at startup
- [ ] Health check endpoint returns `UP`
- [ ] Metrics/monitoring operational
- [ ] CI pipeline passes on the feature branch

---

## 🤖 Compatible LLMs

The agent prompt is designed for LLMs with:
- File read/write tool access to the repository
- Web search or `fetch_webpage` for dependency changelog lookups
- Terminal/shell command execution capability

| LLM | Compatibility |
|---|---|
| Claude Sonnet 4.5+ (GitHub Copilot) | ✅ Recommended |
| Claude Opus 4 | ✅ |
| GPT-4o with tools | ✅ |
| Gemini 1.5 Pro with tools | ✅ |

---

## 📄 License

[Choose: MIT / Apache 2.0 / your organization's preferred license]

---

## 🤝 Contributing

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-improvement`
3. Make your changes to `Gradle-Java-Version-Upgrade-Agent.md`
4. Test against a real Gradle project
5. Submit a pull request with a description of what you improved and why

---

## 📬 Feedback

If you encounter a project pattern this agent doesn't handle, please open an issue with:
- The dependency or configuration pattern that was missed
- The Java version combination (`FROM` → `TO`)
- The error or unexpected behavior observed

---

_This agent prompt is fully discovery-driven. It reads your specific project first and applies only what is needed — no library names or file structures are assumed in advance._
