

\# Grafana Loki – Ingestion Reliability System Design



A system design proposal for improving log ingestion reliability and handling traffic spikes in Grafana Loki.



\## Overview



This proposal introduces tenant-aware adaptive ingestion control to help Loki handle sudden traffic spikes without overwhelming ingesters or affecting other tenants.



\## Key Features



\- Per-tenant ingestion quotas

\- Adaptive rate limiting and backpressure

\- Protection against traffic spikes

\- Fair resource allocation across tenants

\- Retry handling and overload visibility

\- Real-time ingestion health monitoring



\## Architecture



```mermaid

flowchart TD

&#x20;   A\["Log Producers / Grafana Alloy"] --> B\["Loki Distributor"]

&#x20;   B --> C\["Tenant Identification"]

&#x20;   C --> D\["Ingestion Rate Controller"]

&#x20;   D --> E{"Within Quota?"}

&#x20;   E -->|Yes| F\["Ingester Ring"]

&#x20;   E -->|No| G\["Backpressure / Rate Limit"]

&#x20;   G --> H\["429 Response + Retry-After"]

&#x20;   F --> I\["Replicated Log Chunks"]

&#x20;   I --> J\["Object Storage"]

&#x20;   F --> K\["Ingestion Metrics"]

&#x20;   D --> L\["Quota Configuration"]

&#x20;   L --> M\[("Configuration Store")]

&#x20;   K --> N\["Monitoring and Alerting"]

```



\## Data Model



\### Tenant Ingestion Policy



| Field | Type | Description |

|---|---|---|

| tenant\_id | VARCHAR | Tenant identifier |

| max\_ingestion\_rate | BIGINT | Allowed bytes per second |

| burst\_limit | BIGINT | Maximum temporary burst |

| enabled | BOOLEAN | Policy status |

| updated\_at | TIMESTAMP | Last update |



\### Ingestion Usage



| Field | Type | Description |

|---|---|---|

| tenant\_id | VARCHAR | Tenant identifier |

| window\_start | TIMESTAMP | Usage window |

| bytes\_ingested | BIGINT | Total bytes received |

| rejected\_requests | BIGINT | Rejected request count |



\## Request Flow



1\. Log producers send batches to the Loki distributor.

2\. The distributor identifies the tenant.

3\. The ingestion controller checks the tenant's current quota and burst allowance.

4\. Accepted requests continue through Loki's existing ingester ring and replication flow.

5\. Excess traffic receives a rate-limit response with retry guidance.

6\. Usage metrics are collected to support monitoring and quota adjustments.



\## Scaling and Reliability



\- Keep rate-limit checks lightweight and avoid synchronous database queries on every log batch.

\- Cache tenant policies locally and refresh configuration periodically.

\- Use tenant-aware limits to prevent noisy-neighbor problems.

\- Apply bounded queues to protect ingesters during traffic spikes.

\- Scale distributors and ingesters horizontally.

\- Track accepted bytes, rejected requests, ingestion latency, and retry rates.

\- Use graceful degradation when the policy configuration service is unavailable.

\- Preserve Loki's existing replication and object-storage durability mechanisms.



\## API and Operational Considerations



\- Maintain compatibility with existing Loki push APIs.

\- Return HTTP 429 when tenant ingestion limits are exceeded.

\- Include Retry-After guidance for clients.

\- Expose per-tenant ingestion and rejection metrics.

\- Support configurable limits and controlled policy updates.



\## Trade-offs



\*\*Fairness vs. throughput:\*\* Per-tenant quotas prevent one tenant from consuming disproportionate resources but may restrict legitimate bursts.



\*\*Local caching vs. consistency:\*\* Cached policies reduce latency but can temporarily delay configuration updates.



\*\*Backpressure vs. availability:\*\* Rejecting excess traffic protects the cluster, but clients need reliable retry behavior.



\## Expected Impact



The proposed feature aims to improve ingestion stability during traffic spikes, isolate noisy tenants, reduce overload on ingesters, and provide better visibility into ingestion behavior.



\---



This is an independent system design proposal for the Grafana Labs Senior Backend Software Engineer – Loki application. It is not an implementation of Grafana Loki's internal architecture.



