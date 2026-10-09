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

## 9. Official References

- [RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/)
- [Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
