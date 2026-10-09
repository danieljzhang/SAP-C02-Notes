# Amazon RDS and Aurora — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). RDS and Aurora are the primary relational database options. Know Multi-AZ vs read replicas, Aurora Global Database, Aurora Serverless, and when to choose Aurora over RDS.

---

## 1. RDS Overview

Amazon RDS is a managed relational database service supporting: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Db2.

### Multi-AZ Deployment

- Synchronous replication to a standby in another AZ
- Automatic failover in 1–2 minutes (DNS update)
- Standby is **not readable** — for HA only, not read scaling
- Supports Multi-AZ DB Cluster (2 readable standbys) for MySQL and PostgreSQL

### Read Replicas

- Asynchronous replication — slight lag
- Up to 15 read replicas (MySQL/PostgreSQL/MariaDB)
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

Aurora is AWS's cloud-native relational database — compatible with MySQL and PostgreSQL but with significantly better performance and availability.

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
| Storage | Auto-scales to 128 TB | Manual provisioning |
| Replicas | Up to 15, sub-10ms lag | Up to 15, seconds of lag |
| Failover | ~30 seconds | 1–2 minutes |
| Cost | ~20% more than RDS | Lower |
| Multi-AZ | Built-in (6 copies) | Separate standby |
| Backtrack | Yes (MySQL) | No |

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

- Replication lag: typically < 1 second
- Up to 5 secondary regions
- **Managed planned failover:** promotes secondary with no data loss
- **Unplanned failover:** promote secondary; potential data loss up to replication lag
- Use for: global read scaling, multi-region DR, data sovereignty

---

## 4. Aurora Serverless v2

- Automatically scales compute capacity based on load
- Scales in fine-grained increments (Aurora Capacity Units — ACUs)
- Supports all Aurora features (Global Database, Multi-AZ, read replicas)
- Minimum capacity: 0.5 ACU; Maximum: 128 ACU
- Use for: variable workloads, dev/test, multi-tenant SaaS

### Aurora Serverless v1 vs v2

| Feature | v1 | v2 |
|---|---|---|
| Scaling | Coarse, with pause | Fine-grained, instant |
| Pause to zero | Yes | No |
| Multi-AZ | No | Yes |
| Global Database | No | Yes |
| Use case | Infrequent, can tolerate cold start | Variable production workloads |

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

> Use **Aurora** (MySQL or PostgreSQL compatible). Aurora stores 6 copies across 3 AZs. Failover to a read replica takes ~30 seconds. No data loss because storage is shared across all instances.

**Scenario 2:** A company needs a global database with read access in 3 regions and the ability to promote a secondary region to primary in a DR event. What is the solution?

> Use **Aurora Global Database**. Deploy the primary cluster in us-east-1. Add secondary clusters in eu-west-1 and ap-southeast-1. Replication lag is typically < 1 second. In a DR event, promote the secondary cluster using managed failover.

**Scenario 3:** A SaaS application has highly variable database load — near zero at night, high during business hours. What is the most cost-effective database solution?

> Use **Aurora Serverless v2**. It scales automatically based on load, charging only for ACUs consumed. No need to provision for peak capacity. Supports all Aurora features including Multi-AZ.

**Scenario 4:** A developer accidentally deleted critical data from an Aurora MySQL database. How can it be recovered quickly?

> Use **Aurora Backtrack** to rewind the database to a point before the deletion. Backtrack is faster than point-in-time recovery because it does not require restoring from a snapshot — it rewinds the existing cluster.

---

## 8. Exam Tips

- **Multi-AZ** = HA (automatic failover); **Read Replicas** = read scaling + cross-region DR
- **Aurora** = better HA, faster failover, auto-scaling storage — prefer over RDS for most new workloads
- **Aurora Global Database** = multi-region active-read with < 1 second replication lag
- **Aurora Serverless v2** = variable workloads; **v1** = infrequent workloads that can pause
- **Backtrack** = fast rewind for Aurora MySQL — not a backup, not cross-region
- Encryption must be enabled **at creation** — cannot encrypt an existing unencrypted RDS/Aurora instance (must snapshot + restore encrypted)
- **IAM database authentication** = no passwords, short-lived tokens — good for applications using IAM roles

---

## 9. Official References

- [RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/)
- [Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
