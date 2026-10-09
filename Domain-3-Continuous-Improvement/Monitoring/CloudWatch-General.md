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

---

## 🎯 Architecture Decision Notes

### API canary vs heartbeat — test the behavior, not just the reachability

Both are Synthetics canaries, but they answer different questions. A **heartbeat** canary checks that an endpoint responds at all — it catches "the server is down." An **API canary** issues real requests and asserts on the response: status codes, payload shape, latency — it catches "the server is up but returning errors," like a dependency that started replying 200 with an error body, or a third-party API silently returning internal-error payloads.

Decision rules:

- **Heartbeat** for: cron-job liveness, "did the nightly ETL phone home," IoT keep-alives — anywhere failure is defined as *absence of a signal*.
- **API canary** for: any endpoint whose wrong answer is worse than no answer — payment callbacks, pricing feeds, partner integrations. If a bad response would cause your system to act on garbage (the trading-app scenario: acting on a partner's error payload as if it were market data), heartbeat monitoring is the wrong tool.
- **Pair with Route 53 health checks** when the response should drive failover: the canary's alarm becomes the health check that shifts traffic to a secondary region.
- Trap: a heartbeat that hits `/health` proves the load balancer and the web server work; it says nothing about the database, the downstream APIs, or the business logic. Match the canary's assertions to the failure modes you actually fear.

**Fresh example:** A travel-booking platform depends on a partner airline's fare API. A heartbeat on the API's status page stays green while the fare endpoint starts returning malformed prices. An API canary that requests a known route and asserts the price falls within a sane range trips within minutes — before customers book at $0 fares.

---

## 🎯 Architecture Decision Notes

### Container Insights — read metrics at the right altitude

Container Insights reports the same CPU/memory/network numbers at four altitudes: cluster, service, task, and container. The altitude you alert on determines what you can conclude. Cluster-level utilization tells you about bin-packing efficiency; service-level tells you whether to scale the service; task-level tells you whether individual tasks are sized right; container-level is for debugging a noisy neighbor inside a multi-container task.

Decision rules:

- **Scaling decisions live at the service level:** alarm on service-aggregated CPU/memory to drive ECS Service Auto Scaling. Cluster-level alarms are for capacity planning, not scaling actions.
- **Right-sizing lives at the task level:** compare task-level *utilized* CPU/memory against what the task *definition* requested. Persistent under-utilization means shrink the task definition (direct cost saving on Fargate); tasks hitting their limits means grow it before OOM kills start.
- **Collection model differs by launch type:** on Fargate, telemetry is built into the platform — nothing to install. On EC2, the metrics come from agents on the container instances. If Container Insights shows gaps on EC2, check the agent before blaming the workload.
- Trap: service-level averages hide a single hot task. When an alarm fires at the service level, drill to task level before scaling — you may have a skewed workload (one bad partition key, one poisoned container) that more tasks won't fix.

**Fresh example:** A team's Fargate service keeps getting OOM-killed at 2 AM. Service-level memory looks fine (average 40%), but task-level metrics show one task per night spiking to its 512 MB limit while the others idle — a single tenant's export job landing on whatever task it hits. The fix is per-task memory headroom plus a queue-based split of the export workload, not a service-wide scale-out.

## 12. Official References

- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Cross-Account Observability](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Unified-Cross-Account.html)
- [CloudWatch Synthetics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)
