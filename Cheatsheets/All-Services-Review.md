# SAP-C02 All-Services Review

> One-line decision cues for final review. A service name is not an architecture: validate the workload, integration, limits, cost, and failure behavior in the linked notes and current AWS documentation.

## Domain 1: Organizational Complexity

| Service / topic | Recall cue |
|---|---|
| [AWS Organizations](../Domain-1-Organizational-Complexity/Organizations/Organizations-General.md) | Multi-account hierarchy, consolidated governance, delegated administration |
| [Service Control Policies](../Domain-1-Organizational-Complexity/Organizations/SCPs-General.md) | Account/OU maximum permissions; does not grant permissions |
| [AWS Control Tower](../Domain-1-Organizational-Complexity/ControlTower/ControlTower-General.md) | Governed landing zone, account provisioning, controls |
| [Landing zones](../Domain-1-Organizational-Complexity/ControlTower/LandingZone-General.md) | Account, identity, network, logging, security foundations |
| [IAM](../Domain-1-Organizational-Complexity/IAM/IAM-General.md) | Workload/user authorization, roles, policies, federation |
| [IAM Identity Center](../Domain-1-Organizational-Complexity/IAM/IAM-IdentityCenter-General.md) | Workforce SSO and permission sets across accounts |
| [Resource Control Policies](../Domain-1-Organizational-Complexity/IAM/ResourceControlPolicies-General.md) | Resource-side organization guardrails; does not grant access |
| [AWS RAM](../Domain-1-Organizational-Complexity/RAM-General.md) | Share supported resources across accounts/OUs |
| [AWS Config](../Domain-1-Organizational-Complexity/Config-General.md) | Resource configuration history, rules, and compliance |
| [Security Hub](../Domain-1-Organizational-Complexity/SecurityHub-General.md) | Centralize security findings and standards checks |
| [Security service selector](../Domain-1-Organizational-Complexity/Security-Services-Selector.md) | GuardDuty threat detection; Inspector vulnerability scanning; Macie S3 data discovery; WAF/Shield/Firewall Manager boundaries |
| [Cost visibility](../Domain-1-Organizational-Complexity/Cost-Visibility-General.md) | Cost Explorer analysis, Budgets alerts, billing exports, tags, commitments |
| [Deployment strategies](../Domain-2-New-Solutions/Deployment-Strategies-General.md) | CodePipeline orchestration, CodeBuild, CodeDeploy, canary/blue-green/rollback |
| [Purpose-built database selector](../Domain-2-New-Solutions/Databases/Purpose-Built-Database-Selector.md) | Document, wide-column, graph, time-series, warehouse, search, cache, relational |
| [AWS Elastic Disaster Recovery](../Domain-3-Continuous-Improvement/Elastic-Disaster-Recovery-General.md) | Server replication to staging, recovery drills, launch, and failback |
| [Transit Gateway](../Domain-1-Organizational-Complexity/Networking/TransitGateway-General.md) | Scalable routed hub for VPCs and hybrid attachments |
| [Direct Connect](../Domain-1-Organizational-Complexity/Networking/DirectConnect-General.md) | Dedicated hybrid connectivity; encryption is not automatic |
| [Site-to-Site VPN](../Domain-1-Organizational-Complexity/Networking/VPN-General.md) | IPsec tunnels for encrypted hybrid connectivity |
| [Advanced VPC design](../Domain-1-Organizational-Complexity/Networking/VPC-Advanced-General.md) | Routing, endpoints, inspection, segmentation, and hybrid DNS |
| [Route 53](../Domain-1-Organizational-Complexity/Route53-General.md) | DNS, routing policies, health checks, and resolver integration |

## Domain 2: New Solutions

| Service / topic | Recall cue |
|---|---|
| [EC2](../Domain-2-New-Solutions/Compute/EC2-Advanced-General.md) | OS/instance control, specialized software, lift-and-shift |
| [Auto Scaling](../Domain-2-New-Solutions/Compute/AutoScaling-General.md) | Replace/scale fleet capacity from demand and health signals |
| [ECS](../Domain-2-New-Solutions/Compute/ECS-General.md) | AWS-native container orchestration |
| [EKS](../Domain-2-New-Solutions/Compute/EKS-General.md) | Kubernetes API/ecosystem and associated operations |
| [Lambda](../Domain-2-New-Solutions/Compute/Lambda-General.md) | Event-driven functions within runtime and quota constraints |
| [CloudFront](../Domain-2-New-Solutions/Networking/CloudFront-General.md) | Edge content delivery and caching |
| [API Gateway](../Domain-2-New-Solutions/Networking/APIGateway-General.md) | Managed API front door, auth, throttling, and integrations |
| [Elastic Load Balancing](../Domain-2-New-Solutions/Networking/ELB-General.md) | ALB L7, NLB L4, GWLB appliance insertion |
| [Global Accelerator](../Domain-2-New-Solutions/Networking/GlobalAccelerator-General.md) | Anycast static IPs and health-based global traffic routing |
| [S3](../Domain-2-New-Solutions/Storage/S3-Advanced-General.md) | Durable object storage, lifecycle, replication, and access controls |
| [EFS](../Domain-2-New-Solutions/Storage/EFS-General.md) | Shared elastic NFS file storage |
| [FSx](../Domain-2-New-Solutions/Storage/FSx-General.md) | Managed file systems for Windows, Lustre, ONTAP, and OpenZFS use cases |
| [RDS / Aurora](../Domain-2-New-Solutions/Databases/RDS-Aurora-General.md) | Managed relational engines, HA, read scaling, and recovery |
| [DynamoDB](../Domain-2-New-Solutions/Databases/DynamoDB-General.md) | Key-value/document access-pattern-first design |
| [ElastiCache](../Domain-2-New-Solutions/Databases/ElastiCache-General.md) | In-memory cache; plan invalidation and fallback |
| [OpenSearch](../Domain-2-New-Solutions/Databases/OpenSearch-General.md) | Search and log analytics with index/shard operations |
| [Step Functions](../Domain-2-New-Solutions/Serverless/StepFunctions-General.md) | Durable orchestration, retries, branches, and state |
| [EventBridge](../Domain-2-New-Solutions/Serverless/EventBridge-General.md) | Event routing and filtering across services/accounts |
| [SQS / SNS](../Domain-2-New-Solutions/Serverless/SQS-SNS-General.md) | Queue-based decoupling versus pub/sub fanout |
| [Kinesis](../Domain-2-New-Solutions/Serverless/Kinesis-General.md) | Ordered/replayable streaming ingestion patterns |
| [Amazon MSK](../Domain-2-New-Solutions/MSK-General.md) | Managed Apache Kafka for Kafka-compatible workloads |
| [CloudFormation](../Domain-2-New-Solutions/CloudFormation-General.md) | Declarative AWS infrastructure lifecycle and drift management |
| [AWS CDK](../Domain-2-New-Solutions/CDK-General.md) | Programmatic infrastructure synthesized to CloudFormation |

## Domain 3: Continuous Improvement

| Service / topic | Recall cue |
|---|---|
| [CloudWatch](../Domain-3-Continuous-Improvement/Monitoring/CloudWatch-General.md) | Metrics, logs, alarms, dashboards, and operational signals |
| [CloudTrail](../Domain-3-Continuous-Improvement/Monitoring/CloudTrail-General.md) | AWS API audit events and governance evidence |
| [X-Ray](../Domain-3-Continuous-Improvement/Monitoring/XRay-General.md) | Distributed request tracing and service maps |
| [Systems Manager](../Domain-3-Continuous-Improvement/Monitoring/SystemsManager-General.md) | Fleet operations, patching, sessions, automation |
| [Cost Explorer](../Domain-3-Continuous-Improvement/CostOptimization/CostExplorer-General.md) | Analyze historical spend, trends, and forecasts |
| [Compute Optimizer](../Domain-3-Continuous-Improvement/CostOptimization/ComputeOptimizer-General.md) | Utilization-based rightsizing recommendations |
| [Trusted Advisor](../Domain-3-Continuous-Improvement/CostOptimization/TrustedAdvisor-General.md) | Broad best-practice checks; capabilities vary by plan |
| [Well-Architected Tool](../Domain-3-Continuous-Improvement/WellArchitected-Tool-General.md) | Review workloads against the six pillars |
| [AWS Backup](../Domain-3-Continuous-Improvement/Backup-General.md) | Centralized policy-based backup and recovery |
| [Resilience patterns](../Domain-3-Continuous-Improvement/Resilience-Patterns-General.md) | RTO/RPO, DR choices, failover, and fault isolation |
| [Service Quotas](../Domain-3-Continuous-Improvement/ServiceQuotas-General.md) | Monitor/request regional service capacity before limits |

## Domain 4: Migration and Modernization

| Service / topic | Recall cue |
|---|---|
| [AWS Transform MGN](../Domain-4-Migration-Modernization/Migration/MGN-General.md) | Server block replication and EC2 rehost cutover |
| [AWS DMS](../Domain-4-Migration-Modernization/Migration/DMS-General.md) | Database full load, CDC, and supported engine migrations |
| [DataSync](../Domain-4-Migration-Modernization/Migration/DataSync-General.md) | Online managed file/object transfers and recurring tasks |
| [Snow Family](../Domain-4-Migration-Modernization/Migration/SnowFamily-General.md) | Physical data transfer concept; verify current eligibility/availability |
| [Transfer Family](../Domain-4-Migration-Modernization/Migration/TransferFamily-General.md) | Managed SFTP/FTPS/FTP/AS2 and browser transfer interfaces |
| [Migration Hub](../Domain-4-Migration-Modernization/Migration/MigrationHub-General.md) | Inventory and progress tracking, not replication |
| [AWS MAP](../Domain-4-Migration-Modernization/Migration/MAP-General.md) | Migration assessment/mobilization program, not a data mover |
| [Microservices](../Domain-4-Migration-Modernization/Modernization/Microservices-Patterns-General.md) | Decompose by business boundary; migrate incrementally |
| [Container migration](../Domain-4-Migration-Modernization/Modernization/Containers-Migration-General.md) | Package workloads; select ECS/EKS/Fargate based on fit |
| [Serverless migration](../Domain-4-Migration-Modernization/Modernization/Serverless-Migration-General.md) | Event-driven managed compute with retries/idempotency |
| [Database migration patterns](../Domain-4-Migration-Modernization/Modernization/DatabaseMigration-Patterns-General.md) | Separate engine conversion, replication, validation, and cutover |

## Cross-Domain Rules

- Explicit constraints outrank general preferences; reject any option that violates a hard requirement.
- Identify the failure domain: host, AZ, Region, account, data store, network, or dependency.
- Replication is not backup; multi-AZ is not multi-Region; a private connection is not automatically encrypted.
- “Managed” does not remove the need for IAM, network, quotas, observability, testing, and cost controls.
- Verify version-sensitive service availability, limits, pricing, and exam facts in current official sources.

---

## 🎯 Architecture Decision Notes

### Textract — choose document intelligence, not just OCR

When the input is a scanned or photographed document and the requirement
goes beyond "read the characters," prefer a managed document-analysis
service to building an OCR pipeline. The decision ladder:

| Need | Choice |
|---|---|
| Plain text from an image | Basic OCR is enough |
| Typed *and* handwritten text plus structure | Document analysis with form/table extraction |
| "What is the invoice total?" / "Extract the vendor name" | Query-style or purpose-built expense analysis |
| Identity documents (licenses, passports) | Purpose-built ID analysis |

The trap is treating every document problem as an OCR problem: OCR gives
you characters, not the relationship between "Total" and "$1,240.00" in
a table, and not the field semantics of an invoice. Purpose-built
analysis APIs return key-value pairs and line items, which is what a
downstream workflow (approvals, ledger posting) actually consumes.

**Fresh example:** An AP automation flow receives emailed PDF invoices.
Instead of OCR plus fragile regex, the team extracts structured fields
(vendor, date, line items, total) with an expense-oriented analysis API,
matches totals against purchase orders, and routes only mismatches to
humans — reducing manual keying without custom ML.

### Contact-center voice AI — compose the pipeline from single-purpose services

A "smart call center" question is usually a composition problem. Each
service does one job; the architecture chains them:

| Stage | Service role |
|---|---|
| Caller speaks to an IVR / chatbot | Conversational interface (Lex) — understands intent, drives the dialogue |
| System speaks back | Text-to-speech (Polly) — natural voices in contact flows |
| Recordings become text | Speech-to-text (Transcribe) — transcription of stored audio |
| Transcripts become insight | Language analysis (Comprehend) — sentiment, key phrases, entities, topic detection |

Decision guidance: if the scenario asks for *understanding what callers
want* in real time, the conversational piece is the anchor. If it asks
for *reviewing what happened* across thousands of calls, the anchor is
transcribe-then-analyze — speech-to-text feeding language analysis for
sentiment trends, compliance keywords, and coaching insights.

Trap: do not pick a single service to "do AI on calls." Transcription
without analysis is just text; analysis without transcription has no
input; and neither replaces the conversational interface that handles
the live caller.

**Fresh example:** A support center wants to spot frustrated callers
before they churn. Call recordings land in S3, are transcribed to text,
and the transcripts are scored for sentiment and flagged for phrases
like "cancel my account." Supervisors get a daily digest of at-risk
calls — built from three managed services, with no custom ML.


---

### WorkSpaces vs AppStream 2.0 — desktops vs apps

Both deliver Windows/Linux applications to users without local installs, but they answer different questions:

- **WorkSpaces = a persistent virtual desktop (VDI).** Each user gets their own stateful desktop with their profile, files, and settings intact between sessions. Choose it for long-term remote-workforce desktop replacement — developers, back-office staff, anyone who lives in their desktop.
- **AppStream 2.0 = streaming individual applications.** Sessions are stateless and on-demand; the user gets the app, not a desktop. Choose it for short-lived or large-scale access to specific apps — contractors, students, or distributing a legacy Windows app as SaaS without installing it anywhere.

Shared traits the exam may mention: both run inside your VPC, integrate with AD/IAM Identity Center/MFA, and support encrypted storage. Decision rule: "needs *my* desktop that remembers me" → WorkSpaces; "needs *that app* for a while" → AppStream 2.0. Do not answer EC2-with-manual-patching for either — that's the operational-overhead distractor.

### Amazon WorkDocs — discontinued; recognize it only as a distractor

WorkDocs was AWS's managed enterprise document-collaboration service (the AWS-native alternative to SharePoint/Google Drive, pitched on IAM/CloudTrail/KMS integration for regulated industries). **AWS discontinued WorkDocs — the service is end-of-life and closed.** It can only appear in exam questions as a legacy distractor. See [DEPRECATED-SERVICES.md](../DEPRECATED-SERVICES.md). If a scenario asks for document collaboration or managed file sharing today, the current answers are S3-based patterns, WorkSpaces-family services, or third-party SaaS — never WorkDocs.
