# Ledger System Design

**Table of Contents**
1. [Product Requirements Document](#PRD)
2. [System Design](#system-design)

   
## <a name="PRD"></a> Product Requirements Document 
Product Name: Ledger Service (LS)  
Author: João Batista  
Technical Lead: Adriano Silva de Oliveira  
Version: 1.0  
Last Updated: 2025-06-30  

### Purpose
The goal of this product is to design and build a low-latency, immutable ledger service capable of processing millions of financial transactions per day. The system must serve as a system of record for transactional data, offering high throughput and low P99 latency (<100 ms) while supporting integration with multiple domains across the company, including reconciliation, fraud detection, accounting, and compliance.
This ledger will become a critical component of our payment platform infrastructure, ensuring the reliability, integrity, and immutability of all entries to enable trust, traceability, and auditability at scale.

### 🧩 Scope

In Scope:
- Append-only ledger with immutable records (no updates/deletes).
- Write model: record credit and debit transactions with metadata.
- Read model: expose read-optimized views for querying account balances and histories.
- Transactional ingestion pipeline with durability guarantees (Kafka-based ingestion layer).
- Idempotency and deduplication mechanisms.
- Integration interfaces for external systems (gRPC APIs).
- Support for real-time stream processing (e.g., for reconciliation and risk).
- Basic observability: metrics (latency, throughput, error rate), structured logs, traces.

Out of Scope:
- Manual correction or rollback of ledger entries (handled externally by the financial department).
- User interface or dashboard (initial release is backend only).
- Non-financial event ingestion (i.e., the system is scoped to transactional events).

### 🧪 Functional Requirements
| ID  | Requirement                                                                                            |
| --- | ------------------------------------------------------------------------------------------------------ |
| FR1 | The system must accept credit and debit transactions via API                                           |
| FR2 | Each transaction must be persisted immutably and include metadata (timestamp, source, correlation ID). |
| FR3 | The system must support querying balances per account with strong read consistency.                    |
| FR4 | The service must deduplicate transactions with identical idempotency keys.                             |
| FR5 | A write must be acknowledged only after being durably persisted.                                       |
| FR6 | The system must expose an event stream (Kafka topic) of confirmed ledger entries.                      |

### 🚦 Non-Functional Requirements
| Category      | Requirement                                                                                     |
| ------------- | ----------------------------------------------------------------------------------------------- |
| Performance   | P99 write latency < 100ms under normal load. Throughput: ≥ 10,000 tx/sec sustained.             |
| Scalability   | Must support horizontal scaling for ingestion and read/query components.                        |
| Durability    | Writes must survive node failure.                                                               |
| Resilience    | System must be resilient to network partitions; must support retry and backpressure strategies. |
| Auditability  | All writes must be traceable and verifiable. Full audit log exposed internally.                 |
| Security      | All APIs must be authenticated. Transaction integrity must be ensured end-to-end.               |
| Observability | Expose metrics: P95/P99 latency, ingestion throughput, deduplication rate, retry count.         |

### 🔗 Integration Requirements
- The ledger service must be able to receive events via Kafka or HTTP2/gRPC.
- It must expose events to downstream consumers via Kafka.
- Domain teams (e.g., risk, reconciliation, reporting) will subscribe to these event streams and/or query the read model.
- API endpoints must conform to internal API gateway requirements and be discoverable via the service registry.

### 🚀 Milestones
| Phase   | Deliverable                                                                               |
| ------- | ----------------------------------------------------------------------------------------- |
| Phase 1 | MVP with credit/debit ingestion, Kafka ingestion pipeline, append-only storage, basic API |
| Phase 2 | Read model with account balance lookup, deduplication mechanism                           |
| Phase 3 | Event replay support, stream publishing for consumers                                     |
| Phase 4 | SLA enforcement, observability stack (Grafana Loki OSS)                                   |
| Phase 5 | Production hardening: disaster recovery, replication strategy, compliance reviews         |

### 📓 Open Questions / Considerations

Will the ledger support multi-currency transactions? If yes, currency consistency needs to be enforced.  
Will we need versioning or evolution for ledger schemas?  
Will the event ingestion be eventually extended to other business domains (e.g., loyalty points, refunds)?  
Is ordering guaranteed globally or only per account?  

## <a name="system-design"></a>  System Design

WIP

## Decisions
### L4 or L7 Load Balancer
#### Key Considerations
1. Internal traffic, controlled environment. It won't be exposed to the internet.
2. Need to distribute traffic across multiple API Gateway replicas.
3. Low-latency is required, it should add minimal overhead.
4. The API Gateway will handle the SSL at the HTTPS protocol
#### L4 Load Balancer
- Offer maximum performance at a minimum overhead
- Balance the load at TCP connections (port-base, no deep inspection)
##### Pros
- Very low latency
- Simple
- Less CPU-intensive
##### Cons
- No insights into HTTP traffic for smart routing
#### L7 Load Balancer
- Can do health checks based on HTTP status code
- Can route based on HTTP attributes
- Can handle SSL termination at the LB level to offload the API Gateways
##### Pros
- Better control over HTTP traffic
- Can improve fault tolerance with smart health checks
##### Cons
- Slightly high latency due to HTTP packet inspection and processing
- More complex
#### Recommendation
Start with an L4 load balancer for the best latency and simplicity.
