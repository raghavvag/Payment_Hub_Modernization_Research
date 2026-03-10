# Cross-Repository Dependency Graph

> **Scope:** 4 core Payment Hub EE repositories  
> **Date:** March 2026

---

## Dependency Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    Payment Hub EE                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            connector-common (library JAR)            │    │
│  │                                                      │    │
│  │  Spring Boot 3.2.5 ─── Spring Framework 6.1.6       │    │
│  │       │                        │                     │    │
│  │       ├── Jakarta Servlet 6.0  │                     │    │
│  │       ├── Jakarta Validation   │                     │    │
│  │       └── Spring WebFlux ──────┘                     │    │
│  │                                                      │    │
│  │  Camel 4.4.0 ────── RouteBuilder, Exchange, DSL      │    │
│  │  Zeebe 8.5.0 ────── gRPC ────── Netty 4.1.109       │    │
│  │  Auth0 JWT 4.4.0    Jackson 2.15.4                   │    │
│  │  Lombok 1.18.36     org.json 20240303                │    │
│  └──────────┬──────────────────────┬────────────────────┘    │
│             │                      │                         │
│    ┌────────┴────────┐    ┌───────┴────────┐                │
│    │  ams-mifos       │    │  channel       │                │
│    │  Spring Boot 2.6 │    │  Spring Boot 2.6│               │
│    │  Camel 3.4       │    │  Camel 3.12     │               │
│    │  Zeebe 8.1.23    │    │  Zeebe 8.1.1    │               │
│    │  CXF 3.2.5       │    │  Redis          │               │
│    │  Springfox 3.0   │    │  Springfox 3.0  │               │
│    └─────────────────┘    │  SpringDoc 1.6  │               │
│                            └───────┬────────┘                │
│                                    │                         │
│                           ┌────────┴────────┐                │
│                           │ bulk-processor   │                │
│                           │ Spring Boot 2.6  │                │
│                           │ Camel 3.4        │                │
│                           │ Zeebe 8.1.23     │                │
│                           │ Kafka 2.8        │                │
│                           │ AWS SDK 1.11     │                │
│                           │ Azure Blob 12.12 │                │
│                           │ Cucumber 7.8     │                │
│                           └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

---

## Cascading Impact of connector-common Migration

```
connector-common migrates to jakarta.*
         │
         ├──→ ams-mifos: validation DTOs break at RUNTIME
         │    (javax.validation annotations no longer match jakarta.validation on classpath)
         │
         ├──→ channel: interceptor import fails + validation DTOs break
         │    (ErrorHandlerRouteBuilder inheritance = tight coupling)
         │
         └──→ bulk-processor: @EnableJsonWebSignature ClassNotFoundException
              (imports connector-common interceptor package with jakarta.servlet classes)
```

**Conclusion:** Connector-common CANNOT be published with Jakarta namespace until all downstream consumers simultaneously migrate.

---

## Connector-Common Integration Points

### ams-mifos → connector-common

| connector-common class | Used in AMS | Integration type |
|---|---|---|
| `TransactionChannelRequestDTO` | `ZeebeUtil.java`, `ZeebeeWorkers.java` | JSON deserialization |
| `QuoteFspResponseDTO` | `ZeebeeWorkers.java` | JSON deserialization |
| `MoneyData`, `Party`, `PartyIdInfo` | `ZeebeUtil.java` | DTO construction |

### channel → connector-common

| connector-common class | Used in Channel | Integration type |
|---|---|---|
| `ErrorHandlerRouteBuilder` | `ChannelRouteBuilder.java` | **INHERITANCE** (extends) |
| `AuthProcessor` | `ChannelRouteBuilder.java` | Route processor injection |
| `PhErrorDTO` | `IdInterceptor.java` | Error response construction |

### bulk-processor → connector-common

| connector-common class | Used in Bulk | Integration type |
|---|---|---|
| `@EnableJsonWebSignature` | `BulkProcessorApplication.java` | **@Import annotation** |
| `TransactionChannelRequestDTO` | `Utils.java` | DTO construction |

---

## Transitive Dependency Chain (Zeebe → Netty)

```
zeebe-client-java:8.5.0
    └── zeebe-gateway-protocol-impl
        └── io.grpc:grpc-*:1.49.1
            └── io.netty:netty-*:4.1.x (transitive)
```

Spring Boot 3.2.5 BOM manages Netty at **4.1.109.Final** — verified compatible with Zeebe 8.5.0.

