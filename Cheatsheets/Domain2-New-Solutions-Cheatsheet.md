# Domain 2 Cheatsheet: Design for New Solutions

> Deeper notes: [Domain 2 overview](../Domain-2-New-Solutions/README.md) and the linked service guides.

> Focus: choose an architecture that meets scale, availability, security, performance, and cost requirements. Pick the simplest service model that satisfies explicit constraints.

## Compute and Networking

| Requirement | Candidate | Decision cue |
|---|---|---|
| Full OS control / specialized software | EC2 | Own patching, AMI lifecycle, capacity, and host hardening |
| Fleet size changes with demand | Auto Scaling | Pair with health checks and appropriate load balancing |
| AWS-native container orchestration | ECS | Strong default when Kubernetes compatibility is not required |
| Kubernetes APIs/ecosystem | EKS | More cluster/add-on/upgrade operations; justify with requirements/team skills |
| Event-driven short execution | Lambda | Check runtime, duration, concurrency, payload, and downstream limits |
| HTTP application load balancing | ALB | Layer 7 routing; path/host rules and supported targets |
| Static IP / high-performance Layer 4 | NLB | TCP/UDP/TLS use cases; validate source-IP and target behavior |
| Inspect and route network appliances | GWLB | Transparent insertion of virtual appliances |
| Global static anycast IPs and fast endpoint failover | Global Accelerator | TCP/UDP global entry point; not a content cache |
| Cache and accelerate web content | CloudFront | Edge caching, origin protection, TLS/WAF integration |
| Managed API front door | API Gateway | Auth, throttling, stages, transformations, private APIs; select API type by need |

## Storage and Databases

| Need | Candidate | Key distinction |
|---|---|---|
| Object storage, lifecycle, replication | S3 | Strong consistency; choose class from access/retrieval/retention economics |
| Block storage for EC2 | EBS | Choose volume type by IOPS/throughput; snapshots for backup, multi-attach where supported |
| Shared Linux file system | EFS | NFS, elastic shared file access |
| Windows SMB / Windows semantics | FSx for Windows File Server | AD integration and SMB feature fit |
| High-performance parallel/HPC file data | FSx for Lustre | Link to S3 and validate workload throughput/metadata needs |
| NetApp ONTAP feature compatibility | FSx for ONTAP | Multi-protocol, snapshots, replication; assess operational fit |
| Relational database with managed HA | RDS/Aurora | Multi-AZ for HA; read replicas for read scaling/DR patterns, not substitutes |
| Key-value/document at scale | DynamoDB | Access-pattern-first schema, partition design, consistency and capacity mode |
| In-memory cache | ElastiCache | Cache is not the system of record; handle invalidation/failures |
| Search and log analytics | OpenSearch | Search/index workload; plan shard sizing, ingestion, and domain operations |

## Messaging and Integration

| Requirement | Prefer |
|---|---|
| Decouple work with durable competing consumers | SQS |
| Fanout notifications to subscribers | SNS |
| Filter/route domain or SaaS events | EventBridge |
| Ordered/replayable high-volume streaming | Kinesis Data Streams |
| Managed Apache Kafka ecosystem | MSK |
| Durable orchestration with retries/branches | Step Functions |

- Use queues to buffer bursts and protect downstream capacity.
- Design consumers for duplicate delivery and idempotency; use DLQs and replay procedures.
- Choose synchronous APIs only when the caller needs an immediate response and dependency availability/latency are acceptable.

## Infrastructure as Code

- **CloudFormation:** declarative AWS resource stacks, drift/change sets, nested or StackSets patterns.
- **CDK:** define infrastructure in programming languages, synthesize to CloudFormation; review generated templates and construct versions.
- Make deployments repeatable; keep secrets out of templates and parameter values; plan rollback and stateful-resource replacement carefully.

## Common Traps

- Multi-AZ database deployment improves availability; a read replica is for read scale/DR and may require promotion.
- Global Accelerator routes users to healthy endpoints; CloudFront caches content at edge.
- SQS, SNS, EventBridge, Kinesis, and MSK solve different messaging/streaming shapes.
- S3 storage class savings can be offset by retrieval, request, transition, and minimum-duration costs.
- Adding a cache improves performance only if the access pattern and invalidation strategy fit.
- Managed services reduce infrastructure tasks, not IAM, network, quota, monitoring, or cost responsibilities.

## 30-Second Review

1. State latency, data consistency, RTO/RPO, scale, and operational constraints.
2. Select compute and persistence independently; avoid forcing one service to do everything.
3. Identify failure domains and scaling bottlenecks.
4. Add least privilege, encryption, observability, and tested recovery.
5. Compare plausible alternatives on total operational cost, not service price alone.

## Official References

- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Amazon S3 storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [AWS messaging services](https://aws.amazon.com/message-queue/)
