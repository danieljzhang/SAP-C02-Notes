# Operations, Resilience, and Cost

## Observe the right signal

Amazon CloudWatch brings together metrics, logs, alarms, dashboards, and supported telemetry features. Use the signal that answers the operational question:

- **Metrics and alarms** detect changes in measured behavior. Metric math can compare observed usage with a quota, but confirm the metric and quota use compatible units and scope.
- **CloudWatch Synthetics** runs canaries that exercise a user journey or endpoint. A synthetic check can catch a broken experience even when infrastructure metrics look normal.
- **Container Insights** provides container-related operational metrics at supported cluster, service, task, and container levels. It complements, rather than replaces, application logs and traces.
- **VPC Flow Logs** record metadata about IP traffic at configured network interfaces, subnets, or VPCs. They do not capture packet contents.
- **AWS CloudTrail** records supported AWS API activity for audit and investigation. **AWS Config** records resource configuration and evaluates compliance. Use both when you need to know both who called an API and what configuration a resource had.

For a quota dashboard, show actual utilization, the applicable quota, and a threshold that leaves time to respond. A percentage expression is meaningful only when the selected metric and quota represent the same resource scope and unit.

For quotas that publish a compatible CloudWatch usage metric, add the metric as an ID such as `m1` and calculate `m1 / SERVICE_QUOTA(m1) * 100`. AWS usage metrics use the `AWS/Usage` namespace. An alarm can notify an SNS topic when utilization crosses an operational threshold, such as 75%; the threshold is a response-time choice, not an AWS default. Not every quota has a matching usage metric, so verify the service, metric dimensions, quota integration, and units before creating the alarm.

## Recovery objectives and architecture

**Recovery time objective (RTO)** is the target time to restore service. **Recovery point objective (RPO)** is the target maximum age of recoverable data, expressed as acceptable data loss. Set both from business impact, then test that the design can meet them.

Common patterns trade cost and readiness:

- **Backup and restore** keeps recovery copies but reconstructs the environment after an event. It is usually less expensive to keep idle, but recovery can take longer.
- **Pilot light** keeps core data or foundational components ready while other capacity is brought online during recovery.
- **Warm standby** runs a reduced but functional copy that can be scaled or promoted.
- **Multi-site active/active** serves traffic from multiple locations and may reduce recovery time, but increases cost and consistency/operations complexity.

These are patterns, not guaranteed RTO/RPO numbers. Recovery time depends on data volume, service limits, automation, dependencies, DNS/client behavior, and human steps. Replication is not the same as backup: unwanted changes or corruption may replicate. Use isolated recovery points where needed and rehearse failover and failback.

**Fresh example:** A reporting portal can tolerate several hours of rebuild, while its transaction ledger cannot lose more than a few minutes. Give the two tiers separate recovery objectives and designs instead of paying to run every component active/active.

S3 replication, database replicas, snapshots, AWS Backup, and log shipping address different recovery needs. Document retention, cross-account/Region copies, encryption-key access, restore permissions, and validation of recovered data. Measure a real restore before claiming an objective is met.

## Cost controls and organizational visibility

Cost optimization is an architecture constraint, not a final spreadsheet exercise:

- Match compute purchasing commitments to stable usage. Savings Plans and Reserved Instances differ by service coverage, flexibility, and eligible usage. RDS Reserved DB Instances are distinct from compute Savings Plans.
- Evaluate DynamoDB reserved capacity only for eligible, sustained provisioned usage; it does not raise table throughput or remove throttling.
- Use lifecycle policies and storage classes to match retention and retrieval behavior. Include retrieval fees, minimum storage durations, monitoring, replication, and request costs.
- Autoscaling can reduce idle capacity but should preserve enough headroom for bursts and recovery. Spot pricing requires interruption handling.
- AWS Budgets can notify on actual or forecasted costs and can run explicitly configured actions, such as applying an IAM policy or SCP or targeting selected EC2/RDS instances. Budget data is delayed and can update only a few times per day, so these actions are not an immediate, universal spending cap. Cost allocation tags require consistent tagging and activation for cost reporting.
- In consolidated billing, understand which account owns the resource, which account is charged, and how eligible commitment sharing is configured. Cost reports and permissions available to a member account vary by billing setup and organization policy.

**Fresh example:** A media team has steady baseline compute and unpredictable processing spikes. It models a commitment for the baseline, uses interruptible capacity for retryable processing, and measures storage retrieval costs before moving archives to a colder tier.

## A repeatable architecture review

For any proposed service choice, write down:

1. The workload's required behavior and hard constraints.
2. The data path, identity path, and network path.
3. Failure modes, recovery targets, and a test for each.
4. The operational work the team is accepting.
5. The cost drivers and the conditions that would change the choice.

For certification study, practice applying that checklist to newly invented workloads. Explain why an option fits the constraints and which requirement would make it a poor choice. Check the current AWS exam guide; service capabilities and exam coverage evolve.

## References

- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudWatch usage metrics and service quotas](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Service-Quota-Integration.html)
- [CloudWatch Synthetics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics.html)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
- [AWS disaster recovery planning](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)
- [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [AWS Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS Budgets actions](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-controls.html)
