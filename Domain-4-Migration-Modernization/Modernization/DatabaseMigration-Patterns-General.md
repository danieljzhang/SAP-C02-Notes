> Exam relevance: Domain 4. Database migration and modernization patterns: engine choice, conversion, replication, and cutover.

---

## 1. Start with the Outcome

Choose a database migration pattern from business and technical requirements: engine compatibility, data volume, write rate, downtime, RPO/RTO, application coupling, licensing, compliance, performance, operational skills, and cost. “Move to a managed database” is not sufficient design input.

---

## 2. Common Migration Approaches

| Pattern | Fits when | Main risks |
|---|---|---|
| Backup/restore or native dump/load | Same-engine move or outage window is acceptable | Long downtime, large restore duration, version/extension compatibility |
| Snapshot/physical replication | Compatible engines/platforms and supported procedure | Engine-specific limitations, storage/network transfer, failover semantics |
| Full load + CDC with AWS DMS | Low-downtime data movement and supported source/target pair | Log retention, CDC lag, object conversion, transaction consistency |
| Schema conversion + DMS | Heterogeneous engine move | Incomplete automatic conversion, SQL/semantic differences, query performance |
| Logical application migration | Large redesign or different data model is justified | Dual-write consistency, longer project, cutover and reconciliation complexity |

```text
Assess engine and application dependencies
          |
Same engine? -------------------- Different engine?
   |                                      |
native tools / DMS                 assess + convert schema
   |                                      |
full load or replication        full load + CDC where supported
                  \                    /
                validate -> controlled cutover -> observe
```

---

## 3. Heterogeneous Migration Checklist

- Inventory schemas, indexes, constraints, views, procedures, triggers, extensions, jobs, users, encodings, collations, and application SQL.
- Assess conversion coverage and manually remediate unsupported or behaviorally different objects.
- Benchmark representative queries and transaction patterns against the target; data compatibility does not guarantee equivalent performance.
- Select target service/class based on availability, scaling, storage, network, backup, encryption, and operational needs.
- Plan endpoint changes, connection pools, DNS TTL, certificates, secrets, IAM authentication where supported, and rollback.
- Test data types, null behavior, time zones, numeric precision, isolation/locking, transaction semantics, and edge cases.

---

## 4. Low-Downtime Cutover with DMS

1. Prepare and secure source/target endpoints and replication capacity.
2. Convert/apply schema using a supported assessment/conversion path.
3. Run full load and validate table state.
4. Continue CDC; monitor lag, errors, source log retention, and target apply capacity.
5. Exercise application against a test target and run reconciliation.
6. Freeze source writes or use an explicit dual-write strategy; do not improvise during cutover.
7. Wait for final changes to apply, validate critical invariants, switch clients, and monitor.
8. Retain source and rollback access until target writes and business acceptance meet the defined gate.

A low-downtime plan is not automatically zero downtime or zero data loss. Define exactly when the source stops accepting writes and how divergent writes are handled if rollback occurs.

---

## 5. Modernization and Data Ownership

- **Rehost database on EC2:** maximum engine/OS control, but retain patching, HA, backups, and operations responsibilities.
- **Replatform to managed relational service:** reduce infrastructure operations while preserving relational semantics where supported.
- **Adopt a different managed engine:** may improve economics or capabilities, but requires conversion, application changes, and performance validation.
- **Move to purpose-built stores:** use DynamoDB, analytics platforms, or object storage only when access patterns and consistency requirements fit; do not force relational workloads into a NoSQL model to claim modernization.
- **Decompose database per service:** improves ownership autonomy but creates data duplication, distributed transactions, and eventual consistency concerns. Use events, outbox, sagas, and explicit read models where appropriate.

---

## 6. Security, Resilience, and Cost

- Encrypt in transit and at rest; scope migration users/roles; protect credentials and KMS key access; audit schema and data access.
- Place replication paths in controlled subnets, restrict ports, use private connectivity for hybrid transfer where required, and validate cross-account policies.
- Preserve backups/PITR, test restore, define Multi-AZ/Region recovery requirements, and distinguish migration replication from disaster recovery.
- Compare target licensing, instance/storage, I/O, backup, data transfer, reserved commitments, and operational labor. Include parallel source/target cost during migration.
- Confirm engine/version support, DMS limitations, data-type handling, quotas, regional availability, and current pricing in official docs.

---

## 7. Exam Scenarios

**Scenario:** A company runs a commercial relational database on premises and wants to move to Aurora with less than 30 minutes of downtime. The database contains proprietary procedures and a high transaction rate.

**Recommended approach:** Assess engine compatibility and procedure conversion first; measure change rate and source-log retention; convert and test the schema; use a full load plus CDC only after confirming the source/target pair and workload are supported. Benchmark the application, then perform a rehearsed write freeze, final catch-up, validation, and endpoint switch. If conversion risk is too high, consider a staged same-engine move before a later modernization.

**Trap:** Assuming DMS automatically converts every procedure and guarantees the requested outage without measuring CDC lag.

---

## 8. Exam Tips

- DMS handles data movement/CDC; schema conversion and application compatibility are separate work.
- Choose native backup/restore when downtime is acceptable and it meets scale; choose full load + CDC when minimizing downtime is a requirement and supported.
- Validate application semantics, not just row counts.
- Treat migration replication separately from backup and DR; keep recoverable backups throughout.

---

## 9. Official References

- [AWS DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [DMS Schema Conversion](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SchemaConversion.html)
- [Choosing a database on AWS](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/databases-on-aws-how-to-choose.html)
