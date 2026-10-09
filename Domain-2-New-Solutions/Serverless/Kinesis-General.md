# Amazon Kinesis — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). Kinesis is the real-time streaming platform. Know Data Streams vs Firehose vs Data Analytics and when to use Kinesis vs SQS vs MSK.

---

## 1. Kinesis Services Overview

| Service | Purpose |
|---|---|
| Kinesis Data Streams | Real-time data streaming, custom consumers |
| Kinesis Data Firehose | Managed delivery to S3, Redshift, OpenSearch, Splunk |
| Kinesis Data Analytics | Real-time SQL/Apache Flink analytics on streams |
| Kinesis Video Streams | Stream video from devices for ML/analytics |

---

## 2. Kinesis Data Streams

### Architecture

```
Producers → Kinesis Data Stream (shards) → Consumers
(EC2, Lambda,    [shard 1] [shard 2]        (Lambda, KCL app,
 IoT, mobile)   [shard 3] [shard 4]          Firehose, Analytics)
```

### Shards

- Each shard: **1 MB/s write**, **2 MB/s read**
- Scale by adding/removing shards (resharding)
- **Enhanced fan-out:** 2 MB/s per consumer per shard (dedicated throughput)
- Partition key determines which shard receives the record

### Retention

- Default: **24 hours**
- Extended: up to **365 days** (additional cost)
- Records are immutable — consumers track position with sequence numbers

### Consumers

| Consumer Type | Throughput | Use Case |
|---|---|---|
| Standard (polling) | 2 MB/s shared across all consumers | Low-volume, cost-sensitive |
| Enhanced fan-out | 2 MB/s per consumer per shard | Multiple consumers, low latency |

### Lambda Integration

- Lambda polls shards (event source mapping)
- Batch size: 1–10,000 records
- Parallelization factor: up to 10 concurrent Lambda invocations per shard
- Failed batches: retry until success or record expires (use bisect on error + DLQ)

---

## 3. Kinesis Data Firehose

- Fully managed delivery stream — no shards, no consumers to manage
- **Near real-time:** buffers data (60 seconds or 1–128 MB) before delivery
- Destinations: S3, Redshift, OpenSearch, Splunk, HTTP endpoints, Datadog
- Built-in data transformation via Lambda
- Built-in format conversion: JSON → Parquet/ORC (for Athena/Redshift Spectrum)
- Automatic scaling — no capacity management

```
Producers → Firehose → (optional Lambda transform) → S3 / Redshift / OpenSearch
```

---

## 4. Kinesis Data Analytics

- Run SQL queries or Apache Flink applications on streaming data
- Input: Kinesis Data Streams or Firehose
- Output: Kinesis Data Streams, Firehose, Lambda
- Use cases: real-time dashboards, anomaly detection, time-series analytics

---

## 5. Kinesis vs SQS vs MSK

| Feature | Kinesis Data Streams | SQS | MSK (Kafka) |
|---|---|---|---|
| Ordering | Per shard | FIFO only | Per partition |
| Retention | 24h–365 days | Up to 14 days | Configurable (days–forever) |
| Consumers | Multiple (replay) | One per message | Multiple consumer groups |
| Replay | Yes | No | Yes |
| Throughput | MB/s per shard | Unlimited | Very high |
| Management | Low | None | Higher (MSK managed) |
| Use case | Real-time streaming, replay | Work queues, decoupling | Kafka ecosystem, high throughput |

**Key differentiator:** Kinesis and Kafka support **replay** (re-read old records). SQS does not — messages are deleted after consumption.

---

## 6. Exam Scenarios

**Scenario 1:** A company needs to ingest clickstream data from a website and deliver it to S3 for analysis, with minimal operational overhead. What is the solution?

> Use **Kinesis Data Firehose**. Configure the website to send events to Firehose. Firehose buffers and delivers to S3 automatically. Enable format conversion to Parquet for cost-effective Athena queries. No shard management needed.

**Scenario 2:** A company needs multiple independent applications to process the same stream of IoT sensor data in real-time, each at their own pace. What is the solution?

> Use **Kinesis Data Streams** with **Enhanced Fan-Out**. Each consumer application gets dedicated 2 MB/s throughput per shard. Multiple consumers can read the same stream independently without competing for throughput.

**Scenario 3:** A company needs to detect anomalies in real-time from a stream of financial transactions. What is the solution?

> Use **Kinesis Data Streams** as the input. Use **Kinesis Data Analytics** with Apache Flink to run real-time anomaly detection. Route anomalous transactions to an SNS topic via a Kinesis Data Streams output for alerting.

**Scenario 4:** A company needs to replay the last 7 days of events to reprocess data after a bug fix. What streaming service supports this?

> **Kinesis Data Streams** with extended retention (up to 365 days). Consumers can reset their sequence number to replay from any point within the retention window. SQS does not support replay.

---

## 7. Exam Tips

- **Firehose** = managed delivery, near real-time, no consumer code — for S3/Redshift/OpenSearch
- **Data Streams** = real-time, custom consumers, replay — for complex processing
- **Enhanced fan-out** = dedicated throughput per consumer — use when multiple consumers compete
- Kinesis supports **replay**; SQS does not — key differentiator
- **Partition key** determines shard — hot partitions cause throttling; use random or high-cardinality keys
- Firehose is **not real-time** — minimum 60-second buffer; use Data Streams for sub-second latency
- **Kinesis vs MSK:** Kinesis = AWS-native, simpler; MSK = Kafka compatibility, existing Kafka ecosystem

---

---

## 🎯 Architecture Decision Notes

### Kinesis Data Streams vs Firehose — who controls processing

- **Kinesis Data Streams** keeps an ordered, replayable stream that *your*
  consumers read with their own positions. Choose it when multiple
  applications each need the full stream, custom processing, or replay.
- **Firehose** manages delivery and buffering to destinations (S3, Redshift,
  OpenSearch, HTTP endpoints) with built-in transforms. Choose it when the
  goal is managed delivery, not custom stream processing.

Do not pay the operational cost of shard management and consumer-position
bookkeeping when the workload is really "land this data somewhere reliably"
— that is Firehose's job.

## 8. Official References

- [Kinesis Data Streams Developer Guide](https://docs.aws.amazon.com/streams/latest/dev/)
- [Kinesis Data Firehose Developer Guide](https://docs.aws.amazon.com/firehose/latest/dev/)
- [Amazon Managed Service for Apache Flink (formerly Kinesis Data Analytics)](https://docs.aws.amazon.com/managed-flink/latest/java/what-is.html)
