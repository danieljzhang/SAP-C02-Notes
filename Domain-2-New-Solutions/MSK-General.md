# Amazon MSK (Managed Streaming for Apache Kafka) — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). MSK is the answer when the question mentions Kafka, existing Kafka workloads, or Kafka ecosystem tools.

---

## 1. Overview

Amazon MSK is a fully managed Apache Kafka service. It provisions, configures, and maintains Kafka brokers and ZooKeeper nodes. You retain full Kafka API compatibility.

**When to choose MSK over Kinesis:**
- Existing Kafka workloads or Kafka expertise
- Need Kafka ecosystem tools (Kafka Connect, Kafka Streams, Schema Registry)
- Very high throughput requirements
- Long message retention (days to forever)
- Multiple consumer groups with independent offsets

---

## 2. MSK vs Kinesis Data Streams

| Feature | MSK | Kinesis Data Streams |
|---|---|---|
| API | Kafka API | AWS proprietary |
| Ecosystem | Full Kafka ecosystem | AWS-native only |
| Retention | Configurable (unlimited) | 24h–365 days |
| Consumer groups | Multiple independent | Multiple (enhanced fan-out) |
| Management | More (broker config, ZooKeeper) | Less (shard management only) |
| Throughput | Very high | High (per shard) |
| Pricing | Per broker hour | Per shard hour + data |
| Use case | Kafka migration, ecosystem tools | AWS-native streaming |

---

## 3. MSK Serverless

- No broker management — fully serverless
- Automatically scales capacity
- Pay per usage (storage + throughput)
- Supports standard Kafka APIs
- Use for: variable workloads, new Kafka applications without capacity planning

---

## 4. MSK Connect

- Managed Kafka Connect — run connectors without managing workers
- Source connectors: ingest data from databases, S3, etc. into Kafka
- Sink connectors: deliver Kafka data to S3, OpenSearch, databases, etc.
- Auto-scaling connector workers

---

## 5. Security

- **Encryption in transit:** TLS between clients and brokers
- **Encryption at rest:** KMS
- **Authentication:** TLS mutual auth, SASL/SCRAM, IAM (MSK Serverless)
- **VPC:** brokers deployed in your VPC subnets
- **Private connectivity:** access via VPC, VPN, or Direct Connect

---

## 6. Exam Scenarios

**Scenario 1:** A company is migrating an on-premises Kafka cluster to AWS. They use Kafka Connect, Kafka Streams, and Schema Registry. What is the solution?

> Use **Amazon MSK**. It provides full Kafka API compatibility, supports Kafka Connect via MSK Connect, and is compatible with Kafka Streams and Schema Registry. Minimal application changes required.

**Scenario 2:** A company needs to stream data from an RDS database into a Kafka topic for real-time processing. What is the solution?

> Use **MSK Connect** with a Debezium source connector (CDC). Debezium captures row-level changes from RDS and publishes them to MSK topics. Downstream consumers process the change events in real-time.

---

## 7. Exam Tips

- **MSK** = Kafka compatibility, ecosystem tools, existing Kafka workloads
- **Kinesis** = AWS-native, simpler, no Kafka expertise needed
- **MSK Serverless** = no broker management, variable workloads
- **MSK Connect** = managed Kafka Connect — source and sink connectors
- MSK brokers run in **your VPC** — not publicly accessible by default
- If the question mentions **Kafka**, the answer is almost always MSK

---

---

## 🎯 Architecture Decision Notes

### MSK provisioned vs MSK Serverless vs Kafka on EC2

| | MSK provisioned | MSK Serverless | Kafka on EC2 |
|---|---|---|---|
| Broker management | AWS manages brokers; you size them | None — fully serverless | You manage everything |
| Scaling | Manual broker/storage scaling | Automatic | Manual |
| VPC | Your VPC subnets | AWS-managed networking | Your VPC |
| IAM auth | Yes | Yes (required style) | DIY |
| Cost model | Per broker-hour | Per throughput + storage | EC2 + your labor |
| Choose when | Predictable throughput, need broker-level control (instance types, configs) | Variable/spiky workloads, no capacity planning | Deep Kafka customization, unsupported configs |

Decision rule: default to MSK provisioned for steady Kafka workloads where
you want configuration control; MSK Serverless for new apps with
unpredictable traffic. Kafka on EC2 is only justified when MSK's managed
constraints block a required configuration — the exam almost never rewards
it.

### Tiered storage — the cost lever

MSK tiered storage offloads older topic data to S3-backed low-cost storage
while keeping recent data on broker disks. Hot data stays fast; cold data
stays cheap and still consumable.

Exam trap: "retain 2 years of events, minimize cost" — the naive answer
provisions enormous broker EBS volumes. Tiered storage is the designed
answer: long retention without paying hot-storage prices for cold data.
Pair with a retention policy per topic, not one blanket setting.

### Replication and multi-cluster patterns

- **MSK Replicator** (managed MirrorMaker 2): replicate topics across MSK
  clusters, including cross-Region. The DR story for Kafka on AWS —
  "Region failover for Kafka workloads" points here, not to stretching one
  cluster across Regions.
- Do not confuse topic replication (Replicator) with consumer offset
  management — offsets are per-cluster; failover runbooks must address
  consumer position, not just data copy.

### Security decisions that actually get tested

- **Encryption in transit** is TLS; **at rest** is KMS — but the exam tests
  *authentication*: SASL/SCRAM (username/password in Secrets Manager),
  mTLS (client certificates), or **IAM auth** (roles/policies, no secret
  rotation). "No credential rotation overhead" → IAM auth.
- Brokers live in **your VPC subnets** with no public endpoint by default.
  "On-premises producers need to reach MSK" → private connectivity via
  DX/VPN, not opening brokers to the internet.

### Scaling without guessing

- **Storage autoscaling** expands broker EBS automatically — enable it
  rather than over-provisioning "just in case."
- **Broker scaling** (adding brokers / changing instance types) is a
  deliberate operation with partition-rebalancing implications. The exam
  rewards knowing that throughput problems are solved by partitions first
  (more partitions → more parallelism), then brokers.
- Monitor with CloudWatch broker metrics plus JMX/Prometheus scraping for
  consumer lag — lag, not CPU, is the Kafka health signal that matters.

**Fresh example:** A team migrates a Kafka workload with 90-day retention
and spiky traffic. They choose MSK provisioned (steady baseline, custom
broker configs their Streams app needs), enable tiered storage so the
90-day tail doesn't inflate EBS costs, use MSK Replicator to a second
Region for DR, and authenticate producers with IAM to avoid secret
rotation. Each choice answers a distinct requirement word in the scenario.

## 8. Official References

- [MSK Developer Guide](https://docs.aws.amazon.com/msk/latest/developerguide/)
- [MSK Serverless](https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html)
- [MSK Connect](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect.html)
