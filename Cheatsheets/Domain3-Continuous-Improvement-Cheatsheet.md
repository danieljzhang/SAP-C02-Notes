# Domain 3 Cheatsheet: Continuous Improvement

> Deeper notes: [Domain 3 overview](../Domain-3-Continuous-Improvement/README.md) and the linked service guides.

> Focus: improve an existing workload using evidence. Measure first, identify the bottleneck or risk, apply the smallest effective change, and verify the outcome.

## Observability and Operations

| Question | Service / pattern |
|---|---|
| What are metrics, logs, alarms, dashboards? | CloudWatch |
| Who made an AWS API call or changed a resource? | CloudTrail |
| Where is latency/failure occurring across service calls? | X-Ray / distributed tracing |
| How do I automate fleet operations and patching? | Systems Manager |
| How do I assess architecture against best practices? | Well-Architected Tool / Framework |
| How do I monitor/request service quotas? | Service Quotas + CloudWatch alarms where supported |

- Use correlation IDs and traces across service boundaries; metrics alone rarely pinpoint distributed latency.
- Separate audit logs (CloudTrail) from application logs and performance telemetry.
- Automate repeatable operational actions with Systems Manager, controlled roles, approvals, and audit trails.

## Reliability and Recovery

| Requirement | Pattern |
|---|---|
| Recover after AZ failure | Multi-AZ architecture, health checks, automated failover |
| Recover after Region loss with lowest cost | Backup and restore |
| Faster recovery with some standby footprint | Pilot light / warm standby |
| Near-continuous service across Regions | Multi-site active-active, with explicit data consistency design |
| Centralized policy-driven backups | AWS Backup; cross-account/cross-Region copy where supported |

- RTO = acceptable recovery time; RPO = acceptable data loss window.
- Replication is not backup; protect against logical corruption/deletion and test restore.
- Distinguish high availability within one Region from disaster recovery across Regions.
- Test runbooks, failover, failback, DNS/traffic behavior, and data promotion—not only backup completion.

## Cost and Performance

| Tool / practice | Use |
|---|---|
| Cost Explorer | Historical cost, trends, forecasts, filters, rightsizing views |
| Compute Optimizer | Utilization-based rightsizing recommendations for supported resources |
| Trusted Advisor | Broad best-practice checks; available checks/features depend on support plan/current offering |
| Savings Plans / Reserved options | Commit to steady eligible usage after analyzing duration and flexibility |
| S3 lifecycle/storage analysis | Tier data based on access and retrieval economics |
| Load/performance testing | Confirm bottleneck before scaling or caching |

Optimization loop: baseline -> identify idle/oversized or bottleneck -> choose low-risk change -> measure cost/performance/reliability -> document rollback.

## Well-Architected Pillars

1. Operational excellence
2. Security
3. Reliability
4. Performance efficiency
5. Cost optimization
6. Sustainability

Treat trade-offs as explicit. Improving one pillar can affect another (for example, multi-Region increases resilience but also cost and operational complexity).

## Common Traps

- CloudTrail is audit history, not a distributed trace.
- CloudWatch alarms do not automatically fix issues without an action/automation design.
- A low average CPU metric does not prove a system is oversized; inspect peak, latency, memory, queueing, and business demand.
- Savings commitments are risky before usage and growth are understood.
- A backup that has never been restored is unproven.
- Quotas can be regional and resource-specific; request increases before a migration or launch window.

## 30-Second Review

1. Name the objective: reliability, performance, cost, security, or operations.
2. Identify the evidence source and baseline.
3. Choose a service/pattern that addresses the measured issue.
4. Check blast radius, rollback, and secondary effects.
5. Define success metrics and verify after the change.

## Official References

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
