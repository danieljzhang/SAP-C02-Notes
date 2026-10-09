> Exam relevance: Domains 1, 3. CloudTrail is the API audit log for AWS. Know trail types, event types, CloudTrail Lake, and Insights.

---

## 1. Overview

AWS CloudTrail records API calls made to AWS services — who did what, when, from where. It is the primary tool for security auditing, compliance, and operational troubleshooting.

**What CloudTrail captures:**
- AWS Management Console actions
- AWS CLI and SDK calls
- AWS service-to-service calls
- CloudFormation stack operations
- IAM changes, S3 bucket policy changes, security group modifications

---

## 2. Event Types

| Type | Description | Default |
|---|---|---|
| Management events | Control plane operations (create, delete, modify resources) | Enabled (free) |
| Data events | Data plane operations (S3 GetObject, Lambda Invoke, DynamoDB GetItem) | Disabled (additional cost) |
| Insights events | Unusual API activity detection | Disabled (additional cost) |

### Management Events

- Read events: Describe*, List*, Get* (low security value, high volume)
- Write events: Create*, Delete*, Modify*, Put* (high security value)
- Can filter to write-only to reduce cost

### Data Events

High-volume, resource-level operations:
- S3: GetObject, PutObject, DeleteObject (per bucket)
- Lambda: Invoke (per function)
- DynamoDB: GetItem, PutItem, DeleteItem (per table)
- Enable selectively for sensitive resources only

---

## 3. Trail Types

### Single-Region Trail

- Records events in one region only
- Stored in S3 bucket (same or different region)

### Multi-Region Trail (Recommended)

- Records events from all regions in a single S3 bucket
- Catches activity in regions you don't actively use
- Required for comprehensive audit coverage

### Organization Trail

- Created in the management account
- Records events from all member accounts
- Member accounts cannot disable or modify the trail
- Delivers to a centralized S3 bucket in the Log Archive account
- **Best practice for multi-account environments**

---

## 4. CloudTrail Lake

- Managed data lake for CloudTrail events
- Query events using SQL directly in CloudTrail (no Athena setup needed)
- Retention: configurable up to ~7 years (fixed pricing) or ~10 years (extendable retention pricing, the default)
- Availability note: closed to new customers May 31, 2026 — existing customers continue normally
- Supports: AWS events, non-AWS activity (via integrations), Config configuration items
- Faster queries than S3 + Athena for CloudTrail data

```sql
SELECT userIdentity.arn, eventName, eventTime
FROM my-event-data-store
WHERE eventName = 'DeleteBucket'
  AND eventTime > '2024-01-01'
ORDER BY eventTime DESC
LIMIT 100
```

---

## 5. CloudTrail Insights

- Detects unusual write API activity patterns
- Compares current activity against a 7-day baseline
- Generates Insights events when anomalies are detected
- Examples: sudden spike in IAM CreateUser calls, unusual S3 DeleteObject volume
- Delivers Insights events to S3 and CloudWatch Events

---

## 6. Log Integrity Validation

- CloudTrail creates a **digest file** every hour with SHA-256 hashes of log files
- Validate integrity with: `aws cloudtrail validate-logs`
- Detects if log files were modified, deleted, or forged after delivery
- Critical for compliance and forensic investigations

---

## 7. Integration Patterns

### CloudTrail → S3 → Athena

```
CloudTrail → S3 (log archive) → Athena (SQL queries) → QuickSight (dashboards)
```

- Cost-effective for long-term storage and ad-hoc queries
- Partition logs by account/region/date for query performance

### CloudTrail → CloudWatch Logs → Alarms

```
CloudTrail → CloudWatch Logs → Metric Filter → Alarm → SNS
```

- Real-time alerting on specific API calls
- Example: alert on `DeleteTrail`, `StopLogging`, `ConsoleLogin` from root account

### CloudTrail → EventBridge

```
CloudTrail → EventBridge → Lambda / SNS / Step Functions
```

- Near real-time response to API events
- Example: auto-remediate when a security group allows 0.0.0.0/0 on port 22

---

## 8. Exam Scenarios

**Scenario 1:** A company needs to ensure CloudTrail logs from all 30 accounts cannot be deleted by anyone, including account administrators. What is the solution?

> Create an **organization trail** from the management account. Deliver logs to an S3 bucket in the Log Archive account. Apply an SCP that denies `cloudtrail:DeleteTrail`, `cloudtrail:StopLogging`, and `cloudtrail:UpdateTrail` to all member accounts. Apply an S3 bucket policy that denies deletion of log objects.

**Scenario 2:** A security team needs to be alerted within minutes when the AWS root account is used to sign in to the console. What is the solution?

> Enable CloudTrail and deliver to CloudWatch Logs. Create a **metric filter** matching `{ $.userIdentity.type = "Root" && $.eventName = "ConsoleLogin" }`. Create a CloudWatch alarm on the metric. Configure the alarm to publish to an SNS topic that emails the security team.

**Scenario 3:** A company needs to query 2 years of CloudTrail logs to investigate a security incident. They need fast SQL queries without managing infrastructure. What is the solution?

> Use **CloudTrail Lake**. Import historical CloudTrail logs into a Lake event data store. Run SQL queries directly in the CloudTrail console. No Athena table setup or S3 partitioning required.

**Scenario 4:** A company wants to detect if someone is making an unusually high number of IAM role creation calls, which could indicate credential compromise. What is the solution?

> Enable **CloudTrail Insights** on the trail. Insights automatically detects unusual write API activity (including IAM CreateRole spikes) by comparing against the baseline. Insights events are delivered to S3 and can trigger EventBridge rules for automated response.

---

## 9. Exam Tips

- **Organization trail** = covers all accounts, cannot be disabled by members — always recommend for multi-account
- **Data events** are disabled by default and cost extra — enable only for sensitive resources
- **CloudTrail Lake** = managed SQL queries, no Athena setup — for fast incident investigation
- **Insights** = anomaly detection on write API activity — not real-time, compares against 7-day baseline
- **Log integrity validation** = detect tampered logs — required for forensic investigations
- CloudTrail logs have a **15-minute delivery delay** to S3 — not real-time
- For **real-time** response to API events, use CloudTrail → **EventBridge** (near real-time)

---

## 10. Official References

- [CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)
- [CloudTrail Lake](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html)
- [CloudTrail Insights](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-insights-events-with-cloudtrail.html)
- [Organization Trails](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html)
