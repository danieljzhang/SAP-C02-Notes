# Amazon DynamoDB — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). DynamoDB is a managed key-value/document database that can provide low-latency access at scale when the access pattern, partition key, capacity, and consistency model fit. Know partition keys, GSIs, streams, DAX, Global Tables, and capacity modes.

---

## 1. Overview

Amazon DynamoDB is a fully managed NoSQL key-value and document database. It removes database-server provisioning, but table design, capacity mode, quotas, partition-key distribution, consistency, and cost still require architecture decisions. Latency depends on workload and configuration; do not treat a typical service characteristic as an end-to-end guarantee.

**When to choose DynamoDB:**
- Low-latency key-value/document access with well-defined access patterns
- Massive scale (millions of requests/second)
- Simple access patterns (key-value, document)
- Managed operations without database-server provisioning

**When NOT to choose DynamoDB:**
- Complex joins or relational queries → use Aurora/RDS
- ACID transactions across many tables → use Aurora
- Full-text search → use OpenSearch

---

## 2. Data Model

### Primary Key

| Type | Components | Use Case |
|---|---|---|
| Simple (Hash) | Partition key only | Unique items, simple lookups |
| Composite (Hash + Range) | Partition key + sort key | Collections of related items |

- **Partition key:** determines the partition where the item is stored — must be unique for simple PK
- **Sort key:** enables range queries within a partition (begins_with, between, <, >)

### Item Size

- Max item size: **400 KB**
- Attributes can be nested (maps, lists) up to 32 levels deep

---

## 3. Capacity Modes

| Mode | Use Case | Cost |
|---|---|---|
| On-Demand | Unpredictable traffic, new applications | Pay per request |
| Provisioned | Predictable traffic | Pay for provisioned RCU/WCU |
| Provisioned + Auto Scaling | Variable but predictable | Scales RCU/WCU automatically |

- **RCU (Read Capacity Unit):** 1 strongly consistent read of 4 KB/s, or 2 eventually consistent reads
- **WCU (Write Capacity Unit):** 1 write of 1 KB/s
- Switch between modes once per 24 hours

---

## 4. Indexes

### Global Secondary Index (GSI)

- Different partition key and sort key than the base table
- Queries across all partitions
- Has its own provisioned capacity (or on-demand)
- Eventually consistent reads only
- Up to 20 GSIs per table

### Local Secondary Index (LSI)

- Same partition key as base table, different sort key
- Queries within a single partition
- Shares capacity with base table
- Strongly consistent reads supported
- Must be created at table creation time
- Up to 5 LSIs per table

---

## 5. DynamoDB Streams

- Ordered stream of item-level changes (insert, update, delete)
- Retention: 24 hours
- Trigger Lambda functions for real-time processing
- Use cases: replication, audit trail, event-driven workflows, search indexing

```
DynamoDB Table → Stream → Lambda → downstream processing
                                 (ElasticSearch, SQS, another table)
```

---

## 6. DynamoDB Accelerator (DAX)

- In-memory cache for DynamoDB — microsecond read latency
- API-compatible with DynamoDB — minimal code changes
- Write-through cache: writes go to DynamoDB first, then DAX
- Cache TTL: configurable (default 5 minutes for items, 1 minute for queries)
- Deployed in a VPC — requires VPC for Lambda/EC2 access
- Not suitable for: strongly consistent reads, write-heavy workloads

---

## 7. Global Tables

- Multi-Region replicas can serve reads and writes.
- Choose a consistency mode when creating the global table: multi-Region eventual consistency (MREC) or multi-Region strong consistency (MRSC), subject to current Region/configuration constraints.
- MREC replicates asynchronously; concurrent writes can conflict and use last-writer-wins resolution. Design application conflict behavior explicitly.
- MRSC has different consistency, latency, Region, and account constraints; check the current guide before selecting it.
- Replication lag and recovery behavior depend on the selected mode and Regions. Do not treat a typical lag figure as a hard guarantee.
- Capacity and billing choices depend on the global-table mode and current service rules; verify before deployment.

```
us-east-1 Table ←──── replication ────→ eu-west-1 Table
(read/write)                              (read/write)
```

Use when the application benefits from local reads/writes in multiple Regions and can meet the selected consistency/conflict model. A low or zero RPO is not implied by the phrase “Global Tables”; verify the chosen mode, supported topology, and recovery behavior.

---

## 8. Transactions

- **TransactWriteItems:** up to 100 items, all-or-nothing writes across multiple tables
- **TransactGetItems:** up to 100 items, consistent reads across multiple tables
- 2x the cost of standard reads/writes
- Use for: financial transactions, inventory management, order processing

---

## 9. TTL (Time to Live)

- Automatically delete items after a specified timestamp
- No additional cost for TTL deletions
- Deleted items appear in DynamoDB Streams (for audit/archival)
- Use for: session data, temporary tokens, cache entries

---

## 10. Exam Scenarios

**Scenario 1:** A globally distributed application needs local reads and writes in multiple Regions. How should the team choose a DynamoDB Global Tables design?

> Evaluate MREC versus MRSC against consistency, latency, Region, account, and recovery requirements, then validate throughput and partition-key distribution with representative load tests. MREC is asynchronous and requires conflict handling. Do not assume a replication-lag guarantee or that on-demand mode removes throughput/quota design. Add DAX only if its cache consistency model fits the read path.

**Scenario 2:** A company needs to query DynamoDB items by an attribute that is not the primary key. What is the solution?

> Create a **Global Secondary Index (GSI)** on the query attribute as the partition key. GSIs allow queries on any attribute with the same performance as the base table.

**Scenario 3:** A company wants to trigger a Lambda function whenever an item is updated in DynamoDB to update a search index. What is the solution?

> Enable **DynamoDB Streams** on the table. Create a Lambda event source mapping for the stream. Lambda is invoked with the stream records containing the old and new item images. Lambda updates the search index (e.g., OpenSearch).

**Scenario 4:** A read-heavy application is experiencing high DynamoDB costs and latency. What is the solution?

> Add **DynamoDB DAX** as a caching layer. DAX provides microsecond read latency and reduces DynamoDB read capacity consumption. The application uses the DAX client (API-compatible with DynamoDB SDK) with minimal code changes.

---

## 11. Exam Tips

- DynamoDB = managed low-latency key-value/document access when access patterns and partitioning fit — not a relational engine for joins
- **GSI** = query on non-primary-key attributes; **LSI** = alternate sort key, same partition
- **DAX** = microsecond reads, write-through — not for strongly consistent reads
- **Global Tables** = multi-Region reads/writes with MREC or MRSC; conflict and consistency behavior depends on mode
- **Streams** = 24-hour retention, triggers Lambda — use for event-driven patterns
- **On-Demand** = unpredictable traffic; **Provisioned + Auto Scaling** = predictable traffic
- **Transactions** = 2x cost — use only when ACID guarantees are required
- Max item size: **400 KB** — not suitable for large objects (use S3 + DynamoDB pointer)

---

---

## 🎯 Architecture Decision Notes

### DynamoDB design decisions that matter at Professional level

- **Access patterns first:** DynamoDB rewards designs derived from known
  query patterns (single-table design, GSIs for alternate access). A
  relational schema ported unchanged becomes an expensive anti-pattern —
  if the workload needs joins and ad-hoc queries, the answer is usually
  Aurora, not DynamoDB.
- **Capacity mode:** on-demand absorbs spikes without planning; provisioned
  with auto-scaling suits predictable baselines. Model the cost crossover
  instead of defaulting.
- **Recovery:** point-in-time recovery covers the last 35 days; on-demand
  backup covers longer retention. Global tables replicate across Regions
  for multi-active reads/writes — but the data model must fit key-value
  access, and conflict resolution follows last-writer-wins.
- **DAX** caches eligible read paths; it does not fix a bad key design.

---

## 🎯 Architecture Decision Notes

### Reserved capacity — commit where the baseline is boring

For provisioned-mode tables with steady, predictable throughput, DynamoDB reserved capacity lets you pre-commit to a read/write capacity baseline for 1 or 3 years in exchange for a steep discount over on-demand provisioned pricing. It is the DynamoDB analogue of EC2 Reserved Instances: you are buying a cheaper rate for capacity you were going to consume anyway.

Decision rules:

- **Buy reserved capacity** for the stable floor of a provisioned table — the throughput that is there 24/7. Size it to the minimum sustained RCU/WCU, not the peak.
- **Don't reserve the peaks:** bursts and growth belong on auto scaling (or on-demand mode for the spiky portion). Reserved capacity is a commitment — over-committing on a table whose traffic later drops is money burned.
- **Revisit the mode choice first:** if traffic is truly unpredictable, on-demand mode may beat provisioned-plus-reserved on total cost with zero commitment. Model the crossover before locking in.
- Trap: reserved capacity applies to provisioned throughput only. A table on on-demand mode gets no benefit from it — a common exam distractor pairs "unpredictable traffic" with reserved capacity.

**Fresh example:** A SaaS product's `tenants` table sustains 2,000 RCU around the clock with daytime peaks to 6,000. The team reserves 2,000 RCU for one year (the floor, heavily discounted), lets auto scaling handle 2,000–6,000, and keeps a second, spiky analytics table on on-demand. The reservation pays for itself in months; the peaks stay flexible.

---

## 🎯 Architecture Decision Notes

### Throttling is a capacity signal — treat it as one

A throttling exception on a provisioned table means request rate exceeded the table's (or a GSI's) allocated throughput. The SDK retries automatically, so transient throttling is invisible — but sustained throttling is your table telling you the capacity plan is wrong. Don't just "increase retries"; fix the capacity or the access pattern.

Decision playbook, in order:

1. **Check whether it's a hot partition, not a small table.** If one partition key dominates traffic, raising total table throughput barely helps — the partition-level limit still throttles. Fix the key design (add a random suffix, split the hot key) before buying capacity.
2. **Raise provisioned throughput or enable auto scaling** when traffic genuinely grew. Auto scaling handles gradual growth; step increases handle known events (product launch, batch import).
3. **Smooth the write pattern:** batch writes, buffer bursts through SQS/Kinesis, or spread load over time instead of hammering the table in spikes.
4. **Switch modes** if the workload changed character: a table that went from predictable to spiky may belong on on-demand.
5. Remember GSIs have **independent** provisioned throughput — a table that looks healthy can still throttle on a hot index.

**Fresh example:** After a marketing email goes out, a `coupons` table throttles for 20 minutes every campaign. The table's average throughput is tiny, but redemptions all hit the single `campaign-2026-q4` partition key. The fix isn't more RCU — it's salting the partition key (`campaign-2026-q4#<random 1-10>`) so redemptions spread across ten partitions, plus a temporary auto-scaling bump for campaign days.

## 12. Official References

- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [DAX](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
