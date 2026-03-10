# BOM Governance Design

> **Purpose:** Centralized dependency version management for 40+ Payment Hub EE repositories  
> **Status:** ✅ Validated end-to-end with `ph-ee-connector-common`

---

## Architecture

### What is a BOM?

A **Bill of Materials (BOM)** is a Gradle `java-platform` that declares dependency version constraints without including any code. Consuming projects import the BOM and declare dependencies **without version numbers** — the BOM governs versions centrally.

### BOM Structure

```groovy
// mifos-platform-bom/build.gradle
plugins {
    id 'java-platform'
    id 'maven-publish'
}

javaPlatform { allowDependencies() }

group = 'org.mifos'
version = '1.0.0-SNAPSHOT'

dependencies {
    // Import Spring Boot BOM as parent — governs Spring, Jackson, Netty, JUnit
    api platform('org.springframework.boot:spring-boot-dependencies:3.2.5')

    constraints {
        // Mifos-specific dependencies (NOT in Spring Boot BOM)
        api 'jakarta.servlet:jakarta.servlet-api:6.0.0'
        api 'org.hibernate.validator:hibernate-validator:8.0.1.Final'
        api 'org.apache.camel:camel-core:4.4.0'
        api 'io.camunda:zeebe-client-java:8.5.0'
        api 'com.auth0:java-jwt:4.4.0'
        api 'org.json:json:20240303'
        api 'commons-io:commons-io:2.18.0'
        api 'commons-codec:commons-codec:1.17.1'
        api 'org.apache.commons:commons-lang3:3.18.0'
        api 'org.projectlombok:lombok:1.18.36'
        api 'org.junit.jupiter:junit-jupiter-api:5.10.2'
        api 'org.junit.jupiter:junit-jupiter-engine:5.10.2'
    }
}

publishing {
    publications {
        bom(MavenPublication) {
            from components.javaPlatform
            artifactId = 'mifos-platform-bom'
        }
    }
}
```

---

## Consumer Usage

Any `ph-ee-*` repository consumes the BOM:

```groovy
// ph-ee-connector-ams-mifos/build.gradle (AFTER modernization)
dependencies {
    // BOM — single source of truth for ALL versions
    implementation platform('org.mifos:mifos-platform-bom:1.0.0')

    // Version-less declarations — BOM governs:
    implementation 'org.mifos:ph-ee-connector-common'
    implementation 'org.apache.camel:camel-core'
    implementation 'io.camunda:zeebe-client-java'
    implementation 'org.json:json'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## Validation Results

### BOM Resolution Verification

| Dependency | BOM Constraint | Resolved Version | Governed By |
|---|---|---|---|
| `com.auth0:java-jwt` | 4.4.0 | **4.4.0** | ✅ Our BOM |
| `org.apache.camel:camel-core` | 4.4.0 | **4.4.0** | ✅ Our BOM |
| `io.camunda:zeebe-client-java` | 8.5.0 | **8.5.0** | ✅ Our BOM |
| `org.json:json` | 20240303 | **20240303** | ✅ Our BOM |
| `commons-io:commons-io` | 2.18.0 | **2.18.0** | ✅ Our BOM |
| `commons-codec:commons-codec` | 1.17.1 | **1.16.1** | ⚠️ Spring Boot BOM wins |
| `jackson-datatype-jsr310` | 2.17.2 | **2.15.4** | ⚠️ Spring Boot BOM wins |
| `lombok` | 1.18.36 | **1.18.32** | ⚠️ Spring Boot BOM wins |
| `io.netty:netty-*` | — | **4.1.109** | ✅ Spring Boot BOM |

**Key insight:** Spring Boot parent BOM has equal priority. For deps managed by BOTH, Spring Boot wins — this is **correct behavior**. Our BOM only overrides deps Spring Boot doesn't manage.

---

## Benefits of BOM Governance

| Benefit | Without BOM | With BOM |
|---|---|---|
| Version consistency | Each repo hardcodes versions → drift | Single source of truth |
| CVE mitigation | Must update 40+ repos individually | Update BOM → all repos get fix |
| Dependency conflicts | Transitive version clashes | BOM resolves centrally |
| Upgrade effort | ~40 PRs per dependency bump | 1 BOM PR + 0 repo changes |
| Netty/gRPC alignment | Manual coordination required | BOM ensures consistent versions |
| New repo onboarding | Copy-paste versions from other repos | Add single `platform()` declaration |

---

## Version Bump Process

```
STEP 1: CVE discovered (e.g., Netty vulnerability)
         │
         ▼
STEP 2: Update mifos-platform-bom
         ├── Change constraint version
         └── Publish new BOM version
         │
         ▼
STEP 3: All 40+ repos get new version on next build
         └── ZERO changes needed in any ph-ee-* repo
         │
         ▼
STEP 4: CI/CD validates each repo builds with new BOM
```

---

## Rollout Strategy

### Option A — Incremental (Recommended)

1. Publish BOM to mavenLocal
2. Migrate connector-common to consume BOM ✅ (done)
3. Migrate bulk-processor next (medium difficulty)
4. Migrate ams-mifos (hard — CXF)
5. Migrate channel (hard — GSMA stubs)
6. Publish BOM to Artifactory/Maven Central
7. Remaining 36+ repos consume BOM

### Option B — Big Bang (Not Recommended)

Migrate all repos simultaneously. High risk, hard to debug failures.

---

## BOM Governance Limitations

1. **Does NOT eliminate code changes** — javax→jakarta still requires per-repo file edits
2. **Spring Boot BOM overrides some constraints** — deps managed by both BOMs follow Spring Boot
3. **Repos can still override versions** — BOM uses `constraints` (not `force`)
4. **BOM versioning needs a release process** — who owns releases, what CI/CD pipeline

---

## Future Considerations

- **Gradle Convention Plugin:** Could enforce BOM + Java toolchain + Spotless config across all repos
- **Dependabot/Renovate:** Automated BOM version bumps via PR
- **BOM Compatibility Testing:** CI matrix that builds all repos against BOM changes before publishing

