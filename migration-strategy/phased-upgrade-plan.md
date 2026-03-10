# Phased Upgrade Plan — Per-Repository Execution Sequence

> **Methodology:** One variable at a time. Build after every change. Document breakpoints, not just successes.

---

## Execution Principles

1. **One variable at a time** — never upgrade two major frameworks simultaneously
2. **Build after every change** — `./gradlew clean build` is the truth check
3. **Document breakpoints** — failures are data
4. **Revertible steps** — every phase independently revertible via git tag
5. **Commit after each green build** — atomic, bisectable history
6. **Test dependencies with `dependencyInsight`** — understand the graph

---

## Phase 1 — Baseline Dependency Audit

**Applies to:** Every repository  
**Duration:** 1 day per repo

### Steps

1. Capture current dependency tree
   ```powershell
   ./gradlew dependencies --configuration compileClasspath > deps-before.txt
   ```

2. Run CVE scan on current dependencies

3. Document all direct and transitive dependency versions

4. Identify anti-patterns in `build.gradle`:
   - Duplicate `apply plugin` statements
   - `setCanBeResolved(true)` on locked configurations
   - Build plugins as runtime dependencies
   - Mismatched versions (compile vs annotationProcessor)
   - Hardcoded versions that should be BOM-managed

### Document
- Current dependency inventory
- CVE findings
- Anti-pattern list
- Baseline build status

---

## Phase 2 — JDK 21 Compatibility

**Applies to:** Every repository  
**Duration:** 1-2 days per repo

### Steps

1. Set `JAVA_HOME` to JDK 21
   ```powershell
   $env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-21.0.10.7-hotspot"
   $env:PATH = "$env:JAVA_HOME\bin;$env:PATH"
   ```

2. Upgrade Lombok to 1.18.34+
   ```groovy
   compileOnly 'org.projectlombok:lombok:1.18.34'
   annotationProcessor 'org.projectlombok:lombok:1.18.34'
   ```

3. Attempt build (expect Gradle 7.3 failure on JDK 21)

4. Simultaneously upgrade Gradle to 8.12.1
   ```powershell
   ./gradlew wrapper --gradle-version 8.12.1
   ```

5. Build and verify compilation

### Expected Breakpoints
- Lombok `NoSuchFieldError` on `JCTree$JCImport.qualid`
- Gradle 7.3 Groovy 3.0.9 `ClassSignatureParser` failure

### Document
- Lombok version that resolves JDK 21 crash
- Gradle version that resolves Groovy crash
- Compiler warnings (new vs pre-existing)

---

## Phase 3 — Gradle Modernization

**Applies to:** Every repository  
**Duration:** 1-2 days per repo

### Steps

1. Fix Spotless formatting: `./gradlew spotlessApply`
2. Temporarily disable Checkstyle if CRLF issues block build
3. Run `--warning-mode all` and capture deprecation list
4. Experimentally try Gradle 9.0 (revert after)
5. Classify breakpoints by category

### Expected Breakpoints
- JFrog Artifactory `convention` API removal (if used)
- `setCanBeResolved(true)` hard error
- `sourceCompatibility` project-level property removal
- Spring Boot 2.x `getUploadTaskName()` removal

### Document
- Every deprecation warning
- Every breakpoint classified as plugin-level, build-script-level, or API-removal
- Recovery procedure

---

## Phase 4 — Spring Boot Upgrade

**Applies to:** Every repository  
**Duration:** 2-5 days per repo (varies by javax→jakarta surface area)

### Steps

1. Upgrade Spring Boot plugin to 3.2.5
2. Replace `javax.servlet-api` → `jakarta.servlet-api:6.0.0`
3. Upgrade `hibernate-validator` to 8.0.1.Final (jakarta)
4. Migrate javax.servlet imports → jakarta.servlet
5. Migrate javax.validation imports → jakarta.validation
6. Verify javax.crypto is UNTOUCHED
7. Add `bootJar { enabled = false }` for library projects
8. Build and verify

### Expected Breakpoints
- `bootJar` requires main class (library projects)
- JAXB `XmlAccessType` warnings from transitives
- Spring API deprecations in test code
- Mixed javax/jakarta state may compile but fail at runtime

### Document
- Every file changed
- javax.crypto verification
- New warnings vs pre-existing
- Dependency tree diff (Boot 2 vs Boot 3)

---

## Phase 5 — Jakarta Namespace Migration

**Applies to:** Downstream repos after connector-common published  
**Duration:** Included in Phase 4 for connector-common; separate for downstream

### Steps (per namespace)

| Namespace | Find | Replace | Validation |
|---|---|---|---|
| Servlet | `javax.servlet.` | `jakarta.servlet.` | `grep javax.servlet src/` → zero |
| Validation | `javax.validation.` | `jakarta.validation.` | `grep javax.validation src/` → zero |
| Annotation | `javax.annotation.` | `jakarta.annotation.` | `grep javax.annotation src/` → zero |
| WS-RS | `javax.ws.rs.` | `jakarta.ws.rs.` | `grep javax.ws.rs src/` → zero |
| XML Bind | `javax.xml.bind.` | `jakarta.xml.bind.` | `grep javax.xml.bind src/` → zero |
| Crypto | **DO NOT CHANGE** | — | `grep javax.crypto src/` → files exist ✅ |
| SSL | **DO NOT CHANGE** | — | `grep javax.net.ssl src/` → files exist ✅ |

---

## Phase 6 — Camel and Zeebe Compatibility Validation

**Applies to:** Every repository with Camel/Zeebe usage  
**Duration:** 1-2 days per repo

### Steps

1. Upgrade `camel-core` to 4.4.0
2. Build — expect zero breakpoints (based on testing)
3. Upgrade `zeebe-client-java` to 8.5.0
4. Remove explicit Netty pin — let BOM manage
5. Run `dependencyInsight` for Netty/gRPC versions
6. Build and verify warning count

### Expected Result
Based on connector-common testing, both Camel 4 and Zeebe 8.5 are **drop-in upgrades** with zero source code changes required.

### Document
- Camel API compatibility confirmation
- Zeebe API compatibility confirmation
- Netty version resolution (before vs after)
- Warning count change

