# Architecture Decision Scenarios

Scenario-led practice for the AWS Certified Solutions Architect – Professional
(SAP-C02) exam. Each scenario starts from business requirements, works through
the candidate options, and explains the reasoning — including the traps that
make a plausible-looking answer wrong. The companion theme guides cover the
underlying concepts; this guide focuses on choosing between them.

## How to use these scenarios

For each scenario, cover the requirements and try to pick an answer before
reading the reasoning. SAP-C02 rarely asks "what is this service" — it asks
which combination of services meets a set of constraints with the least
operational burden. The traps usually hide in a single overlooked requirement:
an ordering guarantee, an encryption expectation, a failover time, or who
operates what.

---

## 1. Global storefront with regional failover

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

## 2. Order pipeline: queue, topic, bus, or stream?

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

## 3. Acquired company, overlapping networks

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

## 4. Nightly analytics on a budget

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

## 5. SaaS tenant isolation that auditors accept

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

## 6. Spiky relational workload

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

## 7. Ransomware-resilient backups

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

## 8. Cutting a monolith without stopping the business

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

## Decision-pattern cheat sheet

| When you see… | Think first of… | And check… |
|---|---|---|
| Overlapping VPC CIDRs, one service to share | PrivateLink | Endpoint policies, DNS |
| Dynamic API needing fast regional failover | Global Accelerator | Health checks, client behavior |
| Cacheable content, global users | CloudFront | Cache keys, origin TLS |
| Ordered, exactly-once work queue | SQS FIFO + idempotent consumer | DLQ, redrive, visibility timeout |
| Fan-out to many subscribers | SNS (or EventBridge for pattern routing) | Durable subscriptions for down consumers |
| Replayable event history, new consumers later | EventBridge (archive/replay) | Event schema discipline |
| Interruptible, checkpointed batch | Batch + Spot | Interruption handling, fallback capacity |
| Relational + spiky | Aurora Serverless v2 | Scaling limits, backup retention |
| Central kill-switch over many accounts | SCPs (restrictive) + per-account grants | Explicit deny precedence |
| Backups surviving account compromise | Separate account + Vault/Object Lock (Compliance) | Restore testing, key separation |
| Workforce sign-in across accounts | IAM Identity Center | Permission sets, IdP integration |
| App users calling AWS APIs directly | Cognito identity pools (after user-pool sign-in) | Least-privilege role per pool |
| Hybrid network, encryption required | DX + VPN overlay (or MACsec where supported) | Redundant paths |

## References

- [Amazon EC2 Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)
- [AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)
- [SQS FIFO queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html)
- [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [AWS Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)
- [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [DynamoDB global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [AWS Organizations SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [AWS Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [AWS Application Migration Service](https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html)
- [Cognito user pools vs identity pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)
- [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html)
