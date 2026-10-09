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

## Common Traps

- “Highly available” is not automatically multi-Region.
- A read replica is not the same as a Multi-AZ standby or a backup.
- A private network path does not automatically provide encryption or correct routing.
- Server migration does not migrate every external dependency or guarantee application correctness.
- Automated conversion, discovery, and recommendations require validation.
- Lowest storage or compute price may not mean lowest total cost.
- Do not assume every AWS service, feature, Region, quota, or program incentive is universally available.

## Before Submitting

- Answer every item; verify the requested number of selections.
- Revisit marked questions only when you can apply a clearer requirement-based comparison.
- Check remaining time and avoid last-second changes based only on doubt.
- Follow the current exam rules; do not bring prohibited materials or attempt to reproduce protected questions.

## One-Sentence Reminder

Choose the architecture that satisfies the stated constraints with the clearest operational, security, and recovery model—not the option with the most services.

## Official Reference

- [SAP-C02 exam page](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
