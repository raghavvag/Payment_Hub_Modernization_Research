# Migration Pipeline Diagram

## Per-Repository Upgrade Sequence

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UPGRADE PIPELINE (per repo)                      │
│                                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐        │
│  │ Phase A  │──►│ Phase B  │──►│ Phase C  │──►│ Phase D  │        │
│  │ Lombok + │   │ Gradle   │   │ CVE Safe │   │ Build    │        │
│  │ JDK 21   │   │ 8.12.1   │   │ Bumps    │   │ Cleanup  │        │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘        │
│       │              │              │              │                │
│       ▼              ▼              ▼              ▼                │
│    BUILD          BUILD          BUILD          BUILD               │
│    VERIFY         VERIFY         VERIFY         VERIFY              │
│                                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐        │
│  │ Phase E  │──►│ Phase F  │──►│ Phase G  │──►│ Phase H  │        │
│  │ Spring   │   │ Camel 4  │   │ Zeebe +  │   │ Plugins  │        │
│  │ Boot 3 + │   │          │   │ Netty    │   │ + Gradle │        │
│  │ Jakarta  │   │          │   │ Align    │   │   9      │        │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘        │
│       │              │              │              │                │
│       ▼              ▼              ▼              ▼                │
│    BUILD          BUILD          BUILD          BUILD               │
│    VERIFY         VERIFY         VERIFY         VERIFY              │
│                                                                     │
│  ┌──────────┐                                                       │
│  │ Phase I  │   DONE ✅                                             │
│  │ BOM      │                                                       │
│  │ Consume  │                                                       │
│  └──────────┘                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

## Ecosystem-Wide Migration Sequence

```
PHASE 1              PHASE 2              PHASE 3
connector-common    Publish BOM v1.0    ams + channel + bulk
 (COMPLETED ✅)     (COMPLETED ✅)      (NOT STARTED 🔲)
      │                   │                     │
      ▼                   ▼                     ▼
┌─────────────┐   ┌──────────────┐   ┌───────────────────┐
│ JDK 21      │   │ mifos-       │   │ Same upgrade      │
│ Gradle 9    │   │ platform-bom │   │ sequence per repo:│
│ Spring Boot3│   │ published to │   │ 1. Gradle 8.12    │
│ Jakarta EE  │   │ mavenLocal   │   │ 2. Lombok         │
│ Camel 4     │   │              │   │ 3. Spring Boot 3  │
│ Zeebe 8.5   │   └──────────────┘   │ 4. javax→jakarta  │
│ BOM consump.│                      │ 5. Camel 4        │
└─────────────┘                      │ 6. Zeebe 8.5      │
                                     │ 7. BOM consump.   │
                                     │ 8. Gradle 9       │
                                     └───────────────────┘
```

## Ordering Constraints

```
JDK 21 + Lombok ──────►  Gradle 8.12.1
                              │
                              ▼
                        CVE Safe Bumps
                              │
                              ▼
                        Build Cleanup
                              │
                              ▼
                    Spring Boot 3 + Jakarta  ◄── MUST come before Camel 4
                              │
                              ▼
                         Camel 4.4.0  ◄── Requires Jakarta EE
                              │
                              ▼
                    Zeebe 8.5 + Netty Align  ◄── Resolves dependency triangle
                              │
                              ▼
                    Plugin Modernization  ◄── MUST come before Gradle 9
                              │
                              ▼
                         Gradle 9.0
                              │
                              ▼
                      BOM Consumption
```

