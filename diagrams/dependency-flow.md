# Dependency Flow Diagram

## Dependency Resolution Chain

```
                              Zeebe Broker
                                  ▲
                    gRPC (Netty)  │
                                  │
    ┌──────────┐   ┌──────────┐  │  ┌──────────────┐
    │ ams-mifos├──►│connector-├──┤  │bulk-processor│
    │          │   │ common   │  │  │              │
    └────┬─────┘   └────┬─────┘  │  └──────┬───────┘
         │              │        │         │
         │         ┌────▼─────┐  │    ┌────▼─────┐
         │         │ channel  ├──┘    │  Kafka   │
         │         └────┬─────┘       └──────────┘
         │              │
    ┌────▼─────┐   ┌────▼─────┐
    │ Fineract │   │ Mojaloop │
    │   AMS    │   │  Switch  │
    └──────────┘   └──────────┘
```

## Transitive Dependency Chain (Critical Path)

```
mifos-platform-bom
    │
    ├── Spring Boot BOM 3.2.5
    │       ├── Spring Framework 6.1.6
    │       ├── Jackson 2.15.4
    │       ├── Netty 4.1.109.Final ────── ALL Netty modules
    │       ├── Jakarta Servlet 6.0
    │       ├── Tomcat 10.1
    │       └── JUnit 5.10.2
    │
    ├── Camel 4.4.0
    │       ├── camel-core
    │       ├── camel-spring-boot-starter
    │       └── (requires Jakarta EE)
    │
    ├── Zeebe 8.5.0
    │       └── zeebe-gateway-protocol-impl
    │               └── io.grpc:grpc-*:1.49.1
    │                       └── io.netty:netty-*:4.1.x (managed by Spring Boot BOM)
    │
    └── Mifos-specific
            ├── Auth0 JWT 4.4.0
            ├── org.json 20240303
            ├── commons-io 2.18.0
            └── Lombok 1.18.36
```

## Jakarta Namespace Cascade

```
connector-common migrates to jakarta.*
         │
         ├──→ ams-mifos: validation DTOs break at RUNTIME
         │    (javax.validation annotations ≠ jakarta.validation on classpath)
         │
         ├──→ channel: interceptor import fails + validation DTOs break
         │    (ErrorHandlerRouteBuilder inheritance = tight coupling)
         │
         └──→ bulk-processor: @EnableJsonWebSignature ClassNotFoundException
              (imports interceptor package with jakarta.servlet classes)

SOLUTION: BOM + coordinated migration
```

