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
| 2.1 Design deployment strategies | EC2, ECS, EKS, Lambda, Auto Scaling, CodeDeploy |
| 2.2 Design solutions for high availability | ELB, Route 53, Aurora Global, DynamoDB Global Tables, S3 CRR |
| 2.3 Design solutions for business continuity | RDS Multi-AZ, Aurora, Backup, S3 versioning, DR patterns |
| 2.4 Determine security controls | IAM, KMS, Secrets Manager, WAF, Shield, VPC, PrivateLink |
| 2.5 Design reliable and resilient architectures | SQS, SNS, EventBridge, Step Functions, Kinesis |
| 2.6 Design solutions for performance | ElastiCache, CloudFront, Global Accelerator, DynamoDB DAX |
| 2.7 Determine cost optimization strategies | Spot, Savings Plans, S3 tiers, serverless vs EC2 |

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

### Databases/
- [RDS-Aurora-General.md](./Databases/RDS-Aurora-General.md) — Multi-AZ, read replicas, Aurora Global, Serverless
- [DynamoDB-General.md](./Databases/DynamoDB-General.md) — partitions, GSI, streams, DAX, Global Tables
- [ElastiCache-General.md](./Databases/ElastiCache-General.md) — Redis vs Memcached, caching patterns
- [OpenSearch-General.md](./Databases/OpenSearch-General.md) — search, log analytics, domain design

### Serverless/
- [StepFunctions-General.md](./Serverless/StepFunctions-General.md) — Standard vs Express, state types, error handling
- [EventBridge-General.md](./Serverless/EventBridge-General.md) — event buses, rules, pipes, scheduler
- [SQS-SNS-General.md](./Serverless/SQS-SNS-General.md) — queues, topics, fan-out, DLQ, FIFO
- [Kinesis-General.md](./Serverless/Kinesis-General.md) — Data Streams, Firehose, Data Analytics

### Standalone
- [MSK-General.md](./MSK-General.md) — managed Kafka, use cases vs Kinesis
- [CloudFormation-General.md](./CloudFormation-General.md) — stacks, StackSets, drift, nested stacks
- [CDK-General.md](./CDK-General.md) — constructs, stacks, pipelines, vs CloudFormation

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
Route 53 (Latency) → us-east-1 ALB → ECS → Aurora Global (primary)
                  → eu-west-1 ALB → ECS → Aurora Global (replica)
                                              ↕ replication
```

---

## Exam Tips

- **Decouple** with SQS/SNS/EventBridge — tight coupling is always wrong at the professional level
- **Serverless first** for variable workloads — Lambda + DynamoDB + API Gateway
- **Aurora** over RDS for most relational use cases — better HA, faster failover, Global Database
- **DynamoDB** for single-digit millisecond latency at any scale — not for complex queries
- **CloudFront** for static content and global distribution — not just CDN, also security (WAF, Shield)
- **ECS Fargate** vs **EKS** — Fargate for simplicity, EKS for Kubernetes compatibility
- **Auto Scaling** — always combine with health checks and multiple AZs
- Cost optimization: Spot for fault-tolerant, Savings Plans for steady-state, serverless for variable
