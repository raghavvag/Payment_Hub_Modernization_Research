# Repository Analysis — `ph-ee-connector-common`

> **Status:** ✅ FULLY MIGRATED (11 phases, 15 breakpoints documented)

---

## Repository Overview

| Attribute | Value |
|---|---|
| **Purpose** | Shared foundation library for the entire Payment Hub EE ecosystem |
| **Type** | Library JAR — not a runnable application |
| **Consumers** | 40+ downstream `ph-ee-*` microservices |
| **Java files** | ~187 |
| **Source compatibility** | Java 17 → 21 |

**This is the most critical repository in the ecosystem.** Every change here cascades to all downstream repos.

---

## Project Structure Analysis

```
org.mifos.connector.common (187 Java files)
├── ams/dto/           (~20 files)  Fineract AMS data models — 🟢 Zero impact
├── batch/dto/         (2 files)    Batch authorization — 🟢 Zero impact
├── camel/             (8 files)    Apache Camel integration — 🟠 Camel 4 review
├── channel/dto/       (6 files)    Channel API DTOs — 🔴 javax.validation → jakarta
├── exception/         (5 files)    Error framework — 🟢 Zero impact
├── gsma/dto/          (~25 files)  GSMA Mobile Money — 🟢 Zero impact
├── identityaccountmapper/ (6 files) Account lookup — 🟢 Zero impact
├── interceptor/       (12 files)   JWS HTTP interceptor — 🔴 javax.servlet → jakarta
├── mojaloop/          (~20 files)  Mojaloop protocol — 🟡 javax.validation
├── operations/        (3 files)    Transfer entity — 🟢 Zero impact
├── util/              (7 files)    Crypto (javax.crypto = JDK native) — 🟢 Zero impact
├── validation/        (3 files)    Custom validators — 🟢 Zero impact
├── vouchers/dto/      (10 files)   Voucher management — 🟢 Zero impact
├── webclient/         (1 file)     WebFlux adapter — 🟢 Zero impact
└── zeebe/             (1 file)     ZeebeVariables constants — 🟢 Zero impact
```

---

## Dependency List (Original → Migrated)

| Dependency | Original | Migrated |
|---|---|---|
| Spring Boot | 2.6.2 | **3.2.5** |
| Camel | 3.4.0 | **4.4.0** |
| Zeebe | 8.1.1 | **8.5.0** |
| Lombok | 1.18.24 | **1.18.36** |
| Netty | 4.1.68 (pinned) | **4.1.109** (BOM) |
| Auth0 JWT | 3.10.2 | **4.4.0** |
| javax.servlet-api | 3.1.0 | → **jakarta 6.0.0** |
| hibernate-validator | 6.0.20.Final | **8.0.1.Final** |
| org.json | 20190722 | **20240303** |
| commons-io | 2.11.0 | **2.18.0** |
| jackson-datatype-jsr310 | 2.10.1 | **2.15.4** (BOM) |

---

## Build System Configuration

| Attribute | Original | Migrated |
|---|---|---|
| Gradle Wrapper | 7.3 | **9.0** (8.12.1 stable) |
| JFrog Artifactory | 4.24.23 | **5.2.5** |
| Spotless | 6.19.0 | **7.0.2** |
| dep-management | implicit apply | **1.1.7** (plugins block) |
| Source compatibility | `VERSION_17` (project prop) | **Java toolchain 21** |
| BOM | None | **`mifos-platform-bom:1.0.0`** |

---

## Plugin Compatibility

| Plugin | Original | Gradle 9 Compatible? | Action Taken |
|---|---|---|---|
| `com.jfrog.artifactory` | 4.24.23 | ❌ Uses `convention` API | → 5.2.5 |
| `com.diffplug.spotless` | 6.19.0 | ❌ `LenientConfiguration` | → 7.0.2 |
| `org.springframework.boot` | 2.6.2 | ❌ `getUploadTaskName()` | → 3.2.5 |
| `io.spring.dependency-management` | implicit | ⚠️ Imperative apply | → 1.1.7 declarative |

---

## Modernization Challenges (All Resolved)

| Challenge | Resolution |
|---|---|
| Lombok crashes on JDK 21 | Upgraded to 1.18.34+ |
| Gradle 7.3 incompatible with JDK 21 | Upgraded to 8.12.1 |
| javax → jakarta (14 files) | Simple find-replace |
| Netty version conflict | Removed pin, let BOM manage |
| JAXB warnings from Zeebe transitives | Zeebe 8.5.0 cleaned this |
| bootJar fails (no main class) | `bootJar { enabled = false }` |
| Gradle 9 breakpoints (5) | All resolved via plugin + script fixes |

