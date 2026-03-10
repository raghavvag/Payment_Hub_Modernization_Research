# Connector Architecture — Transaction Flow

## Payment Transaction Flow

```
  Client Request
       │
       ▼
  ┌─────────────┐
  │   Channel   │  REST API (Spring Boot)
  │  Connector  │
  └──────┬──────┘
         │ Camel Route
         ▼
  ┌─────────────┐
  │   Zeebe     │  BPMN Workflow Engine
  │   Broker    │
  └──┬───┬───┬──┘
     │   │   │
     │   │   │  Job Workers
     ▼   ▼   ▼
  ┌────┐┌────┐┌────────────┐
  │AMS ││Chan││   Bulk     │
  │    ││nel ││ Processor  │
  │Wkrs││Wkrs││  Workers   │
  └─┬──┘└────┘└──────┬─────┘
    │                 │
    ▼                 ▼
  ┌──────┐     ┌───────────┐
  │Finer.│     │ S3/Azure  │
  │ AMS  │     │ + Kafka   │
  └──────┘     └───────────┘
```

## Connector-Common as Foundation

```
org.mifos.connector.common (187 Java files)
│
├── ams/dto/           (~20 files)  ← Used by ams-mifos workers
├── camel/             (8 files)    ← Route builders extended by all connectors
│   └── ErrorHandlerRouteBuilder   ← channel INHERITS this class
├── channel/dto/       (6 files)    ← Used by channel REST controllers
├── gsma/dto/          (~25 files)  ← Used by channel GSMA controllers
├── interceptor/       (12 files)   ← Enabled by bulk @EnableJsonWebSignature
├── mojaloop/          (~20 files)  ← Used by all connectors for protocol
├── util/              (7 files)    ← Crypto (JDK native), Zeebe utilities
├── vouchers/dto/      (10 files)   ← Used by voucher processing
└── zeebe/             (1 file)     ← Constants shared across all connectors
```

## Zeebe Worker Distribution

```
┌──────────────────────────────────────────────────────┐
│                    Zeebe Broker                       │
│                                                      │
│  Workflow: transfer-process                          │
│  ┌────┐ ┌────────┐ ┌────────┐ ┌────┐ ┌──────────┐  │
│  │init│→│validate│→│  route │→│exec│→│ callback │  │
│  └────┘ └────────┘ └────────┘ └────┘ └──────────┘  │
│    ↕        ↕          ↕        ↕         ↕         │
└────┼────────┼──────────┼────────┼─────────┼─────────┘
     │        │          │        │         │
     ▼        ▼          ▼        ▼         ▼
  channel  channel    channel  ams-mifos  channel
  workers  workers    workers  workers    workers
  (11)                         (3+)
                                │
                            bulk-processor
                            workers (8+)
```

