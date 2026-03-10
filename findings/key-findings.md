# Key Findings

> **Source:** 11 phases of testing on `ph-ee-connector-common` + source analysis of 3 downstream repos  
> **Date:** March 2026

---

## Top 10 Findings

### 1. JDK 21 is Feasible — Lombok is the Only Blocker

Lombok < 1.18.30 crashes on JDK 21 due to internal `javac` API changes (`JCTree$JCImport.qualid` removed). Upgrading to 1.18.34+ resolves it completely. **Zero business code changes needed** for JDK 21 compatibility.

### 2. Gradle 7.3 is Incompatible with JDK 21 (Not Just Lombok)

Gradle 7.3's embedded Groovy 3.0.9 fails at the script compilation level on JDK 21, before any Java code is compiled. **Gradle and Lombok upgrades must happen simultaneously.**

### 3. Gradle 9 Has 5 Breakpoints — All Resolvable

Five distinct breakpoints identified across three categories (plugin API removal, build script issues, dependent blocks). All were resolved, achieving a **fully green Gradle 9 build with zero deprecation warnings**.

### 4. Camel 4 is a Drop-In Upgrade (Zero Code Changes!)

Predicted to have 4 breakpoints. Actual result: **zero breakpoints**. `DefaultExchange(CamelContext)` constructor, `Exchange.getIn()`, and `interceptFrom()` all work unchanged. Total migration effort: **1 line in build.gradle**.

### 5. The Netty/gRPC/Zeebe "Dependency Triangle" Dissolved Naturally

The feared dependency conflict resolved when:
- Spring Boot upgraded to 3.2.5 (new BOM manages Netty 4.1.109)
- Zeebe upgraded to 8.5.0 (compatible transitives)
- Explicit Netty pin removed (let BOM govern)

**8 Netty CVEs resolved** as a result.

### 6. javax→jakarta is Mechanically Simple but Architecturally Complex

Only 14 files changed in connector-common (simple find-replace). But the **cascading impact to 40+ downstream repos** is the real challenge. Mixed javax/jakarta compiles but fails at runtime.

### 7. 15+ CVEs Resolved Through Modernization

Including **Spring4Shell (CRITICAL)**, Netty HTTP/2 Rapid Reset (HIGH), org.json DoS (HIGH), and hibernate-validator XSS (MEDIUM). BOM governance prevents future CVE accumulation.

### 8. BOM Governance Works — Zero Version Strings in build.gradle

`mifos-platform-bom` validated end-to-end. All explicit version numbers eliminated from connector-common's `build.gradle`. Spring Boot parent BOM correctly manages Spring-managed dependencies.

### 9. 85% of Codebase Requires Zero Changes

Out of ~187 Java files, only 14 required modification (7.5%). The vast majority (pure DTOs, enums, utilities) are framework-independent and unaffected by the entire modernization.

### 10. Actual Testing Produces Different Risk Assessments Than Theory

- Camel 4: predicted HIGH risk → actual ZERO breakpoints
- Zeebe 8.5: predicted HIGH risk → actual ZERO breakpoints
- Spring Boot 3: predicted MEDIUM risk → actual 3 breakpoints (all minor)
- Gradle 9: predicted HIGH risk → actual 5 breakpoints (all resolvable)

**Lesson:** Structured experimentation is essential — architectural decisions should be evidence-based.

---

## Summary Statistics

| Metric | Value |
|---|---|
| Total repositories analyzed | 4 |
| Total Java source files scanned | ~400+ |
| javax → jakarta files (ecosystem-wide) | ~93 |
| Phases executed (connector-common) | 11 |
| Breakpoints documented | 15 |
| Build verifications | 50+ |
| CVEs resolved | 15+ |
| Source files modified | 14 out of ~187 (7.5%) |
| Final warning count | 8 compile + 3 test |
| Gradle deprecation warnings | 0 |

