# Technical Risks Assessment

> **Date:** March 2026

---

## Risk Matrix

| # | Risk | Probability | Impact | Severity | Status |
|---|---|---|---|---|---|
| R1 | Netty/gRPC/Zeebe version triangle | High | Critical | 🔴 | ✅ **Mitigated** — BOM alignment resolves |
| R2 | Spring Boot 3 cascades to all 40+ repos | Certain | Critical | 🔴 | ⚠️ Known — requires coordinated migration |
| R3 | connector-common namespace change blocks ecosystem | Certain | Critical | 🔴 | ⚠️ Known — BOM + versioned release strategy |
| R4 | Gradle 9 plugin incompatibility | Certain | High | 🔴 | ✅ **Mitigated** — all plugins upgraded |
| R5 | CXF 3.x → 4.x API break (ams-mifos) | High | High | 🔴 | 🔲 Not yet tested |
| R6 | GSMA stub regeneration (channel, 55+ files) | High | High | 🟠 | 🔲 Not yet tested |
| R7 | Camel 4 API changes | High | Medium | 🟠 | ✅ **Mitigated** — zero breakpoints found |
| R8 | Auth0 JWT v4 API break | Medium | High | 🟠 | ✅ **Mitigated** — backward compatible |
| R9 | javax.crypto accidentally migrated | Low | Critical | 🟡 | ✅ **Mitigated** — verification step in place |
| R10 | Test suite doesn't catch runtime issues | Medium | High | 🟠 | ⚠️ Known — minimal test coverage |
| R11 | Springfox incompatible with Boot 3 | Certain | Medium | 🟠 | 🔲 Must replace with SpringDoc |
| R12 | AWS SDK v1 EOL (bulk-processor) | Low | Medium | 🟡 | 🔲 Separate effort |
| R13 | Apache Tika 1.4 CVEs (bulk-processor) | Medium | Medium | 🟡 | 🔲 Must upgrade to 2.x |
| R14 | Mixed Camel versions in repos | Medium | Medium | 🟡 | 🔲 BOM will enforce single version |
| R15 | Wrapper upgrade is one-way door | Certain | Medium | 🟡 | ✅ **Mitigated** — manual revert documented |

---

## Critical Risks Explained

### R2 — Big Bang Constraint

When connector-common publishes with `jakarta.*` namespace, all downstream repos **must** migrate simultaneously. They cannot consume the new connector-common with `javax.*` on their classpath.

**Mitigation:**
1. Publish connector-common as new major version (e.g., `2.0.0-SNAPSHOT`)
2. Downstream repos upgrade connector-common dependency AND migrate javax→jakarta in same PR
3. BOM ensures version alignment

### R5 — CXF Migration (ams-mifos)

Apache CXF 3.2.5 uses `javax.ws.rs` and is deeply integrated into ams-mifos for Fineract REST API calls. CXF 4.x requires `jakarta.ws.rs` and has API changes. Additionally, `camel-cxf` module names changed in Camel 4.

**Mitigation options:**
1. Upgrade CXF 3.2.5 → 4.x + `camel-cxf-spring-rest` (complex but complete)
2. Replace CXF with Spring RestClient (simpler but requires refactoring)
3. Evaluate if CXF can be eliminated entirely

### R6 — GSMA Stub Regeneration (channel)

55+ Swagger-generated files in `gsmastub/` package all contain `javax.validation` imports. Two approaches:

| Approach | Effort | Risk |
|---|---|---|
| Regenerate with Jakarta codegen | 3-5 days | May change API contract |
| Bulk find-replace | 1-2 days | May miss edge cases |

### R10 — Insufficient Test Coverage

The connector-common repo has only integration tests (requiring external Mojaloop infrastructure), no standalone unit tests. This means:
- Runtime compatibility issues may not be caught until deployment
- Mixed javax/jakarta state compiled successfully despite being runtime-incorrect
- Camel route behavior changes would only surface in integration testing

---

## Risk Trend

| Phase | Risks Remaining |
|---|---|
| Phase 0 (baseline) | 15 risks |
| Phase 3 (Gradle 9 analysis) | 13 risks (R1, R4 confirmed but not yet mitigated) |
| Phase 6 (Spring Boot 3) | 10 risks (R7, R9, R15 mitigated) |
| Phase 8 (Zeebe alignment) | 7 risks (R1, R4, R8 mitigated) |
| Phase 10 (BOM) | 5 risks (R3 partially mitigated, R14 mitigated) |
| Current | **5 open risks** (R2, R5, R6, R10, R11 — all downstream) |

