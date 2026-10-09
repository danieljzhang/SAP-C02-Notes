# AWS Step Functions — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3. Step Functions is the orchestration layer for complex workflows. Know Standard vs Express, state types, error handling, and integration patterns.

---

## 1. Overview

AWS Step Functions is a serverless orchestration service that coordinates distributed components using visual workflows (state machines). It manages state, retries, error handling, and parallel execution.

**When to use Step Functions:**
- Multi-step workflows with branching, parallel execution, or error handling
- Long-running processes (up to 1 year)
- Coordinating multiple Lambda functions or AWS services
- Human approval workflows

---

## 2. Workflow Types

| Feature | Standard | Express |
|---|---|---|
| Max duration | 1 year | 5 minutes |
| Execution semantics | Exactly-once workflow execution unless Retry behavior is configured | Asynchronous: at-least-once; synchronous: at-most-once |
| Execution rate | Region/account quotas vary; check current Service Quotas | Region/account quotas vary; check current Service Quotas |
| Pricing | Per state transition | Per execution, duration, and memory consumed |
| Audit history | Full execution history | CloudWatch Logs only |
| Use case | Long-running, durable, auditable workflows | High-volume, short-duration processing where the chosen delivery semantics fit |

**Express subtypes:**
- **Synchronous Express:** caller waits for result (API Gateway integration)
- **Asynchronous Express:** fire-and-forget, result via CloudWatch

---

## 3. State Types

| State | Purpose |
|---|---|
| Task | Invoke a Lambda, AWS service, or activity |
| Choice | Conditional branching (if/else) |
| Wait | Pause for a duration or until a timestamp |
| Parallel | Execute branches concurrently |
| Map | Iterate over an array, process each item |
| Pass | Pass input to output (transformation) |
| Succeed | End execution successfully |
| Fail | End execution with an error |

---

## 4. Service Integrations

Step Functions can integrate with AWS services in two ways:

| Integration | Behavior |
|---|---|
| Request-Response | Call an integrated service API and proceed when its response is returned; some APIs return before their underlying job completes |
| .sync (Run a Job) | Wait for a supported service job to complete before proceeding |
| .waitForTaskToken | Pause until an external system sends a callback token |

### .waitForTaskToken Pattern (Human Approval)

```
Step Functions → SQS (with task token)
                      └── Human reviews
                              └── Sends task token back to Step Functions
                                      └── Workflow resumes
```

---

## 5. Error Handling

```json
"Retry": [{
  "ErrorEquals": ["Lambda.ServiceException", "Lambda.TooManyRequestsException"],
  "IntervalSeconds": 2,
  "MaxAttempts": 3,
  "BackoffRate": 2
}],
"Catch": [{
  "ErrorEquals": ["States.ALL"],
  "Next": "HandleError"
}]
```

- **Retry:** automatic retry with exponential backoff
- **Catch:** route to error handling state on failure
- **States.ALL:** catches any error
- **States.Timeout:** catches timeout errors

---

## 6. Exam Scenarios

**Scenario 1:** A company needs to orchestrate an order processing workflow: validate order → charge payment → update inventory → send confirmation email. Each step can fail and needs retry logic. What is the solution?

> Use **AWS Step Functions Standard Workflow**. Define each step as a Task state invoking Lambda or AWS services. Add Retry configurations for transient failures. Add Catch states for permanent failures to trigger compensation logic (e.g., refund).

**Scenario 2:** A company processes 500,000 IoT events per second, each requiring a 3-step transformation pipeline. What workflow type?

> Consider **Asynchronous Express Workflows** only if the workflow duration, at-least-once execution semantics, integrations, and current account/Region quotas fit. At this event rate, first decide whether to batch/stream events (for example, through Kinesis or SQS) and size consumers to protect downstream services; do not assume a fixed Step Functions execution quota or exactly-once effects.

**Scenario 3:** A workflow needs a human to approve a document before proceeding. The approval may take hours. What is the pattern?

> Use **Step Functions with .waitForTaskToken**. The workflow sends the document and task token to an SQS queue or SNS topic. A human reviews and approves via an application that calls `SendTaskSuccess` with the token. The workflow resumes.

---

## 7. Exam Tips

- **Standard** = durable execution with exactly-once workflow semantics unless retries are configured
- **Express** = asynchronous at-least-once; synchronous at-most-once — make side effects idempotent as needed
- **.waitForTaskToken** = human approval or external system callback pattern
- **Map state** = process arrays in parallel — replaces Lambda fan-out patterns
- **Parallel state** = concurrent branches — all branches must complete before proceeding
- Step Functions replaces complex Lambda-chaining code with visual, auditable workflows

---

## 8. Official References

- [Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/)
- [Standard vs Express](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-standard-vs-express.html)
- [Workflow type execution guarantees](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html)
- [Service Integrations](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-service-integrations.html)
