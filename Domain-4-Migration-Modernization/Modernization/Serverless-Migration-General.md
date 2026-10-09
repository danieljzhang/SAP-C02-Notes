# Serverless Migration and Modernization — SAP-C02 Study Notes

## 1. When Serverless Fits

Serverless architectures are useful when teams want managed scaling and reduced infrastructure operations for event-driven, variable, or independently deployable workloads. AWS services such as Lambda, API Gateway, EventBridge, SQS, SNS, Step Functions, and managed data stores can form the target. Serverless is not automatically cheaper, and it does not remove security, reliability, integration, or capacity planning.

## 2. Modernization Pattern

```text
Users / existing application
          |
   API Gateway / ALB
          |
  Lambda or container service
          |
  +-------+----------+
  |                  |
EventBridge / SQS   managed data store
  |
Step Functions for durable multi-step orchestration
```

- Identify a bounded capability or event flow; avoid a big-bang rewrite unless requirements justify it.
- Use the strangler pattern to route selected traffic from a legacy application to new APIs/functions.
- Move long-running or multi-step work to queues and workflows; return quickly to synchronous callers.
- Externalize state and make handlers idempotent. Define event contracts, retries, dead-letter handling, and replay procedures.
- Keep an explicit source of truth during dual-running periods and reconcile data before retiring the legacy path.

## 3. Service and Integration Choices

| Need | Candidate pattern |
|---|---|
| HTTP API with managed request routing/authentication | API Gateway + Lambda, or ALB with an appropriate compute target |
| Event routing to multiple AWS services | EventBridge with rules/targets and retry/dead-letter design |
| Durable buffering and consumer decoupling | SQS between producers and workers |
| Pub/sub fanout | SNS or EventBridge based on filtering/routing and event-bus needs |
| Multi-step workflow, retries, branching, human approval | Step Functions; select workflow type and integrations for duration/latency needs |
| Shared relational data and transactions | Managed relational database; serverless compute does not imply a NoSQL database |

## 4. Design Constraints and Security

- Check current Lambda runtime, package, execution-duration, concurrency, payload, and service quota constraints against the workload. Use containers, ECS/EKS, or another compute model when the workload does not fit.
- Account for cold starts, initialization work, connection management, burst concurrency, downstream throttling, and reserved/provisioned capacity choices where applicable.
- Use least-privilege execution roles, resource policies, API authentication/authorization, secret management, KMS, TLS, and VPC controls as required.
- Design for at-least-once delivery and retries where services provide them; make side effects idempotent and avoid retry storms.
- Use asynchronous integration for long-running work and expose status/correlation IDs; do not hold a synchronous API connection open for an unbounded process.
- Observe function errors/throttles/duration, queue age/depth, workflow failures, API latency, and business outcomes. Establish tracing/correlation across boundaries.

## 5. Reliability, Cost, and Operations

- Apply timeouts, bounded retries, backoff/jitter, circuit breakers, DLQs, and replay controls at the right layer.
- Prevent overload by controlling concurrency and queue consumption relative to database and downstream capacity.
- Define deployment/versioning and rollback for code, event schemas, and data migrations. Use aliases/gradual traffic shifting where supported.
- Cost is driven by requests, duration, memory/compute, API calls, workflow transitions, messaging, logs, data transfer, and underlying data stores. Always include downstream services in the comparison.
- Serverless can reduce idle infrastructure cost for spiky workloads, but continuous high utilization or chatty workflows may favor containers or another architecture.

## 6. Common Pitfalls

- Recreating a monolith as hundreds of tiny functions without clear service boundaries.
- Treating ephemeral function storage as durable state.
- Calling many downstream services synchronously and creating a latency/failure chain.
- Ignoring duplicate delivery, poison messages, DLQ ownership, and replay safety.
- Connecting functions directly to databases without connection/concurrency management.
- Assuming pay-per-use always means lowest total cost or zero operational work.

## 7. Original Scenario

**Scenario:** An insurer's nightly batch process takes several hours and must be modernized gradually. Claims arrive through an API, enrichment can be retried, and some steps require human review.

**Recommended approach:** Introduce an API/event boundary, place durable asynchronous work on SQS or EventBridge as appropriate, orchestrate multi-step processing and approval with Step Functions, and use idempotent Lambda/container workers with a managed data store. Migrate one stage at a time and compare end-to-end cost, throughput, and operational burden.

**Why not one long Lambda invocation?** The workload has multi-hour duration and human waits; use durable orchestration and asynchronous state instead of a request-bound function.

## 8. Exam Tips

- Serverless fits event-driven, variable demand, and reduced-ops goals when constraints and integration model fit.
- Queue/workflow patterns handle long-running work better than synchronous request chains.
- “Managed” still requires IAM, observability, quotas, retries, idempotency, and cost controls.
- Modernize incrementally when risk and downtime matter; preserve a rollback path and data consistency.

## 9. Official References

- [AWS Well-Architected Serverless Applications Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [AWS Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- [AWS Prescriptive Guidance: Modernization](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-intro/welcome.html)
