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

## 8. Official References

- [MSK Developer Guide](https://docs.aws.amazon.com/msk/latest/developerguide/)
- [MSK Serverless](https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html)
- [MSK Connect](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect.html)
