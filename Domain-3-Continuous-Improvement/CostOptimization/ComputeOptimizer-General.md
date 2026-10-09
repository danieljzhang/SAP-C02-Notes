> Exam relevance: Domain 3 (25%). Compute Optimizer uses ML to recommend optimal resource configurations. Know supported resources, recommendation types, and how it differs from Cost Explorer rightsizing.

---

## 1. Overview

AWS Compute Optimizer analyzes historical utilization metrics using machine learning and recommends optimal AWS resource configurations to reduce cost and improve performance.

**Compute Optimizer vs Cost Explorer Rightsizing:**

| Feature | Compute Optimizer | Cost Explorer Rightsizing |
|---|---|---|
| ML-based | Yes | No (threshold-based) |
| Supported resources | EC2, ASG, Lambda, ECS Fargate, EBS, RDS | EC2 only |
| Performance risk | Shows risk of under-provisioning | Not shown |
| Enhanced metrics | Yes (CloudWatch Agent memory) | Yes (CloudWatch Agent) |
| Cost | Free (standard), paid (enhanced) | Free |

---

## 2. Supported Resources

| Resource | What It Optimizes |
|---|---|
| EC2 instances | Instance type, size |
| EC2 Auto Scaling Groups | Instance type, size |
| EBS volumes | Volume type, size, IOPS |
| Lambda functions | Memory size |
| ECS services on Fargate | CPU and memory |
| RDS DB instances | Instance class (preview) |

---

## 3. Recommendation Types

Each recommendation includes:
- **Current configuration** and its cost
- **Recommended configuration** and estimated cost
- **Performance risk:** risk of the recommendation being under-provisioned
- **Savings opportunity:** estimated monthly savings
- **Finding:** Over-provisioned, Under-provisioned, Optimized, Not enough data

---

## 4. Enhanced Infrastructure Metrics

- Paid feature: uses 3 months of metrics instead of 14 days
- More accurate recommendations for workloads with seasonal patterns
- Requires opt-in per account or organization-wide

---

## 5. Organization-Wide Recommendations

- Enable Compute Optimizer at the organization level from the management account
- View recommendations across all member accounts in one place
- Aggregate savings opportunities across the organization

---

## 6. Exam Scenarios

**Scenario 1:** A company wants ML-based recommendations to rightsize EC2 instances, Lambda functions, and EBS volumes across all accounts. What is the solution?

> Enable **AWS Compute Optimizer** at the organization level. It analyzes utilization metrics for EC2, Lambda, EBS, ECS Fargate, and ASGs using ML. View consolidated recommendations across all accounts from the management account.

**Scenario 2:** Compute Optimizer recommends downsizing an EC2 instance but the team is concerned about performance impact. What information does Compute Optimizer provide?

> Compute Optimizer includes a **performance risk** indicator for each recommendation — Low, Medium, or High. High risk means the recommended instance may be under-provisioned for peak workloads. The team can choose a recommendation with lower performance risk at a slightly higher cost.

---

## 7. Exam Tips

- Compute Optimizer = **ML-based**, covers **EC2, Lambda, EBS, ECS Fargate, ASG**
- Cost Explorer rightsizing = **EC2 only**, threshold-based
- **Performance risk** is unique to Compute Optimizer — shows risk of under-provisioning
- Enable at **organization level** for consolidated recommendations
- Requires **opt-in** — not enabled by default
- **Enhanced metrics** (paid) = 3 months of data for seasonal workloads

---

## 8. Official References

- [Compute Optimizer User Guide](https://docs.aws.amazon.com/compute-optimizer/latest/ug/)
- [Supported Resources](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)
- [Enhanced Infrastructure Metrics](https://docs.aws.amazon.com/compute-optimizer/latest/ug/enhanced-infrastructure-metrics.html)
