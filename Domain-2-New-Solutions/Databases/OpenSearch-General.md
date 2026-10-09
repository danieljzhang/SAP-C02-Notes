# Amazon OpenSearch Service — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). OpenSearch is the answer for full-text search and log analytics. Know ingestion patterns, domain design, and integration with CloudWatch and Kinesis.

---

## 1. Overview

Amazon OpenSearch Service (formerly Elasticsearch Service) is a managed search and analytics engine. It is used for full-text search, log analytics, real-time application monitoring, and clickstream analytics.

**When to choose OpenSearch:**
- Full-text search with relevance scoring
- Log aggregation and analysis (ELK stack replacement)
- Real-time analytics on streaming data
- Geospatial search

---

## 2. Core Concepts

- **Domain:** an OpenSearch cluster (collection of nodes)
- **Index:** collection of documents (like a database table)
- **Document:** JSON object stored in an index
- **Shard:** subdivision of an index for distribution across nodes
- **Replica shard:** copy of a shard for redundancy and read scaling

---

## 3. Domain Design

### Node Types

| Type | Purpose |
|---|---|
| Data nodes | Store and search data |
| Master nodes | Cluster management (dedicated recommended for production) |
| UltraWarm nodes | Low-cost warm storage for infrequently accessed data |
| Cold storage | S3-backed, lowest cost for rarely accessed data |

### High Availability

- Deploy across **3 AZs** with 3 dedicated master nodes
- Use **Multi-AZ with Standby** for automatic failover (OpenSearch 2.x+)
- Replica shards in different AZs for data redundancy

---

## 4. Ingestion Patterns

| Source | Method |
|---|---|
| CloudWatch Logs | CloudWatch Logs subscription filter → Lambda → OpenSearch |
| Kinesis Data Firehose | Firehose delivery stream → OpenSearch (built-in integration) |
| DynamoDB Streams | DynamoDB Streams → Lambda → OpenSearch |
| S3 | S3 → Lambda → OpenSearch |
| Application | Direct HTTP API (REST) |
| IoT | IoT Rules → Kinesis → OpenSearch |

**Kinesis Data Firehose** is the recommended pattern for high-volume log ingestion — managed, buffered, no Lambda required.

---

## 5. Index State Management (ISM)

Automate index lifecycle:

```
Hot (active writes) → Warm (read-only, UltraWarm) → Cold (S3-backed) → Delete
```

- Define policies to transition indexes based on age or size
- Reduces storage costs for time-series data (logs, metrics)

---

## 6. Security

- **VPC:** deploy in VPC for private access; use security groups
- **Fine-grained access control:** document and field-level security
- **Encryption:** at rest (KMS) and in transit (TLS)
- **Cognito authentication:** for Kibana/OpenSearch Dashboards access
- **IAM:** resource-based policies for API access

---

## 7. OpenSearch Dashboards

- Built-in visualization tool (formerly Kibana)
- Create dashboards, visualizations, and saved searches
- Accessible via HTTPS endpoint
- Secure with Cognito or SAML authentication

---

## 8. Exam Scenarios

**Scenario 1:** A company wants to search through application logs stored in CloudWatch Logs using full-text search. What is the solution?

> Create a **CloudWatch Logs subscription filter** that streams log data to **Kinesis Data Firehose**. Configure Firehose to deliver to **OpenSearch Service**. Use OpenSearch Dashboards for search and visualization.

**Scenario 2:** A company needs to add product search functionality to an e-commerce application with relevance ranking and faceted filtering. What is the solution?

> Use **Amazon OpenSearch Service**. Index product catalog documents. Use OpenSearch's full-text search with relevance scoring for search queries. Use aggregations for faceted filtering (category, price range, brand).

**Scenario 3:** A company stores 2 years of log data in OpenSearch. Recent logs (last 7 days) are queried frequently; older logs are rarely accessed. How to reduce costs?

> Use **Index State Management (ISM)** to transition indexes older than 7 days to **UltraWarm** storage and indexes older than 30 days to **Cold storage**. This significantly reduces storage costs while keeping data queryable.

---

## 9. Exam Tips

- OpenSearch = **full-text search + log analytics** — not a relational database
- **Kinesis Firehose → OpenSearch** = standard high-volume ingestion pattern
- **3 AZs + 3 dedicated master nodes** = production HA configuration
- **UltraWarm/Cold storage** = cost optimization for time-series log data
- OpenSearch Dashboards (Kibana) requires **Cognito or SAML** for secure access
- OpenSearch is **not** a replacement for DynamoDB or RDS — different use cases

---

## 10. Official References

- [OpenSearch Service Developer Guide](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/)
- [UltraWarm Storage](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ultrawarm.html)
- [Index State Management](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ism.html)
