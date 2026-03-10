# Repository Analysis — `ph-ee-bulk-processor`

> **Status:** 🔲 Analyzed — Migration NOT started  
> **Estimated Effort:** 🟡 MEDIUM (8-12 days)

---

## Repository Overview

| Attribute | Value |
|---|---|
| **Purpose** | Bulk payment processing engine — batch transaction files |
| **Type** | Spring Boot application (bootable) |
| **Gradle Wrapper** | 7.3 |
| **Spring Boot** | 2.6.2 |
| **sourceCompatibility** | `'17'` |

---

## Project Structure Analysis

| Package | Files | Purpose | Migration Impact |
|---|---|---|---|
| `bulk/api/` | 6 | REST controllers | 🔴 `javax.servlet` |
| `bulk/camel/routes/` | 5 | RouteBuilder classes | 🟠 Camel 4 |
| `bulk/camel/config/` | 1 | SSL trust-all config | 🟢 `javax.net.ssl` = JDK |
| `bulk/zeebe/` | 2 | ZeebeWorkers, ProcessStarter | 🟡 Zeebe alignment |
| `bulk/zeebe/worker/` | 8 | Workers (Splitting, Formatting, etc.) | 🟡 `javax.annotation`, `DefaultExchange` |
| `bulk/config/` | 4 | Configuration validators | 🔴 `javax.annotation.PostConstruct` |
| `bulk/connectors/` | varies | External API integration | 🟠 `exchange.getIn()` |
| `bulk/file/` | varies | File parsing/storage (S3, Azure) | 🟢 Low risk |
| `bulk/kafka/` | varies | Kafka producers/consumers | 🟢 Low risk |

---

## Jakarta Migration Impact — 11 files, 18 javax imports

| javax namespace | Files | Migration |
|---|---|---|
| `javax.servlet.*` | 4 | → `jakarta.servlet` |
| `javax.annotation.PostConstruct` | 6 | → `jakarta.annotation` |
| `javax.net.ssl.*` | 2 | 🟢 NO CHANGE — JDK native |

**No Swagger-generated stubs** — this makes the jakarta migration straightforward compared to channel and ams-mifos.

---

## Dependency List

| Dependency | Version | Risk |
|---|---|---|
| `com.amazonaws:aws-java-sdk` | 1.11.486 | 🔴 Very old (2019) — should be AWS SDK v2 |
| `com.amazonaws:aws-java-sdk-s3` | 1.11.486 | 🔴 Very old |
| `com.amazonaws:aws-java-sdk-dynamodb` | 1.11.486 | 🔴 Marked "To be removed" |
| `com.azure:azure-storage-blob` | 12.12.0 | 🟡 Update to 12.25+ |
| `org.apache.tika:tika-core` | 1.4 | 🔴 Ancient (2013) — many CVEs |
| `org.apache.commons:commons-io` | 1.3.2 | 🔴 Ancient + conflicts with `commons-io:2.11.0` |
| `io.cucumber:cucumber-*` | 7.8.1 | 🟡 Test scope — update to 7.18+ |
| `io.rest-assured:rest-assured` | 4.4.0 | 🟡 Update to 5.x for Jakarta |
| Mixed Camel versions | 3.4.0 | 🔴 Version inconsistency |
| Mixed Spring Boot starters | 2.5.2/2.6.2 | 🔴 Version inconsistency |

---

## Build System Anti-Patterns

- Duplicate `commons-io` (`1.3.2` and `2.11.0` on classpath)
- Mixed Camel/Spring Boot versions
- `setCanBeResolved(true)` on `implementation`
- Duplicate `apply plugin` statements
- Spotless configuration issues

---

## Modernization Challenges

| Challenge | Difficulty | Notes |
|---|---|---|
| AWS SDK 1.11 → v2 | 🔴 Hard | Separate effort, major API change |
| Apache Tika 1.4 → 2.x | 🟡 Medium | HIGH CVEs, must upgrade |
| Duplicate commons-io | 🟢 Easy | Remove 1.3.2, keep 2.18+ |
| javax.annotation (6 files) | 🟢 Easy | Simple find-replace |
| Camel 4 upgrade | 🟡 Medium | Drop-in based on common testing |
| Kafka Spring Boot 3 | 🟢 Low | BOM-managed, auto-compatible |
| Cucumber test framework | 🟢 Low | Compatible with Spring Boot 3 |

---

## Integration with connector-common

```groovy
implementation 'org.mifos:ph-ee-connector-common:1.8.1-SNAPSHOT'
```

**⚠️ CRITICAL:** Uses `@EnableJsonWebSignature` which triggers `@ComponentScan` + `@Import` of connector-common's interceptor package. If connector-common migrates to Jakarta while this repo still has javax on classpath → `ClassNotFoundException` at startup.

**Classes consumed:**
- `interceptor.annotation.EnableJsonWebSignature` — **annotation import** (pulls entire interceptor package)
- `channel.dto.TransactionChannelRequestDTO` — DTO with `jakarta.validation`
- `gsma.dto.*` — pure POJOs (safe)
- `mojaloop.dto.*` — DTOs with `jakarta.validation`

---

## Recommended Migration Sequence

This repo is the recommended **second** target (after connector-common) because:
1. Medium difficulty — no generated stubs
2. Good validation target for BOM consumption
3. Tests Kafka + S3/Azure integration with Jakarta
4. Validates `@EnableJsonWebSignature` cross-repo annotation import

