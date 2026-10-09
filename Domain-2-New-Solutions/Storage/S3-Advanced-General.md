# Amazon S3 Advanced — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3, 4. S3 is foundational to almost every architecture. At the professional level, focus on storage classes, lifecycle, replication, encryption, and access patterns.

---

## 1. Storage Classes

| Class | Use Case | Retrieval | Min Duration | Min Size |
|---|---|---|---|---|
| S3 Standard | Frequently accessed | Immediate | None | None |
| S3 Intelligent-Tiering | Unknown/changing access | Immediate | None | None |
| S3 Standard-IA | Infrequent, rapid access | Immediate | 30 days | 128 KB |
| S3 One Zone-IA | Infrequent, single AZ | Immediate | 30 days | 128 KB |
| S3 Glacier Instant Retrieval | Archive, millisecond access | Milliseconds | 90 days | 128 KB |
| S3 Glacier Flexible Retrieval | Archive, minutes-hours | Minutes–hours | 90 days | 40 KB |
| S3 Glacier Deep Archive | Long-term archive | 12–48 hours | 180 days | 40 KB |

**Intelligent-Tiering** automatically moves objects between access tiers based on usage — no retrieval fees, small monitoring fee per object.

---

## 2. Lifecycle Policies

Automate transitions and expirations:

```
New object → Standard (30 days) → Standard-IA (60 days) → Glacier Flexible (90 days) → Delete (365 days)
```

```json
{
  "Rules": [{
    "Status": "Enabled",
    "Transitions": [
      {"Days": 30, "StorageClass": "STANDARD_IA"},
      {"Days": 90, "StorageClass": "GLACIER"}
    ],
    "Expiration": {"Days": 365}
  }]
}
```

- Apply to entire bucket or filtered by prefix/tag
- Minimum 30 days in Standard before transitioning to Standard-IA
- Expire incomplete multipart uploads to avoid storage costs

---

## 3. Replication

### Cross-Region Replication (CRR)

- Replicates objects to a bucket in a **different region**
- Use cases: compliance, DR, latency reduction for global users
- Requires versioning enabled on both source and destination

### Same-Region Replication (SRR)

- Replicates objects within the **same region**
- Use cases: log aggregation, live replication between prod and test

### Replication Rules

- Can filter by prefix or tag
- Replicate existing objects: use **S3 Batch Replication**
- Replication does not replicate: delete markers (optional), lifecycle actions, objects encrypted with SSE-C

### Replication Time Control (RTC)

- Guarantees 99.99% of objects replicated within **15 minutes**
- Provides replication metrics and notifications
- Additional cost

---

## 4. Encryption

| Type | Key Management | Use Case |
|---|---|---|
| SSE-S3 | AWS manages keys (AES-256) | Default, simplest |
| SSE-KMS | AWS KMS manages keys | Audit trail, key rotation, cross-account |
| SSE-C | Customer provides keys | Customer manages keys outside AWS |
| DSSE-KMS | Dual-layer KMS encryption | Compliance requiring double encryption |
| Client-side | Encrypt before upload | Full client control |

**SSE-KMS** is the most exam-relevant — enables CloudTrail audit of key usage, cross-account access via key policy, and automatic key rotation.

Enforce encryption at rest via bucket policy:
```json
{
  "Effect": "Deny",
  "Action": "s3:PutObject",
  "Condition": {
    "StringNotEquals": {"s3:x-amz-server-side-encryption": "aws:kms"}
  }
}
```

---

## 5. Access Control

### Block Public Access

- Account-level and bucket-level settings
- Overrides bucket policies and ACLs that grant public access
- Enable at account level to prevent any public S3 access organization-wide

### Bucket Policies vs ACLs

- **Bucket policies:** JSON-based, support conditions, cross-account access — preferred
- **ACLs:** legacy, limited — disable ACLs and use bucket policies (Object Ownership: Bucket owner enforced)

### S3 Access Points

- Named network endpoints with their own access policies
- Simplify access management for shared datasets
- Can be restricted to a specific VPC

### S3 Object Lambda

- Transform objects on retrieval using Lambda
- Use cases: redact PII, convert formats, add watermarks
- No need to store multiple versions of the same object

---

## 6. Performance

- **Prefix parallelism:** S3 scales to 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD per second **per prefix**
- Use multiple prefixes for high-throughput workloads
- **Multipart upload:** recommended for objects > 100 MB, required for > 5 GB
- **S3 Transfer Acceleration:** uses CloudFront edge locations for faster uploads from distant locations
- **Byte-range fetches:** parallel downloads of large objects

---

## 7. Versioning and MFA Delete

- **Versioning:** preserves all versions of an object; delete creates a delete marker
- **MFA Delete:** requires MFA to permanently delete versions or disable versioning
- Versioning is required for replication and Object Lock

### S3 Object Lock

- WORM (Write Once Read Many) protection
- **Retention modes:**
  - **Compliance:** no one (including root) can delete before retention period
  - **Governance:** users with special permission can override
- **Legal hold:** indefinite protection, no retention period

---

## 8. Event Notifications

| Destination | Use Case |
|---|---|
| SNS | Fan-out to multiple subscribers |
| SQS | Buffered processing |
| Lambda | Real-time processing |
| EventBridge | Advanced filtering, multiple targets |

**EventBridge** is the most flexible — supports advanced filtering, multiple targets, and cross-account delivery.

---

## 9. Exam Scenarios

**Scenario 1:** A company stores compliance documents that must not be deleted for 7 years, even by administrators. What is the solution?

> Enable **S3 Object Lock** in **Compliance mode** with a 7-year retention period. In Compliance mode, no user — including root — can delete or modify the object before the retention period expires.

**Scenario 2:** A company wants to replicate S3 objects to another region for DR with a guarantee that replication completes within 15 minutes. What is the solution?

> Enable **Cross-Region Replication** with **Replication Time Control (RTC)**. RTC guarantees 99.99% of objects are replicated within 15 minutes and provides metrics for monitoring replication lag.

**Scenario 3:** A data lake has a single S3 bucket accessed by 10 different teams with different access requirements. How to simplify access management?

> Create **S3 Access Points** — one per team. Each access point has its own policy restricting access to the team's data prefix. Teams use their access point ARN instead of the bucket ARN.

**Scenario 4:** A company wants to automatically move objects to cheaper storage after 30 days and delete them after 1 year. What is the solution?

> Create an **S3 Lifecycle policy** that transitions objects to Standard-IA after 30 days, to Glacier after 90 days, and expires (deletes) objects after 365 days.

---

## 10. Exam Tips

- **Intelligent-Tiering** = unknown access patterns, no retrieval fees — good default for variable workloads
- **SSE-KMS** = audit trail + cross-account + key rotation — preferred for compliance
- **Object Lock Compliance mode** = no one can delete — even root
- **RTC** = 15-minute replication SLA — needed when the question specifies a replication time guarantee
- **S3 Access Points** = simplify multi-team access to shared buckets
- **Block Public Access at account level** = organization-wide protection via SCP or account setting
- Multipart upload: recommended > 100 MB, required > 5 GB — also use lifecycle to expire incomplete uploads

---

## 11. Official References

- [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/)
- [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [S3 Access Points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points.html)
