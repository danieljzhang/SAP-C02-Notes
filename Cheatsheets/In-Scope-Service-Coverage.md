# SAP-C02 In-Scope Service Coverage Map

> **Version:** SAP-C02, compared with AWS's official in-scope service list and domain tasks on October 8, 2026. AWS states that the service list is non-exhaustive and subject to change; inclusion does not imply equal exam weight. This repo is not SAP-C03 material.

Use this map to distinguish deep study notes from short awareness cues. A service without a standalone file can still appear in an architecture scenario; use its official guide when a requirement depends on detailed feature behavior.

## Objective Coverage

| SAP-C02 task | Primary repo material | Review focus |
|---|---|---|
| 1.1 Network connectivity | [Domain 1 overview](../Domain-1-Organizational-Complexity/README.md), Transit Gateway, Direct Connect, VPN, VPC, Route 53 notes | Hybrid DNS, transitive routing, segmentation, endpoints, traffic troubleshooting |
| 1.2 Security controls | [Security selector](../Domain-1-Organizational-Complexity/Security-Services-Selector.md), IAM, SCP/RCP, Security Hub, Config | Identity, keys/certificates, network ACLs/security groups, findings and remediation |
| 1.3 Reliable/resilient architectures | [DR patterns](../Domain-3-Continuous-Improvement/Resilience-Patterns-General.md), [AWS DRS](../Domain-3-Continuous-Improvement/Elastic-Disaster-Recovery-General.md), Backup | RTO/RPO, recovery drills, scaling, restoration and data protection |
| 1.4 Multi-account environment | Organizations, Control Tower, Landing Zone, IAM Identity Center, RAM | Account structure, delegated administration, central logging/events, governance |
| 1.5 Cost optimization and visibility | [Cost visibility](../Domain-1-Organizational-Complexity/Cost-Visibility-General.md), Cost Explorer, Trusted Advisor, Compute Optimizer | Tags, budgets, billing exports, forecasts, purchasing options, cost allocation |
| 2.1 Deployment strategy | [Deployment strategies](../Domain-2-New-Solutions/Deployment-Strategies-General.md), CloudFormation, CDK, Systems Manager | CI/CD, change controls, rollout, rollback, managed services |
| 2.2 Business continuity | DR patterns, AWS DRS, Backup, S3/RDS/Aurora notes | Data replication, recovery drills, DNS, backups, multi-Region readiness |
| 2.3 Security controls | Security selector, IAM, CloudFront/WAF, VPC, KMS, Secrets Manager | Least privilege, network flows, encryption, endpoint choice, patching |
| 2.4 Reliability | DR patterns, Auto Scaling, SQS/SNS, EventBridge, Step Functions, Service Quotas | Failure isolation, HA, quotas, recovery, loose coupling |
| 2.5 Performance | CloudWatch, X-Ray, Compute Optimizer, ElastiCache, CloudFront, Global Accelerator | Metrics, bottlenecks, caching, buffering, rightsizing, access patterns |
| 2.6 Cost optimization | Cost visibility, Cost Explorer, Compute Optimizer, S3 lifecycle | Cost/usage reports, commitment model, tiering, transfer cost |
| 3.1 Operational excellence | CloudWatch, CloudTrail, Systems Manager, CloudFormation | Monitoring, deployments, automation, runbooks, recovery exercises |
| 3.2 Security improvement | Security selector, Security Hub, Config, IAM | Data retention, secrets, least privilege, traceability, patching, remediation |
| 3.3 Performance improvement | CloudWatch, X-Ray, Compute Optimizer, service notes | Define KPIs/SLAs, test remediation, find bottlenecks |
| 3.4 Reliability improvement | DR patterns, AWS DRS, Backup, Service Quotas | Remove single points of failure, replication, self-healing, quota readiness |
| 3.5 Cost opportunities | Cost visibility, Cost Explorer, Trusted Advisor | Usage reports, unused resources, alarms, rightsizing and purchasing options |
| 4.1 Select workloads for migration | [Domain 4 overview](../Domain-4-Migration-Modernization/README.md), MAP, Migration Hub | Portfolio assessment, 7Rs, waves, dependencies, TCO |
| 4.2 Choose migration approach | MGN, DMS, DataSync, Transfer Family, Snow Family | Match server/database/file/protocol unit; secure the migration path |
| 4.3 Design target architecture | Compute, container, storage, and database selector notes | Select platform and services from workload constraints |
| 4.4 Modernize/enhance | Microservices, containers, serverless, database migration patterns | Decoupling, purpose-built data, serverless and integration fit |

## Covered in Depth

The repository has dedicated guides for Organizations/Control Tower/IAM/networking, EC2/Auto Scaling/ECS/EKS/Lambda, CloudFront/API Gateway/load balancing, S3/EFS/FSx, RDS/Aurora/DynamoDB/ElastiCache/OpenSearch, SQS/SNS/EventBridge/Kinesis/MSK/Step Functions, CloudFormation/CDK, monitoring/cost/reliability, and the core migration services. See [All-Services Review](./All-Services-Review.md) for direct links.

## Awareness-Level Services

These service families appear in the official SAP-C02 in-scope list but do not all have standalone guides here. Learn the one-line selection cue and check current documentation when a scenario depends on a specific feature.

| Family | Services with brief or no standalone coverage | Awareness cue |
|---|---|---|
| Analytics | Athena, Glue, Lake Formation, EMR, Data Firehose, Managed Service for Apache Flink, QuickSight, Data Exchange | SQL on S3<br>ETL-catalog<br>data governance<br>Hadoop-Spark<br>managed delivery<br>stream processing<br>BI<br>third-party datasets |
| Application integration | AppFlow, AppSync, Amazon MQ | SaaS data transfer<br>managed GraphQL APIs<br>broker compatibility for existing middleware |
| Cloud financial management | Cost and Usage Report/Data Exports, Savings Plans | Granular billing datasets and commitment options; detailed cost workflows are in the cost visibility guide |
| Compute and containers | App Runner, Batch, Elastic Beanstalk, Lightsail, Outposts, Wavelength, ECS Anywhere, EKS Anywhere | Managed web app<br>batch jobs<br>application platform<br>simple VPS<br>on-premises AWS hardware<br>edge zones<br>hybrid container operations |
| Databases | DocumentDB, Keyspaces, Neptune, Redshift, Timestream | MongoDB-compatible document<br>Cassandra-compatible wide column<br>graph<br>warehouse<br>time-series; see [database selector](../Domain-2-New-Solutions/Databases/Purpose-Built-Database-Selector.md) |
| Developer tools | CodeArtifact, CodeGuru | Package repository<br>code quality and performance insights; CodeBuild, CodeDeploy, and CodePipeline are in the deployment guide |
| Blockchain and business applications | Amazon Managed Blockchain, Amazon SES | Managed blockchain network awareness<br>transactional email integration |
| End-user and frontend | AppStream 2.0, WorkSpaces, Amplify, Device Farm, Pinpoint | App streaming<br>virtual desktops<br>full-stack web-mobile development<br>device testing<br>customer engagement messaging |
| IoT | IoT Core, Device Defender, Device Management, IoT Events, Greengrass, SiteWise, IoT Things Graph, IoT 1-Click | Secure device messaging<br>fleet security and management<br>edge processing<br>industrial data; verify lifecycle and current support |
| Machine learning | Comprehend, Fraud Detector, Kendra, Lex, Personalize, Polly, Rekognition, SageMaker AI, Textract, Transcribe, Translate | Text analytics<br>fraud<br>search<br>conversational<br>personalization<br>speech<br>vision<br>ML platform<br>document extraction<br>transcription<br>translation |
| Media | Elastic Transcoder, Kinesis Video Streams | Legacy media conversion awareness<br>video ingestion and streaming; verify current product lifecycle |
| Management and governance | Health Dashboard, License Manager, Managed Grafana/Prometheus, Proton, Service Catalog | Service events<br>license tracking<br>managed observability<br>platform delivery<br>governed product provisioning |
| Migration and transfer | Application Discovery Service, Schema Conversion Tool (SCT) | Inventory/dependency assessment<br>schema assessment and conversion; pair with the execution services in Domain 4 |
| Security | Artifact, Audit Manager, ACM, CloudHSM, Cognito, Detective, Directory Service, Firewall Manager, Network Firewall, STS, WAF, Shield | Compliance documents<br>evidence collection<br>certificates<br>dedicated HSM<br>user identity<br>investigation<br>managed directories<br>central policy<br>network inspection<br>temporary credentials<br>web filtering<br>DDoS protection |
| Storage | EBS, S3 Glacier, Storage Gateway | Block volumes<br>archival classes<br>hybrid storage access; compare with S3, DataSync, AWS Backup, and the DRS guide |

## Study Priority

1. Master every official domain task and the detailed guides linked above.
2. Learn the selector cues in the awareness table; do not memorize every console setting for every listed service.
3. For any question hinging on a service feature, quota, integration, or lifecycle, verify the current AWS guide.
4. Recheck this map against the live SAP-C02 guide before the exam; switch to the SAP-C03 guide for exams scheduled after SAP-C02 retires.

## Official References

- [SAP-C02 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02.html)
- [SAP-C02 in-scope services](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/sap-02-in-scope-services.html)
- [AWS certification page and version dates](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
