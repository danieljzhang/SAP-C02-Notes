# Microservices Modernization Patterns — SAP-C02 Study Notes

## 1. When Microservices Fit

Microservices can help when independently deployable business capabilities need separate scaling, release cadence, fault boundaries, or team ownership. They are not a default upgrade for every monolith. A well-structured monolith may be cheaper and simpler when the domain is cohesive, scaling needs are shared, or teams cannot support distributed operations.

## 2. Decomposition and Migration

```text
Legacy monolith                         AWS target
+----------------------+       +----------------------------+
| Orders | Billing |   |       | API Gateway / ALB           |
| Catalog | Shipping  | ----->| Order service -> own data   |
+----------------------+       | Billing service -> own data |
       shared DB               | Events / queues between     |
                               | bounded contexts            |
                               +----------------------------+
```

- Decompose around business capabilities and bounded contexts, not technical layers or arbitrary table boundaries.
- Use the **strangler** pattern: place a routing layer in front of the legacy system, move one capability at a time, and route selected calls to the new service.
- Establish clear service ownership, APIs/events, data contracts, SLOs, and rollback behavior before broad decomposition.
- Avoid a distributed monolith: services that must deploy together, share a schema, and synchronously call each other across every request have moved complexity without gaining independence.

## 3. Communication and Data Patterns

| Need | Pattern | Trade-off |
|---|---|---|
| Immediate query/command response | Synchronous API (API Gateway, ALB, service endpoint) | Simpler caller flow, but latency and availability are coupled |
| Loose coupling and burst absorption | SQS queue | Durable buffer and retry; adds eventual consistency and queue operations |
| Publish domain events to multiple subscribers | EventBridge or SNS | Fan-out and routing; consumers must be idempotent and tolerate reordering/duplicates where applicable |
| Multi-step business workflow | Step Functions or application saga | Explicit state/retry/compensation; workflow design becomes a first-class concern |
| Atomic database update plus event publication | Transactional outbox/CDC pattern | Avoids dual-write loss; adds outbox processing and deduplication |

Prefer data ownership per service. Cross-service joins and shared writable databases couple release and failure domains. Replicate read models or compose APIs for cross-domain views; define consistency and staleness expectations explicitly.

## 4. Reliability, Security, and Operations

- Use timeouts, bounded retries with backoff/jitter, circuit breakers, bulkheads, and idempotency keys to prevent cascading failures.
- Authenticate service identities with IAM roles or an approved workload identity pattern; authorize at API boundaries and keep secrets in managed secret stores.
- Use TLS, private network paths where required, encryption at rest, and centralized audit/trace/log collection.
- Version APIs and event schemas; support backward-compatible evolution and dead-letter/replay procedures.
- Deploy gradually with canaries/blue-green strategies, health checks, and automated rollback. A database rollback may not be simple after a new service writes new-format data.
- Monitor business-level indicators, distributed traces, queue age/depth, retries, errors, and dependency health.
- Define ownership and on-call boundaries; service autonomy without operational ownership raises recovery risk.

## 5. Cost and Migration Trade-offs

Microservices add network calls, serialization, observability, deployment pipelines, security policies, and often duplicated data. Cost may rise before teams realize benefits. Start with a capability that has a clear scaling, release, reliability, or ownership driver; measure the result before decomposing more.

A safe sequence is: map dependencies -> create an anti-corruption boundary -> route a narrow capability -> migrate data/traffic -> validate -> remove the legacy path. Keep a stable source of truth during transition and plan reconciliation for dual-running periods.

## 6. Original Scenario

**Scenario:** A monolithic order application causes deployment bottlenecks. The catalog changes weekly, while billing is tightly regulated and rarely changes. The company wants faster catalog releases but cannot risk a broad rewrite.

**Recommended approach:** Use a strangler pattern to extract catalog as a bounded capability behind a stable API. Keep billing in the monolith initially. Define catalog data ownership, event/API contracts, observability, and rollback; migrate traffic gradually. Avoid extracting billing solely for uniformity.

**Trap:** Splitting every module into a separate service creates distributed operations without proving independent scaling or release value.

## 7. Exam Tips

- Choose microservices when independent business capabilities need independent ownership/scale/release, not because they are fashionable.
- A strangler migration limits blast radius and avoids a big-bang rewrite.
- Asynchrony reduces temporal coupling but introduces eventual consistency, retries, duplicate handling, and ordering concerns.
- Shared database writes and synchronous dependency chains undermine service autonomy.

## 8. Official References

- [AWS Prescriptive Guidance: Decomposing monoliths](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-decomposing-monoliths/)
- [AWS Prescriptive Guidance: Strangler fig pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-decomposing-monoliths/strangler-fig.html)
- [AWS Well-Architected Serverless Applications Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [AWS Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
