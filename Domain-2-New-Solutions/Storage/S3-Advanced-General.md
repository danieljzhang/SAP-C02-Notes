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

- AWS currently describes the RTC target as replicating 99.9% of objects within **15 minutes**; this is not a guarantee that every object completes within 15 minutes
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

> Enable **Cross-Region Replication** with **Replication Time Control (RTC)** when the requirement aligns with its documented 15-minute replication SLA. AWS currently states that 99.9% of objects replicate within 15 minutes and provides metrics/events for objects that exceed the threshold; it does not guarantee every object completes within 15 minutes. Check current SLA exclusions and request-rate/transfer quotas.

**Scenario 3:** A data lake has a single S3 bucket accessed by 10 different teams with different access requirements. How to simplify access management?

> Create **S3 Access Points** — one per team. Each access point has its own policy restricting access to the team's data prefix. Teams use their access point ARN instead of the bucket ARN.

**Scenario 4:** A company wants to automatically move objects to cheaper storage after 30 days and delete them after 1 year. What is the solution?

> Create an **S3 Lifecycle policy** that transitions objects to Standard-IA after 30 days, to Glacier after 90 days, and expires (deletes) objects after 365 days.

---

## 10. Exam Tips

- **Intelligent-Tiering** = unknown access patterns, no retrieval fees — good default for variable workloads
- **SSE-KMS** = audit trail + cross-account + key rotation — preferred for compliance
- **Object Lock Compliance mode** = no one can delete — even root
- **RTC** = documented 15-minute replication SLA for the stated percentage of objects, with exclusions; do not promise every object completes within 15 minutes
- **S3 Access Points** = simplify multi-team access to shared buckets
- **Block Public Access at account level** = organization-wide protection via SCP or account setting
- Multipart upload: recommended > 100 MB, required > 5 GB — also use lifecycle to expire incomplete uploads

---

## 11. 🎯 Architecture Decision Notes

### Choosing a storage class — decide from the access pattern

Pick from retrieval frequency, acceptable delay, and resilience — not from
the class name:

| Signal | Direction |
|---|---|
| Accessed frequently, latency-sensitive | S3 Standard |
| Access unknown or shifting | Intelligent-Tiering (watch monitoring fees and archive-tier fit) |
| Infrequent but must be instant | Standard-IA (multi-AZ) — check retrieval fees and minimum duration |
| Infrequent and recreatable | One Zone-IA — never the only copy of irreplaceable data |
| Archive, rare access, ms retrieval needed | Glacier Instant Retrieval |
| Archive, restore step acceptable | Glacier Flexible Retrieval (minutes–hours) |
| Archive, almost never touched | Glacier Deep Archive (hours; cheapest) |
| Single-AZ, microsecond object access | S3 Express One Zone — a performance tier, not an archive |

For IA and archive tiers, compare **minimum storage duration, minimum
billable object size, retrieval charges, and recovery time** against the real
workload. Intelligent-Tiering's automation is not free — model it.

### Layered access control — each layer answers a different question

- **Block Public Access:** preventive guardrail against public grants.
- **Bucket/IAM policies and access points:** who may do what.
- **Access Analyzer:** finds policy-based exposure you did not intend.
- **CloudTrail data events:** records object-level API calls — only when
  enabled, and at extra cost.
- **EventBridge alerts on logged events:** detection, not prevention.

Versioning protects against overwrites/deletes but old versions keep
billing until lifecycle rules expire them. Enabling versioning does not
retroactively version existing objects — they stay as null-version objects.

---

### Requester Pays — who foots the bill for someone else's downloads

By default the bucket owner pays for storage *and* for every request and byte downloaded. **Requester Pays** flips the transfer/request cost to whoever reads the data (the owner still pays for storage). Requests against the bucket must be authenticated so AWS knows whom to bill — anonymous access and Requester Pays don't mix.

Use it when you host a dataset that *other parties* consume heavily and you don't want their download bills: shared research datasets, a media library licensed to partners, a data vendor's product bucket. The exam pattern is usually cost attribution — "the startup wants the media company, not itself, to pay for transfer out of its buckets."

Traps: (1) the requester must include the payer flag in their requests, so the consuming application needs code/config changes — it's not transparent; (2) it shifts cost, it doesn't reduce total cost — the requester may push back, so it's a commercial decision as much as a technical one; (3) don't confuse with S3 access points or bucket policies, which control *permission*, not *billing*.

**Fresh example:** A genomics startup hosts a 50 TB public dataset that three pharma partners download repeatedly. Enabling Requester Pays on the dataset bucket means each partner's AWS account is billed for its own downloads, while the startup keeps paying only for the 50 TB of storage.

### Multi-Region Access Points — when a single global S3 endpoint matters

Standard S3 Access Points are regional. **Multi-Region Access Points (MRAP)**
provide one global endpoint that routes requests to the closest bucket,
with active-active failover between Regions:

- **Use when:** a global application needs low-latency S3 access from multiple Regions with automatic failover — without building the routing logic in the application.
- **How it works:** you define the set of buckets (one per Region); AWS routes each request to the lowest-latency bucket. Failover controls let you shift traffic between Regions.
- **Replication:** MRAP does not replicate data — pair it with S3 Cross-Region Replication (CRR) so each Regional bucket actually has the objects. An MRAP pointing at empty buckets fails over to nothing.
- **Not for:** single-Region workloads (a regional access point is simpler), or when you need custom routing logic beyond latency/failover.

**Fresh example:** A media company serves user uploads from three Regions. MRAP gives the mobile app one endpoint; uploads land in the nearest bucket, and CRR keeps all three buckets in sync. When a Region has an event, traffic control shifts requests without an app update.

**Trap:** Assuming MRAP copies the data. It routes requests — replication is a separate configuration you must set up.

---

### S3 Express One Zone — the missing row in the storage-class table

**S3 Express One Zone** (launched November 2023) is a high-performance,
single-Availability-Zone storage class built for latency-sensitive workloads —
single-digit-millisecond data access, far faster than S3 Standard. It belongs
in the storage-class decision alongside the existing table:

| Class | AZ resilience | Latency | Fit |
|---|---|---|---|
| S3 Express One Zone | Single AZ | Single-digit ms | Hot, latency-critical data that can be re-created or is replicated at the application layer |

Decision rules:

- **Pick Express One Zone** when the workload needs the lowest S3 latency
  (ML training data, high-frequency analytics scratch, media processing) *and*
  the data is reproducible or the application handles AZ-level replication
  itself.
- **Do not pick it** for data that must survive an AZ loss without
  application help — that is what S3 Standard (multi-AZ) is for. The exam's
  trap is offering Express One Zone's speed for a durability requirement.
- It is **not** in the Intelligent-Tiering family and does not participate in
  the same lifecycle-transition rules as the regional classes — check current
  documentation before designing lifecycle policies around it.

**Fresh example:** A video-transcoding pipeline stages raw uploads for
processing within minutes, then deletes them; the originals live durably in
S3 Standard. The staging prefix uses Express One Zone for speed, with a
lifecycle expiration — not a transition — because the data is temporary by
design.

## 12. Official References

- [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/)
- [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [S3 Access Points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points.html)
