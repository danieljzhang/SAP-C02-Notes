# AWS Service Quotas — SAP-C02 Study Notes

> Exam relevance: Domain 3 (25%). Service Quotas is the centralized way to view and request quota increases. Know how to monitor quotas proactively and integrate with CloudWatch.

---

## 1. Overview

AWS Service Quotas provides a central location to view and manage AWS service quotas (formerly called limits) across all services and regions. It enables proactive quota management before hitting limits.

---

## 2. Key Concepts

| Concept | Description |
|---|---|
| Quota | Maximum value for a resource or operation (e.g., 5 VPCs per region) |
| Default quota | AWS-defined starting value |
| Applied quota | Current quota for your account (may be higher than default after increase) |
| Adjustable quota | Can be increased via request |
| Non-adjustable quota | Hard limit — cannot be increased |

---

## 3. Quota Increase Requests

- Submit increase requests directly from Service Quotas console
- Requests go to AWS Support automatically
- Track request status in Service Quotas console
- Some quotas are auto-approved; others require manual review

```bash
# Request quota increase via CLI
aws service-quotas request-service-quota-increase \
  --service-code ec2 \
  --quota-code L-1216C47A \
  --desired-value 100
```

---

## 4. CloudWatch Integration

Monitor quota utilization proactively:

- Service Quotas publishes utilization metrics to CloudWatch for supported services
- Create CloudWatch alarms when utilization exceeds a threshold (e.g., 80%)
- Alert before hitting the limit — not after

```bash
# Create alarm for EC2 instance quota utilization
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-vCPU-Quota-Warning" \
  --namespace "AWS/Usage" \
  --metric-name "ResourceCount" \
  --dimensions Name=Service,Value=EC2 Name=Resource,Value=vCPU \
  --statistic Maximum \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:quota-alerts
```

---

## 5. Quota Templates (Organizations)

- Define quota increase requests as templates
- Apply templates to new accounts automatically when they join the organization
- Ensures new accounts start with appropriate quotas for your workloads

---

## 6. Common Quotas to Monitor

| Service | Quota | Default |
|---|---|---|
| EC2 | Running On-Demand Standard instances (vCPUs) | 32–1152 vCPUs (varies) |
| VPC | VPCs per region | 5 |
| Lambda | Concurrent executions | 1,000 |
| RDS | DB instances per region | 40 |
| ELB | Load balancers per region | 50 |
| IAM | Roles per account | 1,000 |

> Always verify current defaults — they change over time.

---

## 7. Exam Scenarios

**Scenario 1:** A company is launching a new product and expects to need 500 EC2 instances in a region where the default limit is 32 vCPUs. What should they do proactively?

> Use **Service Quotas** to request an increase for EC2 On-Demand vCPUs well before the launch. Submit the request with justification. Monitor the request status. Set a CloudWatch alarm on the quota utilization metric to alert if approaching the new limit.

**Scenario 2:** A company wants all new AWS accounts to automatically have higher Lambda concurrency limits. What is the solution?

> Create a **quota template** in Service Quotas for Lambda concurrent executions. Apply the template to the organization. New accounts joining the organization automatically have the quota increase request submitted.

---

## 8. Exam Tips

- Service Quotas = **proactive** quota management — don't wait for `LimitExceeded` errors
- **CloudWatch alarms on quota utilization** = alert at 80% before hitting 100%
- **Quota templates** = automatically apply quota increases to new accounts in the organization
- Some quotas are **per region**, some are **per account** — check the scope
- **Trusted Advisor** also shows service limit warnings — complementary to Service Quotas
- Request increases **well in advance** — some require manual AWS review and take days

---

## 9. Official References

- [Service Quotas User Guide](https://docs.aws.amazon.com/servicequotas/latest/userguide/)
- [CloudWatch Usage Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Usage-Metrics.html)
- [Quota Templates](https://docs.aws.amazon.com/servicequotas/latest/userguide/organization-templates.html)
