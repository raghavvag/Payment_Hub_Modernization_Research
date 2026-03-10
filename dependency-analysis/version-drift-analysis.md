# Version Drift Analysis

> **Scope:** 4 core Payment Hub EE repositories  
> **Date:** March 2026

---

## Cross-Repository Version Matrix

| Dependency | common (migrated) | ams-mifos | channel | bulk-processor | Drift? |
|---|---|---|---|---|---|
| Spring Boot | **3.2.5** | 2.6.2 | 2.6.2 | 2.6.2 | 🔴 Major |
| Apache Camel | **4.4.0** | 3.4.0 | 3.12.0 (mixed!) | 3.4.0 (mixed!) | 🔴 Major |
| Zeebe Client | **8.5.0** | 8.1.23 | 8.1.1 | 8.1.23 | 🟠 Minor |
| Lombok | **1.18.36** (BOM) | 1.18.24 | 1.18.22/24 (mismatch!) | 1.18.24 | 🟡 Patch |
| Jackson | **2.15.4** (BOM) | 2.6.0 | 2.13.1 | 2.12.3 | 🔴 Major |
| org.json | **20240303** | 20210307 | 20211205 | 20210307 | 🟠 Major |
| commons-io | **2.18.0** | 2.11.0 | 2.11.0 | 2.11.0 + 1.3.2! | 🟠 Minor |
| Netty | **4.1.109** (BOM) | via Zeebe | via Zeebe | via Zeebe | 🔴 Major |
| javax.servlet | → **jakarta 6.0** | 3.1.0 (stubs) | 3.1.0 (explicit) | via Spring | 🔴 Namespace |
| hibernate-validator | **8.0.1** (jakarta) | via validation-api | via Spring | via Spring | 🔴 Namespace |
| Namespace | **jakarta** | javax | javax | javax | 🔴 Incompatible |

---

## Gradle Build System Drift

| Attribute | common | ams-mifos | channel | bulk-processor |
|---|---|---|---|---|
| **Gradle Wrapper** | **9.0** | 7.3 | 7.3 | 7.3 |
| **Spotless Plugin** | **7.0.2** | 6.19.0 | 6.19.0 | 6.19.0 |
| **JFrog Artifactory** | **5.2.5** | Not used | Not used | Not used |
| **ErrorProne Plugin** | Not used | 3.1.0 | 3.1.0 | Not used |
| **`setCanBeResolved(true)`** | ❌ Removed | ✅ Present | ✅ Present | ✅ Present |
| **Duplicate `apply plugin`** | ❌ Cleaned | ✅ Present | ✅ Present | ✅ Present |
| **Spotless as impl dep** | ❌ Removed | Not present | ✅ Present | ✅ Present |
| **`sourceCompatibility`** | → toolchain | `VERSION_13` ⚠️ | `VERSION_17` | `'17'` |

---

## Outdated Dependency Identification

### Critical (EOL / Dead Libraries)

| Dependency | Repo | Version | Status |
|---|---|---|---|
| `io.springfox:springfox-*` | ams, channel | 3.0.0 | 🔴 **Dead** — no Spring Boot 3 support |
| `com.sun.xml.ws:jaxws-ri` | ams | 2.3.2 | 🔴 **Dead** — javax JAXB |
| `org.apache.tika:tika-core` | bulk | 1.4 | 🔴 **Ancient (2013)** — HIGH CVEs |
| `com.amazonaws:aws-java-sdk` | bulk | 1.11.486 | 🔴 **Ancient (2019)** — no module support |
| `org.apache.commons:commons-io` | bulk | 1.3.2 | 🔴 **Ancient** — conflicts with 2.11.0 |

### Outdated (Behind by Major Versions)

| Dependency | Repo | Current | Latest | Gap |
|---|---|---|---|---|
| Spring Boot | all 3 downstream | 2.6.2 | 3.4.x | 2 major |
| Apache CXF | ams | 3.2.5 | 4.x | 1 major |
| Jackson | ams | 2.6.0 | 2.18.x | 12 minor |
| org.json | all downstream | 20190722-20211205 | 20240303 | 2-5 years |
| commons-io | all downstream | 2.11.0 | 2.18.x | 7 minor |

### Version Inconsistencies Within Repos

| Issue | Repo | Details |
|---|---|---|
| Mixed Camel versions | channel | `3.12.0` and `3.4.0` coexist |
| Mixed Spring Boot starters | bulk | `2.5.2` and `2.6.2` coexist |
| Lombok scope mismatch | channel | `implementation '1.18.22'` vs `annotationProcessor '1.18.24'` |
| Duplicate commons-io | bulk | `1.3.2` and `2.11.0` on classpath |

---

## Impact of Version Drift

1. **Security:** 17+ known CVEs across current dependency versions
2. **Compatibility:** javax/jakarta namespace split makes repos incompatible
3. **Maintenance:** Each repo has different versions of the same dependencies
4. **Reproducibility:** Builds may produce different results on different machines
5. **Onboarding:** New developers must understand per-repo version differences

---

## BOM as Version Drift Solution

With `mifos-platform-bom` governing versions:
- **All 4 repos get identical dependency versions** on next build
- **Zero per-repo version hunting** — update BOM once, all repos align
- **Version inconsistencies eliminated** — BOM enforces single version
- **CVE mitigation centralized** — bump in BOM, all repos patched

