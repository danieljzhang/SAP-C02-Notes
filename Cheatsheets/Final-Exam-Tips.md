# SAP-C02 Final Exam Tips

## The Last Review

- Recheck the official exam guide, domain names, question count, duration, and test-day policies.
- Review weak-domain cheatsheets first; spend remaining time on service comparisons and original scenarios.
- Prioritize architecture decisions and trade-offs over memorizing console click paths or unstable quotas.
- Sleep, prepare identification and appointment details, and follow the current testing-provider instructions.

## High-Value Distinctions

- **SCP/RCP:** permission ceilings, not grants.
- **Direct Connect/VPN:** dedicated connectivity versus encrypted IPsec tunnels; DX is not encrypted by default.
- **ALB/NLB/GWLB:** Layer 7 application routing / Layer 4 network performance / virtual appliance insertion.
- **CloudFront/Global Accelerator:** content caching / global anycast routing to healthy endpoints.
- **SQS/SNS/EventBridge/Kinesis/MSK:** queue / fanout / event routing / streaming data / Kafka-compatible event streaming.
- **Multi-AZ/read replica/backup/DR:** availability / read scaling or recovery option / point-in-time protection / regional or broader recovery strategy.
- **CloudWatch/CloudTrail/X-Ray:** telemetry / AWS API audit / distributed tracing.
- **MGN/DMS/DataSync/Transfer Family/Migration Hub:** server rehost / database migration / file-object copy / partner protocol endpoint / migration tracking.

## Scenario Habits

1. Underline non-negotiable requirements: downtime, data loss, encryption, access boundary, latency, compatibility, and budget.
2. Determine if the problem is new design, migration, modernization, or improvement of an existing system.
3. Draw the data and control flows. Mark trust boundaries, failure domains, and source of truth.
4. Pick the smallest viable architecture that meets the hard requirements.
5. Check the failure case, recovery path, and operational owner.
6. Eliminate options that add unneeded code changes, manual steps, or cost when a managed fit exists.

## Common Traps — the 30 that decide pass/fail

**Identity & access**
1. SCPs/RCPs are ceilings, never grants — an SCP can't give permission, only take it away.
2. Cross-account access needs BOTH the identity policy AND the resource policy to allow.
3. An explicit DENY anywhere beats any number of ALLOWs.
4. Cognito User Pools ≠ Identity Pools: one authenticates users, the other grants AWS access.

**Networking**
5. Direct Connect is NOT encrypted by default — add VPN or MACsec for encryption in transit.
6. VPC peering is non-transitive; overlapping CIDRs kill peering (use PrivateLink instead).
7. "Highly available" ≠ multi-Region. Multi-AZ survives AZ failure, not Regional failure.
8. A private network path ≠ encrypted. PrivateLink/VPC endpoints still need TLS for encryption.
9. Route 53 alone is not failover — health checks + routing policy do the failing over.

**Compute & containers**
10. Lambda bills per 1-ms, not per 100ms. SnapStart covers Java 11+, Python 3.12+, .NET 8+.
11. Provisioned Concurrency ≠ Reserved Concurrency: one pre-warms, the other caps.
12. Fargate needs VPC endpoints (ECR, CloudWatch, Secrets Manager) in private subnets — or tasks can't pull images.
13. "The team already runs Kafka" → MSK. "AWS-native streaming" → Kinesis.

**Storage & databases**
14. A read replica is NOT a backup. A Multi-AZ standby is NOT a backup. Replication ≠ backup.
15. S3 Versioning ≠ Object Lock: versioning recovers from overwrites; Object Lock blocks deletion (Compliance mode even blocks root).
16. Aurora Global Database is single-writer. Multi-Region writes → DynamoDB Global Tables.
17. DynamoDB Global Tables resolve conflicts by last-writer-wins — design for the loser.
18. "Lowest storage price" ≠ lowest total cost — retrieval fees and minimum durations change the math.

**Messaging — pick by pattern**
19. Queue (buffered work) → SQS. Fan-out (one message, many consumers) → SNS. Pattern-routed events → EventBridge. Ordered replay → Kinesis.
20. FIFO costs throughput — don't pay the FIFO tax where ordering isn't required.
21. SNS fan-out loses ordering; SQS standard loses "every consumer gets every message."

**Resilience & migration**
22. RTO/RPO decide the DR strategy — backup/restore vs pilot light vs warm standby vs active/active.
23. Pilot light keeps *data* replicated and *core* running; warm standby keeps a scaled-down *full* system.
24. MGN rehosts servers; DMS moves databases; DataSync copies files; don't mix them up.
25. The 7 Rs include Relocate (not just the classic 6).

**Cost & operations**
26. "Cheapest per-hour" ≠ cheapest overall — data transfer, requests, and idle capacity dominate.
27. CloudWatch = metrics/logs/alarms. CloudTrail = who did what API call. Config = what changed. Don't mix the three.
28. A Config rule is detective (finds drift); an SCP is preventive (blocks it). Use both.

**Exam-day mindset**
29. Four answers that all "work" → re-read for the ONE requirement word that picks the winner (static IP, single-tenant, zero downtime, no code changes).
30. "No code changes" / "least operational overhead" / "most cost-effective" are usually the actual question — the scenario is just context.

## Before Submitting

- Answer every item; verify the requested number of selections.
- Revisit marked questions only when you can apply a clearer requirement-based comparison.
- Check remaining time and avoid last-second changes based only on doubt.
- Follow the current exam rules; do not bring prohibited materials or attempt to reproduce protected questions.

## One-Sentence Reminder

Choose the architecture that satisfies the stated constraints with the clearest operational, security, and recovery model—not the option with the most services.

## Official Reference

- [SAP-C02 exam page](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
