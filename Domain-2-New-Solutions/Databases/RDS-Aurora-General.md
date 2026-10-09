# Amazon RDS and Aurora — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). Compare managed relational options by engine compatibility, application requirements, availability, replication, operations, and cost. Know Multi-AZ versus read replicas, Aurora Global Database, and current Aurora Serverless options.

---

## 1. RDS Overview

Amazon RDS is a managed relational database service supporting: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Db2.

### Multi-AZ Deployment

- Synchronous replication to a standby in another AZ
- Automatic failover; duration and client recovery depend on engine/deployment, connection handling, and DNS/client caching
- Standby is **not readable** — for HA only, not read scaling
- Supports Multi-AZ DB Cluster (2 readable standbys) for MySQL and PostgreSQL

### Read Replicas

- Asynchronous replication — slight lag
- Read-replica counts and supported engines/features vary; check current quotas and engine documentation
- Can be in different regions (cross-region read replicas)
- Can be promoted to standalone DB (for DR or migration)
- Readable — offload read traffic from primary

### Multi-AZ vs Read Replicas

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High availability | Read scaling, DR |
| Replication | Synchronous | Asynchronous |
| Readable | No (standby) | Yes |
| Failover | Automatic | Manual promotion |
| Cross-region | No (same region) | Yes |

---

## 2. Aurora Overview

Aurora is an AWS relational database engine compatible with MySQL and PostgreSQL. Compatibility does not guarantee identical engine behavior; performance and availability depend on engine/version, cluster design, instance class, and workload. Benchmark representative queries and transactions.

### Aurora Architecture

```
Aurora Cluster
├── Primary instance (read/write)
├── Aurora Replica 1 (read-only, same region)
├── Aurora Replica 2 (read-only, same region)
└── Shared cluster storage (6 copies across 3 AZs)
```

- **Cluster storage:** automatically replicates 6 copies across 3 AZs — survives loss of 2 copies for writes, 3 for reads
- **Cluster endpoint:** always points to primary (read/write)
- **Reader endpoint:** load balances across all replicas
- **Instance endpoint:** connects to a specific instance

### Aurora vs RDS

| Feature | Aurora | RDS |
|---|---|---|
| Storage | Cluster storage scales within engine/service limits | Storage behavior depends on engine and configuration |
| Replicas | Aurora replicas share cluster storage; lag/failover behavior depends on topology | Read replicas are asynchronous; behavior and limits vary by engine |
| Failover | Depends on cluster health, replica readiness, client reconnection, and engine | Depends on deployment type, engine, client reconnection, and DNS caching |
| Cost | Compare instance, storage, I/O, replication, and operations for the workload | Compare the complete target configuration and licensing |
| Multi-AZ | Storage is distributed across AZs; compute topology still matters | Multi-AZ options vary; distinguish standby deployments from readable Multi-AZ DB clusters |
| Backtrack | Available for supported Aurora MySQL configurations | Not an RDS general feature |

---

## 3. Aurora Global Database

Spans multiple AWS regions with fast replication:

```
Primary Region (us-east-1)
└── Aurora Primary Cluster (read/write)
        └── Replication (~1 second lag)
                └── Secondary Region (eu-west-1)
                        └── Aurora Secondary Cluster (read-only)
                                └── Can be promoted in DR event
```

- Replication lag is typically under a second, but is not a hard end-to-end guarantee
- One primary writer Region and read-only secondary Regions; supported write forwarding sends secondary-region writes to the primary
- Secondary-region count, supported engine versions, and Region combinations change; check current configuration requirements
- **Managed planned failover:** promotes secondary with no data loss
- **Unplanned failover:** promote secondary; potential data loss up to replication lag
- Use for: global read scaling, multi-region DR, data sovereignty

---

## 4. Aurora Serverless v2

- Automatically scales compute capacity based on load
- Scales in fine-grained increments (Aurora Capacity Units — ACUs)
- Feature compatibility depends on engine/version and configuration; verify current support before selecting Global Database, replicas, or other features
- Capacity range and scale-to-zero behavior depend on current engine/version and configuration; check current documentation and quotas rather than memorizing a fixed range
- Use for: variable workloads, dev/test, multi-tenant SaaS

### Aurora Serverless v1 vs v2

| Feature | v1 | v2 |
|---|---|---|
| Scaling | Legacy generation; verify current lifecycle and migration notices | Current Serverless generation; behavior depends on engine/version/configuration |
| Pause to zero | Historical behavior; verify current availability | Check current auto-pause support and minimum capacity |
| Multi-AZ | Verify current support before use | Feature support varies by engine/version |
| Global Database | Verify current support before use | Feature support varies by engine/version |
| Use case | Existing deployments only where current support permits | Variable workloads when scaling behavior and economics fit |

---

## 5. Backups and Recovery

| Feature | RDS | Aurora |
|---|---|---|
| Automated backups | 1–35 days retention | 1–35 days retention |
| Manual snapshots | Yes | Yes |
| Point-in-time recovery | Yes (5-minute granularity) | Yes (1-second granularity) |
| Backtrack (MySQL) | No | Yes — rewind without restore |
| Cross-region backup | Copy snapshot | Copy snapshot |

**Aurora Backtrack:** rewind the database to a specific point in time without restoring from backup. Faster than PITR. Supported for Aurora MySQL only.

---

## 6. Security

- **Encryption at rest:** KMS (must enable at creation — cannot enable later on existing unencrypted DB)
- **Encryption in transit:** SSL/TLS
- **IAM database authentication:** use IAM tokens instead of passwords (MySQL, PostgreSQL)
- **VPC:** deploy in private subnets; use security groups to control access
- **Secrets Manager:** rotate database credentials automatically

---

## 7. Exam Scenarios

**Scenario 1:** A company needs a relational database that can survive an AZ failure with automatic failover and no data loss. What is the solution?

> Compare an Aurora cluster with suitable replicas and a supported RDS Multi-AZ deployment against the specific RPO/failover requirement. Aurora's distributed cluster storage and failover-ready compute can be a strong fit, but validate the exact engine, topology, client reconnect behavior, and recovery objective. Do not infer a numeric failover time or zero data loss from the service name alone.

**Scenario 2:** A company needs a global database with read access in 3 regions and the ability to promote a secondary region to primary in a DR event. What is the solution?

> Use **Aurora Global Database**. Deploy the primary cluster in us-east-1. Add secondary clusters in eu-west-1 and ap-southeast-1. Replication lag is typically < 1 second. In a DR event, promote the secondary cluster using managed failover.

**Scenario 3:** A SaaS application has highly variable database load — near zero at night, high during business hours. What is the most cost-effective database solution?

> Evaluate **Aurora Serverless v2** if its supported engine, scaling behavior, and workload economics fit. Model database compute, storage, I/O, networking, and baseline capacity; compare against provisioned Aurora/RDS and test scaling under realistic load. Serverless does not remove all capacity planning or guarantee the lowest total cost.

**Scenario 4:** A developer accidentally deleted critical data from an Aurora MySQL database. How can it be recovered quickly?

> Use **Aurora Backtrack** to rewind the database to a point before the deletion. Backtrack is faster than point-in-time recovery because it does not require restoring from a snapshot — it rewinds the existing cluster.

---

## 8. Exam Tips

- **Multi-AZ** = HA (automatic failover); **Read Replicas** = read scaling + cross-region DR
- **Aurora vs RDS** = compare requirements and total cost; do not assume Aurora is best for every relational workload
- **Aurora Global Database** = one writer Region with read-only secondaries; replication is typically fast, not an unconditional lag guarantee
- **Aurora Serverless v2** = variable workloads when current feature support and economics fit; treat v1 as legacy and verify lifecycle notices
- **Backtrack** = fast rewind for Aurora MySQL — not a backup, not cross-region
- Encryption must be enabled **at creation** — cannot encrypt an existing unencrypted RDS/Aurora instance (must snapshot + restore encrypted)
- **IAM database authentication** = no passwords, short-lived tokens — good for applications using IAM roles

---

## 9. 🎯 Architecture Decision Notes

### Choosing the relational engine and capacity model

- **RDS** fits steady relational workloads with a known engine; read
  replicas scale reads, but writes still funnel through one primary.
- **Aurora** adds storage auto-scaling, faster failover, and Global
  Database for cross-Region reads with minute-scale failover.
- **Aurora Serverless v2** scales compute in fine increments — the answer
  for spiky relational workloads (e.g., 10x ticket on-sales) where
  provisioned-for-peak would idle expensively. It keeps MySQL/PostgreSQL
  compatibility, so no data-layer rewrite.
- **Do not pick DynamoDB for a relational workload** just because "it
  scales" — the key-value remodel is the real project cost.

For point-in-time recovery, verify the backup retention window covers the
required RPO rather than assuming defaults. Backtrack is not a substitute
for tested restores.

---

### Aurora replica auto-scaling vs storage auto-scaling — two different knobs

Aurora scales in two independent dimensions, and exam questions test them separately:

- **Storage auto-scaling is automatic.** You never pre-provision Aurora storage; the cluster volume grows (and can shrink) with actual data. No policy, no action — this is just how Aurora works.
- **Compute auto-scaling is a policy you define.** Aurora Auto Scaling adds/removes read replicas based on CloudWatch metrics (e.g., CPU or connections), bounded by the min/max replica counts in your scaling policy, with a cooldown to prevent flapping. It needs a primary plus at least one replica to start from, and it manages CloudWatch alarms behind the scenes.

Decision rule: storage handles itself; if the workload is *read-spiky*, add an Aurora Auto Scaling policy on replicas. If the workload is *write-heavy*, replica scaling won't help — the writer is still one instance (consider Aurora Serverless v2 or sharding instead).

**Fresh example:** A ticketing app sees 10x read traffic for two hours on onsale days. A policy "min 1, max 8 replicas, target 60% CPU" absorbs the spike and scales back in overnight — cheaper than running 8 replicas year-round.

### Aurora's fast crash recovery — why restart times are short

After a crash, a traditional database replays redo logs from the last checkpoint before serving traffic. Aurora skips this: its storage layer is distributed and self-healing, and the buffer cache lives outside the database process, so a restart typically completes in under a minute with a warm cache — no brownout while the cache repopulates. Practical upshot: Aurora failovers and instance reboots are less disruptive than the same events on classic RDS, but client reconnection and DNS behavior still matter (see below).

### RDS Multi-AZ failover — what actually changes for the client

On automatic failover, RDS updates the DB instance endpoint's DNS record to point at the promoted standby. Applications that connect through the endpoint reconnect transparently — *if* they honor the new DNS answer. The classic trap: an app that caches the old IP (JVM DNS caching, long-lived connection pools) keeps hammering the dead primary. Mitigations: use the endpoint (never hardcode IPs), keep DNS TTLs short on the client side, and consider RDS Proxy for workloads that can't tolerate the failover window. RDS Proxy also bypasses client DNS caches entirely, cutting Aurora failover times for the application.

### When the workload is Oracle — know what RDS can't do

RDS for Oracle is managed, but it does **not** support Oracle RAC, multitenant containers, Database Vault, or Unified Auditing. If the requirement names RAC (or any unsupported feature), the answer is Oracle on EC2 — typically EBS-backed instances across AZs with automated snapshots (Data Lifecycle Manager) and patching via SSM Patch Manager, not CI/CD tools.

For getting Oracle *into* AWS: match the tool to the downtime budget. DMS with continuous replication for minimal-downtime migrations (plus SCT/AWS DMS Schema Conversion for heterogeneous moves); Oracle Data Pump for bulk export/import; RMAN backup-to-S3 with restore onto RDS for offline migrations. Licensing is BYOL vs license-included — a cost decision, not a technical one. RMAN also remains the answer for block-level Oracle backups with compression, encryption via KMS, and cross-region copies.

**Fresh example:** A bank's Oracle RAC cluster must move to AWS with near-zero downtime and keep RAC. Answer: EC2-based Oracle with RAC (RDS can't), DMS for the data move with CDC, SSM Patch Manager for ongoing patching. If RAC weren't required, RDS for Oracle with Multi-AZ would be the simpler answer.

### RDS Proxy — when a proxy beats a bigger instance

RDS Proxy sits between the application and the database, holding a pool of warm connections to RDS/Aurora. Pick it when:

- **Connection churn is the bottleneck**, not query load — classic with Lambda functions, where each invocation opens fresh connections (especially expensive with TLS). The proxy reuses connections so the database stops paying per-connection setup cost.
- **Failovers must be fast and boring.** The proxy preserves application connections across failover, routes to the new writer without waiting on DNS propagation, and queues incoming requests while the writer is down. Only in-flight transactions are interrupted.
- **You need more concurrency than the engine allows.** Infrequently-used connections are shared, so thousands of app-side connections collapse to a small pool on the database.

Trap: the proxy doesn't make slow queries fast and doesn't add read capacity — pair it with read replicas for read scaling. And it isn't free: model proxy cost against the instance downsizing it enables.

**Fresh example:** 200 Lambda functions burst on uploads, each opening a TLS connection to Aurora — the database spends more time handshaking than querying. RDS Proxy in front collapses this to a steady warm pool; failover during a deploy becomes a blip instead of a connection storm.

### Sharding — horizontal scaling when vertical scaling runs out

When one RDS instance can't grow further (or grows too expensively), split the data across multiple instances — shards — each holding a subset (e.g., by customer ID range or hash). Sharding gives horizontal scale-out and can be cheaper than one giant instance, but it moves complexity into the application: cross-shard queries, rebalancing, and transactions across shards are all your problem now.

Decision order before sharding: (1) read replicas for read-heavy skew, (2) Aurora for a higher ceiling, (3) caching (ElastiCache) for hot keys, (4) sharding as the last resort. Don't shard a workload that a replica set or a bigger instance would have solved.

Trap: CDN and static-hosting answers are distractors for database-tier questions — CloudFront and S3 website hosting do nothing for a relational bottleneck.

---

### Global relational vs global key-value — the write pattern decides

Three services promise "global." Only one matches your write pattern:

| | Aurora Global Database | DynamoDB global tables | RDS cross-Region read replicas |
|---|---|---|---|
| Writers | **One** primary Region | **Multi-active** writes in every Region | One primary; replicas are read-only |
| Data model | Relational (MySQL/Postgres) | Key-value / document | Relational |
| Failover | Promote a secondary (minutes) | No failover — already writing everywhere | Promote a replica |
| Pick when | Relational + single writer + global reads | Writes happen in multiple Regions | Read scaling on standard RDS |

**Fresh example:** A booking platform takes reservations in
us-east-1, eu-west-1, and ap-southeast-1, all writing to the same dataset.
Aurora Global Database fails the requirement — it has one writer Region.
DynamoDB global tables is the answer *if* the data model fits key-value
access; if it must stay relational with multi-Region writes, the honest
answer is "redesign the data model or accept single-writer."

### Trap callouts

- **"Global" ≠ "active-active writes."** Aurora Global Database is
  single-writer; the secondaries serve reads. Multi-Region writes is a
  DynamoDB-global-tables (or conflict-resolving custom) conversation.
- **RPO language:** "low RPO" for DynamoDB global tables depends on the
  consistency mode chosen — the phrase alone promises nothing. Verify mode
  and topology.
- **Promote ≠ failover magic:** promoting an Aurora secondary or RDS
  replica is a planned operation with application consequences (connection
  strings, DNS, write forwarding). Test it.

### Aurora cloning vs snapshots vs backtrack — three different copies

| | Clone | Snapshot | Backtrack |
|---|---|---|---|
| What | Copy-on-write instant copy of the cluster | Durable point-in-time backup | In-place rewind of the existing cluster |
| Speed | Near-instant regardless of size | Restore time scales with data | Fast rewind, no restore |
| Storage cost | ~Zero until writes diverge | Full copy | Minimal |
| Survives cluster deletion? | No — tied to the source | Yes | No |
| Pick when | Dev/test copies, schema experiments | Durable backups, cross-account/Region copies | "Oops" recovery (Aurora MySQL only) |

**Fresh example:** QA needs a production-data copy every Monday for
testing. Snapshot-restore duplicates terabytes weekly and takes hours.
Aurora cloning gives QA an instant copy at near-zero storage cost; writes
diverge on copy-on-write. The snapshot schedule stays — for *backups*, not
for test data.

**Trap:** Backtrack is not a backup — it cannot survive cluster deletion
and is not a substitute for snapshots. Cloning is not a backup either —
delete the source and the clone's basis is gone.

## 10. Official References

- [RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/)
- [Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
