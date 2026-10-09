# AWS Certified Solutions Architect – Professional (SAP-C02) Study Notes

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Exam: SAP-C02](https://img.shields.io/badge/Exam-SAP--C02-orange.svg)](https://aws.amazon.com/certification/certified-solutions-architect-professional/)

## 🎯 Exam-Focused Study Notes

> Study materials for AWS Certified Solutions Architect – Professional (SAP-C02), with service notes, architecture patterns, and original scenarios across all four exam domains.

This is a free study resource organized around SAP-C02 exam objectives, with service guides, architecture patterns, and original scenario exercises. It is not a full mock exam or paid question bank, and it contains no actual exam questions. Some services in AWS's non-exhaustive in-scope list receive selector or awareness coverage rather than a standalone guide; see the [coverage map](./Cheatsheets/In-Scope-Service-Coverage.md) and consult current AWS documentation for feature-specific decisions.

> **Exam-version notice (checked October 8, 2026):** [AWS states](https://aws.amazon.com/certification/certified-solutions-architect-professional/) that SAP-C02's last exam date is November 16, 2026, and registration for SAP-C03 opens October 27, 2026. This repository is mapped to SAP-C02. Confirm the version you are booking and use the corresponding official exam guide; SAP-C03 objectives may differ.

---

## 📊 Exam Information

| Item | Detail |
|---|---|
| Exam code | SAP-C02 |
| Questions | 75 (65 scored + 10 unscored) |
| Duration | 180 minutes |
| Passing score | 750 / 1000 (scaled score) |
| Cost | $300 USD |
| Validity | 3 years |

### Domain Weights

| Domain | Name | Weight | Est. Questions |
|---|---|---|---|
| 1 | Design Solutions for Organizational Complexity | 26% | ~17 |
| 2 | Design for New Solutions | 29% | ~19 |
| 3 | Continuous Improvement for Existing Solutions | 25% | ~16 |
| 4 | Accelerate Workload Migration and Modernization | 20% | ~13 |

> Verify domain names and weights against the current official exam guide at:
> https://aws.amazon.com/certification/certified-solutions-architect-professional/
> The current SAP-C02 guide also identifies emerging responsible-AI/security topics as possible unscored pretest content. Review the live guide for updates.

---

## 📚 Repository Structure

### [Domain 1: Design Solutions for Organizational Complexity (26%)](./Domain-1-Organizational-Complexity/)

- **Governance:** AWS Organizations, SCPs, Control Tower, Landing Zones, RAM
- **Identity:** IAM, IAM Identity Center, Resource Control Policies
- **Networking:** Transit Gateway, Direct Connect, VPN, VPC Advanced, Route 53
- **Security:** Config, Security Hub, GuardDuty, Inspector, Macie, WAF/Shield, Firewall Manager
- **Cost visibility:** Cost Explorer, Budgets, Cost and Usage Reports, allocation tags
- **Coverage:** Multi-account design, cross-account access, enterprise networking, governance guardrails

### [Domain 2: Design for New Solutions (29%)](./Domain-2-New-Solutions/)

- **Compute:** EC2 Advanced, ECS, EKS, Lambda, Auto Scaling
- **Networking:** CloudFront, API Gateway, ELB, Global Accelerator
- **Storage:** S3 Advanced, EFS, FSx
- **Databases:** RDS/Aurora, DynamoDB, ElastiCache, OpenSearch
- **Serverless & Messaging:** Step Functions, EventBridge, SQS/SNS, Kinesis, MSK
- **IaC and deployment:** CloudFormation, CDK, CodePipeline, CodeBuild, CodeDeploy
- **Coverage:** Architecture design, scalability, resilience, security, cost optimization

### [Domain 3: Continuous Improvement for Existing Solutions (25%)](./Domain-3-Continuous-Improvement/)

- **Monitoring:** CloudWatch, CloudTrail, X-Ray, Systems Manager
- **Cost Optimization:** Cost Explorer, Compute Optimizer, Trusted Advisor
- **Reliability:** Well-Architected Tool, AWS Backup, Elastic Disaster Recovery, Resilience Patterns, Service Quotas
- **Coverage:** Operational excellence, performance tuning, cost reduction, reliability improvement

### [Domain 4: Accelerate Workload Migration and Modernization (20%)](./Domain-4-Migration-Modernization/)

- **Migration Tools:** MGN, DMS, DataSync, Snow Family, Transfer Family, Migration Hub, MAP
- **Modernization Patterns:** Microservices, containers, serverless, database modernization
- **Coverage:** Migration strategies (7 Rs), lift-and-shift, re-platforming, re-architecting
- **Domain guide:** [Domain 4 overview and study notes](./Domain-4-Migration-Modernization/README.md)

### [Cheatsheets](./Cheatsheets/) — Last-Minute Review

- Domain-specific cheatsheets for rapid review
- All-services quick reference
- Exam strategy and time management
- Final exam tips
- Original practice scenarios
- [Cheatsheets index](./Cheatsheets/README.md)
- [In-scope service coverage map](./Cheatsheets/In-Scope-Service-Coverage.md)
- [Readiness tracker](./Cheatsheets/Readiness-Tracker.md)

---

## 🚀 Quick Start Guide

### Comprehensive Study (6–8 weeks)

1. **Week 1–2:** Domain 2 (New Solutions) — highest weight, broadest service coverage
2. **Week 3:** Domain 1 (Organizational Complexity) — multi-account, networking, governance
3. **Week 4–5:** Domain 3 (Continuous Improvement) — monitoring, cost, reliability
4. **Week 6:** Domain 4 (Migration) — migration strategies and modernization patterns
5. **Week 7–8:** Cheatsheets, practice scenarios, mock exams, weak-area review

### Review / Refresh (2 weeks)

1. **Days 1–7:** All domain cheatsheets → identify weak areas
2. **Days 8–11:** Deep dive into weak areas using detailed service files
3. **Days 12–14:** All-Services-Review + Final Exam Tips + Practice Scenarios

### Last-Night Preparation (2–3 hours)

1. [Final Exam Tips](./Cheatsheets/Final-Exam-Tips.md) — 30 min
2. [Exam Strategy](./Cheatsheets/Exam-Strategy.md) — 30 min
3. [All Services Review](./Cheatsheets/All-Services-Review.md) — 60 min
4. Weak-area domain cheatsheets — 30 min

---

## 🔥 High-Impact Study Areas

### Must-Know Services (Appear Across Multiple Domains)

| Service | Domains | Why It Matters |
|---|---|---|
| AWS Organizations + SCPs | 1, 3, 4 | Foundation of multi-account governance |
| IAM + IAM Identity Center | 1, 2, 3 | Cross-account access, federation, least privilege |
| Transit Gateway | 1, 2 | Hub-and-spoke networking at scale |
| CloudFormation | 2, 3, 4 | IaC for repeatable, auditable infrastructure |
| CloudWatch | 2, 3, 4 | Monitoring foundation across all domains |
| Lambda | 2, 3, 4 | Serverless compute, event-driven patterns |
| S3 | 2, 3, 4 | Storage foundation, replication, lifecycle |
| RDS / Aurora | 2, 3, 4 | Relational database HA, DR, migration |
| Route 53 | 1, 2, 3 | DNS, routing policies, health checks, failover |
| Direct Connect | 1, 2, 4 | Hybrid connectivity, bandwidth, latency |

### Critical Architecture Patterns

- **Multi-account landing zone** — Organizations + Control Tower + SCPs + IAM Identity Center
- **Hub-and-spoke networking** — Transit Gateway + Direct Connect + VPN
- **Multi-Region traffic and data** — Route 53 / Global Accelerator for traffic; DynamoDB Global Tables can support multi-Region writes, while Aurora Global Database has a primary writer and read-only secondary Regions (optional write forwarding sends writes to the primary)
- **Event-driven architecture** — EventBridge + Lambda + SQS/SNS + Step Functions
- **Data lake architecture** — S3 + Glue + Athena + Lake Formation
- **Lift-and-shift migration** — MGN + DMS + DataSync + Migration Hub
- **Serverless modernization** — Lambda + API Gateway + DynamoDB + EventBridge

---

## 📁 File Organization

```
Domain-X-Name/
├── README.md                        # Domain overview and navigation
├── SubFolder/                       # Grouped by service category
│   ├── ServiceName-General.md       # Comprehensive service guide
│   └── ServiceName-DeepDive-*.md    # Advanced topics
└── ServiceName-General.md           # Standalone service files
```

### Content Structure (Each Service File)

1. **Overview** — Service purpose, problems it solves, exam relevance
2. **Core Concepts** — Fundamental components and architecture
3. **Configuration Examples** — YAML/JSON/CLI examples
4. **IAM and Security** — Roles, policies, encryption, network controls
5. **Integration Patterns** — How it works with other AWS services
6. **High Availability and DR** — Resilience, backup, failover options
7. **Performance and Cost** — Scaling, optimization, trade-offs
8. **Monitoring and Troubleshooting** — Metrics, logs, failure modes
9. **Exam Scenarios** — Original scenario-based questions with reasoning
10. **Exam Tips** — Decision criteria, common traps, elimination hints

---

## 🎓 Prerequisites

- 2+ years of AWS experience in architecture or engineering roles
- Familiarity with core AWS services (EC2, S3, VPC, IAM, RDS)
- AWS Solutions Architect Associate (recommended, not required)
- Hands-on experience with multi-account environments and IaC

---

## 🔗 Official Resources

- [SAP-C02 Exam Guide](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Skill Builder](https://skillbuilder.aws/)
- [AWS Whitepapers](https://aws.amazon.com/whitepapers/)

---

## 🤝 Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). Contributions, corrections, and improvements are welcome.

## 📄 License

MIT License — see the [MIT license text](https://opensource.org/licenses/MIT).

## 🙏 Acknowledgments

Created with [Amazon Q Developer](https://aws.amazon.com/q/developer/) using the prompt templates in [prompts.md](./prompts.md).

---

**Good luck with your SAP-C02 preparation! Focus on architecture decisions and trade-offs — not just service features. 🚀**
