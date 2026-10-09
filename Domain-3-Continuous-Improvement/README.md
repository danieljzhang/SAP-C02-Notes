> Third highest weight domain. Focuses on making existing architectures better — more observable, more cost-efficient, more reliable, and more performant. Expect ~16 questions.

---

## What This Domain Tests

- Improving observability with CloudWatch, CloudTrail, X-Ray, and Systems Manager
- Optimizing costs with Cost Explorer, Compute Optimizer, and Trusted Advisor
- Improving reliability with Well-Architected reviews, Backup, and resilience patterns
- Identifying and resolving performance bottlenecks
- Automating operational tasks with Systems Manager
- Managing service quotas and capacity

---

## Subdomain Breakdown

| Subdomain | Key Services |
|---|---|
| 3.1 Determine a strategy to improve overall operational excellence | Systems Manager, CloudFormation, Config, CloudTrail |
| 3.2 Determine a strategy to improve security | Security Hub, GuardDuty, IAM Access Analyzer, Macie |
| 3.3 Determine a strategy to improve performance | CloudWatch, X-Ray, Compute Optimizer, ElastiCache |
| 3.4 Determine a strategy to improve reliability | Backup, Resilience Hub, Route 53, Auto Scaling |
| 3.5 Identify opportunities for cost optimizations | Cost Explorer, Trusted Advisor, Savings Plans, S3 lifecycle |

> Task names aligned to the [official SAP-C02 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain3.html). The guide may change; recheck before exam day.

---

## Files in This Domain

### Monitoring/
- [CloudWatch-General.md](./Monitoring/CloudWatch-General.md) — metrics, alarms, dashboards, Logs Insights, Container Insights
- [CloudTrail-General.md](./Monitoring/CloudTrail-General.md) — trails, event history, Lake, Insights
- [XRay-General.md](./Monitoring/XRay-General.md) — distributed tracing, service maps, sampling
- [SystemsManager-General.md](./Monitoring/SystemsManager-General.md) — Parameter Store, Secrets Manager, Run Command, Patch Manager, Session Manager

### CostOptimization/
- [CostExplorer-General.md](./CostOptimization/CostExplorer-General.md) — cost analysis, forecasting, rightsizing
- [ComputeOptimizer-General.md](./CostOptimization/ComputeOptimizer-General.md) — ML-based rightsizing recommendations
- [TrustedAdvisor-General.md](./CostOptimization/TrustedAdvisor-General.md) — checks across six categories; plan availability changes over time

### Standalone
- [WellArchitected-Tool-General.md](./WellArchitected-Tool-General.md) — framework reviews, lenses, improvement plans
- [Backup-General.md](./Backup-General.md) — centralized backup, policies, cross-region, cross-account
- [Resilience-Patterns-General.md](./Resilience-Patterns-General.md) — DR strategies, RTO/RPO, multi-region patterns
- [Elastic-Disaster-Recovery-General.md](./Elastic-Disaster-Recovery-General.md) — block replication, drills, recovery launch, failback
- [ServiceQuotas-General.md](./ServiceQuotas-General.md) — quota management, monitoring, increase requests

---

## Key Architecture Patterns

### Observability Stack
```
Application → CloudWatch Metrics + Logs
           → X-Ray Traces
           → CloudTrail API Audit
           → CloudWatch Dashboards + Alarms → SNS → PagerDuty/Slack
```

### Cost Optimization Loop
```
Cost Explorer (analyze) → Compute Optimizer (rightsize)
→ Trusted Advisor (check) → Savings Plans (commit)
→ S3 Lifecycle (tier) → Reserved Capacity (plan)
```

### Reliability Improvement
```
Well-Architected Review → identify gaps
→ AWS Backup (protect data)
→ Multi-AZ / Multi-Region (reduce selected failure risks)
→ Auto Scaling (handle load)
→ Route 53 health checks (detect failures)
```

---

## Exam Tips

- **CloudWatch** is the answer for metrics, alarms, and log aggregation — know Logs Insights queries
- **X-Ray** is the answer for distributed tracing and latency analysis across microservices
- **Systems Manager** replaces SSH/RDP — Session Manager, Run Command, Patch Manager
- **Compute Optimizer** = ML-based rightsizing; **Trusted Advisor** = broad best practice checks
- **AWS Backup** = centralized, policy-driven backup across services and accounts
- **Well-Architected Tool** = structured review against the 6 pillars — know the pillars cold
- DR strategies: Backup/Restore → Pilot Light → Warm Standby → Multi-Site Active-Active (increasing cost and decreasing RTO)
- **Service Quotas** + CloudWatch alarms = proactive quota monitoring before hitting limits
