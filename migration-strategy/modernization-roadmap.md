# Modernization Roadmap

> **Scope:** Payment Hub EE Ecosystem (40+ repositories)  
> **Date:** March 2026  
> **Target:** Java 21, Gradle 9, Spring Boot 3, Jakarta EE 10, Camel 4, Zeebe 8.5+, BOM governance

---

## Roadmap Overview

```
Phase 1 ──── Phase 2 ──── Phase 3 ──── Phase 4 ──── Phase 5 ──── Phase 6
connector    BOM          bulk-        ams-mifos    channel      Ecosystem
common       Creation     processor                              Validation
✅ DONE      ✅ DONE      🔲 NEXT      🔲           🔲           🔲
```

---

## Phase 1 — Foundation (✅ COMPLETE)

**Target:** `ph-ee-connector-common`  
**Duration:** Week 1-3

| Step | Action | Status |
|---|---|---|
| 1.1 | JDK 21 + Lombok upgrade | ✅ |
| 1.2 | Gradle 8.12.1 + build script cleanup | ✅ |
| 1.3 | CVE-critical dependency bumps | ✅ |
| 1.4 | Spring Boot 3.2.5 + javax→jakarta (14 files) | ✅ |
| 1.5 | Camel 4.4.0 | ✅ |
| 1.6 | Zeebe 8.5.0 + Netty alignment | ✅ |
| 1.7 | Plugin modernization + Gradle 9 | ✅ |

---

## Phase 2 — BOM Governance (✅ COMPLETE)

**Target:** `mifos-platform-bom`  
**Duration:** Week 3-4

| Step | Action | Status |
|---|---|---|
| 2.1 | Create BOM repo with `java-platform` plugin | ✅ |
| 2.2 | Import Spring Boot BOM as parent | ✅ |
| 2.3 | Add Mifos-specific constraints (Camel, Zeebe, Auth0, etc.) | ✅ |
| 2.4 | Publish to mavenLocal | ✅ |
| 2.5 | Validate connector-common builds with zero version strings | ✅ |

---

## Phase 3 — Bulk Processor (🔲 NOT STARTED)

**Target:** `ph-ee-bulk-processor`  
**Duration:** Week 4-6  
**Estimated effort:** 8-12 days

| Step | Action | Est. |
|---|---|---|
| 3.1 | Gradle 7.3 → 8.12.1 + Lombok 1.18.34 | 1 day |
| 3.2 | Build script cleanup | 1 day |
| 3.3 | Spring Boot 3.2.5 + javax→jakarta (11 files) | 2 days |
| 3.4 | connector-common → 2.0.0-SNAPSHOT (jakarta) | 1 day |
| 3.5 | Camel 4.4.0 | 1 day |
| 3.6 | Zeebe 8.5.0 | 0.5 day |
| 3.7 | BOM consumption | 1 day |
| 3.8 | Gradle 9 verification | 1 day |
| 3.9 | AWS SDK v2 migration (if time permits) | 3 days |
| 3.10 | Tika 1.4 → 2.9+ | 0.5 day |

---

## Phase 4 — AMS-Mifos (🔲 NOT STARTED)

**Target:** `ph-ee-connector-ams-mifos`  
**Duration:** Week 6-9  
**Estimated effort:** 15-19 days (highest difficulty)

| Step | Action | Est. |
|---|---|---|
| 4.1 | Gradle 8.12.1 + Lombok | 1 day |
| 4.2 | Build script cleanup + fix `sourceCompatibility = VERSION_13` | 1 day |
| 4.3 | Remove Springfox → add SpringDoc | 2 days |
| 4.4 | Spring Boot 3.2.5 | 1 day |
| 4.5 | javax→jakarta (13 files) | 2 days |
| 4.6 | **CXF 3.2.5 → 4.x** (or replace with Spring RestClient) | **5 days** ⚠️ |
| 4.7 | JAXB javax.xml.bind → jakarta.xml.bind | 1 day |
| 4.8 | BOM consumption | 1 day |
| 4.9 | Camel 4 (including camel-cxf restructuring) | 2 days |
| 4.10 | Zeebe 8.5.0 | 0.5 day |
| 4.11 | fineractstub regeneration (Jakarta-compatible) | 2 days |

---

## Phase 5 — Channel Connector (🔲 NOT STARTED)

**Target:** `ph-ee-connector-channel`  
**Duration:** Week 9-12  
**Estimated effort:** 12-17 days

| Step | Action | Est. |
|---|---|---|
| 5.1 | Gradle 8.12.1 + Lombok fix | 1 day |
| 5.2 | Build script cleanup | 1 day |
| 5.3 | Springfox + SpringDoc 1.x → SpringDoc 2.x | 2 days |
| 5.4 | Spring Boot 3.2.5 | 1 day |
| 5.5 | javax→jakarta core (5 files) | 1 day |
| 5.6 | javax→jakarta GSMA stubs API (15 files) | 2 days |
| 5.7 | javax→jakarta GSMA models (40 files) — OR regenerate | **3-5 days** ⚠️ |
| 5.8 | BOM consumption | 1 day |
| 5.9 | Camel 4 (deduplicate mixed versions) | 1 day |
| 5.10 | Zeebe 8.5.0 | 0.5 day |
| 5.11 | Redis Spring Boot 3 validation | 1 day |

---

## Phase 6 — Ecosystem Validation (🔲 NOT STARTED)

**Target:** All repos together  
**Duration:** Week 12-13

| Step | Action |
|---|---|
| 6.1 | Publish connector-common + BOM to Artifactory |
| 6.2 | Build all repos against published artifacts |
| 6.3 | Integration testing: ams ↔ channel ↔ bulk ↔ Zeebe |
| 6.4 | Docker image builds with JDK 21 base |
| 6.5 | Kubernetes deployment validation |
| 6.6 | Final CVE audit |
| 6.7 | Documentation and contribution guide |

---

## Timeline

```
Week:  1   2   3   4   5   6   7   8   9  10  11  12  13
       ├───┴───┴───┤───┤───┴───┤───┴───┴───┤───┴───┴───┤───┤
       │  Phase 1  │ 2 │Phase 3│  Phase 4  │  Phase 5  │ 6 │
       │  common   │BOM│ bulk  │ ams-mifos │  channel  │Val│
       │   ✅ DONE │ ✅│       │           │           │   │
       └───────────┴───┴───────┴───────────┴───────────┴───┘
```

**Total estimated effort: 48-61 days across 13 weeks**

