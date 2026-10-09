# Original SAP-C02 Practice Scenarios

These are free, original architecture exercises, not actual or reconstructed exam questions and not a timed/scored mock exam. Use a separate practice exam for full-length test simulation. Map each exercise to the current guide and decide before reading the recommendation.

## Domain 1: Organizational Complexity

### 1. Organization-Wide Data Guardrails

**Problem:** A company has dozens of AWS accounts. Security requires that teams cannot disable central audit logging, while workload teams must still create resources independently. A small set of security administrators needs centralized findings.

**Decision:** Apply preventive organization guardrails without turning the management account into a workload account; centralize audit and security operations.

**Recommended architecture:** Use AWS Organizations and Control Tower for account governance, SCPs for maximum-permission guardrails, centralized CloudTrail/Config, and delegated administration for supported security services. Keep workload deployment permissions in account IAM roles and automate approved exceptions.

**Why alternatives are weaker:** A shared administrator IAM role alone does not prevent an account administrator from changing controls. SCPs do not grant permissions and should not be used as the only access policy.

**Security / reliability / cost:** Restrict management-account use, log policy changes centrally, test guardrail impact in a pilot OU, and monitor service quotas before account rollout.

**Trap / takeaway:** Separate governance ceiling from workload authorization.

### 2. Encrypted Hybrid Connectivity

**Problem:** An enterprise needs predictable private connectivity between a data center and many VPCs. Policy requires encryption in transit, and a single circuit failure must not isolate workloads.

**Recommended architecture:** Use redundant Direct Connect connectivity for predictable private capacity and add IPsec VPN over the DX path where encryption is required, with a separate VPN/second DX failure path as appropriate. Connect VPCs through Transit Gateway and design routing/segmentation deliberately.

**Why alternatives are weaker:** DX alone is not encrypted by default and one circuit is a failure point. A full mesh of VPC peering is difficult to operate and non-transitive.

**Security / reliability / cost:** Validate route propagation, asymmetric paths, encryption, provider diversity, and failover behavior. Balance DX port/circuit costs against required bandwidth and resilience.

**Trap / takeaway:** “Private” does not mean encrypted or redundant.

### 3. Workforce Access Across Accounts

**Problem:** A company uses a central identity provider and wants employees to access multiple AWS accounts without distributing IAM user credentials. Developers should have different access from auditors.

**Recommended architecture:** Integrate IAM Identity Center with the identity source, assign users/groups to accounts through permission sets, and scope roles to job duties. Use Organizations for account structure and SCPs only as maximum-permission guardrails.

**Why alternatives are weaker:** Long-lived IAM access keys increase credential risk and lifecycle burden. Broad cross-account roles make separation of duties difficult.

**Security / reliability / cost:** Require MFA according to policy, test joiner/mover/leaver automation, log role activity, and maintain a carefully controlled emergency access path.

**Trap / takeaway:** Federation authenticates; permission sets and IAM policies authorize.

### 4. Cost Allocation and Early Warning

**Problem:** A company needs to assign cloud spend to products and alert owners before monthly budgets are exceeded. Its finance team also needs line-item analysis when an alert fires.

**Recommended architecture:** Define and activate cost-allocation tags and Cost Categories, use Cost Explorer for trends, configure AWS Budgets for actual/forecast notifications, and export detailed billing data to S3 for analysis. Assign owners and access controls to the reports.

**Why alternatives are weaker:** The Pricing Calculator estimates a proposed design; it does not analyze actual spend. Cost Explorer alone is not a hard spend cap and may not provide the detailed data workflow required.

**Trap / takeaway:** Separate allocation, analysis, alerts, detailed billing data, and pre-deployment estimates.

### 5. Multi-Account Security Findings

**Problem:** A security team needs to detect suspicious behavior, find vulnerable EC2 software, and discover sensitive data in S3 across member accounts. Findings must reach a central operations team.

**Recommended architecture:** Use GuardDuty for threat detection, Inspector for supported vulnerability findings, and Macie for S3 sensitive-data discovery. Configure organization enrollment/delegated administration and aggregate findings with Security Hub CSPM; route selected findings through EventBridge to a triage/remediation workflow.

**Why alternatives are weaker:** Security Hub CSPM aggregates posture and findings; it does not replace each detection source. Config evaluates resource configuration, not malicious activity.

**Trap / takeaway:** Choose the detection service from the type of risk, then centralize and respond.

## Domain 2: New Solutions

### 6. Global Static Content and Dynamic API

**Problem:** A media service has global users. Images and video should load quickly; a dynamic API needs stable entry points and fast failover between regional endpoints.

**Recommended architecture:** Use CloudFront for cacheable content and origin protection. Use Global Accelerator for the dynamic TCP/UDP application entry path when static anycast IPs and health-based endpoint routing are required. Keep application APIs and data services deployed resiliently in their Regions.

**Why alternatives are weaker:** Global Accelerator is not an object cache; CloudFront does not replace every arbitrary TCP/UDP routing need. Route 53 can route DNS but clients/resolvers may cache records during failover.

**Security / reliability / cost:** Use TLS, origin access restrictions, WAF where appropriate, health checks, and cache policies that avoid exposing private or user-specific content. Model edge requests, data transfer, and regional capacity.

**Trap / takeaway:** Match the global service to content caching versus connection routing.

### 7. Burst-Heavy Order Processing

**Problem:** A retail API sees sharp bursts. Orders must not be dropped if workers or the database are temporarily unavailable, and clients need a quick acknowledgement.

**Recommended architecture:** Put API Gateway or an appropriate load-balanced API in front of an ingestion service, durably enqueue accepted work in SQS, and process asynchronously with bounded concurrency. Use idempotency keys, DLQ/replay operations, and database capacity controls.

**Why alternatives are weaker:** Synchronous API-to-database processing couples customer latency and availability to worker/database load. SNS alone is pub/sub fanout, not necessarily the durable competing-consumer queue required.

**Security / reliability / cost:** Authenticate/authorize API requests, encrypt queue/data, scope producer/consumer roles, monitor queue age/depth and DLQ, and cap concurrency to protect the database.

**Trap / takeaway:** A queue absorbs bursts only when consumers, retries, and poison-message handling are designed.

### 8. Relational Workload with Read Spikes

**Problem:** A customer-facing application has a relational system of record, high read volume, and a requirement for automatic in-Region failover. Reads can tolerate modest replica lag, but writes must remain relationally consistent.

**Recommended architecture:** Use a managed relational service with Multi-AZ for availability and add read replicas or a cache for read scaling based on workload. Keep writes on the primary and explicitly route/read-test replica behavior.

**Why alternatives are weaker:** A read replica is not the Multi-AZ failover mechanism. Moving to DynamoDB solely for read scale requires access-pattern redesign and is not a drop-in relational substitute.

**Security / reliability / cost:** Use private subnets, TLS, managed credentials/secrets, backups/PITR, connection pooling, and load tests. Consider replica and cache costs plus invalidation complexity.

**Trap / takeaway:** Separate HA, read scaling, and caching requirements.

### 9. Stream Processing with Replay

**Problem:** An analytics platform ingests events at high volume. Consumers need ordered records per entity and the ability to replay a retention window after a processing bug.

**Recommended architecture:** Use a partitioned streaming service such as Kinesis Data Streams or managed Kafka (MSK) based on ecosystem/compatibility needs. Select partition keys for per-entity ordering and throughput distribution; make consumers checkpoint and handle duplicates.

**Why alternatives are weaker:** SQS queues do not provide the same retained ordered stream/replay semantics. A single hot partition key limits throughput.

**Security / reliability / cost:** Encrypt stream data, apply least-privilege producer/consumer roles, monitor lag and throttling, and set retention to match replay needs. Compare shard/capacity or broker costs against throughput.

**Trap / takeaway:** Ordering is scoped by partition/key, not necessarily global.

### 10. Safe Lambda Release

**Problem:** A payments API needs to release a new Lambda version gradually and automatically restore the prior version if error rates increase.

**Recommended architecture:** Use a versioned Lambda function and alias with a supported CodeDeploy canary or linear traffic-shifting deployment. Connect CloudWatch alarms and validation hooks to deployment stop/rollback behavior; keep changes backward-compatible with shared data stores.

**Why alternatives are weaker:** Updating `$LATEST` directly gives no controlled versioned traffic shift. A successful deployment action alone does not prove business behavior is healthy.

**Trap / takeaway:** Pair deployment strategy with measurable alarms and state-compatible rollback.

### 11. Mixed Database Workload

**Problem:** An application has transaction records, time-series device measurements, and multi-hop relationship queries. The existing relational database is overloaded by analytical scans and graph traversals.

**Recommended architecture:** Keep transactional records in a relational store if relational semantics are required. Evaluate Timestream for time-series access and Neptune for graph traversal only when their data models and query needs fit. Send analytical history to S3 and an analytics/warehouse path such as Redshift where appropriate; define data ownership and synchronization.

**Why alternatives are weaker:** Adding indexes or a cache to one relational engine may not address fundamentally different graph and time-series query shapes. Multiple stores add operational and consistency costs, so split only where measured benefits justify them.

**Trap / takeaway:** Purpose-built databases solve different access patterns; a polyglot design needs explicit consistency and cost controls.

### 12. Recover an On-Premises Server Estate

**Problem:** A company must recover supported on-premises servers into AWS after a site outage. It needs periodic non-disruptive drills and a short measured recovery objective, but it does not need a second full compute fleet running continuously.

**Recommended architecture:** Evaluate AWS Elastic Disaster Recovery for continuous replication to a staging area, prepare the recovery VPC and dependencies, and run isolated recovery drills. Measure server launch, application validation, traffic routing, and failback against the required RTO/RPO. Keep independent backups for point-in-time protection.

**Why alternatives are weaker:** MGN is primarily a migration/cutover tool; it is not a substitute for an ongoing DR plan. A replication dashboard is not evidence that the complete application meets its recovery target.

**Trap / takeaway:** Distinguish ongoing DR (DRS) from migration rehosting (MGN), and prove objectives with drills.

## Domain 3: Continuous Improvement

### 13. Regional Recovery Requirement

**Problem:** A business application has a one-hour RTO and five-minute RPO. A secondary Region must be ready without paying for two full-size production fleets continuously.

**Recommended architecture:** Use a warm-standby or carefully sized pilot-light design based on tested startup time, replicate data with a technology that meets the RPO, and automate promotion, scaling, and traffic failover. Rehearse failover/failback and measure actual recovery objectives.

**Why alternatives are weaker:** Backup/restore may not meet the RTO/RPO. Full active-active may exceed the cost requirement and requires conflict/data-consistency design.

**Security / reliability / cost:** Replicate encrypted data with controlled cross-account keys, protect backups from deletion, test application dependencies in the DR Region, and budget for replicated storage and standby compute.

**Trap / takeaway:** The label “warm standby” is not proof the measured RTO/RPO is met.

### 14. Find a Distributed Latency Bottleneck

**Problem:** A user-facing API has intermittent slow requests across several microservices. CPU averages are normal, and operators need to identify which dependency is responsible.

**Recommended architecture:** Instrument distributed traces with X-Ray or compatible tracing, correlate with CloudWatch metrics/logs, and review service maps and request-level latency. Add targeted metrics and alarms for the identified dependency.

**Why alternatives are weaker:** CloudTrail records AWS API activity, not application request spans. Scaling every service without evidence may increase cost and hide the cause.

**Security / reliability / cost:** Avoid sensitive data in traces, establish sampling appropriate to traffic, and use alarms/runbooks for actionable signals.

**Trap / takeaway:** Choose observability based on the question: audit, metrics/logs, or distributed traces.

### 15. Reduce Compute Spend Safely

**Problem:** A company suspects over-provisioned EC2 instances but has seasonal peaks and strict latency SLOs. It wants evidence before committing to a long-term discount plan.

**Recommended architecture:** Use Cost Explorer and Compute Optimizer recommendations, inspect peak and seasonal utilization plus application latency, test rightsizing in stages, then evaluate Savings Plans for stable eligible usage. Keep autoscaling and capacity headroom for peaks.

**Why alternatives are weaker:** Committing before understanding usage can lock in waste. Average CPU alone may miss memory, I/O, or burst demand.

**Security / reliability / cost:** Define rollback thresholds, use load tests, and monitor SLOs and spend after each change.

**Trap / takeaway:** Rightsize from representative evidence; commit only after the baseline is understood.

## Domain 4: Migration and Modernization

### 16. Data Center Exit with Poorly Documented Servers

**Problem:** A company must leave a data center in six months. It has hundreds of supported Windows/Linux servers, undocumented dependencies, and little capacity for application rewrites. Downtime must be short.

**Recommended architecture:** Build an application inventory and waves, use AWS Transform MGN for eligible server rehosting, test representative workloads in the target VPC, and coordinate DMS/DataSync for databases and files. Use Migration Hub for supported portfolio progress tracking.

**Why alternatives are weaker:** Refactoring everything before exit adds schedule and scope risk. Migration Hub does not replicate workloads.

**Security / reliability / cost:** Prepare the landing zone, IAM, network, logging, backups, and quotas. Test dependencies and rollback; measure replication lag and target performance. Run source and target in parallel only as long as required.

**Trap / takeaway:** Rehost can be a deliberate first step; it is not the final modernization design.

### 17. Heterogeneous Database Migration

**Problem:** A high-write database must move to a different engine with minimal downtime. The schema uses stored procedures and engine-specific functions.

**Recommended architecture:** Assess conversion coverage first, remediate code/schema incompatibilities, benchmark the target, and use DMS full load plus CDC only if the pair and workload are supported. Rehearse a controlled write freeze, final CDC catch-up, validation, endpoint switch, and rollback decision.

**Why alternatives are weaker:** DMS does not automatically convert every object or application query. A one-time export may exceed the downtime window.

**Security / reliability / cost:** Use least-privilege endpoints, TLS/KMS where supported, private routing, source-log retention, CDC-lag monitoring, and backup/restore tests. Include parallel database and replication costs.

**Trap / takeaway:** Data movement, schema conversion, application compatibility, and cutover are separate workstreams.

### 18. Multi-Terabyte File Migration

**Problem:** An organization must migrate a large NFS dataset to S3. Files continue to change during transfer. The WAN may or may not meet the deadline.

**Recommended architecture:** Measure available throughput and elapsed-time feasibility. Use DataSync for an initial online transfer plus repeatable deltas and integrity checks when the network fits. Freeze or queue writes for a final delta, validate, then redirect clients. If online transfer cannot meet the window, validate currently available physical-transfer alternatives and eligibility before planning shipment.

**Why alternatives are weaker:** Transfer Family is for protocol-facing user/partner workflows, not bulk NAS synchronization. Snowball Edge should not be assumed available to new customers.

**Security / reliability / cost:** Use encrypted paths, scoped IAM/storage permissions, manifests/checksums, logging, and source retention until acceptance. Compare network, service, storage, and physical logistics costs.

**Trap / takeaway:** Choose by transfer shape, throughput, and current service availability—not raw data size alone.

### 19. Modernize a Monolith Incrementally

**Problem:** A monolith’s catalog feature needs independent releases, while billing is stable and regulated. The company cannot fund a full rewrite.

**Recommended architecture:** Use a strangler pattern to extract the catalog bounded context behind an API or event boundary. Give it clear data ownership, deploy it on ECS/EKS or serverless based on runtime and operational fit, route a small portion of traffic, monitor, and retain rollback. Leave billing in place initially.

**Why alternatives are weaker:** A big-bang rewrite expands risk; extracting every module creates a distributed monolith without proving value.

**Security / reliability / cost:** Define service identities, API/event authorization, idempotency, versioned contracts, tracing, deployment rollback, and full cost including network/operations.

**Trap / takeaway:** Modernize the capability that has a measurable driver; do not transform architecture for its own sake.

---

## Cross-Domain Decision Scenarios

The scenarios below are organized by *decision* rather than by domain — each
one forces a choice between easily confused services. Work them
requirements-first: cover the recommendation, decide, then read the reasoning.

## 20. Global storefront with regional failover

**Scenario.** A retail company runs its storefront in `us-east-1`. Shoppers are
global. The business requires: checkout must stay available if a Region has an
impaired event, product catalog reads must be fast worldwide, and order data
must not be lost if the primary Region fails.

**Key decisions.** (a) How global traffic reaches the application. (b) How the
data tier survives a regional event.

**Reasoning.**

- For traffic entry, distinguish caching from failover. CloudFront caches
  HTTP(S) content at edge locations and is the right front door for the
  catalog and static assets. For the checkout API — dynamic, non-cacheable —
  AWS Global Accelerator gives static anycast IPs and fast regional failover
  for TCP/UDP. Using both is normal: CloudFront in front of cacheable
  content, Global Accelerator in front of the API endpoints.
- For the data tier, match the recovery requirement to the replication model.
  Aurora Global Database gives a primary cluster with read replicas in other
  Regions and a typical cross-Region failover measured in minutes, with
  control over the recovery point. DynamoDB global tables replicate with
  multi-active writes and single-digit-millisecond reads, but the data model
  must fit key-value access patterns. A relational order model with
  transactions points to Aurora; a session/cart model with simple lookups
  could suit DynamoDB.
- DNS failover with Route 53 health checks can steer traffic, but DNS answers
  are cached by clients — do not rely on DNS alone for fast failover of an
  API. Pair it with Global Accelerator or accept the propagation delay.

**Traps.**

- Picking CloudFront alone for the checkout API and assuming it provides
  regional failover for dynamic requests.
- Assuming a read replica is a backup: replicas replicate corruption too.
  Keep isolated recovery points (snapshots, AWS Backup) independent of
  replication.
- Treating "multi-AZ" as "multi-Region." An AZ event and a Region event are
  different blast radii with different answers.

## 21. Order pipeline: queue, topic, bus, or stream?

**Scenario.** An e-commerce platform must process orders through payment,
inventory, and notification steps. Requirements: no order lost if a downstream
step is temporarily down; inventory deduction must happen exactly once per
order; marketing wants to add new consumers of order events later without
changing the pipeline; analytics wants to replay the last 24 hours of orders
to rebuild a dashboard.

**Key decision.** Choose the messaging backbone.

**Reasoning.**

- **Amazon SQS** decouples producers from a single consumer group with
  durable buffering. FIFO queues preserve ordering and support
  content-based deduplication — the fit for inventory deduction where
  exactly-once processing matters. Standard queues offer higher throughput
  but at-least-once delivery without ordering.
- **Amazon SNS** is publish/subscribe fan-out: one message to many
  subscribers. It fits the notification step but is not a durable work
  queue — use it to fan out, not to buffer work.
- **Amazon EventBridge** routes events by pattern to many targets and fits
  the "add consumers later" requirement: new rules can match order events
  without touching the producer. Its archive/replay feature answers the
  analytics replay ask for events it has seen.
- **Kinesis Data Streams** fits when multiple consumers each need the full
  ordered stream with their own position — but it adds shard management
  and consumer-position bookkeeping the other options avoid.

A natural composition: EventBridge as the order-event bus (extensible
consumers, replay), SQS FIFO for the inventory step (ordering, dedup),
SNS for fan-out notifications. Do not force one service to do all four
jobs.

**Traps.**

- Using SNS where durable buffering is needed — a down subscriber loses
  messages unless the subscription itself is durable (e.g., an SQS
  subscription).
- Assuming FIFO "exactly-once" means the consumer can skip idempotency.
  Design the inventory update to tolerate a duplicate delivery anyway.
- Choosing Kinesis for a simple queue workload and paying the operational
  cost of stream management for no benefit.

## 22. Acquired company, overlapping networks

**Scenario.** A company acquires a competitor. Both run production in VPCs
using `10.0.0.0/16`. The parent needs to call the acquired company's
catalog API; full network merger is a next-year project. The API must not be
exposed to the internet, and the parent's security team will not allow
routing between the two VPCs.

**Key decision.** Connect one service across overlapping address space
without merging networks.

**Reasoning.**

- **AWS PrivateLink** is built for this: the API owner publishes an
  endpoint service (typically behind a Network Load Balancer); the consumer
  creates an interface endpoint with private IPs in its own VPC. No routes
  are exchanged between the VPCs, so overlapping CIDRs do not matter, and
  the API never touches the internet.
- VPC peering is the wrong tool here: peering is not transitive and, more
  importantly, requires non-overlapping ranges for routed communication.
  Transit Gateway has the same addressing constraint for routed traffic.
- For the longer-term merger, plan CIDR re-addressing or NAT-based
  isolation before attempting routed connectivity — do not start with
  peering and hope the overlap resolves itself.

**Fresh example.** The parent's checkout service calls the catalog API
through the interface endpoint's private DNS name. When the merger
completes next year and networks are re-addressed, the PrivateLink
connection can be retired without changing the calling code's endpoint
configuration pattern.

**Traps.**

- Proposing peering or Transit Gateway for overlapping CIDRs.
- Forgetting that an interface endpoint needs security-group and endpoint
  policy review — reachability is not authorization.
- Exposing the API through a public ALB "temporarily" and never
  revisiting it.

## 23. Nightly analytics on a budget

**Scenario.** A data team runs a nightly 3-hour analytics job over 2 TB of
data in S3. The job is checkpointed and can resume after interruption. The
CFO wants the lowest compute cost; the team wants no capacity management.

**Key decision.** Pick the compute purchasing and orchestration model.

**Reasoning.**

- **AWS Batch** with a Spot-backed compute environment matches this shape:
  queued jobs, automatic placement onto suitable capacity, checkpointing
  tolerates interruption. Fargate-backed Batch removes even the EC2
  capacity planning if the team will pay a premium per vCPU-hour for zero
  host management.
- **Spot Instances** are the cost lever: up to ~90% off for interruptible
  capacity. The checkpointing requirement is what makes Spot viable — an
  uncheckpointed 3-hour job is a bad Spot candidate.
- **Savings Plans** cover the steady baseline if part of the workload is
  predictable, but they are a billing commitment, not an architecture
  answer. Do not present a Savings Plan as the resilience plan.
- **Lambda** is wrong here: 15-minute execution limit and per-millisecond
  pricing punish long-running batch work.

**Traps.**

- Running the job on On-Demand "for simplicity" and ignoring a 70–90%
  cost reduction the requirements already unlock.
- Using Spot without interruption handling and discovering at 2 AM that
  the job restarts from zero.
- Confusing Fargate (no host management, higher unit cost) with "cheaper"
  — it trades cost for operational simplicity, which is the right trade
  only if the team values it.

## 24. SaaS tenant isolation that auditors accept

**Scenario.** A SaaS company hosts 200 tenants' workloads in one AWS
organization. Requirements: no tenant can reach another tenant's data plane;
central security can revoke any tenant's access quickly; tenant onboarding
must not require manual network changes; auditors need evidence of the
boundaries.

**Key decision.** Combine account strategy, network isolation, and
permission guardrails.

**Reasoning.**

- **One account per tenant** (or per tenant tier) is the strongest
  isolation boundary AWS offers. AWS Organizations with consolidated
  structure makes 200 accounts manageable; Control Tower-style automation
  handles onboarding.
- **Service Control Policies (SCPs)** set the maximum permissions in
  member accounts — the central kill-switch and guardrail. Remember SCPs
  do not grant anything; each account still needs identity policies for
  its workloads. An explicit deny in an SCP overrides any allow.
- **PrivateLink** exposes shared platform services (logging, metering)
  to tenant VPCs without cross-VPC routing.
- **Cognito user pools** handle tenant end-user sign-in per application;
  do not mix workforce access (IAM Identity Center) with customer
  identities.
- For evidence: IAM Access Analyzer findings, CloudTrail organization
  trails, and AWS Config rules give auditors the boundary proof. An
  architecture diagram is not evidence; logged enforcement is.

**Traps.**

- Putting all tenants in one account separated only by tags and IAM
  policies — one mis-scoped policy away from cross-tenant access.
- Assuming an SCP grants access ("we attached the SCP so the role can
  read the bucket") — SCPs only restrict.
- Using Cognito identity pools where user pools belong, or vice versa:
  pools authenticate users, identity pools vend AWS credentials.

## 25. Spiky relational workload

**Scenario.** A ticketing platform has 10x traffic spikes during on-sales,
then idles. The data model is relational with transactions. Requirements:
handle spikes without pre-provisioned waste; no data loss; recovery to any
point in the last 7 days.

**Key decision.** Choose the relational engine and capacity model.

**Reasoning.**

- **Aurora Serverless v2** scales compute in fine-grained increments with
  the workload — the fit for 10x spikes on a relational model. It keeps
  the MySQL/PostgreSQL compatibility the application already uses.
- **RDS with read replicas** handles read-heavy spikes but writes still
  funnel to one primary; it does not auto-scale compute.
- **DynamoDB on-demand** scales seamlessly but demands a key-value data
  model rethink — wrong when the application needs relational
  transactions and the team cannot rewrite the data layer before
  on-sale season.
- For point-in-time recovery: Aurora backtracks and restores to a
  point in time within the backup retention window; verify the retention
  covers the required 7 days rather than assuming defaults.

**Traps.**

- Choosing DynamoDB for a relational workload "because it scales" and
  discovering the data-model rewrite mid-project.
- Provisioned RDS sized for peak running 24/7 — the exact waste the
  requirements rule out.
- Forgetting that serverless scaling has limits and warm-up behavior;
  test the spike, do not assume instant infinite scale.

## 26. Ransomware-resilient backups

**Scenario.** After an industry ransomware incident, a company needs backups
that survive a compromised AWS account. Requirements: backups immutable for
90 days; recovery tested quarterly; backup data in a second Region; the
backup administrator must not be able to delete the backups they manage.

**Key decision.** Design the backup account and immutability controls.

**Reasoning.**

- Put backups in a **dedicated backup account** inside the organization,
  with an SCP denying deletion of recovery points. Cross-account backup
  copies (AWS Backup supports cross-account and cross-Region copy) mean a
  compromised workload account cannot reach the vault.
- **Vault Lock** (Compliance mode) makes recovery points immutable for the
  retention period — not even the root user of the backup account can
  delete them early. This is the control that answers "the admin must not
  be able to delete."
- **S3 Object Lock** (Compliance mode) is the equivalent for object data
  with its own retention rules; use it where backups land in S3 directly.
- Quarterly restore tests are part of the design, not a follow-up task:
  an untested backup is a hope, and SAP-C02 treats "tested" as a
  requirement word, not decoration.

**Traps.**

- Keeping backups in the same account as the workload — one compromised
  credential set can delete both.
- Using S3 Versioning alone as "immutability" — versions can be deleted;
  Object Lock cannot (in Compliance mode).
- Replicating to a second Region but with the same KMS key and admin
  — shared fate dressed as redundancy.

## 27. Cutting a monolith without stopping the business

**Scenario.** A 10-year-old monolith on EC2 must move to AWS with minimal
downtime and minimal rewrite. The database is 4 TB. Requirements: cutover
weekend under 4 hours; rollback possible; the application cannot be
redesigned this year.

**Key decision.** Pick the migration path and the data-migration method.

**Reasoning.**

- **Rehost first** with AWS Application Migration Service: replicate
  servers continuously, cut over in the window, keep the source running
  for rollback. The "no redesign this year" constraint rules out
  re-architecting now — sequence it later.
- **AWS DMS** with ongoing replication moves the 4 TB database with
  minimal downtime: full load plus change data capture, then a short
  cutover. Size the replication instance and test with production-like
  change rates; DMS is not magic for unsupported source/target
  combinations — check the matrix.
- For the cutover itself: pre-warm the target (AMIs, auto-scaling minima,
  database parameter groups), rehearse the DNS cutover, and define the
  rollback trigger before the weekend starts.

**Traps.**

- Choosing re-architecting because it is "better" while ignoring the
  stated constraints — SAP-C02 grades against requirements, not ideals.
- Treating the migration tools as the plan: discovery (dependencies,
  licensing), data transfer time for 4 TB, and rollback criteria are the
  plan; the tools execute it.
- No rollback rehearsal. The rollback path untested is the rollback path
  unavailable.

---

## How to Review Your Answers

For each scenario, ask: Which hard requirement decided the answer? Which plausible alternative failed it? What is the data source of truth during transition? What happens when a dependency fails? How will the team test, observe, secure, roll back, and pay for the design?
