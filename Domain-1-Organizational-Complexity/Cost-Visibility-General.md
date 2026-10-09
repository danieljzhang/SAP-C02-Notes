# Cost Visibility and Optimization — SAP-C02 Study Notes

> Official Domain 1 task 1.5 covers cost optimization and visibility. The objective is not merely to choose a cheaper instance: establish who owns spend, how usage is measured, and how the architecture can change safely.

## 1. Cost Management Tool Selection

| Need | Tool | Decision cue |
|---|---|---|
| Explore historical spend, trends, forecasts, and filters | AWS Cost Explorer | Analysis and investigation; not a real-time billing ledger |
| Notify when actual or forecast spend crosses a threshold | AWS Budgets | Budget alerts, actions, and anomaly notifications; configure lead time and recipients |
| Detailed line-item billing data for custom queries | AWS Cost and Usage Report (CUR / Data Exports) | Deliver billing data to S3, then query with Athena or visualize with QuickSight |
| Estimate a proposed architecture before deployment | AWS Pricing Calculator | Estimate only; assumptions, regional prices, discounts, and usage drive accuracy |
| Detect unusual spend patterns | Cost Anomaly Detection | Identify anomalous spend and route alerts; it does not replace budgets or cost allocation |
| Attribute spend to teams/products | Cost allocation tags, Cost Categories, Organizations billing views | Define owners, tag policies, activation, and a remediation process for untagged resources |
| Find rightsizing opportunities | Compute Optimizer and Trusted Advisor | Recommendations require workload validation and have plan/coverage constraints |

## 2. A Practical Visibility Loop

```text
Define account/OU ownership and cost-allocation model
                  |
Apply tags and Cost Categories; activate cost-allocation tags
                  |
Use Cost Explorer for trends + CUR/Data Exports for detail
                  |
Set Budgets and anomaly alerts with named owners
                  |
Investigate drivers -> validate with workload telemetry
                  |
Change architecture/commitments -> measure actual outcome
```

- Use AWS Organizations consolidated billing and account separation to align spend with teams, products, and environments.
- Standardize tag keys and allowed values. Tags are not retroactive for every report and do not automatically allocate all shared costs.
- Allocate shared networking, support, security, and platform costs with an explicit model; do not leave them invisible.
- Give teams dashboards and alerts, but central FinOps should own shared policies and commitment decisions.

## 3. Purchasing and Architecture Decisions

- **On-Demand:** flexible, no usage commitment; useful for uncertain or short-lived demand.
- **Savings Plans / Reserved Instances:** commitment discounts for eligible usage; analyze the stable baseline, flexibility, term, payment option, and utilization before purchase.
- **Spot:** discounted spare capacity with interruption risk; use for fault-tolerant, restartable, or checkpointed work.
- **Storage tiers:** compare capacity price with retrieval, request, transition, minimum-duration, and restore-time costs.
- **Data transfer:** model inter-AZ, inter-Region, internet egress, NAT, Direct Connect, and replication traffic; high-availability designs may increase data movement.
- **Managed versus self-managed:** include operational labor, patching, backup, resilience, and idle capacity, not just hourly resource price.

## 4. Governance and Security

- Separate payer/management responsibilities from workload administration; restrict access to billing data using least privilege.
- Use tag policies and account vending controls to improve allocation at creation time.
- Protect CUR/Data Export buckets with scoped IAM and KMS policies; billing data reveals resource and usage metadata.
- Use budgets and anomaly alerts as detection/notification controls, not as a guaranteed hard spending cap. Budget actions can restrict selected resources only after configuration and should be tested carefully.

## 5. Original Scenario

**Scenario:** A company sees a 30% increase in cloud spend, but its monthly invoice only identifies account totals. Product owners need cost responsibility, and leadership wants early warning before budget overruns.

**Recommended approach:** Standardize and activate allocation tags, define Cost Categories for shared and product costs, use Cost Explorer to identify trends, and deliver CUR/Data Exports to S3 for detailed analysis with Athena. Configure AWS Budgets for actual/forecast thresholds and Cost Anomaly Detection for unusual patterns. Assign owners to investigate alerts before buying commitments.

**Trap:** A budget threshold is not a universal spend cap, and an estimate from the Pricing Calculator is not an invoice forecast without realistic assumptions.

## 6. Exam Tips

- Cost Explorer = analyze; Budgets = threshold notifications/actions; CUR/Data Exports = detailed billing records; Pricing Calculator = estimate before deployment.
- First improve allocation and visibility; then commit to discounts based on measured, stable usage.
- Savings are not successful if they violate resilience, performance, or recovery requirements.
- Verify current billing product names, report schemas, tag activation behavior, and service availability in AWS documentation.

## 7. Official References

- [AWS Cost Management User Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-costmanagement.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS Cost and Usage Reports](https://docs.aws.amazon.com/cur/latest/userguide/what-is-cur.html)
- [AWS Pricing Calculator](https://calculator.aws/)
- [Cost allocation tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)
- [SAP-C02 Domain 1 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain1.html)
