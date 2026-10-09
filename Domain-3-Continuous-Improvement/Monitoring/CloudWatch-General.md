# Amazon CloudWatch — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3, 4. CloudWatch is the observability foundation. Know metrics, alarms, Logs Insights, dashboards, Container Insights, and cross-account observability.

---

## 1. Overview

Amazon CloudWatch is the unified observability service for AWS. It collects metrics, logs, and traces, and enables alarms, dashboards, and automated actions.

---

## 2. Metrics

### Namespaces and Dimensions

- **Namespace:** logical grouping (e.g., `AWS/EC2`, `AWS/RDS`, custom)
- **Dimension:** key-value pair that identifies a metric (e.g., `InstanceId=i-1234`)
- **Resolution:** standard (1-minute) or high-resolution (1-second) for custom metrics

### Key Metrics by Service

| Service | Key Metrics |
|---|---|
| EC2 | CPUUtilization, NetworkIn/Out, StatusCheckFailed — **no memory/disk by default** |
| RDS/Aurora | CPUUtilization, FreeStorageSpace, DatabaseConnections, ReadLatency |
| ALB | RequestCount, TargetResponseTime, HTTPCode_Target_5XX, HealthyHostCount |
| Lambda | Invocations, Errors, Duration, Throttles, ConcurrentExecutions |
| SQS | ApproximateNumberOfMessagesVisible, NumberOfMessagesSent, ApproximateAgeOfOldestMessage |
| DynamoDB | ConsumedReadCapacityUnits, ConsumedWriteCapacityUnits, ThrottledRequests |

> EC2 memory and disk metrics require the **CloudWatch Agent** — not available by default.

### CloudWatch Agent

- Collects system-level metrics: memory, disk, swap, processes
- Collects custom application logs
- Configured via JSON config file or Systems Manager Parameter Store
- Required for: memory utilization, disk space, custom application metrics

---

## 3. Alarms

### Alarm States

| State | Meaning |
|---|---|
| OK | Metric within threshold |
| ALARM | Metric breached threshold |
| INSUFFICIENT_DATA | Not enough data to evaluate |

### Alarm Actions

- **SNS notification** → email, SMS, Lambda, SQS
- **EC2 action** → stop, terminate, reboot, recover instance
- **Auto Scaling action** → scale in/out
- **Systems Manager OpsCenter** → create OpsItem

### Composite Alarms

- Combine multiple alarms with AND/OR logic
- Reduce alarm noise — only alert when multiple conditions are true
- Example: alert only when CPU > 80% AND memory > 90%

### Metric Math

- Perform calculations on multiple metrics
- Example: error rate = `errors / requests * 100`
- Use in alarms and dashboards

---

## 4. CloudWatch Logs

### Log Groups and Streams

- **Log group:** collection of log streams with shared retention and access settings
- **Log stream:** sequence of log events from a single source
- **Retention:** 1 day to 10 years (default: never expire — set retention to control costs)

### Log Insights

Query language for analyzing log data:

```
fields @timestamp, @message
| filter @message like /ERROR/
| stats count(*) as errorCount by bin(5m)
| sort errorCount desc
| limit 20
```

- Interactive query editor in CloudWatch console
- Query multiple log groups simultaneously
- Save queries for reuse
- Visualize results as time series, bar charts, pie charts

### Metric Filters

- Extract metric data from log events
- Example: count ERROR occurrences in application logs → create CloudWatch metric → alarm on it

```
Filter pattern: [timestamp, requestId, level="ERROR", ...]
Metric name: ApplicationErrors
```

### Subscription Filters

Stream log data in real-time to:
- **Kinesis Data Streams** — real-time processing
- **Kinesis Data Firehose** — delivery to S3, OpenSearch
- **Lambda** — real-time processing
- **OpenSearch** — log analytics

---

## 5. CloudWatch Dashboards

- Visualize metrics and alarms across services and accounts
- **Cross-account dashboards:** view metrics from multiple accounts in one dashboard
- Automatic refresh: 10s, 1m, 2m, 5m, 15m
- Share dashboards publicly or with specific IAM users

---

## 6. Container Insights

- Collect metrics and logs from ECS, EKS, and Kubernetes
- Metrics: CPU, memory, network, disk per cluster/service/task/pod
- Requires CloudWatch agent or Fluent Bit as a sidecar/DaemonSet
- Performance dashboards available in CloudWatch console

---

## 7. Application Signals

- Automatic instrumentation for application performance monitoring
- Tracks: service health, SLOs, dependencies, latency
- Integrates with X-Ray for distributed tracing
- Supported runtimes: Java, Python, Node.js

---

## 8. Cross-Account Observability

- Share metrics, logs, and traces across accounts
- **Monitoring account:** central account that receives data from source accounts
- **Source accounts:** send metrics/logs/traces to monitoring account
- Configure via CloudWatch console or Organizations integration
- Enables centralized dashboards and alarms across the organization

---

## 9. CloudWatch Synthetics

- Create **canaries** — scripts that monitor endpoints and APIs
- Run on a schedule (every 1 minute to 1 hour)
- Check: availability, latency, broken links, UI workflows
- Alerts via CloudWatch alarms when canary fails
- Blueprints: heartbeat, API canary, broken link checker, visual monitoring

---

## 10. Exam Scenarios

**Scenario 1:** A company needs to monitor memory utilization on EC2 instances and alert when it exceeds 85%. CloudWatch does not show memory metrics by default. What is the solution?

> Install the **CloudWatch Agent** on EC2 instances. Configure it to collect memory metrics. The agent publishes memory utilization to a custom CloudWatch namespace. Create a CloudWatch alarm on the custom metric with an 85% threshold.

**Scenario 2:** A company wants to detect when their application logs contain more than 10 errors per minute and trigger an SNS notification. What is the solution?

> Create a **CloudWatch Logs Metric Filter** on the application log group that counts ERROR occurrences. Create a CloudWatch metric from the filter. Create an alarm that triggers when the metric exceeds 10 per minute. Configure the alarm action to publish to an SNS topic.

**Scenario 3:** A company has 20 AWS accounts and wants a single dashboard showing key metrics across all accounts. What is the solution?

> Enable **CloudWatch cross-account observability**. Configure source accounts to share metrics with a central monitoring account. Create a **CloudWatch dashboard** in the monitoring account that displays metrics from all source accounts.

**Scenario 4:** A company wants to proactively detect if their public API endpoint becomes unavailable before customers report it. What is the solution?

> Create a **CloudWatch Synthetics canary** using the heartbeat blueprint. Configure it to call the API endpoint every minute. Set up a CloudWatch alarm on the canary's `SuccessPercent` metric. Alert via SNS when the canary fails.

**Scenario 5:** A company needs to analyze application logs to find the top 10 slowest API endpoints over the last 24 hours. What is the solution?

> Use **CloudWatch Logs Insights**. Write a query that parses the response time from log events, groups by endpoint, calculates average or p99 latency, and sorts descending. Run against the application log group for the last 24 hours.

---

## 11. Exam Tips

- EC2 **memory and disk** metrics require CloudWatch Agent — not available by default
- **Composite alarms** reduce noise — combine multiple conditions before alerting
- **Logs Insights** = interactive log analysis; **Metric Filters** = ongoing metric extraction from logs
- **Subscription filters** = real-time log streaming to Kinesis/Lambda/OpenSearch
- Set log group **retention** — default is never expire, which increases costs
- **Cross-account observability** = centralized monitoring across organization accounts
- **Synthetics canaries** = proactive endpoint monitoring — detect issues before users do
- CloudWatch alarms evaluate metrics in **periods** — ensure period aligns with metric resolution

---

---

## 🎯 Architecture Decision Notes

### Choose the signal that answers the operational question

- **Metrics and alarms** detect measured behavior changes. Metric math can
  express utilization vs quota — but only when metric and quota share scope
  and units. For quotas with a CloudWatch usage metric (`AWS/Usage`
  namespace), `m1 / SERVICE_QUOTA(m1) * 100` gives a real percentage; alarm
  at a threshold that leaves response time (e.g., 75%), which is an
  operational choice, not an AWS default.
- **Synthetics canaries** exercise the user journey — they catch a broken
  experience while infrastructure metrics still look green.
- **VPC Flow Logs** record IP metadata, not packet contents — right for
  connectivity debugging, wrong for payload inspection.
- **CloudTrail vs Config:** CloudTrail records *who called which API*;
  Config records *what the configuration was* and evaluates compliance.
  Incident investigation usually needs both.

## 12. Official References

- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Cross-Account Observability](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Unified-Cross-Account.html)
- [CloudWatch Synthetics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)
