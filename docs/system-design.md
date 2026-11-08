

\# Loki Adaptive Ingestion Control



\## 1. Problem Statement



In a multi-tenant logging platform, a sudden increase in log volume from one tenant can put pressure on distributors and ingesters.



This proposal introduces adaptive ingestion control to enforce tenant-level limits, handle bursts, and protect the ingestion pipeline.



\## 2. Design Goals



\- Maintain predictable ingestion performance.

\- Prevent noisy tenants from affecting other tenants.

\- Support controlled traffic bursts.

\- Minimize latency added by quota checks.

\- Preserve compatibility with existing Loki ingestion APIs.



\## 3. Architecture Diagram



```mermaid

flowchart TD

&#x20;   A\["Log Producers"] --> B\["Distributor"]

&#x20;   B --> C\["Tenant Identification"]

&#x20;   C --> D\["Adaptive Quota Controller"]

&#x20;   D --> E{"Quota Available?"}

&#x20;   E -->|Yes| F\["Ingester Ring"]

&#x20;   E -->|No| G\["Backpressure Handler"]

&#x20;   G --> H\["HTTP 429 + Retry-After"]

&#x20;   F --> I\["Replicated Chunks"]

&#x20;   I --> J\["Object Storage"]

&#x20;   D --> K\["Cached Tenant Policies"]

&#x20;   K --> L\[("Policy Configuration Store")]

&#x20;   D --> M\["Usage Metrics"]

&#x20;   M --> N\["Monitoring and Alerts"]

```



\## 4. Component Responsibilities



| Component | Responsibility |

|---|---|

| Distributor | Receives and routes log batches |

| Tenant Resolver | Identifies the requesting tenant |

| Quota Controller | Checks rate and burst limits |

| Backpressure Handler | Rejects or delays excess traffic |

| Ingester Ring | Processes and replicates accepted logs |

| Configuration Store | Maintains tenant policies |

| Monitoring | Tracks ingestion health and usage |



\## 5. Data Model



\### Tenant Policy



\- tenant\_id: Tenant identifier

\- max\_ingestion\_rate: Allowed bytes per second

\- burst\_limit: Maximum burst capacity

\- enabled: Policy status

\- updated\_at: Last modification time



\### Usage Window



\- tenant\_id: Tenant identifier

\- window\_start: Start of measurement window

\- bytes\_ingested: Accepted ingestion volume

\- rejected\_requests: Number of rejected batches



Policies can be stored in a relational configuration store, while high-frequency counters should remain in memory or an appropriate distributed counter system.



\## 6. Scaling Strategy



\- Scale distributors horizontally behind a load balancer.

\- Partition quota state by tenant to avoid centralized bottlenecks.

\- Use local policy caches to reduce configuration-store traffic.

\- Use bounded queues to protect downstream ingesters.

\- Keep usage aggregation asynchronous.

\- Apply separate limits for individual tenants.

\- Monitor high-cardinality and noisy-tenant patterns carefully.



\## 7. Reliability and Failure Handling



\- Reject requests that exceed configured limits with HTTP 429.

\- Provide Retry-After guidance to clients.

\- Use timeouts and controlled retries for configuration refreshes.

\- Continue using the last valid cached policy during temporary configuration-store outages.

\- Avoid unbounded queues and uncontrolled retries.

\- Preserve existing replication behavior for accepted log batches.



\## 8. Monitoring



Track the following metrics:



\- Bytes ingested per tenant

\- Accepted and rejected requests

\- Distributor request latency

\- Ingester utilization

\- Quota enforcement latency

\- Retry rates

\- Configuration refresh failures



\## 9. Trade-offs



Local caching improves request latency but can temporarily delay policy changes.



Strict rate limiting protects cluster resources but can reject legitimate bursts.



Distributed quota coordination provides stronger global enforcement but introduces additional communication and operational complexity.



The proposed approach prioritizes low-latency checks and predictable ingestion behavior.



\## 10. Conclusion



Adaptive ingestion control can help Loki maintain stable performance during traffic spikes while providing fair resource allocation across tenants.



The design extends the existing ingestion workflow rather than replacing Loki's distributor, ingester, replication, or object-storage architecture.



This is a proposed design, not an implemented feature.

```



Press \*\*Ctrl + S\*\*.



\## Step 5 — Check your folder structure



Your project should now look like this:



```text

grafana-loki-ingestion-system-design/

│

├── README.md

│

└── docs/

&#x20;   └── system-design.md

```



\## Step 6 — Push everything to GitHub



In CMD, run these commands one by one:



```cmd

git add README.md docs/

```



```cmd

git commit -m "Add Loki ingestion reliability system design"

```



```cmd

git push origin main

```



\## Step 7 — Verify the repository



<box border radius="lg" padding={3} gap={2}>

&#x20; \*\*Your GitHub repository\*\*



&#x20; <Link url="https://github.com/varunkarkera/grafana-loki-ingestion-system-design" title="grafana-loki-ingestion-system-design"/>



&#x20; \*\*Your detailed design document\*\*



&#x20; <Link url="https://github.com/varunkarkera/grafana-loki-ingestion-system-design/blob/main/docs/system-design.md" title="Open system design document"/>

</box>



Check that the Mermaid diagram renders and the tables display correctly.



\## Step 8 — Submit the mission



Once the files are pushed, copy this command into your application:



<box border radius="lg" padding={3} gap={2}>

&#x20; \*\*Submission command\*\*

&#x20; `mission submit https://github.com/varunkarkera/grafana-loki-ingestion-system-design`

&#x20; <button block onClick={() => GenUI.copy("mission submit https://github.com/varunkarkera/grafana-loki-ingestion-system-design")}>

&#x20;   <icon name="copy" size="sm"/> Copy submission command

&#x20; </button>

</box>



\*\*Final repository URL:\*\*



https://github.com/varunkarkera/grafana-loki-ingestion-system-design



One important point: this is a design proposal, not a working Loki implementation. Keep that distinction clear when discussing it with Grafana's engineering team.

