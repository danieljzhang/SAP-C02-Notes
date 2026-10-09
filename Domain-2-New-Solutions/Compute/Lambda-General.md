# AWS Lambda — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3, 4. Lambda is the serverless compute foundation. Know concurrency, triggers, destinations, VPC integration, and SnapStart.

---

## 1. Overview

AWS Lambda runs code in response to events without provisioning or managing servers. You pay only for compute time consumed (per 100ms increments).

**Key characteristics:**
- Max execution timeout: **15 minutes**
- Max memory: **10,240 MB** (10 GB)
- Max deployment package: 50 MB (zipped), 250 MB (unzipped), 10 GB (container image)
- Ephemeral storage (/tmp): up to **10 GB**

---

## 2. Concurrency

| Type | Description |
|---|---|
| Unreserved concurrency | Shared pool across all functions in the account |
| Reserved concurrency | Guarantees capacity for a function; limits its max concurrency |
| Provisioned concurrency | Pre-initialized execution environments — eliminates cold starts |

- Default account limit: **1,000 concurrent executions** (can be increased)
- **Reserved concurrency = 0** → function is throttled (useful to disable a function)
- **Provisioned concurrency** → use for latency-sensitive functions; costs more

### Cold Starts

- Occur when a new execution environment is initialized
- Affected by: runtime (Java/C# slower than Python/Node), VPC attachment, package size
- Mitigations: Provisioned Concurrency, SnapStart (Java), keep-warm pings (not recommended at scale)

### SnapStart (Java)

- Takes a snapshot of initialized execution environment
- Restores from snapshot instead of initializing from scratch
- Reduces cold start latency for Java functions by up to 90%
- Supported for Java 11+ on Corretto runtime

---

## 3. Triggers and Event Sources

| Trigger | Invocation Type | Notes |
|---|---|---|
| API Gateway / ALB | Synchronous | Request/response |
| S3 | Asynchronous | Object events |
| DynamoDB Streams | Polling (stream) | Ordered, per-shard |
| Kinesis Data Streams | Polling (stream) | Ordered, per-shard |
| SQS | Polling (queue) | Batch processing, DLQ support |
| SNS | Asynchronous | Fan-out pattern |
| EventBridge | Asynchronous | Event-driven |
| CloudWatch Events/Logs | Asynchronous | Scheduled, log-based |
| Cognito | Synchronous | User pool triggers |
| Step Functions | Synchronous/Async | Orchestration |

### Invocation Types

- **Synchronous:** caller waits for response (API Gateway, ALB, Cognito)
- **Asynchronous:** Lambda queues the event; retries on failure (S3, SNS, EventBridge)
- **Polling:** Lambda polls the source (SQS, Kinesis, DynamoDB Streams)

---

## 4. Destinations (Async Invocations)

Configure what happens after an async invocation succeeds or fails:

```
Lambda (async) → Success → SQS / SNS / EventBridge / another Lambda
               → Failure → SQS / SNS / EventBridge / another Lambda (DLQ alternative)
```

Destinations are more flexible than DLQs — they capture both success and failure events with full context.

---

## 5. VPC Integration

- Lambda can be placed inside a VPC to access private resources (RDS, ElastiCache, internal APIs)
- Requires subnets and security groups
- Uses **Hyperplane ENIs** (shared, not per-function) — eliminates the old cold start penalty from ENI creation
- Lambda in VPC still needs NAT Gateway or VPC endpoints to access internet or AWS services

```
Lambda (in VPC)
├── Private subnet → RDS, ElastiCache (private resources)
└── NAT Gateway → Internet (if needed)
    or VPC Endpoint → AWS services (S3, DynamoDB, Secrets Manager)
```

---

## 6. Layers

- Share code, libraries, and dependencies across multiple functions
- Up to 5 layers per function
- Max total unzipped size (function + layers): 250 MB
- Common use: shared utilities, AWS SDK versions, custom runtimes

---

## 7. Lambda@Edge and CloudFront Functions

| Feature | Lambda@Edge | CloudFront Functions |
|---|---|---|
| Runtime | Node.js, Python | JavaScript (ES5.1) |
| Execution location | CloudFront edge locations | CloudFront edge locations |
| Max execution time | 5s (viewer) / 30s (origin) | 1ms |
| Use case | Complex logic, auth, redirects | Simple header manipulation, URL rewrites |
| Cost | Higher | Lower |

---

## 8. Security

- **Execution role:** IAM role assumed by Lambda — defines what AWS services it can call
- **Resource-based policy:** controls who can invoke the Lambda function
- **Environment variables:** encrypt with KMS; use Secrets Manager for sensitive values
- **Code signing:** enforce that only signed deployment packages are deployed
- **VPC:** isolate Lambda from internet when accessing private resources

---

## 9. Exam Scenarios

**Scenario 1:** A Java Lambda function has unacceptable cold start latency for a customer-facing API. What is the solution?

> Enable **SnapStart** for the Java function (Java 11+ Corretto). SnapStart takes a snapshot of the initialized environment and restores from it, reducing cold start latency by up to 90%. Alternatively, use **Provisioned Concurrency** to keep execution environments pre-initialized.

**Scenario 2:** A Lambda function processes SQS messages. Some messages consistently fail and need to be preserved for analysis. What is the configuration?

> Configure a **Dead Letter Queue (DLQ)** on the SQS queue (not the Lambda function) for SQS-triggered Lambda. Failed messages are sent to the DLQ after the maximum receive count is exceeded. Alternatively, configure **Lambda Destinations** for failure to capture the failed event with full context.

**Scenario 3:** A Lambda function needs to access an RDS database in a private subnet. What is required?

> Configure the Lambda function to run inside the VPC. Assign it to the same VPC as RDS, in private subnets. Create a security group for Lambda that allows outbound to the RDS security group on port 3306. The RDS security group must allow inbound from the Lambda security group.

**Scenario 4:** A company wants to run authentication logic at the edge before requests reach the origin. What is the solution?

> Use **Lambda@Edge** with a CloudFront viewer request trigger. The Lambda function runs at the nearest CloudFront edge location, validates the request (e.g., JWT verification), and either forwards or rejects it before it reaches the origin.

---

## 10. Exam Tips

- Lambda timeout max is **15 minutes** — not suitable for long-running processes
- **Provisioned Concurrency** eliminates cold starts but costs more — use for latency-sensitive APIs
- **SnapStart** is Java-only — reduces cold starts without the cost of Provisioned Concurrency
- Lambda in VPC uses **Hyperplane ENIs** — no longer causes cold start delays
- **Reserved concurrency = 0** disables the function — useful for emergency shutoff
- **Destinations** > DLQ for async invocations — more context, handles both success and failure
- Lambda@Edge runs at **CloudFront edge** — not in a specific region; cannot access VPC resources

---

---

## 🎯 Architecture Decision Notes

### Concurrency controls — what each one actually does

- **Reserved concurrency** sets both an allocation and a ceiling for one
  function. Use it to guarantee capacity for a critical function *and* to
  cap a runaway function so it cannot starve others or overwhelm a
  downstream database.
- **Provisioned concurrency** keeps execution environments warm to cut cold
  starts. It addresses latency, not throughput limits.
- Neither control removes downstream limits. A function scaled to 1,000
  concurrent executions still throttles against the database connection
  limit or the third-party API rate limit — monitor throttles, duration,
  retries, *and* downstream capacity together.

## 11. Official References

- [Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)
- [SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)
- [Lambda@Edge](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html)
- [Lambda VPC](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html)
