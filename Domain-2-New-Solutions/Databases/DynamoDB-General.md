# Amazon DynamoDB — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). DynamoDB is the go-to for single-digit millisecond latency at any scale. Know partition keys, GSIs, streams, DAX, Global Tables, and capacity modes.

---

## 1. Overview

Amazon DynamoDB is a fully managed, serverless NoSQL key-value and document database. It delivers single-digit millisecond performance at any scale with no infrastructure management.

**When to choose DynamoDB:**
- Single-digit millisecond latency requirement
- Massive scale (millions of requests/second)
- Simple access patterns (key-value, document)
- Serverless, no capacity planning

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

- Multi-region, multi-active replication
- All replicas are readable and writable
- Replication lag: typically < 1 second
- Conflict resolution: last-writer-wins (based on timestamp)
- Requires on-demand capacity mode or auto-scaling provisioned

```
us-east-1 Table ←──── replication ────→ eu-west-1 Table
(read/write)                              (read/write)
```

Use for: global applications, multi-region active-active, DR with near-zero RPO.

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

**Scenario 1:** A mobile application needs a database that can handle 1 million writes per second with single-digit millisecond latency globally. What is the solution?

> Use **DynamoDB with Global Tables**. Global Tables provides multi-region active-active replication with < 1 second lag. On-demand capacity mode handles unpredictable traffic spikes. DAX can be added for microsecond read latency.

**Scenario 2:** A company needs to query DynamoDB items by an attribute that is not the primary key. What is the solution?

> Create a **Global Secondary Index (GSI)** on the query attribute as the partition key. GSIs allow queries on any attribute with the same performance as the base table.

**Scenario 3:** A company wants to trigger a Lambda function whenever an item is updated in DynamoDB to update a search index. What is the solution?

> Enable **DynamoDB Streams** on the table. Create a Lambda event source mapping for the stream. Lambda is invoked with the stream records containing the old and new item images. Lambda updates the search index (e.g., OpenSearch).

**Scenario 4:** A read-heavy application is experiencing high DynamoDB costs and latency. What is the solution?

> Add **DynamoDB DAX** as a caching layer. DAX provides microsecond read latency and reduces DynamoDB read capacity consumption. The application uses the DAX client (API-compatible with DynamoDB SDK) with minimal code changes.

---

## 11. Exam Tips

- DynamoDB = **single-digit ms latency at any scale** — not for complex queries or joins
- **GSI** = query on non-primary-key attributes; **LSI** = alternate sort key, same partition
- **DAX** = microsecond reads, write-through — not for strongly consistent reads
- **Global Tables** = multi-region active-active — last-writer-wins conflict resolution
- **Streams** = 24-hour retention, triggers Lambda — use for event-driven patterns
- **On-Demand** = unpredictable traffic; **Provisioned + Auto Scaling** = predictable traffic
- **Transactions** = 2x cost — use only when ACID guarantees are required
- Max item size: **400 KB** — not suitable for large objects (use S3 + DynamoDB pointer)

---

## 12. Official References

- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [DAX](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
