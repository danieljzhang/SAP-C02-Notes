# Purpose-Built Database Selector — SAP-C02 Study Notes

> Start from access patterns, consistency, latency, scale, query shape, and operational constraints. “NoSQL” and “managed” are not sufficient selection criteria.

## 1. Fast Selection Table

| Workload shape | Candidate | Strong fit when | Main design work |
|---|---|---|---|
| Relational OLTP, joins, transactions | Amazon RDS / Aurora | SQL and relational integrity are central | Engine/version fit, HA, connection behavior, backups, read scaling |
| Key-value/document with known access patterns | DynamoDB | Horizontally scaled, low-latency key access fits the data model | Partition/sort keys, indexes, consistency, hot keys, capacity |
| MongoDB-compatible document workload | DocumentDB | Application uses supported MongoDB APIs/semantics | Compatibility gaps, indexes, query behavior, migration testing |
| Apache Cassandra-compatible wide-column workload | Keyspaces | Cassandra-compatible APIs and partitioned workloads fit | Partition keys, clustering, consistency, throughput, query restrictions |
| Highly connected graph traversal | Neptune | Relationships and graph queries are first-class | Graph model/query language, traversals, replication, workload testing |
| Time-stamped operational metrics/events | Timestream | Time-series ingest, retention, and time-window queries dominate | Memory/magnetic store retention, dimensions, late-arriving data, query cost |
| Data warehouse / BI analytics | Redshift | Large analytical scans, joins, aggregations, and warehouse SQL fit | Distribution/sort strategy, concurrency, ingestion, workload isolation |
| Search, log analytics, text retrieval | OpenSearch | Full-text/search and indexed analytics are required | Index mapping, shard sizing, ingestion, refresh, lifecycle, security |
| In-memory cache | ElastiCache | Repeated reads or ephemeral low-latency state justify caching | Eviction, TTL, invalidation, failover, cache stampede |
| Object data lake / durable files | S3 + analytics services | Data is object-shaped and consumers can query/process it | Layout, catalog, governance, lifecycle, query/scan cost |

## 2. Decision Sequence

1. **Define operations:** point lookup, range, joins, graph traversal, full-text search, time-window aggregation, or analytical scans.
2. **Define correctness:** transactions, read/write consistency, ordering, conflict resolution, and acceptable staleness.
3. **Define scale and shape:** read/write rate, item size, skew/hot keys, retention, growth, and regional distribution.
4. **Choose the data model before the service:** relational schema, key-value/document, wide column, graph, time series, search index, or warehouse.
5. **Plan migration:** engine compatibility, schema/code conversion, data movement, CDC, validation, cutover, rollback, and application changes.
6. **Measure total cost:** compute, storage, I/O, replicas, backup, requests, data transfer, retrieval, licensing, and operational effort.

## 3. Common Near-Misses

- **DynamoDB vs Keyspaces:** access-pattern-first key-value/document design versus Cassandra-compatible wide-column APIs and query model.
- **DocumentDB vs RDS:** document compatibility does not replace relational joins/transactions; validate MongoDB API compatibility for the actual application.
- **Neptune vs OpenSearch:** graph relationship traversals versus text/search and indexed-document retrieval.
- **Timestream vs DynamoDB:** time-series-specific retention/query behavior versus a key-value/document model that can store time-stamped items.
- **Redshift vs RDS/Aurora:** analytical warehouse scans/aggregations versus transactional OLTP.
- **Cache vs source of truth:** ElastiCache can reduce latency/load, but data correctness, invalidation, and cache failure behavior remain application concerns.
- **S3 data lake vs warehouse:** flexible object storage and decoupled analytics versus a managed SQL warehouse with dedicated analytical compute.

## 4. Original Scenario

**Scenario:** A telemetry platform stores device measurements, customer profiles, and device-to-device relationships. Analysts need daily fleet-wide aggregates; the application needs millisecond profile lookups and relationship traversal.

**Recommended approach:** Do not force every workload into one database. Evaluate Timestream for time-series measurements, DynamoDB for known profile access patterns, Neptune for graph traversal if relationship queries justify it, and S3/Redshift or other analytics services for large historical aggregates. Define ingestion, identity, consistency, and data lifecycle between stores.

**Trap:** A single-purpose database rarely optimizes OLTP, graph traversal, time-series retention, and warehouse scans equally well.

## 5. Exam Tips

- Purpose-built databases usually mean multiple fit-for-purpose stores with explicit data ownership and synchronization.
- Engine compatibility is not full behavioral compatibility; test queries, types, transactions, and application drivers.
- A service being managed does not remove schema, consistency, availability, migration, security, or cost decisions.
- Check current feature/engine/Region support, quotas, and pricing before choosing a version-sensitive option.

---

## 🎯 Architecture Decision Notes

### OLAP vs OLTP — pick the database by the query shape, not the data size

The first question about any database requirement is what the queries look like:

- **OLTP (transactional):** many short reads/writes on current data, ACID semantics — shopping carts, banking, CRM. Answers: RDS, Aurora, DynamoDB, DocumentDB.
- **OLAP (analytical):** complex aggregations and joins over large historical datasets — BI dashboards, forecasting, trend analysis. Answers: Redshift, Athena, EMR.

Trap: stuffing an OLAP workload into an OLTP database (or vice versa) "because we already run MySQL" is the most common real-world mistake the exam probes. A retailer's demand-forecasting job scanning years of sales belongs on Redshift or a data lake with Athena, not on the transactional MySQL that serves the storefront — even though both hold "sales data."

### Redshift DR — automated snapshots are on; cross-region copy is not

Redshift enables automated snapshots by default (roughly every 8 hours or per 5 GB-per-node of changes, 1-day default retention, adjustable). But the disaster-recovery pattern the exam tests is the part you must configure yourself: **cross-region snapshot copy**. Automated snapshots stay in their region unless you explicitly enable copy to a DR region with its own retention. If the scenario says "replicate the cluster to another region for DR," the answer is enabling cross-region snapshot copy — not just "turn on automated snapshots," which is already on.

### Timestream — when the data is a timestamp with a value attached

Reach for Timestream when the workload is time-stamped measurements at scale: IoT sensor streams, operational metrics, clickstream analytics. Three properties make it beat a relational or document database here:

- **Purpose-built time-series functions** (aggregates, derivatives, interpolation, correlation) run in the query engine — no hand-rolled window math in application code.
- **Tiered storage**: a memory store for recent data (fast queries) and a magnetic store for history, with lifecycle management between them — hundreds of terabytes without manual partitioning.
- **Scheduled queries**: continuously pre-compute rollups (per-minute, hourly, daily) so dashboards read aggregates instead of scanning raw points.

Decision rule: if the access pattern is "ingest millions of timestamped events, query by time window," Timestream outperforms MySQL-on-EC2 or DocumentDB by orders of magnitude with less operational work. If you need ad-hoc joins across entities or document flexibility, it's the wrong tool — pair it with DynamoDB/DocumentDB for the entity side.

Mini-example: a weather company ingests billions of sensor events daily and builds hourly/daily/weekly trend forecasts; retrieval from MySQL keeps getting slower as data grows. Replatform the database tier to Timestream (time-window queries over the memory store are fast), keep long-running forecast jobs on containers (ECS), and use scheduled queries to pre-compute the hourly rollups the forecasters read.

## 6. Official References

- [AWS database decision guide](https://aws.amazon.com/getting-started/decision-guides/databases-on-aws-how-to-choose/)
- [Amazon DocumentDB](https://docs.aws.amazon.com/documentdb/latest/developerguide/what-is.html)
- [Amazon Keyspaces](https://docs.aws.amazon.com/keyspaces/latest/devguide/what-is-keyspaces.html)
- [Amazon Neptune](https://docs.aws.amazon.com/neptune/latest/userguide/intro.html)
- [Amazon Timestream](https://docs.aws.amazon.com/timestream/latest/developerguide/what-is-timestream.html)
- [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html)
- [SAP-C02 Domain 2 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain2.html)
