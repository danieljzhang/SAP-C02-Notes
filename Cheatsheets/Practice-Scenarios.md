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

### Additional Scenario: Cost Allocation and Early Warning

**Problem:** A company needs to assign cloud spend to products and alert owners before monthly budgets are exceeded. Its finance team also needs line-item analysis when an alert fires.

**Recommended architecture:** Define and activate cost-allocation tags and Cost Categories, use Cost Explorer for trends, configure AWS Budgets for actual/forecast notifications, and export detailed billing data to S3 for analysis. Assign owners and access controls to the reports.

**Why alternatives are weaker:** The Pricing Calculator estimates a proposed design; it does not analyze actual spend. Cost Explorer alone is not a hard spend cap and may not provide the detailed data workflow required.

**Trap / takeaway:** Separate allocation, analysis, alerts, detailed billing data, and pre-deployment estimates.

### Additional Scenario: Multi-Account Security Findings

**Problem:** A security team needs to detect suspicious behavior, find vulnerable EC2 software, and discover sensitive data in S3 across member accounts. Findings must reach a central operations team.

**Recommended architecture:** Use GuardDuty for threat detection, Inspector for supported vulnerability findings, and Macie for S3 sensitive-data discovery. Configure organization enrollment/delegated administration and aggregate findings with Security Hub CSPM; route selected findings through EventBridge to a triage/remediation workflow.

**Why alternatives are weaker:** Security Hub CSPM aggregates posture and findings; it does not replace each detection source. Config evaluates resource configuration, not malicious activity.

**Trap / takeaway:** Choose the detection service from the type of risk, then centralize and respond.

## Domain 2: New Solutions

### 4. Global Static Content and Dynamic API

**Problem:** A media service has global users. Images and video should load quickly; a dynamic API needs stable entry points and fast failover between regional endpoints.

**Recommended architecture:** Use CloudFront for cacheable content and origin protection. Use Global Accelerator for the dynamic TCP/UDP application entry path when static anycast IPs and health-based endpoint routing are required. Keep application APIs and data services deployed resiliently in their Regions.

**Why alternatives are weaker:** Global Accelerator is not an object cache; CloudFront does not replace every arbitrary TCP/UDP routing need. Route 53 can route DNS but clients/resolvers may cache records during failover.

**Security / reliability / cost:** Use TLS, origin access restrictions, WAF where appropriate, health checks, and cache policies that avoid exposing private or user-specific content. Model edge requests, data transfer, and regional capacity.

**Trap / takeaway:** Match the global service to content caching versus connection routing.

### 5. Burst-Heavy Order Processing

**Problem:** A retail API sees sharp bursts. Orders must not be dropped if workers or the database are temporarily unavailable, and clients need a quick acknowledgement.

**Recommended architecture:** Put API Gateway or an appropriate load-balanced API in front of an ingestion service, durably enqueue accepted work in SQS, and process asynchronously with bounded concurrency. Use idempotency keys, DLQ/replay operations, and database capacity controls.

**Why alternatives are weaker:** Synchronous API-to-database processing couples customer latency and availability to worker/database load. SNS alone is pub/sub fanout, not necessarily the durable competing-consumer queue required.

**Security / reliability / cost:** Authenticate/authorize API requests, encrypt queue/data, scope producer/consumer roles, monitor queue age/depth and DLQ, and cap concurrency to protect the database.

**Trap / takeaway:** A queue absorbs bursts only when consumers, retries, and poison-message handling are designed.

### 6. Relational Workload with Read Spikes

**Problem:** A customer-facing application has a relational system of record, high read volume, and a requirement for automatic in-Region failover. Reads can tolerate modest replica lag, but writes must remain relationally consistent.

**Recommended architecture:** Use a managed relational service with Multi-AZ for availability and add read replicas or a cache for read scaling based on workload. Keep writes on the primary and explicitly route/read-test replica behavior.

**Why alternatives are weaker:** A read replica is not the Multi-AZ failover mechanism. Moving to DynamoDB solely for read scale requires access-pattern redesign and is not a drop-in relational substitute.

**Security / reliability / cost:** Use private subnets, TLS, managed credentials/secrets, backups/PITR, connection pooling, and load tests. Consider replica and cache costs plus invalidation complexity.

**Trap / takeaway:** Separate HA, read scaling, and caching requirements.

### 7. Stream Processing with Replay

**Problem:** An analytics platform ingests events at high volume. Consumers need ordered records per entity and the ability to replay a retention window after a processing bug.

**Recommended architecture:** Use a partitioned streaming service such as Kinesis Data Streams or managed Kafka (MSK) based on ecosystem/compatibility needs. Select partition keys for per-entity ordering and throughput distribution; make consumers checkpoint and handle duplicates.

**Why alternatives are weaker:** SQS queues do not provide the same retained ordered stream/replay semantics. A single hot partition key limits throughput.

**Security / reliability / cost:** Encrypt stream data, apply least-privilege producer/consumer roles, monitor lag and throttling, and set retention to match replay needs. Compare shard/capacity or broker costs against throughput.

**Trap / takeaway:** Ordering is scoped by partition/key, not necessarily global.

### Additional Scenario: Safe Lambda Release

**Problem:** A payments API needs to release a new Lambda version gradually and automatically restore the prior version if error rates increase.

**Recommended architecture:** Use a versioned Lambda function and alias with a supported CodeDeploy canary or linear traffic-shifting deployment. Connect CloudWatch alarms and validation hooks to deployment stop/rollback behavior; keep changes backward-compatible with shared data stores.

**Why alternatives are weaker:** Updating `$LATEST` directly gives no controlled versioned traffic shift. A successful deployment action alone does not prove business behavior is healthy.

**Trap / takeaway:** Pair deployment strategy with measurable alarms and state-compatible rollback.

### Additional Scenario: Mixed Database Workload

**Problem:** An application has transaction records, time-series device measurements, and multi-hop relationship queries. The existing relational database is overloaded by analytical scans and graph traversals.

**Recommended architecture:** Keep transactional records in a relational store if relational semantics are required. Evaluate Timestream for time-series access and Neptune for graph traversal only when their data models and query needs fit. Send analytical history to S3 and an analytics/warehouse path such as Redshift where appropriate; define data ownership and synchronization.

**Why alternatives are weaker:** Adding indexes or a cache to one relational engine may not address fundamentally different graph and time-series query shapes. Multiple stores add operational and consistency costs, so split only where measured benefits justify them.

**Trap / takeaway:** Purpose-built databases solve different access patterns; a polyglot design needs explicit consistency and cost controls.

### Additional Scenario: Recover an On-Premises Server Estate

**Problem:** A company must recover supported on-premises servers into AWS after a site outage. It needs periodic non-disruptive drills and a short measured recovery objective, but it does not need a second full compute fleet running continuously.

**Recommended architecture:** Evaluate AWS Elastic Disaster Recovery for continuous replication to a staging area, prepare the recovery VPC and dependencies, and run isolated recovery drills. Measure server launch, application validation, traffic routing, and failback against the required RTO/RPO. Keep independent backups for point-in-time protection.

**Why alternatives are weaker:** MGN is primarily a migration/cutover tool; it is not a substitute for an ongoing DR plan. A replication dashboard is not evidence that the complete application meets its recovery target.

**Trap / takeaway:** Distinguish ongoing DR (DRS) from migration rehosting (MGN), and prove objectives with drills.

## Domain 3: Continuous Improvement

### 8. Regional Recovery Requirement

**Problem:** A business application has a one-hour RTO and five-minute RPO. A secondary Region must be ready without paying for two full-size production fleets continuously.

**Recommended architecture:** Use a warm-standby or carefully sized pilot-light design based on tested startup time, replicate data with a technology that meets the RPO, and automate promotion, scaling, and traffic failover. Rehearse failover/failback and measure actual recovery objectives.

**Why alternatives are weaker:** Backup/restore may not meet the RTO/RPO. Full active-active may exceed the cost requirement and requires conflict/data-consistency design.

**Security / reliability / cost:** Replicate encrypted data with controlled cross-account keys, protect backups from deletion, test application dependencies in the DR Region, and budget for replicated storage and standby compute.

**Trap / takeaway:** The label “warm standby” is not proof the measured RTO/RPO is met.

### 9. Find a Distributed Latency Bottleneck

**Problem:** A user-facing API has intermittent slow requests across several microservices. CPU averages are normal, and operators need to identify which dependency is responsible.

**Recommended architecture:** Instrument distributed traces with X-Ray or compatible tracing, correlate with CloudWatch metrics/logs, and review service maps and request-level latency. Add targeted metrics and alarms for the identified dependency.

**Why alternatives are weaker:** CloudTrail records AWS API activity, not application request spans. Scaling every service without evidence may increase cost and hide the cause.

**Security / reliability / cost:** Avoid sensitive data in traces, establish sampling appropriate to traffic, and use alarms/runbooks for actionable signals.

**Trap / takeaway:** Choose observability based on the question: audit, metrics/logs, or distributed traces.

### 10. Reduce Compute Spend Safely

**Problem:** A company suspects over-provisioned EC2 instances but has seasonal peaks and strict latency SLOs. It wants evidence before committing to a long-term discount plan.

**Recommended architecture:** Use Cost Explorer and Compute Optimizer recommendations, inspect peak and seasonal utilization plus application latency, test rightsizing in stages, then evaluate Savings Plans for stable eligible usage. Keep autoscaling and capacity headroom for peaks.

**Why alternatives are weaker:** Committing before understanding usage can lock in waste. Average CPU alone may miss memory, I/O, or burst demand.

**Security / reliability / cost:** Define rollback thresholds, use load tests, and monitor SLOs and spend after each change.

**Trap / takeaway:** Rightsize from representative evidence; commit only after the baseline is understood.

## Domain 4: Migration and Modernization

### 11. Data Center Exit with Poorly Documented Servers

**Problem:** A company must leave a data center in six months. It has hundreds of supported Windows/Linux servers, undocumented dependencies, and little capacity for application rewrites. Downtime must be short.

**Recommended architecture:** Build an application inventory and waves, use AWS Transform MGN for eligible server rehosting, test representative workloads in the target VPC, and coordinate DMS/DataSync for databases and files. Use Migration Hub for supported portfolio progress tracking.

**Why alternatives are weaker:** Refactoring everything before exit adds schedule and scope risk. Migration Hub does not replicate workloads.

**Security / reliability / cost:** Prepare the landing zone, IAM, network, logging, backups, and quotas. Test dependencies and rollback; measure replication lag and target performance. Run source and target in parallel only as long as required.

**Trap / takeaway:** Rehost can be a deliberate first step; it is not the final modernization design.

### 12. Heterogeneous Database Migration

**Problem:** A high-write database must move to a different engine with minimal downtime. The schema uses stored procedures and engine-specific functions.

**Recommended architecture:** Assess conversion coverage first, remediate code/schema incompatibilities, benchmark the target, and use DMS full load plus CDC only if the pair and workload are supported. Rehearse a controlled write freeze, final CDC catch-up, validation, endpoint switch, and rollback decision.

**Why alternatives are weaker:** DMS does not automatically convert every object or application query. A one-time export may exceed the downtime window.

**Security / reliability / cost:** Use least-privilege endpoints, TLS/KMS where supported, private routing, source-log retention, CDC-lag monitoring, and backup/restore tests. Include parallel database and replication costs.

**Trap / takeaway:** Data movement, schema conversion, application compatibility, and cutover are separate workstreams.

### 13. Multi-Terabyte File Migration

**Problem:** An organization must migrate a large NFS dataset to S3. Files continue to change during transfer. The WAN may or may not meet the deadline.

**Recommended architecture:** Measure available throughput and elapsed-time feasibility. Use DataSync for an initial online transfer plus repeatable deltas and integrity checks when the network fits. Freeze or queue writes for a final delta, validate, then redirect clients. If online transfer cannot meet the window, validate currently available physical-transfer alternatives and eligibility before planning shipment.

**Why alternatives are weaker:** Transfer Family is for protocol-facing user/partner workflows, not bulk NAS synchronization. Snowball Edge should not be assumed available to new customers.

**Security / reliability / cost:** Use encrypted paths, scoped IAM/storage permissions, manifests/checksums, logging, and source retention until acceptance. Compare network, service, storage, and physical logistics costs.

**Trap / takeaway:** Choose by transfer shape, throughput, and current service availability—not raw data size alone.

### 14. Modernize a Monolith Incrementally

**Problem:** A monolith’s catalog feature needs independent releases, while billing is stable and regulated. The company cannot fund a full rewrite.

**Recommended architecture:** Use a strangler pattern to extract the catalog bounded context behind an API or event boundary. Give it clear data ownership, deploy it on ECS/EKS or serverless based on runtime and operational fit, route a small portion of traffic, monitor, and retain rollback. Leave billing in place initially.

**Why alternatives are weaker:** A big-bang rewrite expands risk; extracting every module creates a distributed monolith without proving value.

**Security / reliability / cost:** Define service identities, API/event authorization, idempotency, versioned contracts, tracing, deployment rollback, and full cost including network/operations.

**Trap / takeaway:** Modernize the capability that has a measurable driver; do not transform architecture for its own sake.

## How to Review Your Answers

For each scenario, ask: Which hard requirement decided the answer? Which plausible alternative failed it? What is the data source of truth during transition? What happens when a dependency fails? How will the team test, observe, secure, roll back, and pay for the design?
