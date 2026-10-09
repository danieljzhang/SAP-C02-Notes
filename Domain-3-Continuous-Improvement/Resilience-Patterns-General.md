# Resilience Patterns and DR Strategies — SAP-C02 Study Notes

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
Both regions serve traffic simultaneously
```

- **RTO:** near zero (traffic rerouted automatically)
- **RPO:** near zero (synchronous or near-synchronous replication)
- **Cost:** highest — full infrastructure in multiple regions
- **Use case:** mission-critical, zero-downtime requirement

---

## 3. DR Strategy Comparison

| Strategy | RTO | RPO | Cost | Complexity |
|---|---|---|---|---|
| Backup & Restore | Hours | Hours | $ | Low |
| Pilot Light | Minutes–Hours | Minutes | $$ | Medium |
| Warm Standby | Minutes | Seconds–Minutes | $$$ | Medium-High |
| Multi-Site Active-Active | Seconds | Seconds | $$$$ | High |

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
- **Aurora Global Database** — active in multiple regions
- **DynamoDB Global Tables** — active in multiple regions
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
- RPO: near zero (synchronous replication)

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

> **Multi-Site Active-Active**. Deploy the full application in two regions. Use DynamoDB Global Tables or Aurora Global Database for data replication. Use Route 53 latency routing or Global Accelerator to distribute traffic. Both regions serve traffic simultaneously — no failover needed.

---

## 7. Exam Tips

- Know the **4 DR strategies** and their RTO/RPO/cost trade-offs cold
- **Backup & Restore** = cheapest, slowest; **Multi-Site Active-Active** = most expensive, fastest
- **Aurora Global Database** = < 1 second replication lag — use for Pilot Light and Warm Standby
- **DynamoDB Global Tables** = multi-region active-active — use for Multi-Site Active-Active
- **Route 53 failover routing** = DNS-based failover — has TTL delay (set low TTL for faster failover)
- **Global Accelerator** = faster failover than Route 53 (anycast, no DNS TTL)
- Multi-AZ ≠ Multi-Region — Multi-AZ protects against AZ failure, not region failure

---

## 8. Official References

- [AWS DR Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [AWS Resilience Hub](https://docs.aws.amazon.com/resilience-hub/latest/userguide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
