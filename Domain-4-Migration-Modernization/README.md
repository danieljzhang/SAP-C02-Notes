> Focus on selecting workloads, migration strategies, target architectures, and modernization opportunities. Use the current official SAP-C02 exam guide to verify objective wording and weights before exam day.

## What This Domain Tests

- Assessing workload portfolios, dependencies, business outcomes, and migration readiness
- Choosing among retire, retain, rehost, relocate, repurchase, replatform, and refactor/re-architect approaches
- Selecting migration tooling for servers, databases, files, and partner transfers
- Building repeatable waves, test plans, cutovers, and rollback paths
- Modernizing with containers, serverless, microservices, and managed database services
- Balancing downtime, data consistency, security, operating effort, risk, and cost

## Official Task Outline

| Task | Focus |
|---|---|
| 4.1 Select existing workloads and processes for potential migration | Portfolio assessment, asset planning, prioritization, waves, TCO, and the 7Rs |
| 4.2 Determine the optimal migration approach for existing workloads | Data/application/database transfer, networking, identity, and governance choices |
| 4.3 Determine a new architecture for existing workloads | Compute, containers, storage, and database platform selection |
| 4.4 Determine opportunities for modernization and enhancements | Serverless, containers, purpose-built data stores, and application integration |

> Task names aligned to the [official SAP-C02 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain4.html). The guide may change; recheck before exam day.

## Migration Strategy at a Glance

| Strategy | Typical decision |
|---|---|
| Retire | Workload is no longer needed; validate dependencies before shutdown |
| Retain | Keep temporarily or permanently for business, technical, compliance, or timing reasons |
| Rehost | Move server with minimal changes; MGN can support eligible server migrations |
| Relocate | Move a supported virtualized environment with minimal redesign; validate current service and platform support |
| Repurchase | Replace with SaaS or a different product |
| Replatform | Make limited optimizations while preserving the core architecture |
| Refactor / re-architect | Change application architecture to meet business goals that cannot be met by smaller changes |

The official SAP-C02 guide uses the **seven common migration strategies (7Rs)**. Use its labels and definitions, and verify the live guide before the exam.

## Files in This Domain

### Migration services and programs

- [AWS Transform MGN](./Migration/MGN-General.md) — continuous server block replication and EC2 test/cutover launches
- [AWS DMS](./Migration/DMS-General.md) — full load, ongoing CDC, and database migration design
- [AWS DataSync](./Migration/DataSync-General.md) — managed online file/object transfer and recurring copies
- [AWS Snow Family](./Migration/SnowFamily-General.md) — offline-transfer decision and current availability caveat
- [AWS Transfer Family](./Migration/TransferFamily-General.md) — managed partner-facing file-transfer protocols
- [AWS Migration Hub](./Migration/MigrationHub-General.md) — application inventory and portfolio progress tracking
- [AWS Migration Acceleration Program](./Migration/MAP-General.md) — assessment, mobilization, migration methodology, and conditional program support

### Modernization patterns

- [Microservices patterns](./Modernization/Microservices-Patterns-General.md) — decomposition, strangler migration, service boundaries, and data ownership
- [Container migration](./Modernization/Containers-Migration-General.md) — packaging, orchestration selection, and operational trade-offs
- [Serverless migration](./Modernization/Serverless-Migration-General.md) — event-driven decomposition and serverless fit
- [Database migration patterns](./Modernization/DatabaseMigration-Patterns-General.md) — engine choice, conversion, replication, and cutover

## Workload Decision Flow

```text
Discover goals, dependencies, constraints, and ownership
                         |
                 Is workload needed?
                  /             \
                no               yes
                |                 |
              Retire       Can it remain for now?
                                /       \
                              yes       no / move
                              |            |
                            Retain    Choose migration R
                                           |
                           +---------------+----------------+
                           |               |                |
                         Server          Database        Files / partner
                         MGN             DMS             DataSync / Transfer
                                           |
                                   test + cutover + validate
                                           |
                                  optimize / modernize later
```

## Exam Decision Cues

- **Facility deadline, undocumented application, minimal change:** rehost in waves, test, then modernize later.
- **Different database engine and short outage:** assess/convert schema, full load plus CDC where supported, and plan a controlled write cutover.
- **Large file share with online bandwidth:** DataSync; use a current eligible physical-transfer alternative if the network cannot meet the window.
- **External partners require SFTP/AS2:** Transfer Family, not DataSync.
- **Portfolio tracking across tools:** Migration Hub; it does not execute the copy.
- **Business asks for a program and migration factory:** MAP-style assessment/mobilization; it is not the replication service.
- **Business capability cannot be met by simple movement:** consider refactor/re-architecture with explicit value, cost, and risk justification.

## Exam Tips

- Separate **portfolio planning**, **data movement**, **target platform**, and **application modernization**; a single service rarely does all four.
- Define source-of-truth, downtime, RPO/RTO, rollback, and validation before selecting a migration tool.
- Do not equate a successful infrastructure launch with an accepted application migration.
- Modernization increases potential agility but also scope, testing, and delivery risk; align it with a measurable outcome.
- Verify service lifecycle, compatibility, Region coverage, quotas, and pricing against current AWS documentation.

## Official References

- [SAP-C02 exam page and guide](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
- [AWS Migration Hub](https://aws.amazon.com/transform/)
- [AWS Migration Acceleration Program](https://aws.amazon.com/migration-acceleration-program/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Transform](https://aws.amazon.com/transform/)
