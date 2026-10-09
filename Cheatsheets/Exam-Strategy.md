# SAP-C02 Exam Strategy

> Confirm the current exam format and policies on the official AWS certification page before exam day. The study materials currently use 75 questions and 180 minutes as a planning baseline; if that changes, adjust all pacing below.

## Time and Pacing

At 75 questions in 180 minutes, the arithmetic average is about 2.4 minutes per question. Long scenarios may need more time; short ones should give time back.

- **First pass:** answer questions you can reason through confidently; mark uncertain or unusually long items and move on when progress stalls.
- **Second pass:** return to marked items, compare the remaining options against the hard constraints, and use remaining time deliberately.
- **Final check:** review unanswered questions, multi-select count, and accidental changes. Keep a time buffer rather than spending the whole exam on one scenario.
- Avoid imposing an exact per-question deadline if the current exam interface or format differs; use checkpoints and adjust.

## Parse Each Scenario

1. **Name the business outcome:** migrate by a deadline, reduce cost, increase availability, improve performance, meet compliance, or simplify operations.
2. **Extract hard constraints:** RTO/RPO, outage window, data loss, throughput/latency, protocols, region/account boundaries, encryption, licensing, budget, and “no code change” requirements.
3. **Separate constraints from preferences:** words such as “must,” “cannot,” and “minimum” usually disqualify options; “prefer” and “ideally” are optimization goals.
4. **Identify the workload unit:** account, network, server, database, file set, event stream, API, or application capability.
5. **Choose the right layer:** governance/planning, connectivity, migration execution, compute, data, or observability.
6. **Check operational ownership:** who patches, scales, restores, rotates credentials, tests failover, and responds to errors?

## Eliminate Options

Reject options that:

- Violate a stated outage, recovery, data residency, protocol, or operational constraint.
- Use a service for the wrong data/workload unit (for example, Migration Hub as a data mover).
- Depend on unmentioned application changes when no code change is allowed.
- Introduce a new single point of failure or an unsupported cross-region/account assumption.
- Claim zero downtime/data loss without a write-consistency and cutover design.
- Optimize one cost metric while ignoring retrieval, data transfer, idle capacity, or operational labor.

For the remaining options, compare fit first, then reliability/security/operations, then total cost. “Most managed” is not automatically correct if it fails an explicit requirement.

## Multi-Select and Long Scenarios

- Count the number of selections requested and stop at that count.
- Evaluate every option independently against the requirements; do not choose an option merely because it is true in isolation.
- For multi-account questions, distinguish identity authentication, IAM authorization, Organizations guardrails, and resource policies.
- For hybrid networking, trace both directions, DNS, routes, encryption, redundancy, and inspection.
- For DR, map each option to RTO, RPO, data consistency, failover trigger, and failback plan.
- For migration, distinguish inventory/planning from server, database, and file transfer; add testing, cutover, rollback, and validation.

## Review Checkpoints

- Around one-third of the way through: check that your pace is sustainable and you are not over-reading every distractor.
- Around two-thirds: leave enough time for flagged questions and unanswered items.
- Final minutes: resolve blanks first, verify selection counts, and avoid changing answers without a concrete reason.

Use the test interface’s actual clock and rules; these checkpoints are adaptable study guidance, not official exam policy.

## SAP-C02 Emerging Topics

The current SAP-C02 guide lists emerging security and responsible-AI topics, including content filtering/regulatory controls for generative AI, identity controls for generative/agentic applications, and human-approval workflows. AWS identifies these as possible pretest questions that do not affect the score; use the guide for updates and do not mistake this notice for a guarantee of scored coverage.

## Final Checklist

- I can state the explicit business and technical constraint before selecting a service.
- I can explain why the closest alternative is weaker.
- I have not confused replication with backup, HA with DR, or a dashboard with an execution service.
- I have checked security, failure behavior, operations, and total cost.
- I can move on from uncertainty and return with remaining time.

## Post-Exam Reflection

After the exam, record objective areas that felt weak, reasoning habits that helped or hurt, and any study material that needs updating. Do not record or share actual exam questions or protected content.

## Official Reference

- [AWS Certified Solutions Architect – Professional](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
