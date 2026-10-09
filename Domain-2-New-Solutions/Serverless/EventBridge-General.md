# Amazon EventBridge — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3. EventBridge is the event routing backbone. Know event buses, rules, pipes, scheduler, and cross-account patterns.

---

## 1. Overview

Amazon EventBridge is a serverless event bus that connects AWS services, SaaS applications, and custom applications using events. It enables loosely coupled, event-driven architectures.

---

## 2. Event Buses

| Bus Type | Source | Use Case |
|---|---|---|
| Default | AWS services | Receive events from all AWS services automatically |
| Custom | Your applications | Route application events |
| Partner | SaaS partners (Zendesk, Datadog, etc.) | Receive SaaS events |

---

## 3. Rules

Rules match incoming events and route them to targets:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

- Up to 5 targets per rule
- Targets: Lambda, SQS, SNS, Step Functions, Kinesis, API Gateway, ECS, CodePipeline, and more
- **Input transformation:** modify event before sending to target
- **Dead-letter queue:** capture failed event deliveries

---

## 4. EventBridge Pipes

Point-to-point integration between a source and a target with optional filtering and enrichment:

```
Source (SQS, DynamoDB Streams, Kinesis, Kafka)
    └── Filter (optional)
            └── Enrichment (Lambda, Step Functions, API Gateway)
                    └── Target (Lambda, SQS, SNS, Step Functions, etc.)
```

- Reduces boilerplate Lambda code for simple source-to-target integrations
- Built-in batching and filtering
- Supports enrichment step for data transformation

---

## 5. EventBridge Scheduler

Create scheduled tasks at scale:

- **Rate-based:** every N minutes/hours/days
- **Cron-based:** specific times (cron expressions)
- **One-time:** single future execution
- Supports flexible time windows (run within a window, not exactly at time)
- Targets: any AWS service API (Lambda, SQS, Step Functions, ECS, etc.)
- Scales to millions of schedules

---

## 6. Cross-Account Event Routing

```
Account A (producer)
  └── Custom event bus → EventBridge rule → sends to Account B event bus

Account B (consumer)
  └── Custom event bus → rule → Lambda/SQS/etc.
```

- Add resource-based policy to the target event bus allowing the source account
- Enables centralized event processing across an organization

---

## 7. Schema Registry

- Automatically discovers and stores event schemas
- Generate code bindings (Java, Python, TypeScript) from schemas
- Enables type-safe event handling in applications

---

## 8. Exam Scenarios

**Scenario 1:** A company wants to trigger a Lambda function whenever an EC2 instance is terminated in any account in the organization. What is the solution?

> Create an EventBridge rule on the **default event bus** in each account matching EC2 termination events. Route events to a **central event bus** in the security account using cross-account event routing. Create a rule on the central bus to trigger Lambda.

**Scenario 2:** A company wants to process DynamoDB Stream records, filter out irrelevant events, enrich with additional data from an API, and send to SQS. What is the cleanest solution?

> Use **EventBridge Pipes**. Configure DynamoDB Streams as the source, add a filter for relevant events, use an API Gateway enrichment step, and SQS as the target. No Lambda boilerplate code needed.

**Scenario 3:** A company needs to run a database cleanup job every Sunday at 2 AM UTC. What is the solution?

> Use **EventBridge Scheduler** with a cron expression (`cron(0 2 ? * SUN *)`). Target a Lambda function or ECS task. Scheduler handles the scheduling infrastructure — no EC2 cron jobs needed.

---

## 9. Exam Tips

- **Default bus** = AWS service events; **Custom bus** = application events
- EventBridge is **push-based** — events are delivered to targets (unlike SQS which is pull-based)
- **Pipes** = point-to-point with filter + enrichment; **Rules** = fan-out to multiple targets
- **Scheduler** = millions of schedules, flexible time windows — replaces CloudWatch Events scheduled rules
- Cross-account routing requires **resource policy** on the target event bus
- EventBridge has **at-least-once delivery** — design targets to be idempotent

---

## 10. Official References

- [EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)
- [EventBridge Pipes](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-pipes.html)
- [EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/)
- [Cross-Account Events](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-cross-account.html)
