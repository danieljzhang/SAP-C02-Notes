# Domain 2: Design for New Solutions (29%)

> Highest-weight domain. Covers the broadest range of AWS services. Expect ~19 questions focused on architecture decisions, trade-offs, and integration patterns.

---

## What This Domain Tests

- Selecting the right compute, storage, database, and networking services for given requirements
- Designing scalable, highly available, and fault-tolerant architectures
- Applying security, encryption, and access controls to new workloads
- Choosing between serverless, container, and EC2-based architectures
- Designing event-driven and decoupled architectures
- Applying IaC best practices with CloudFormation and CDK

---

## Subdomain Breakdown

| Subdomain | Key Services |
|---|---|
| 2.1 Design a deployment strategy to meet business requirements | IaC, CI/CD, change management, Systems Manager, deployment and rollback strategies |
| 2.2 Design a solution to ensure business continuity | Global infrastructure, RTO/RPO, DR, backup, replication, failover |
| 2.3 Determine security controls based on requirements | IAM, network controls, encryption, endpoints, credentials, managed security services |
| 2.4 Design a strategy to meet reliability requirements | Multi-AZ/Region, replication, quotas, scaling, loosely coupled integration |
| 2.5 Design a solution to meet performance objectives | Monitoring, storage/compute/database selection, caching, buffering, rightsizing |
| 2.6 Determine a cost optimization strategy | Cost visibility, pricing models, storage tiers, data transfer, rightsizing |

> Task names aligned to the [official SAP-C02 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain2.html). The guide may change; recheck before exam day.

---

## Files in This Domain

### Compute/
- [EC2-Advanced-General.md](./Compute/EC2-Advanced-General.md) — placement groups, Spot, Reserved, instance types
- [ECS-General.md](./Compute/ECS-General.md) — Fargate, task definitions, service discovery
- [EKS-General.md](./Compute/EKS-General.md) — managed node groups, Fargate profiles, networking
- [Lambda-General.md](./Compute/Lambda-General.md) — concurrency, layers, destinations, SnapStart
- [AutoScaling-General.md](./Compute/AutoScaling-General.md) — scaling policies, predictive, warm pools

### Networking/
- [CloudFront-General.md](./Networking/CloudFront-General.md) — distributions, origins, cache behaviors, security
- [APIGateway-General.md](./Networking/APIGateway-General.md) — REST vs HTTP vs WebSocket, throttling, caching
- [ELB-General.md](./Networking/ELB-General.md) — ALB, NLB, GWLB — when to use each
- [GlobalAccelerator-General.md](./Networking/GlobalAccelerator-General.md) — anycast, traffic dials, vs CloudFront

### Storage/
- [S3-Advanced-General.md](./Storage/S3-Advanced-General.md) — storage classes, lifecycle, replication, encryption
- [EFS-General.md](./Storage/EFS-General.md) — shared file storage, performance modes, replication
- [FSx-General.md](./Storage/FSx-General.md) — FSx for Windows, Lustre, NetApp ONTAP, OpenZFS
- [EBS-General.md](./Storage/EBS-General.md) — volume types, snapshots, encryption, multi-attach

### Databases/
- [RDS-Aurora-General.md](./Databases/RDS-Aurora-General.md) — Multi-AZ, read replicas, Aurora Global, Serverless
- [DynamoDB-General.md](./Databases/DynamoDB-General.md) — partitions, GSI, streams, DAX, Global Tables
- [ElastiCache-General.md](./Databases/ElastiCache-General.md) — Redis vs Memcached, caching patterns
- [OpenSearch-General.md](./Databases/OpenSearch-General.md) — search, log analytics, domain design
- [Purpose-Built-Database-Selector.md](./Databases/Purpose-Built-Database-Selector.md) — DocumentDB, Keyspaces, Neptune, Redshift, Timestream and selection cues

### Serverless/
- [StepFunctions-General.md](./Serverless/StepFunctions-General.md) — Standard vs Express, state types, error handling
- [EventBridge-General.md](./Serverless/EventBridge-General.md) — event buses, rules, pipes, scheduler
- [SQS-SNS-General.md](./Serverless/SQS-SNS-General.md) — queues, topics, fan-out, DLQ, FIFO
- [Kinesis-General.md](./Serverless/Kinesis-General.md) — Data Streams, Firehose, Data Analytics

### Standalone
- [MSK-General.md](./MSK-General.md) — managed Kafka, use cases vs Kinesis
- [CloudFormation-General.md](./CloudFormation-General.md) — stacks, StackSets, drift, nested stacks
- [CDK-General.md](./CDK-General.md) — constructs, stacks, pipelines, vs CloudFormation
- [Deployment-Strategies-General.md](./Deployment-Strategies-General.md) — CodePipeline, CodeBuild, CodeDeploy, rollout and rollback patterns

---

## Key Architecture Patterns

### Three-Tier Web Application
```
Internet → CloudFront → ALB → ECS/EC2 (Auto Scaling) → Aurora Multi-AZ
                                    ↓
                              ElastiCache (Redis)
```

### Serverless API
```
Client → API Gateway → Lambda → DynamoDB
                    ↓
              SQS → Lambda → downstream processing
```

### Event-Driven Architecture
```
Producer → EventBridge → Rules → Multiple consumers
                              ├── Lambda (real-time processing)
                              ├── SQS (buffered processing)
                              └── Step Functions (orchestration)
```

### Multi-Region Active-Active
```
Route 53 / Global Accelerator
      ├── Region A: ALB → ECS → Aurora Global primary (writer)
      └── Region B: ALB → ECS → Aurora Global secondary (read-only)
                              └── optional write forwarding to primary

For multi-Region writes, use a data store designed for that model (for example,
DynamoDB Global Tables) and design for its consistency/conflict semantics.
```

---

## Exam Tips

- **Decouple** with SQS/SNS/EventBridge when asynchronous processing, burst buffering, or independent consumers meet the requirements; account for retries and eventual consistency
- **Serverless** can fit variable or event-driven workloads; validate runtime, duration, concurrency, data, latency, and total-cost constraints
- **Aurora vs RDS** is workload-specific: compare engine compatibility, HA/replication needs, features, performance, operations, and cost
- **DynamoDB** fits access-pattern-driven key-value/document workloads; design partition keys, indexes, throughput, consistency, and hot-key behavior
- **CloudFront** for static content and global distribution — not just CDN, also security (WAF, Shield)
- **ECS** vs **EKS** — ECS (use the Fargate launch type for serverless compute) for AWS-native simplicity, EKS for Kubernetes compatibility
- **Auto Scaling** — use health checks and multiple AZs where availability requirements call for them
- Cost optimization: Spot for fault-tolerant, Savings Plans for steady-state, serverless for variable
