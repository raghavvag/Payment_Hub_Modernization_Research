# Repository Analysis — `ph-ee-connector-channel`

> **Status:** 🔲 Analyzed — Migration NOT started  
> **Estimated Effort:** 🔴 HIGH (12-17 days)

---

## Repository Overview

| Attribute | Value |
|---|---|
| **Purpose** | Channel connector — REST API gateway for payment initiation |
| **Type** | Spring Boot application (bootable) |
| **Gradle Wrapper** | 7.3 |
| **Spring Boot** | 2.6.2 |
| **sourceCompatibility** | `JavaVersion.VERSION_17` |

---

## Project Structure Analysis

| Package | Files | Purpose | Migration Impact |
|---|---|---|---|
| `channel/api/` | 4 | REST controllers | 🟡 `DefaultExchange` usage |
| `channel/camel/routes/` | 4 | Route builders | 🟠 Heavy `exchange.getIn()` |
| `channel/zeebe/` | 3 | ZeebeClient config, Workers | 🟡 Zeebe alignment |
| `channel/gsma_api/` | 5 | GSMA API controllers | 🟡 `javax.servlet`, Camel Exchange |
| `channel/interceptor/` | 1 | IdInterceptor | 🔴 `javax.servlet` |
| `channel/utils/` | 2 | AMSUtils, SpringWrapperUtil | 🟡 `javax.annotation`, `DefaultExchange` |
| `gsmastub/api/` | ~15 | Swagger-generated GSMA stubs | 🔴 **MASSIVE:** `javax.servlet`, `javax.validation`, `javax.xml.bind` |
| `gsmastub/model/` | ~40 | Swagger-generated GSMA models | 🔴 **MASSIVE:** `javax.validation` (every model) |

---

## Jakarta Migration Impact — 55+ files, 150+ javax imports ⚠️

| javax namespace | Files | Import Count |
|---|---|---|
| `javax.servlet.*` | 15 | ~20 |
| `javax.validation.*` | ~40 | ~120+ |
| `javax.xml.bind.annotation.*` | 1 | 2 |
| `javax.annotation.PostConstruct` | 2 | 2 |
| `javax.net.ssl.*` | 1 | 🟢 NO CHANGE |

**The GSMA stub package is the largest migration surface in the entire ecosystem.**

### Decision Required: Regenerate vs Find-Replace

| Approach | Pros | Cons |
|---|---|---|
| **Regenerate stubs** | Clean, Jakarta-native output | Requires swagger-codegen 7.x, may change API |
| **Bulk find-replace** | Simple, preserves existing behavior | Technical debt, may miss edge cases |

---

## Dependency List

| Dependency | Version | Risk |
|---|---|---|
| `com.google.code.gson:gson` | 2.8.9 | 🟡 DoS CVE (CVE-2022-25647) |
| `org.apache.camel:camel-jetty` | 3.12.0 | 🟠 Jetty version alignment |
| `org.apache.camel:camel-spring-redis` | 3.12.0 | 🟠 Module name change |
| `io.springfox:springfox-swagger-ui` | 3.0.0 | 🔴 Dead library |
| `org.springdoc:springdoc-openapi-ui` | 1.6.11 | 🟠 Need starter-webmvc-ui for Boot 3 |
| `org.projectlombok:lombok` (implementation!) | 1.18.22 | 🔴 Wrong scope + mismatched with annotationProcessor (1.18.24) |
| `com.diffplug.spotless` (implementation!) | 2.4.1 | 🔴 Build plugin as runtime dep |

---

## Build System Anti-Patterns

- **Mixed Camel versions** — `3.12.0` and `3.4.0` coexist
- **Lombok version mismatch** — `implementation '1.18.22'` vs `annotationProcessor '1.18.24'`
- **Spotless as runtime dependency** — ships build plugin in published JAR
- **Dual API documentation** — both Springfox and SpringDoc present
- `setCanBeResolved(true)` on `implementation`
- Duplicate `apply plugin` statements

---

## Modernization Challenges

| Challenge | Difficulty | Notes |
|---|---|---|
| GSMA stub package (55+ files) | 🔴 **Hard** | Decision: regenerate or bulk find-replace |
| Springfox + SpringDoc coexistence | 🟠 Medium | Remove Springfox, upgrade SpringDoc to 2.x |
| Mixed Camel versions | 🟡 Medium | Consolidate to single version via BOM |
| Lombok version mismatch | 🟢 Easy | Fix scope + align version |
| Redis Spring Boot 3 | 🟢 Low | BOM-managed, auto-compatible |

---

## Integration with connector-common

```groovy
implementation 'org.mifos:ph-ee-connector-common:1.4.1-SNAPSHOT'
```

**⚠️ CRITICAL:** Channel uses `ErrorHandlerRouteBuilder` via **INHERITANCE** — the deepest integration point in the ecosystem. If Camel 4 changes this base class API, channel breaks.

**Classes consumed:**
- `camel.ErrorHandlerRouteBuilder` — **extends** (tight coupling)
- `camel.AuthProcessor` — route processor injection
- `camel.AuthProperties` — configuration
- `channel.dto.*` — DTOs with `jakarta.validation`
- `interceptor.*` — JWS interceptor with `jakarta.servlet`

