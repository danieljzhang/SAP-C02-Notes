# SAP-C02 Readiness Tracker

> This is a study aid, not a score predictor. AWS reports a scaled score and does not define a raw-practice percentage that guarantees a pass. Use the official exam guide and AWS practice resources alongside this tracker.

## Requested 90% Practice Target

Use **90% raw accuracy on two separate, fresh, timed full-length practice sets** as a personal stretch target, with no domain below 85%. Take those mock exams separately through AWS or a provider such as Tutorial Dojo; this repository contains study notes and original scenario exercises, not full-length mock exams. Review every miss and retest the underlying objective with different questions before treating a result as evidence of readiness.

This 90% target is not an AWS scoring conversion or pass guarantee. SAP-C02 uses scaled scoring; its official guide states a 750/1000 passing score and includes unscored questions that are not identified. Do not infer your exam score directly from practice accuracy.

## Objective Self-Assessment

For each task, record a date, a timed practice score, the weakest decision pattern, and the next action. Do not mark an objective ready based only on rereading notes.

| Objective | Confidence (1–5) | Latest timed evidence | Next review |
|---|---:|---|---|
| 1.1 Network connectivity strategies |  |  |  |
| 1.2 Security controls |  |  |  |
| 1.3 Reliable and resilient architectures |  |  |  |
| 1.4 Multi-account AWS environment |  |  |  |
| 1.5 Cost optimization and visibility |  |  |  |
| 2.1 Deployment strategy |  |  |  |
| 2.2 Business continuity |  |  |  |
| 2.3 Security controls |  |  |  |
| 2.4 Reliability |  |  |  |
| 2.5 Performance |  |  |  |
| 2.6 Cost optimization |  |  |  |
| 3.1 Operational excellence |  |  |  |
| 3.2 Security improvement |  |  |  |
| 3.3 Performance improvement |  |  |  |
| 3.4 Reliability improvement |  |  |  |
| 3.5 Cost optimization opportunities |  |  |  |
| 4.1 Select workloads for migration |  |  |  |
| 4.2 Choose migration approach |  |  |  |
| 4.3 Design target architecture |  |  |  |
| 4.4 Modernization and enhancements |  |  |  |

## Miss Review

For every missed or guessed question, record:

| Field | What to capture |
|---|---|
| Objective | The exact task from the current guide |
| Decisive requirement | The latency, RTO/RPO, security, protocol, scale, cost, or operational constraint that controls the answer |
| Error type | Knowledge gap, misread constraint, service confusion, unsupported assumption, or time-pressure guess |
| Why the selected answer failed | State the violated constraint, not just the service name |
| Correct decision rule | Explain when the recommended pattern fits and when it does not |
| Retest date | Revisit later with a new scenario, not the same memorized question |

## Readiness Checklist

- I can explain all 20 official tasks in my own words and link each to representative architecture decisions.
- I can compare plausible services and identify the explicit requirement that disqualifies the weaker option.
- I can reason through multi-account, hybrid, multi-Region, migration, DR, deployment, and cost scenarios.
- I can state what changes are version-sensitive and find the corresponding current AWS documentation.
- I have completed at least two fresh timed practice sets, reviewed all errors, and have no unexplained weak objective.
- I can pace a 75-question/180-minute exam using the current exam page, while recognizing the format can change for a new version.

## Study Loop

1. Take a timed baseline and classify misses by official objective.
2. Study the linked notes for the two weakest objectives; verify version-sensitive details in AWS docs.
3. Draw or explain a design from memory, including failure path, security boundary, cost driver, and rollback.
4. Retest with a different scenario after a delay.
5. Repeat until the stretch target and readiness checklist are met; schedule only for the exam version you intend to take.

## Official References

- [SAP-C02 exam guide and scoring](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02.html)
- [AWS certification page and SAP-C03 transition](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
- [Cheatsheets and practice scenarios](./README.md)
