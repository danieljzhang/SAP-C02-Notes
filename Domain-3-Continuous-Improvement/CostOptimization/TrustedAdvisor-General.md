# AWS Trusted Advisor — SAP-C02 Study Notes

> Exam relevance: Domains 1 and 3. Trusted Advisor provides checks across six categories. These are not the six Well-Architected pillars. Check availability depends on the current AWS Support plan and is changing; verify the current plan and check reference before relying on a specific check.

---

## 1. Overview

AWS Trusted Advisor inspects your AWS environment and provides recommendations based on AWS best practices. Check refresh behavior and availability depend on the plan and check; do not assume every result is real-time.

---

## 2. Check Categories

| Category | Examples |
|---|---|
| Cost Optimization | Idle EC2 instances, underutilized EBS volumes, unassociated EIPs, low-utilization RDS |
| Performance | High-utilization EC2, CloudFront content delivery optimization, EBS throughput |
| Security | S3 bucket permissions, security groups with unrestricted access, MFA on root, IAM use |
| Fault Tolerance | EBS snapshots, RDS Multi-AZ, Auto Scaling, Route 53 health checks |
| Service Limits | Usage approaching service quotas (EC2 instances, VPCs, EIPs, etc.) |
| Operational Excellence | Operational best-practice checks; consult the current check reference |

---

## 3. Support Plan Requirements

As of October 2026, AWS documentation lists AWS Business Support+, AWS Enterprise Support, and AWS Unified Operations for access to all Trusted Advisor checks and the API. Basic Support has access to the Service Limits category and selected Security/Fault Tolerance checks; some checks require manual refresh. Confirm current plan entitlements before designing automation.

AWS has also announced that Developer Support, Business Support, and Enterprise On-Ramp will be discontinued in most Regions on January 1, 2027; existing customers are directed to current replacement plans. Service availability and entitlements can vary by Region. Check the current AWS Support plan notices.

Examples of selected checks available to Basic/Developer Support customers in the current guide include:
- S3 bucket permissions
- Security groups with specific ports unrestricted
- MFA on root account
- EBS public snapshots
- RDS public snapshots
- STS global endpoint usage across Regions

The exact list and refresh behavior are plan-dependent and may change; do not describe these as a fixed “seven core checks always free” bundle.

---

## 4. Organizational View

- Centralize supported check results across AWS Organizations accounts
- Availability and setup requirements depend on the current Support plan and organization configuration; verify before promising a particular tier
- View Trusted Advisor checks across all accounts in the organization
- Aggregate findings in a central dashboard
- Export reports for all accounts

---

## 5. Trusted Advisor vs Other Services

| Service | Focus |
|---|---|
| Trusted Advisor | Broad checks across cost optimization, performance, security, fault tolerance, service limits, and operational excellence |
| Compute Optimizer | ML-based rightsizing for compute resources |
| Cost Explorer | Cost analysis and forecasting |
| Security Hub | Security findings aggregation and compliance |
| Config | Configuration compliance against rules |
| Well-Architected Tool | Structured architecture review against 6 pillars |

---

## 6. Exam Scenarios

**Scenario 1:** A company wants to identify EC2 instances that have been running but have very low CPU utilization (potential waste). What is the quickest solution?

> Check the relevant **Trusted Advisor Cost Optimization** check if it is available under the account's current plan. Use the current check reference for its utilization thresholds and evaluation window, then validate seasonal demand and application SLOs before rightsizing.

**Scenario 2:** A company wants to ensure no security groups allow unrestricted inbound access on sensitive ports (SSH, RDP). What is the solution?

> Use a suitable **Trusted Advisor Security** check where the current plan exposes it, and/or continuously evaluate security groups with Config/security tooling. Confirm the check's exact ports, IPv4/IPv6 coverage, refresh behavior, and plan entitlement; do not assume it detects every unrestricted rule.

**Scenario 3:** A company is approaching EC2 instance limits in a region and wants proactive notification. What is the solution?

> Use **Service Quotas** to monitor supported quotas and configure proactive notifications/alarms where available; request increases ahead of demand. Trusted Advisor can provide complementary checks, but verify current metric and alert integration rather than assuming every quota is surfaced the same way.

---

## 7. Exam Tips

- Trusted Advisor has six check categories, including **Operational Excellence**; do not confuse them with Well-Architected pillars
- Full check/API access and selected Basic-plan checks depend on current Support plan entitlements
- Support-plan names and availability are changing through January 2027; use current AWS documentation
- Trusted Advisor is broad; use service-specific tools when exact configuration, compliance, or quota monitoring is required
- For **ML-based rightsizing** → Compute Optimizer; for **broad best practices** → Trusted Advisor
- **Organizational view** requires Enterprise support — for multi-account consolidated view
- Trusted Advisor checks run **weekly** by default — not real-time

---

## 8. Official References

- [Trusted Advisor User Guide](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html)
- [Trusted Advisor Check Reference](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor-check-reference.html)
- [Organizational View](https://docs.aws.amazon.com/awssupport/latest/user/organizational-view.html)
- [AWS Support plans](https://aws.amazon.com/premiumsupport/plans/)
