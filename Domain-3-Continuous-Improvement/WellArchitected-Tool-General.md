> Exam relevance: Domain 3 (25%). The Well-Architected Framework and Tool are the structured approach to architecture improvement. Know the 6 pillars, key design principles, and how the tool works.

---

## 1. The Six Pillars

| Pillar | Focus |
|---|---|
| Operational Excellence | Run and monitor systems, improve processes |
| Security | Protect data, systems, and assets |
| Reliability | Recover from failures, meet demand |
| Performance Efficiency | Use resources efficiently |
| Cost Optimization | Avoid unnecessary costs |
| Sustainability | Minimize environmental impact |

---

## 2. Key Design Principles per Pillar

### Operational Excellence
- Perform operations as code (IaC, runbooks)
- Make frequent, small, reversible changes
- Anticipate failure — test failure scenarios
- Learn from operational events

### Security
- Implement a strong identity foundation (least privilege, IAM)
- Enable traceability (CloudTrail, CloudWatch)
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest

### Reliability
- Automatically recover from failure
- Test recovery procedures
- Scale horizontally to increase availability
- Stop guessing capacity — use Auto Scaling
- Manage change through automation

### Performance Efficiency
- Democratize advanced technologies (use managed services)
- Go global in minutes (multi-region)
- Use serverless architectures
- Experiment more often
- Consider mechanical sympathy (match resource to workload)

### Cost Optimization
- Implement cloud financial management
- Adopt a consumption model (pay for what you use)
- Measure overall efficiency
- Stop spending money on undifferentiated heavy lifting
- Analyze and attribute expenditure

### Sustainability
- Understand your impact
- Establish sustainability goals
- Maximize utilization
- Use managed services
- Reduce downstream impact

---

## 3. AWS Well-Architected Tool

A free service in the AWS console for conducting architecture reviews:

### Workload Review Process

1. Define a workload (name, description, environment, regions)
2. Answer questions for each pillar (50–70 questions total)
3. Review identified risks (High Risk Issues, Medium Risk Issues)
4. Create improvement plan with prioritized recommendations
5. Track progress over time with milestone snapshots

### Risk Levels

| Level | Meaning |
|---|---|
| High Risk Issue (HRI) | Significant architectural gap — address immediately |
| Medium Risk Issue (MRI) | Improvement opportunity — address in near term |
| None | Best practice followed |

---

## 4. Lenses

Extend the Well-Architected Framework for specific domains:

| Lens | Focus |
|---|---|
| Serverless Lens | Lambda, API Gateway, DynamoDB architectures |
| SaaS Lens | Multi-tenant SaaS architectures |
| Machine Learning Lens | ML workloads and pipelines |
| Data Analytics Lens | Data lake and analytics architectures |
| IoT Lens | IoT device and data architectures |
| Custom Lenses | Define your own questions and best practices |

---

## 5. Well-Architected Reviews at Scale

- **AWS Well-Architected Partners:** APN partners trained to conduct reviews
- **Custom Lenses:** share across accounts in the organization
- **Profiles:** pre-answer questions based on workload type to focus the review
- **Trusted Advisor integration:** some checks align with Well-Architected best practices

---

## 6. Exam Scenarios

**Scenario 1:** A company wants to assess whether their existing three-tier web application follows AWS best practices before a major scaling event. What is the process?

> Use the **AWS Well-Architected Tool**. Define the workload. Answer questions across all six pillars. Review High Risk Issues and Medium Risk Issues. Create an improvement plan prioritizing HRIs. Address identified gaps before the scaling event.

**Scenario 2:** A company is building a serverless application and wants architecture guidance specific to Lambda and API Gateway. What is the solution?

> Apply the **Serverless Lens** in the Well-Architected Tool. The Serverless Lens adds serverless-specific questions covering Lambda concurrency, cold starts, API Gateway throttling, event-driven patterns, and DynamoDB design.

---

## 7. Exam Tips

- Know all **6 pillars** by name — they appear in scenario questions as evaluation criteria
- **Reliability pillar** = HA, DR, fault tolerance — most relevant for architecture scenarios
- **Cost Optimization pillar** = right-sizing, consumption model, reserved capacity
- The Well-Architected Tool is **free** — no cost to conduct reviews
- **HRI** = fix now; **MRI** = fix soon — prioritize HRIs in improvement plans
- **Lenses** extend the framework for specific workload types — Serverless Lens is most exam-relevant

---

## 8. 🎯 Architecture Decision Notes

### Make the review repeatable, not ceremonial

Run Well-Architected reviews on a cadence tied to change (major releases,
new integrations), not on the calendar alone. For each pillar, convert
findings into tracked remediation with owners — a review whose output is a
PDF nobody opens is theater. Use the review to force the tradeoff
conversations teams avoid: cost vs resilience, speed vs control, build vs
buy. Re-run after remediation and compare; the delta is the value.

## 9. Official References

- [Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Well-Architected Tool](https://docs.aws.amazon.com/wellarchitected/latest/userguide/)
- [Serverless Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
