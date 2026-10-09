# AWS Trusted Advisor — SAP-C02 Study Notes

> Exam relevance: Domain 3 (25%). Trusted Advisor provides best practice checks across 5 pillars. Know check categories, support plan requirements, and organizational view.

---

## 1. Overview

AWS Trusted Advisor inspects your AWS environment and provides real-time recommendations across five categories based on AWS best practices.

---

## 2. Check Categories

| Category | Examples |
|---|---|
| Cost Optimization | Idle EC2 instances, underutilized EBS volumes, unassociated EIPs, low-utilization RDS |
| Performance | High-utilization EC2, CloudFront content delivery optimization, EBS throughput |
| Security | S3 bucket permissions, security groups with unrestricted access, MFA on root, IAM use |
| Fault Tolerance | EBS snapshots, RDS Multi-AZ, Auto Scaling, Route 53 health checks |
| Service Limits | Usage approaching service quotas (EC2 instances, VPCs, EIPs, etc.) |

---

## 3. Support Plan Requirements

| Check Type | Required Plan |
|---|---|
| Core checks (7 checks) | All plans (including Basic/Free) |
| Full checks (hundreds) | Business, Enterprise On-Ramp, or Enterprise |
| Programmatic access (API) | Business, Enterprise On-Ramp, or Enterprise |
| Organizational view | Enterprise On-Ramp or Enterprise |

**Core checks (always free):**
- S3 bucket permissions
- Security groups (unrestricted ports)
- IAM use
- MFA on root account
- EBS public snapshots
- RDS public snapshots
- Service limits

---

## 4. Organizational View

- Available with Enterprise support plans
- View Trusted Advisor checks across all accounts in the organization
- Aggregate findings in a central dashboard
- Export reports for all accounts

---

## 5. Trusted Advisor vs Other Services

| Service | Focus |
|---|---|
| Trusted Advisor | Broad best practice checks across 5 pillars |
| Compute Optimizer | ML-based rightsizing for compute resources |
| Cost Explorer | Cost analysis and forecasting |
| Security Hub | Security findings aggregation and compliance |
| Config | Configuration compliance against rules |
| Well-Architected Tool | Structured architecture review against 6 pillars |

---

## 6. Exam Scenarios

**Scenario 1:** A company wants to identify EC2 instances that have been running but have very low CPU utilization (potential waste). What is the quickest solution?

> Check **Trusted Advisor** under the **Cost Optimization** category. The "Low Utilization Amazon EC2 Instances" check identifies instances with < 10% CPU utilization over 14 days. Requires Business or Enterprise support plan for full access.

**Scenario 2:** A company wants to ensure no security groups allow unrestricted inbound access on sensitive ports (SSH, RDP). What is the solution?

> Check **Trusted Advisor** under the **Security** category. The "Security Groups - Unrestricted Access" check is available on all support plans (core check). It flags security groups with 0.0.0.0/0 or ::/0 on any port.

**Scenario 3:** A company is approaching EC2 instance limits in a region and wants proactive notification. What is the solution?

> Use **Trusted Advisor** Service Limits checks combined with **CloudWatch alarms**. Trusted Advisor exposes service limit utilization as CloudWatch metrics. Create alarms when utilization exceeds 80% of the limit. Alternatively, use **Service Quotas** with CloudWatch alarms for more granular monitoring.

---

## 7. Exam Tips

- **Business/Enterprise support** required for full Trusted Advisor access and API
- **7 core checks** are free for all accounts — know which ones
- Trusted Advisor is **broad** — covers cost, performance, security, fault tolerance, limits
- For **ML-based rightsizing** → Compute Optimizer; for **broad best practices** → Trusted Advisor
- **Organizational view** requires Enterprise support — for multi-account consolidated view
- Trusted Advisor checks run **weekly** by default — not real-time

---

## 8. Official References

- [Trusted Advisor User Guide](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html)
- [Trusted Advisor Check Reference](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor-check-reference.html)
- [Organizational View](https://docs.aws.amazon.com/awssupport/latest/user/organizational-view.html)
