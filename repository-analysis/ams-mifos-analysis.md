# Repository Analysis — `ph-ee-connector-ams-mifos`

> **Status:** 🔲 Analyzed — Migration NOT started  
> **Estimated Effort:** 🔴 HIGH (15-19 days)

---

## Repository Overview

| Attribute | Value |
|---|---|
| **Purpose** | Integrates Payment Hub EE with Mifos/Fineract AMS backend |
| **Type** | Spring Boot application (bootable) |
| **Gradle Wrapper** | 7.3 |
| **Spring Boot** | 2.6.2 |
| **sourceCompatibility** | `JavaVersion.VERSION_13` ⚠️ (unusual) |

---

## Project Structure Analysis

| Package | Files | Purpose | Migration Impact |
|---|---|---|---|
| `ams/camel/` | 2 | SSL config, CXF trust manager | 🟢 `javax.net.ssl` = JDK native |
| `ams/interop/` | 7 | Camel RouteBuilders, Processors | 🟠 Camel 4, `exchange.getIn()` |
| `ams/zeebe/` | 4 | ZeebeClient config, Workers | 🟡 Zeebe alignment |
| `ams/tenant/` | 2 | Multi-tenancy service | 🔴 `javax.ws.rs`, `javax.annotation` |
| `fineractstub/api/` | 7 | Swagger-generated stubs | 🔴 `javax.servlet`, `javax.validation`, `javax.xml.bind` |
| `fineractstub/model/` | 5+ | Generated model classes | 🟡 `javax.validation` |

---

## Dependency List

| Dependency | Version | Risk |
|---|---|---|
| `com.sun.xml.ws:jaxws-ri` | 2.3.2 | 🔴 Dead — javax JAXB |
| `org.apache.cxf:cxf-rt-rs-client` | 3.2.5 | 🔴 Very old — CXF 4.x needed |
| `org.apache.cxf:cxf-rt-frontend-jaxrs` | 3.2.5 | 🔴 Uses `javax.ws.rs` |
| `io.springfox:springfox-oas` | 3.0.0 | 🔴 Dead library — replace with SpringDoc |
| `io.springfox:springfox-swagger-ui` | 3.0.0 | 🔴 Dead library |
| `javax.validation:validation-api` | any | 🔴 Dead namespace |
| `org.springframework:spring-web` | 3.0.2.RELEASE | 🔴 Ancient (2013!) |
| `io.camunda:zeebe-client-java` | 8.1.23 | 🟡 Behind |
| `camel-cxf` | 3.4.0 | 🔴 Heavy Jakarta impact |

---

## Jakarta Migration Impact — 13 files, 27 javax imports

| javax namespace | Files | Migration |
|---|---|---|
| `javax.servlet.*` | 4 | → `jakarta.servlet` |
| `javax.validation.*` | 5 | → `jakarta.validation` |
| `javax.ws.rs.core.HttpHeaders` | 1 | → `jakarta.ws.rs` (requires CXF 4.x) |
| `javax.xml.bind.annotation.*` | 1 | → `jakarta.xml.bind` |
| `javax.annotation.PostConstruct` | 1 | → `jakarta.annotation` |
| `javax.net.ssl.*` | 2 | 🟢 NO CHANGE — JDK native |

---

## Build System Anti-Patterns

- `sourceCompatibility = VERSION_13` — targeting JDK 13 bytecode is unusual
- `setCanBeResolved(true)` on `implementation` — Gradle 9 hard error
- Duplicate `apply plugin` statements
- ErrorProne plugin 3.1.0 — Gradle 9 compatibility unknown

---

## Modernization Challenges

| Challenge | Difficulty | Notes |
|---|---|---|
| CXF 3.2.5 → 4.x | 🔴 **Hard** | Uses `javax.ws.rs`, requires major API migration |
| Springfox → SpringDoc | 🟠 Medium | Dead library, must replace entirely |
| JAXB annotations in stubs | 🟠 Medium | Need regeneration with Jakarta codegen |
| fineractstub regeneration | 🟡 Medium | Swagger-generated code needs Jakarta |
| Camel CXF module restructuring | 🔴 **Hard** | `camel-cxf` → `camel-cxf-spring-rest` |

---

## Integration with connector-common

```groovy
implementation 'org.mifos:ph-ee-connector-common:1.5.1-SNAPSHOT'
```

**Classes consumed from connector-common:**
- `ams.dto.*` — DTOs (safe, Lombok-only)
- `channel.dto.TransactionChannelRequestDTO` — uses `jakarta.validation` after migration
- `mojaloop.dto.*` — uses `jakarta.validation` after migration
- `gsma.dto.*` — pure POJOs (safe)

**⚠️ When connector-common migrates to jakarta, this repo MUST also migrate simultaneously** — validation DTOs will fail at runtime otherwise.

