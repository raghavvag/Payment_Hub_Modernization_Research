# Payment Hub EE — Platform Modernization Research

> **Author:** GSoC 2026 Contributor Candidate  
> **Project:** Mifos Initiative — Payment Hub EE Dependency Modernization  
> **Date:** March 2026  
> **Status:** Research Complete — Proposal Ready

---

## Purpose

This repository contains **structured technical research** conducted as preparation for a Google Summer of Code 2026 proposal targeting the **Mifos Initiative's Payment Hub Enterprise Edition (PHEE)** dependency modernization project.

The research objective was to answer, **with evidence**, the following questions:

| Question | Status | Evidence |
|---|---|---|
| Can the ecosystem move to JDK 21 cleanly? | ✅ **Answered** | Lombok ≥ 1.18.30 required; Gradle 7.3 incompatible with JDK 21 |
| What actually breaks in Gradle 9 vs 8.12? | ✅ **Answered** | 5 breakpoints across 3 categories identified |
| How deep is the javax → jakarta migration impact? | ✅ **Answered** | 93 files across 4 repos; connector-common done (14 files) |
| Where exactly does Zeebe conflict? | ✅ **Answered** | Netty/gRPC "dependency triangle" — resolved via BOM alignment |
| What would a centralized BOM look like? | ✅ **Answered** | `mifos-platform-bom` validated end-to-end |
| How would we enforce it across 40+ repos? | ✅ **Answered** | `java-platform` plugin + `platform()` consumption — zero version strings |

---

## What is Payment Hub EE?

Payment Hub EE is a **Digital Public Good (DPG)** — an open-source payment orchestration platform built on microservices architecture. It integrates with:

- **Mojaloop** — interoperability switch for instant payments
- **Apache Fineract** — core banking / Account Management System
- **Zeebe** (Camunda) — BPMN workflow orchestration
- **Apache Camel** — enterprise integration routing
- **Spring Boot** — application framework

The ecosystem comprises **40+ interconnected `ph-ee-*` repositories** that share a foundational library (`ph-ee-connector-common`) and depend on aligned versions of Spring Boot, Camel, Zeebe, and Netty.

---

## Repositories Analyzed

| Repository | Purpose | Migration Status |
|---|---|---|
| [`ph-ee-connector-common`](https://github.com/openMF/ph-ee-connector-common) | Foundation library — DTOs, interceptors, Camel routes, crypto | ✅ **Fully migrated** (11 phases, 15 breakpoints) |
| [`ph-ee-connector-ams-mifos`](https://github.com/openMF/ph-ee-connector-ams-mifos) | Fineract AMS integration connector | 🔲 Analyzed — not started |
| [`ph-ee-connector-channel`](https://github.com/openMF/ph-ee-connector-channel) | REST API gateway for payment initiation | 🔲 Analyzed — not started |
| [`ph-ee-bulk-processor`](https://github.com/openMF/ph-ee-bulk-processor) | Bulk payment file processing engine | 🔲 Analyzed — not started |

---

## Experiments Conducted

| Experiment | Scope | Key Finding |
|---|---|---|
| [JDK 21 Validation](experiments/jdk21-validation.md) | Compile compatibility | Lombok 1.18.24 crashes; 1.18.34+ works. Gradle 7.3 incompatible with JDK 21. |
| [Gradle Upgrade Analysis](experiments/gradle-upgrade-analysis.md) | Gradle 7.3 → 8.12.1 | Stable intermediate target. All failures are formatting, not compilation. |
| [Gradle 9 Experiment](experiments/gradle9-experiment.md) | Gradle 9.0 breakpoints | 5 breakpoints: plugin API removals, configuration role enforcement, sourceCompatibility removal |
| [Spring Boot Migration](experiments/springboot-migration-notes.md) | Spring Boot 2.6.2 → 3.2.5 | 14 files changed (javax→jakarta). Mixed namespace state compiles but fails at runtime. |
| [Camel Upgrade](experiments/camel-upgrade-notes.md) | Camel 3.4.0 → 4.4.0 | **Zero breakpoints** — complete source compatibility. Drop-in upgrade. |
| [Zeebe Compatibility](experiments/zeebe-compatibility.md) | Zeebe 8.1.1 → 8.5.0 | Drop-in upgrade. Netty triangle resolved. 8 JAXB warnings eliminated. |

---

## Key Technical Findings

### 1. Connector-Common is Fully Modernizable
The foundation library successfully migrated through **11 phases** from `Gradle 7.3 + Java 17 + Spring Boot 2.6.2` to `Gradle 9.0 + Java 21 + Spring Boot 3.2.5 + Jakarta EE 10 + Camel 4.4.0 + Zeebe 8.5.0` with **only 14 Java files modified** out of ~187.

### 2. BOM Governance is Viable
A centralized `mifos-platform-bom` was created and validated. It eliminated all explicit version strings from `build.gradle`, proving that 40+ repos can be version-governed from a single source of truth.

### 3. The Dependency Triangle is Manageable
The feared Netty/gRPC/Zeebe version conflict resolved naturally when Spring Boot 3's BOM managed Netty versions and the explicit `netty-all` pin was removed.

### 4. Camel 4 Backward Compatibility is Excellent
Despite being a major version bump, Camel 4.4.0 maintained full source compatibility with all Camel 3.x APIs used in the project. Zero code changes required.

### 5. 15+ CVEs Resolved
The modernization resolves **2 CRITICAL** (Spring4Shell), **8 HIGH** (Netty HTTP/2 Rapid Reset, org.json DoS, commons-io DoS), and **5+ MEDIUM** severity CVEs across the dependency tree.

### 6. Ecosystem Cascade is the Real Challenge
The technical migration is mechanically simple. The **coordination challenge** — upgrading connector-common forces all 40+ downstream repos to simultaneously migrate javax → jakarta — is where the BOM governance architecture provides the solution.

---

## Repository Structure

```
paymenthub-modernization-research/
│
├── README.md                              ← You are here
│
├── experiments/                           ← Detailed experiment logs
│   ├── jdk21-validation.md
│   ├── gradle-upgrade-analysis.md
│   ├── gradle9-experiment.md
│   ├── springboot-migration-notes.md
│   ├── camel-upgrade-notes.md
│   └── zeebe-compatibility.md
│
├── repository-analysis/                   ← Per-repo deep analysis
│   ├── connector-common-analysis.md
│   ├── ams-mifos-analysis.md
│   ├── connector-channel-analysis.md
│   └── bulk-processor-analysis.md
│
├── dependency-analysis/                   ← Cross-repo dependency intelligence
│   ├── dependency-graph.md
│   ├── version-drift-analysis.md
│   └── cve-risk-analysis.md
│
├── migration-strategy/                    ← Modernization architecture
│   ├── modernization-roadmap.md
│   ├── phased-upgrade-plan.md
│   └── bom-governance-design.md
│
├── diagrams/                              ← Architecture diagrams
│   ├── architecture-diagram.md
│   ├── dependency-flow.md
│   ├── migration-pipeline.md
│   └── connector-architecture.md
│
├── logs/                                  ← Raw build logs
│   ├── gradle-8-build-log.txt
│   ├── gradle-9-build-log.txt
│   ├── jdk21-build-log.txt
│   └── spotless-errors-log.txt
│
└── findings/                              ← Synthesized insights
    ├── key-findings.md
    ├── technical-risks.md
    └── recommendations.md
```

---

## How This Research Supports the GSoC Proposal

This research demonstrates:

1. **Technical Depth** — Every finding is backed by actual build execution, not theoretical analysis
2. **Systematic Methodology** — 11-phase Architecture Validation Spike with documented breakpoints
3. **Architectural Thinking** — BOM governance design for 40+ repos, not just version bumps
4. **Risk Awareness** — Dependency triangle, runtime linkage, namespace cascade all identified
5. **Practical Evidence** — Working Gradle 9 build, published BOM, zero CVEs remaining

---

## Technology Stack (Before → After)

| Component | Before | After | Status |
|---|---|---|---|
| Java | 17 | **21 (LTS)** | ✅ Validated |
| Gradle | 7.3 | **9.0** (8.12.1 stable) | ✅ Validated |
| Spring Boot | 2.6.2 | **3.2.5** | ✅ Validated |
| Spring Framework | 5.3.x | **6.1.6** | ✅ Validated |
| Apache Camel | 3.4.0 | **4.4.0** | ✅ Validated |
| Zeebe Client | 8.1.1 | **8.5.0** | ✅ Validated |
| Namespace | `javax.*` | **`jakarta.*`** | ✅ Validated |
| Netty | 4.1.68 (8 CVEs) | **4.1.109** (0 CVEs) | ✅ Validated |
| BOM Governance | None | **`mifos-platform-bom:1.0.0`** | ✅ Validated |

---

## Quick Links

- [Key Findings Summary](findings/key-findings.md)
- [Technical Risks Assessment](findings/technical-risks.md)
- [Modernization Roadmap](migration-strategy/modernization-roadmap.md)
- [BOM Governance Design](migration-strategy/bom-governance-design.md)
- [CVE Risk Analysis](dependency-analysis/cve-risk-analysis.md)

---

> **This repository is a research artifact — not a migration implementation.**  
> It provides the evidence base for a GSoC proposal demonstrating readiness to execute the Payment Hub EE dependency modernization at scale.

