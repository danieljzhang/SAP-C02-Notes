# Data, Storage, and Analytics

## Object, block, file, and transfer choices

Amazon S3 is object storage. Choose a storage class and lifecycle policy from access frequency, retrieval delay, resiliency, and total cost rather than selecting a class by name alone. Intelligent-Tiering can move eligible objects between access tiers; archive tiers have different retrieval behavior. Versioning preserves object versions and supports recovery from some overwrites or deletes, but old versions consume storage until expired or removed.

| Class | Useful when | Decision point |
| --- | --- | --- |
| S3 Standard | Data is accessed frequently | General-purpose, multi-AZ object storage with millisecond access. |
| S3 Intelligent-Tiering | Access is unknown or changes over time | Automatic tiering; review monitoring fees and whether optional archive tiers fit the retrieval needs. |
| S3 Standard-IA | Long-lived data is accessed infrequently but must be available quickly | Multi-AZ resilience; retrieval fees and a minimum storage duration apply. |
| S3 One Zone-IA | Infrequent data can be recreated if its AZ is lost | Lower resilience because data is stored in one AZ; not the only copy of irreplaceable data. |
| S3 Glacier Instant Retrieval | Rarely accessed archive data still needs millisecond access | Archive pricing and minimum-duration/retrieval charges apply. |
| S3 Glacier Flexible Retrieval | Archive data can tolerate a restore step | Retrieval takes minutes to hours depending on the selected retrieval option. |
| S3 Glacier Deep Archive | Long-term data is very rarely accessed | Lowest-cost archival choice among these classes; retrieval takes hours and requires restoration. |
| S3 Express One Zone | A workload needs very low-latency object access in one AZ | High-performance, single-AZ choice, not a general archive tier. |

For IA and archive classes, compare minimum storage duration, minimum billable object size, retrieval charges, and recovery time against the real workload. Exact pricing and class behavior can change.

For access control, use S3 Block Public Access as a preventive guardrail, then review bucket policies, IAM policies, access points, and IAM Access Analyzer findings for public or cross-account sharing. These controls answer different questions: Block Public Access blocks public grants, Access Analyzer identifies policy-based exposure, and CloudTrail data events record object-level API activity only when configured (and incur additional charges). EventBridge can route supported logged events for alerting, but an alert is not a prevention control.

When versioning is first enabled, existing objects remain as null-version objects; it does not create historical versions retroactively. Subsequent writes create versioned objects. Add lifecycle rules for noncurrent versions when retention requirements allow, or storage can continue to grow after overwrites and deletes.

Amazon Macie helps discover and classify sensitive data in S3 and can surface findings for investigation. It is a discovery and security-posture tool, not a backup service or a substitute for access controls.

- **Requester Pays** can shift eligible request and transfer charges to the requester; the bucket owner still pays for stored data.
- **S3 Transfer Acceleration** can help some long-distance uploads by routing them through AWS edge locations. Measure it for the expected geography and file sizes before accepting the additional cost.
- **Multipart upload** is useful for large objects and retryable parallel parts. Set lifecycle cleanup for abandoned multipart uploads.
- **Cross-Region Replication** copies eligible objects between buckets. It is not a substitute for database replication or an application-consistent backup plan.
- **AWS Snowball Edge** is for large offline data transfers or supported edge processing when network transfer is impractical. Plan lead time, device capacity, export/import steps, and any compute needs; verify current regional availability and service lifecycle before recommending a device.

**Fresh example:** A research team receives large image archives from remote sites. It tests multipart uploads over ordinary S3 endpoints first, then compares Transfer Acceleration using real locations and objects. It keeps public access blocked and gives each analysis role access only to its assigned prefix.

## DynamoDB design and recovery

DynamoDB is a managed key-value and document database. The access pattern should drive the partition key, sort key, and secondary-index design. Provisioned capacity supports predictable workloads; on-demand capacity reduces capacity planning for variable workloads, subject to service quotas and burst behavior.

A `ProvisionedThroughputExceededException` is a signal to inspect request rate, hot keys, partition-key distribution, capacity settings, and downstream retry behavior. Use exponential backoff with jitter where retries are appropriate; retries alone do not fix a hot partition or a request pattern that exceeds sustainable capacity.

- **Global tables** replicate table data across Regions for multi-Region access and recovery. Understand replication behavior, conflict handling, consistency, and regional failure procedures.
- **Time to Live (TTL)** marks expired items for asynchronous removal. Do not treat TTL as an exact-time scheduler or as an immediate security deletion guarantee.
- DynamoDB can support session-state patterns when access is key-based and the data model fits. TTL can clean up expired session records eventually; application authorization and session expiration checks must not rely on physical deletion happening at a precise time.
- **DAX** is an in-memory cache for DynamoDB read-heavy patterns. Check consistency and access-pattern requirements before adding it; it does not replace table design.
- **Point-in-time recovery and backups** address restoration needs. A backup is only useful if recovery steps and recovery objectives are tested.
- DynamoDB now supports resource-based policies for tables, indexes, and streams. Recheck current service documentation rather than relying on older notes that say otherwise.
- **Reserved capacity** is a pricing commitment for eligible provisioned capacity, not a mechanism for making a workload resilient or increasing its throughput limits.

## Relational databases and analytics stores

Amazon RDS and Aurora provide managed relational databases, but availability, read scaling, and cross-Region recovery are distinct design goals:

- A **Multi-AZ deployment** is primarily an availability and failover design. A **read replica** is primarily a read-scaling or replication option; it may also be part of a recovery plan, but promotion and application cutover must be planned.
- Aurora storage grows automatically as the cluster's data grows, avoiding manual storage-volume preallocation. Aurora reader auto scaling changes replica count in response to configured workload metrics; Aurora Serverless v2 adjusts database compute capacity; Aurora Global Database addresses cross-Region replication and recovery. These are separate controls, not synonyms for storage auto-growth.
- Confirm engine compatibility, Region support, failover behavior, scaling limits, and replication lag for the workload.
- **RDS Proxy** pools and manages database connections for supported engines. It can reduce connection-management pressure, but it does not fix inefficient queries or insufficient database capacity.
- **Point-in-time recovery (PITR)** restores to a selected time within the service's available backup window. A snapshot is a point-in-time image; neither should be confused with a cross-Region failover plan.
- **OLTP** systems handle frequent transactional reads and writes; **OLAP** systems support analytical queries over larger datasets. Redshift is a data warehouse, not a drop-in substitute for a transactional relational database.
- Redshift snapshots are recovery copies of warehouse data. Review automated snapshot retention, manual snapshot lifecycle, cross-Region copy needs, encryption, and restore testing against the recovery objective.
- **Sharding** partitions data across multiple database nodes to scale a workload beyond one database's limits. It adds application and operational complexity; first evaluate indexing, query design, vertical scaling, read replicas, and managed service options.
- When moving Oracle workloads, assess engine features, licensing, extensions, data volume, downtime tolerance, and application compatibility. Choose native migration, replication, or a managed AWS target only after validating feature parity and cutover requirements.

**Fresh example:** A booking service needs fast regional reads and a recoverable write region. The architect separately evaluates database availability, read replicas, cross-Region replication, and application routing instead of assuming one Multi-AZ setting satisfies every requirement.

Amazon Timestream is a managed time-series database for timestamped measurements and event data. Its time-series functions, separate memory and magnetic storage tiers, and scheduled queries can fit workloads that need frequent ingestion, time-window analysis, and retained history. Model dimensions and time-based access patterns deliberately, and compare its retention and query model with storing events in S3 or using a streaming analytics pipeline.

## Hybrid storage and content analysis

AWS Storage Gateway connects on-premises workloads to AWS-backed storage:

- **File Gateway** presents file interfaces backed by S3 objects.
- **Cached Volume Gateway** keeps the primary dataset in AWS and a frequently accessed subset in the local cache. **Stored Volume Gateway** keeps the primary dataset locally and asynchronously backs it up to AWS. Choose based on local data-residency/latency needs and the capacity available on site.
- **Tape Gateway** presents virtual tapes for backup workflows and archival retention; it is not a low-latency file system for interactive analytics.
- iSCSI access should use network controls and supported authentication such as CHAP where appropriate. Authentication does not replace encryption, host authorization, or network isolation.

Amazon Athena runs SQL queries over data in S3; organize data formats, partitions, permissions, and query output locations to control performance and cost. Amazon QuickSight provides analytics and visualization over supported data sources. CloudWatch VPC Flow Logs can be delivered to storage for analysis, but flow logs are not a packet capture and do not show application payloads.

Amazon Textract extracts text and structured fields from supported documents. Amazon Rekognition analyzes supported image and video content. They are managed analysis services, not storage gateways; provide the input through a supported integration and protect source data with least-privilege permissions.

## References

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [CloudTrail events for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cloudtrail-logging-s3-info.html)
- [Amazon S3 storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)
- [AWS Snowball](https://docs.aws.amazon.com/snowball/latest/developer-guide/whatissnowball.html)
- [DynamoDB resource-based policies](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-resource-based.html)
- [DynamoDB global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [DynamoDB point-in-time recovery](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html)
- [Amazon Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [Amazon RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [Amazon RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)
- [AWS Storage Gateway](https://docs.aws.amazon.com/storagegateway/latest/userguide/WhatIsStorageGateway.html)
- [Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
- [Amazon Redshift snapshots](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-snapshots.html)
- [Amazon Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html)
- [Amazon Timestream](https://docs.aws.amazon.com/timestream/latest/developerguide/what-is-timestream.html)
- [Amazon Textract](https://docs.aws.amazon.com/textract/latest/dg/what-is.html)
- [Amazon Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html)
