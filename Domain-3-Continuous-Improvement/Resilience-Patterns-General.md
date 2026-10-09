> Exam relevance: Domains 2, 3. DR strategies and resilience patterns are core SAP-C02 topics. Know the 4 DR strategies, RTO/RPO trade-offs, and multi-region patterns cold.

---

## 1. RTO and RPO

| Metric | Definition | Lower = |
|---|---|---|
| RTO (Recovery Time Objective) | Max acceptable downtime after a disaster | Faster recovery, higher cost |
| RPO (Recovery Point Objective) | Max acceptable data loss (time since last backup) | Less data loss, higher cost |

---

## 2. DR Strategies (AWS)

### 1. Backup and Restore

```
Active Region                    DR Region
Primary workload ──── backup ──→ S3 / AWS Backup (snapshots)
                                        ↓ (disaster occurs)
                                 Restore from backup (hours)
```

- **RTO:** hours
- **RPO:** hours (depends on backup frequency)
- **Cost:** lowest — pay only for storage
- **Use case:** non-critical workloads, cost-sensitive

### 2. Pilot Light

```
Active Region                    DR Region
Full workload ──── replicate ──→ Core services only (DB replica, minimal infra)
                                        ↓ (disaster occurs)
                                 Scale up remaining services (minutes to hours)
```

- **RTO:** minutes to hours (need to scale up)
- **RPO:** minutes (near real-time replication)
- **Cost:** low — only core services running in DR
- **Use case:** critical data, moderate recovery time acceptable

### 3. Warm Standby

```
Active Region                    DR Region
Full workload ──── replicate ──→ Scaled-down but functional workload
                                        ↓ (disaster occurs)
                                 Scale up to full capacity (minutes)
```

- **RTO:** minutes
- **RPO:** seconds to minutes
- **Cost:** moderate — DR environment always running at reduced capacity
- **Use case:** business-critical workloads

### 4. Multi-Site Active-Active

```
Active Region (us-east-1)        Active Region (eu-west-1)
Full workload ←── Route 53 ──→ Full workload
              (latency routing)
Both application stacks can serve traffic; the data write model needs its own consistency/conflict design
```

- **RTO:** can be low with healthy capacity and tested traffic controls; not guaranteed to be zero
- **RPO:** depends on the replication and consistency model; asynchronous replication can lose acknowledged writes
- **Cost:** highest — full infrastructure in multiple regions
- **Use case:** mission-critical, zero-downtime requirement

---

## 3. DR Strategy Comparison

| Strategy | RTO | RPO | Cost | Complexity |
|---|---|---|---|---|
| Backup & Restore | Usually longest; measured restore time | Depends on backup frequency and consistency | Lowest ongoing infrastructure cost | Lower steady-state, higher recovery effort |
| Pilot Light | Depends on provisioning, scaling, and validation | Depends on replication/backup design | Low-to-moderate | Medium |
| Warm Standby | Faster when continuously tested and ready | Depends on replication mode and lag | Moderate-to-high | Medium-high |
| Multi-Site Active-Active | Can reduce traffic-recovery time; not automatically zero | Depends on consistency and replication model | Highest | High |

These are relative patterns, not fixed service guarantees. Measure failover, recovery, and data-loss windows against the workload's actual RTO/RPO.

---

## 4. AWS Services for Each Strategy

### Backup and Restore
- **AWS Backup** — centralized backup across services
- **S3 Cross-Region Replication** — replicate data to DR region
- **RDS snapshots** — copy to DR region
- **AMI copy** — copy EC2 AMIs to DR region

### Pilot Light
- **Aurora Global Database** — read replica in DR region (promote on failover)
- **RDS cross-region read replica** — promote on failover
- **Route 53 failover routing** — switch DNS to DR region
- **CloudFormation** — pre-define DR infrastructure, deploy on demand

### Warm Standby
- **Aurora Global Database** — near real-time replication
- **DynamoDB Global Tables** — multi-region active-active
- **Auto Scaling** — scale up DR environment on failover
- **Route 53 health checks** — automatic DNS failover

### Multi-Site Active-Active
- **DynamoDB Global Tables** — can support multi-Region reads/writes; select MREC or MRSC and design for its consistency/conflict model
- **Aurora Global Database** — one primary writer Region with read-only secondary Regions; supported write forwarding sends secondary writes to the primary, not independent regional writers
- **Route 53 latency routing** — route users to nearest region
- **Global Accelerator** — anycast routing to nearest healthy endpoint
- **CloudFront** — global content delivery

---

## 5. Resilience Patterns

### Multi-AZ (Single Region)

```
AZ-a: EC2 + RDS Primary
AZ-b: EC2 + RDS Standby (auto-failover)
AZ-c: EC2 + RDS Read Replica
ALB distributes across all AZs
```

- Protects against AZ failure
- RTO: minutes (RDS Multi-AZ auto-failover)
- RPO: typically near zero for acknowledged writes in supported synchronous deployments; verify engine and commit semantics

### Circuit Breaker

- Detect when a downstream service is failing
- Stop sending requests to the failing service (open circuit)
- Periodically test if service recovered (half-open)
- Resume traffic when healthy (closed circuit)
- Prevents cascade failures

### Bulkhead

- Isolate components so failure in one doesn't affect others
- Example: separate thread pools, connection pools, or queues per downstream service
- In AWS: separate SQS queues, separate Lambda concurrency reservations

### Retry with Exponential Backoff and Jitter

```python
import random, time

def retry_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except TransientError:
            wait = (2 ** attempt) + random.uniform(0, 1)  # jitter
            time.sleep(wait)
    raise MaxRetriesExceeded()
```

- Jitter prevents thundering herd when many clients retry simultaneously
- AWS SDKs implement this by default

---

## 6. Exam Scenarios

**Scenario 1:** A company has an RTO of 4 hours and RPO of 1 hour for a non-critical internal application. What is the most cost-effective DR strategy?

> **Backup and Restore**. Take hourly RDS snapshots and copy to the DR region (meets 1-hour RPO). Store CloudFormation templates and AMIs in the DR region. In a disaster, restore from snapshots and deploy infrastructure from CloudFormation (meets 4-hour RTO).

**Scenario 2:** A company has an RTO of 15 minutes and RPO of 5 minutes for a business-critical application. What DR strategy?

> **Warm Standby**. Run a scaled-down version of the application in the DR region with an Aurora Global Database replica. Use Route 53 failover routing with health checks. On disaster, promote the Aurora replica and scale up the DR Auto Scaling group. Route 53 automatically redirects traffic.

**Scenario 3:** A financial services company requires zero data loss and near-zero downtime. What DR strategy?

> First clarify whether zero data loss is mandatory for every failure and whether the application can accept the latency and availability trade-offs of synchronous or strongly consistent cross-Region writes. Multi-site application capacity can reduce recovery time, but it does not itself guarantee zero data loss or zero downtime. For DynamoDB, evaluate MRSC where the required topology is supported; Aurora Global Database has a primary writer and read-only secondaries, with regional writes forwarded to the primary only when supported. Define traffic failover, conflict handling, and failback, then test the stated RTO/RPO.

---

## 7. Exam Tips

- Know the **4 DR strategies** and their RTO/RPO/cost trade-offs cold
- **Backup & Restore** = cheapest, slowest; **Multi-Site Active-Active** = most expensive, fastest
- **Aurora Global Database** = primary writer plus read-only secondaries; typical replication lag is not a hard RPO guarantee
- **DynamoDB Global Tables** = choose between MREC and MRSC; topology, latency, consistency, and conflict handling matter
- **Route 53 failover routing** = DNS-based failover — has TTL delay (set low TTL for faster failover)
- **Global Accelerator** = faster failover than Route 53 (anycast, no DNS TTL)
- Multi-AZ ≠ Multi-Region — Multi-AZ protects against AZ failure, not region failure

---

## 8. 🎯 Architecture Decision Notes

### Set RTO/RPO from business impact, then test the design

RTO (how long to restore) and RPO (how much data loss is acceptable) are
business decisions first, architecture second. Different tiers deserve
different objectives — a reporting portal tolerating hours of rebuild and a
transaction ledger tolerating minutes of loss should not share one
active/active bill.

| Pattern | Readiness | Cost profile |
|---|---|---|
| Backup and restore | Rebuild after event | Cheapest idle; slowest recovery |
| Pilot light | Core data/components ready | Moderate |
| Warm standby | Reduced functional copy, scale on event | Higher |
| Multi-site active/active | Serving from multiple locations | Highest; adds consistency complexity |

These are patterns, not guaranteed numbers — actual recovery depends on
data volume, automation, dependencies, DNS behavior, and human steps.
Rehearse failover *and* failback; measure a real restore before claiming an
objective is met.

## 9. Official References

- [AWS DR Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [AWS Resilience Hub](https://docs.aws.amazon.com/resilience-hub/latest/userguide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
