# AWS Cost Explorer — SAP-C02 Study Notes

> Exam relevance: Domain 3 (25%). Cost Explorer is the primary tool for analyzing and forecasting AWS costs. Know filtering, grouping, rightsizing recommendations, and Savings Plans analysis.

---

## 1. Overview

AWS Cost Explorer provides interactive visualizations and analysis of AWS costs and usage. It helps identify cost drivers, forecast future spend, and find optimization opportunities.

---

## 2. Key Features

### Cost and Usage Analysis

- Visualize costs by: service, account, region, tag, instance type, purchase option
- Filter by: time range (up to 13 months historical), service, linked account, tag
- Group by multiple dimensions simultaneously
- Daily, monthly, or hourly granularity (hourly requires additional cost)

### Forecasting

- ML-based cost forecast for up to 12 months
- Based on historical usage patterns
- Confidence intervals included
- Use for: budget planning, capacity planning

### Rightsizing Recommendations

- Identifies over-provisioned EC2 instances
- Recommends instance type changes based on CloudWatch utilization metrics
- Shows estimated monthly savings
- Requires **Cost Explorer rightsizing recommendations** enabled (free)
- Considers: CPU, memory (requires CloudWatch Agent), network utilization

### Savings Plans Recommendations

- Recommends Compute, EC2 Instance, or SageMaker Savings Plans
- Based on historical On-Demand usage
- Shows: recommended commitment, estimated savings, coverage
- Lookback period: 7, 30, or 60 days

---

## 3. Cost Allocation Tags

- Tag resources to track costs by project, team, environment, cost center
- **AWS-generated tags:** `aws:createdBy` (who created the resource)
- **User-defined tags:** custom key-value pairs
- Activate tags in Cost Explorer for cost allocation
- Tags appear in Cost Explorer after 24 hours

---

## 4. Budgets Integration

- Create budgets based on Cost Explorer data
- Alert when actual or forecasted costs exceed thresholds
- Budget types: cost, usage, Savings Plans coverage, reservation coverage
- Actions: notify via SNS, apply IAM policy, apply SCP (to restrict spending)

---

## 5. Exam Scenarios

**Scenario 1:** A company wants to identify which AWS accounts and services are driving the most cost increase over the last 3 months. What is the solution?

> Use **Cost Explorer**. Filter by the last 3 months. Group by **Linked Account** and **Service**. Sort by cost to identify the top cost drivers. Drill down into specific accounts or services for further analysis.

**Scenario 2:** A company wants to reduce EC2 costs. They suspect many instances are over-provisioned. What is the solution?

> Enable **Cost Explorer Rightsizing Recommendations**. Install the CloudWatch Agent on instances to collect memory metrics. Cost Explorer analyzes CPU, memory, and network utilization and recommends smaller instance types with estimated savings.

**Scenario 3:** A company has steady EC2 usage and wants to commit to Savings Plans but doesn't know the right commitment amount. What is the solution?

> Use **Cost Explorer Savings Plans Recommendations**. Set the lookback period to 30 days. Cost Explorer recommends the optimal commitment amount based on historical On-Demand usage, showing estimated savings and payback period.

---

## 6. Exam Tips

- Cost Explorer shows **13 months** of historical data
- **Rightsizing recommendations** require CloudWatch metrics — memory needs CloudWatch Agent
- **Cost allocation tags** must be **activated** in Cost Explorer before they appear in reports
- Cost Explorer is **free** to use; hourly granularity has an additional cost
- **Savings Plans recommendations** are based on historical usage — use 30-day lookback for stable workloads
- For **real-time** cost alerts, use **AWS Budgets** (Cost Explorer is for analysis, not alerting)

---

---

## 🎯 Architecture Decision Notes

### Cost optimization starts at architecture time

Treat cost as a design constraint, not a spreadsheet exercise at the end:

- Match **commitment instruments to stable baselines**: Savings Plans and
  Reserved Instances discount predictable usage; they do not make spiky
  workloads cheap — right-sizing and serverless do.
- **Savings Plans vs RIs:** Plans flex across instance families, regions,
  and services (Compute plans cover EC2, Lambda, Fargate); RIs lock to
  configuration. Flexibility has a price — model both.
- Tag-driven allocation only works with a **governed tag vocabulary**
  enforced by policy; untagged resources make every cost report fiction.
- The cheapest architecture that misses RTO/RPO is not cheap — it is just
  unpriced risk.

## 7. Official References

- [Cost Explorer User Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [Rightsizing Recommendations](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-rightsizing.html)
- [Savings Plans Recommendations](https://docs.aws.amazon.com/savingsplans/latest/userguide/purchase-rec.html)
