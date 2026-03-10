# Spring Boot Migration Notes — 2.6.2 → 3.2.5

> **Repository:** `ph-ee-connector-common`  
> **Date:** March 2026  
> **Phase:** 6

---

## Objective

Upgrade Spring Boot from 2.6.2 to 3.2.5, which forces the `javax` → `jakarta` namespace migration. This is the **largest single phase** — affecting 14 Java source files and fundamentally changing the dependency namespace.

---

## Environment Setup

| Component | Before | After |
|---|---|---|
| Spring Boot Plugin | 2.6.2 | **3.2.5** |
| Spring Framework | 5.3.14 | **6.1.6** |
| `javax.servlet-api` | 3.1.0 | → `jakarta.servlet-api:6.0.0` |
| `hibernate-validator` | 6.2.5.Final | → **8.0.1.Final** (jakarta) |

---

## Steps Performed

### Step 1 — Update build.gradle

```groovy
// Plugin version change
id 'org.springframework.boot' version '3.2.5'  // was 2.6.2

// Dependency swap
implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'  // was javax.servlet:javax.servlet-api:3.1.0
compileOnly 'org.hibernate.validator:hibernate-validator:8.0.1.Final'  // was 6.2.5.Final

// Library configuration
bootJar { enabled = false }  // This is a library, not a bootable app
```

### Step 2 — Migrate javax.servlet → jakarta.servlet (6 files)

| File | Imports Changed |
|---|---|
| `ByteArrayServletInputStream.java` | `ReadListener`, `ServletInputStream` |
| `MultiReadHttpServletRequest.java` | `ServletException`, `ServletInputStream`, `HttpServletRequest`, `HttpServletRequestWrapper`, `Part` |
| `JWSFilterStrategy.java` | `FilterChain`, `ServletException`, `ServletRequest`, `ServletResponse`, `HttpServletResponse` |
| `JWSUtil.java` | `HttpServletRequest`, `HttpServletResponse` |
| `MultiReadFilter.java` | `FilterChain`, `ServletException`, `ServletRequest`, `ServletResponse`, `HttpServletRequest` |
| `WebSignatureInterceptor.java` | `HttpServletRequest`, `HttpServletResponse` |

### Step 3 — Migrate javax.validation → jakarta.validation (8 files)

| File | Imports Changed |
|---|---|
| `EnumNamePattern.java` | `Constraint` |
| `EnumNamePatternValidator.java` | `ConstraintValidator`, `ConstraintValidatorContext` |
| `PhErrorDTO.java` | `constraints.NotNull` |
| `RegisterAliasRequestDTO.java` | `constraints.NotEmpty`, `constraints.NotNull` |
| `TransactionChannelRequestDTO.java` | `constraints.NotNull` |
| `MoneyData.java` | `constraints.NotEmpty` |
| `Party.java` | `constraints.NotNull` |
| `PartyIdInfo.java` | `constraints.NotEmpty`, `constraints.NotNull` |

### Step 4 — Verify javax.crypto UNTOUCHED

```powershell
Select-String -Path "src\main\java\**\*.java" -Pattern "javax\.crypto" -Recurse
```

**Result:** 4 files correctly still using `javax.crypto` (JDK-native — never migrates):
- `SecurityUtil.java`
- `JsonWebSignature.java` (util)
- `JsonWebSignature.java` (interceptor/service)
- `JsonWebSignatureService.java`

### Step 5 — Build

```powershell
./gradlew clean build -x spotlessGroovyGradleCheck -x spotlessCheck
```

---

## Results

### Build Status: ✅ BUILD SUCCESSFUL

```
> Task :compileJava — 17 warnings (9 pre-existing + 8 new JAXB)
> Task :compileTestJava — 3 warnings (getStatusCodeValue deprecated)
> Task :test
> Task :build
BUILD SUCCESSFUL in 20s
```

---

## Warnings / Errors Encountered

### Breakpoint #1 — bootJar Fails (Library Has No Main Class)

```
Execution failed for task ':bootJar'.
> Main class name has not been configured and it could not be resolved
```

**Fix:** `bootJar { enabled = false }` — standard for library projects on Spring Boot 3.  
**Category:** 🟡 Build Script — expected

### Breakpoint #2 — 8 New JAXB XmlAccessType Warnings

```
warning: unknown enum constant XmlAccessType.FIELD
  reason: class file for javax.xml.bind.annotation.XmlAccessType not found
```

**Root cause:** Zeebe/Camel transitive dependencies use JAXB annotations. JAXB was removed from JDK in Java 11.  
**Impact:** Non-blocking warnings only.  
**Resolution:** Eliminated when Zeebe upgraded to 8.5.0 in Phase 8.

### Breakpoint #3 — Test Deprecation Warnings

```
OnboardDfsps.java:329: warning: [deprecation] getStatusCodeValue() deprecated
```

**Root cause:** Spring Boot 3 / Spring Framework 6 deprecated `getStatusCodeValue()`.  
**Fix:** Replace with `getStatusCode().value()`.  
**Impact:** Test code only, non-blocking.

### 🚨 Critical Finding: Mixed javax/jakarta Compiles Successfully

Before migrating source files, after only changing `build.gradle` dependencies:
- `compileJava` → ✅ PASSED
- `test` → ✅ PASSED

**Why this is dangerous:** Both javax and jakarta namespaces coexist on the classpath because Camel 3.4.0 pulls javax transitively while Spring Boot 3 provides jakarta. The project compiles and tests in this **mixed state**, but at runtime (Tomcat 10+), only `jakarta.servlet` is available. **Migration is mandatory even though the build passes.**

---

## Migration Scope Summary

```
Total files affected:     14
  javax.servlet:           6  (find-replace: javax.servlet → jakarta.servlet)
  javax.validation:        8  (find-replace: javax.validation → jakarta.validation)
  javax.crypto:            0  (JDK native — correctly untouched)
Total files unchanged:   173
```

---

## Verification Commands

```powershell
# Verify zero javax.servlet remaining
Select-String -Path "src\main\java\**\*.java" -Pattern "javax\.servlet" -Recurse
# Result: ZERO ✅

# Verify zero javax.validation remaining
Select-String -Path "src\main\java\**\*.java" -Pattern "javax\.validation" -Recurse
# Result: ZERO ✅

# Verify javax.crypto preserved
Select-String -Path "src\main\java\**\*.java" -Pattern "javax\.crypto" -Recurse
# Result: 4 files ✅ (correct — JDK native)
```

---

## Conclusions

1. **14 Java files modified** — exactly as predicted (6 servlet + 8 validation)
2. **Mixed javax/jakarta state compiles but is runtime-dangerous** — critical insight for downstream repos
3. **bootJar must be disabled for library projects** — Spring Boot 3 enforces main class resolution
4. **JAXB warnings from Zeebe transitives** — resolved later by Zeebe 8.5.0 upgrade
5. **javax.crypto correctly untouched** — JDK-native, never migrates to jakarta
6. **The migration is mechanically simple** — the complexity is the downstream cascade to 40+ repos
7. **Spring Framework upgraded from 5.3.x to 6.1.6** — major but transparent through BOM

