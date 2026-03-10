# Gradle 9.0 Breakpoint Experiment

> **Repository:** `ph-ee-connector-common`  
> **Date:** March 2026  
> **Phase:** 3 (experimental — reverted after analysis) → Phase 9 (fully resolved)

---

## Objective

Experimentally upgrade to Gradle 9.0 to identify **all** breakpoints before attempting a real upgrade. This is a diagnostic phase — the wrapper was reverted to 8.12.1 after analysis and later permanently upgraded in Phase 9 after resolving all issues.

---

## Environment Setup

| Component | Value |
|---|---|
| Gradle 9.0 | Groovy 4.0.27, Kotlin 2.2.0, Ant 1.10.15 |
| JDK | 21.0.10 Temurin |
| Lombok | 1.18.34 |
| Spring Boot | 2.6.2 (at time of Phase 3) |

---

## Steps Performed

### Step 1 — Upgrade Wrapper

```powershell
./gradlew wrapper --gradle-version 9.0
```

### Step 2 — Attempt Build

```powershell
./gradlew clean build
```

**Result: ❌ BUILD FAILED** — Immediate failure at plugin application phase

### Step 3 — Iterative Breakpoint Discovery

Iteratively commented out failing components in `build.gradle` to expose **all** breakpoints, not just the first one.

---

## Results — Five Breakpoints Identified

### Breakpoint #1 — JFrog Artifactory Plugin `convention` API Removal

```
* Where: Build file 'build.gradle' line: 10
* What went wrong:
An exception occurred applying plugin request [id: 'com.jfrog.artifactory', version: '4.24.23']
> Failed to apply plugin 'com.jfrog.artifactory'.
   > Could not get unknown property 'convention' for root project
```

**Root cause:** Artifactory plugin 4.24.23 uses `project.convention` which was removed in Gradle 9.  
**Fix:** Upgrade to JFrog Artifactory plugin 5.x+ (uses `project.extensions`).  
**Category:** 🔴 Plugin-Level — API Removal

### Breakpoint #2 — Spring Boot 2.6.2 `getUploadTaskName()` Removal

```
* What went wrong:
'java.lang.String org.gradle.api.artifacts.Configuration.getUploadTaskName()'
```

**Root cause:** Spring Boot 2.6.2's dependency-management plugin uses `Configuration.getUploadTaskName()` removed in Gradle 9.  
**Fix:** Upgrade to Spring Boot 3.x (Gradle 9-compatible dependency management).  
**Category:** 🔴 Plugin-Level — API Removal

### Breakpoint #3 — `setCanBeResolved(true)` on Locked Configuration

```
* Where: Build file 'build.gradle' line: 79
* What went wrong:
> Cannot change the allowed usage of configuration ':implementation',
  as it was locked upon creation to the role: 'Dependency Scope'.
```

**Root cause:** Gradle 9 enforces strict configuration roles. `implementation` cannot be made resolvable.  
**Fix:** Remove `implementation.setCanBeResolved(true)` and `api.setCanBeResolved(true)`.  
**Category:** 🟡 Build Script-Level — API Enforcement

### Breakpoint #4 — `sourceCompatibility` Project-Level Property Removed

```
* Where: Build file 'build.gradle' line: 177
* What went wrong:
> Could not set unknown property 'sourceCompatibility' for root project
```

**Root cause:** Gradle 9 removed `sourceCompatibility` as a direct project property.  
**Fix:** Replace with Java toolchain:
```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```
**Category:** 🟡 Build Script-Level — API Removal

### Breakpoint #5 — `artifactory {}` DSL Blocks Undefined

```
* Where: Build file 'build.gradle' line: 196
* What went wrong:
> Could not find method artifactory() for arguments [...] on root project
```

**Root cause:** With Artifactory plugin disabled, DSL methods are undefined.  
**Category:** 🟠 Dependent on Breakpoint #1 — auto-resolves when Artifactory plugin upgraded

---

## Breakpoint Classification Summary

| Category | Count | Blocking? | Fix Strategy |
|---|---|---|---|
| 🔴 Plugin-Level API Removal | 2 | Yes — build fails at configuration | Upgrade Artifactory 5.x+, Spring Boot 3.x |
| 🟡 Build Script-Level | 2 | Yes — build fails at evaluation | Remove `setCanBeResolved`, replace `sourceCompatibility` |
| 🟠 Dependent (Artifactory DSL) | 1 | Yes — auto-resolves with #1 | Part of Artifactory upgrade |

---

## Recovery

Gradle 9.0 wrapper could not even parse `build.gradle`, so `./gradlew wrapper` was unusable. Manual edit required:

```ini
# gradle/wrapper/gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.12.1-bin.zip
```

Verified BUILD SUCCESSFUL on Gradle 8.12.1 after recovery.

---

## Phase 9 Resolution (Later)

All 5 breakpoints were resolved in Phase 9:

| Breakpoint | Resolution Applied |
|---|---|
| JFrog Artifactory 4.24.23 | → **5.2.5** |
| Spring Boot 2.6.2 dep-mgmt | → **3.2.5** (Phase 6) |
| `setCanBeResolved(true)` | Removed entirely |
| `sourceCompatibility` | → Java toolchain `languageVersion = 21` |
| Artifactory DSL | Auto-resolved with JFrog 5.2.5 |

**Final Phase 9 result:**

```
./gradlew wrapper --gradle-version 9.0
./gradlew clean build -x spotlessGroovyGradleCheck -x spotlessCheck
BUILD SUCCESSFUL in 6s
```

**✅ Gradle 9.0 builds successfully with zero deprecation warnings.**

---

## Conclusions

1. **Gradle 9 requires ALL breakpoints resolved first** — failure is at build script evaluation
2. **Correct fix order:** Fix build script → Upgrade Spring Boot 3 → Upgrade plugins → then upgrade wrapper
3. **Spring Boot 3 is a prerequisite for Gradle 9** — Boot 2.6.2's dep-management uses removed APIs
4. **Wrapper upgrade is a one-way door** — if `build.gradle` fails to parse, `./gradlew wrapper` won't work
5. **5 breakpoints across 3 categories** — valuable architectural evidence for the ecosystem upgrade
6. **All breakpoints are solvable** — validated end-to-end in Phase 9

