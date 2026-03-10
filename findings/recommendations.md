# Recommendations

> **Based on:** 11 phases of testing on connector-common + analysis of 3 downstream repos  
> **Date:** March 2026

---

## Recommended Modernization Path

### Immediate Priority (Weeks 1-4)

1. **Publish modernized connector-common as `2.0.0-SNAPSHOT`** — This is the foundation. Without this published artifact, no downstream repo can migrate.

2. **Publish `mifos-platform-bom:1.0.0`** — to Maven Central or JFrog Artifactory. This enables all repos to consume centralized version governance.

3. **Start with bulk-processor** — it's the easiest downstream target (11 javax files, no Swagger stubs, no CXF). It validates the BOM consumption pattern and `@EnableJsonWebSignature` cross-repo annotation import.

### Medium Priority (Weeks 4-9)

4. **Migrate ams-mifos** — hardest repo due to CXF 3.2.5. Evaluate whether to upgrade CXF to 4.x or replace with Spring RestClient. Remove Springfox, add SpringDoc.

5. **Migrate channel** — largest file count. Decision needed on GSMA stub regeneration vs bulk find-replace. Deduplicate mixed Camel versions.

### Long-Term (Weeks 9-13)

6. **Ecosystem validation** — build all repos against published artifacts, integration test, Docker/K8s deployment validation.

7. **Extend BOM to remaining 36+ repos** — publish BOM to Maven Central, provide migration guide for community.

---

## Architectural Recommendations

### 1. Centralized BOM is Non-Optional

With 40+ repos sharing dependencies, version drift is inevitable without governance. The `mifos-platform-bom` should be:
- Published to Maven Central (not just mavenLocal)
- Versioned independently from connector repos
- Updated via CI/CD pipeline with automated testing

### 2. Use Java Toolchain Instead of sourceCompatibility

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```

This is the modern, Gradle 9-compatible way to declare JDK version. It also enables cross-compilation.

### 3. Consider Gradle Convention Plugin

A shared Gradle convention plugin could enforce:
- BOM consumption
- Java toolchain configuration
- Spotless formatting rules
- Checkstyle configuration
- Common test infrastructure

This would reduce per-repo boilerplate from ~50 lines to ~5 lines.

### 4. Establish CI/CD Dependency Scanning

Regular automated dependency scanning (e.g., GitHub Dependabot, Snyk, OWASP Dependency-Check) should be part of the CI pipeline to catch CVEs early.

### 5. Regenerate Swagger Stubs with Jakarta Codegen

For ams-mifos and channel, rather than find-replacing 68+ generated files, regenerate them using a Jakarta-compatible Swagger/OpenAPI codegen. This produces cleaner output and reduces maintenance burden.

---

## Per-Repository Effort Estimates

| Repository | Effort | Key Challenge | BOM Simplifies? |
|---|---|---|---|
| connector-common | ✅ Done | — | ✅ |
| bulk-processor | 8-12 days | AWS SDK, Tika | ✅ |
| ams-mifos | 15-19 days | CXF 4.x, Springfox | ✅ |
| channel | 12-17 days | GSMA stubs (55+ files) | ✅ |
| **Total** | **35-48 days** | | |

---

## What NOT to Do

1. **Don't upgrade everything at once** — sequential, tested phases prevent cascading failures
2. **Don't replace javax.crypto** — it's JDK-native, not Jakarta
3. **Don't skip Gradle 8.12.1** — jumping from 7.3 to 9.0 is too risky
4. **Don't mix javax/jakarta on classpath** — it compiles but fails at runtime
5. **Don't upgrade Netty independently** — it must coordinate with Spring Boot BOM + Zeebe
6. **Don't ignore CXF** — it's the hardest migration point and needs dedicated attention

---

## GSoC Project Success Criteria

A successful GSoC project would deliver:

| Deliverable | Evidence |
|---|---|
| All 4 core repos on JDK 21 | Green builds on JDK 21 |
| All 4 core repos on Spring Boot 3 | javax→jakarta complete |
| All 4 core repos on Gradle 8.12.1+ | Gradle 9 verified |
| Centralized BOM published | Maven artifact |
| Zero CRITICAL/HIGH CVEs | Dependency scan report |
| Migration guide for remaining repos | Documentation |
| Downstream repos can adopt BOM | Working examples |

