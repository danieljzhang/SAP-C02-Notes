> Exam relevance: Domain 4. DMS moves data between supported database engines with full load and ongoing change data capture (CDC); schema and code conversion are separate concerns.

---

## 1. Purpose and Exam Relevance

AWS Database Migration Service (AWS DMS) moves data between supported database and analytics engines, including migrations into, within, or out of AWS. It supports one-time full-load migrations and ongoing change data capture (CDC) so a target can stay close to the source until cutover. DMS is central when a database must move independently from its application servers or when downtime must be minimized.

**Key distinction:** DMS moves and replicates data; schema and code conversion are separate concerns. Use DMS Schema Conversion or AWS Schema Conversion Tool (AWS SCT) where appropriate to assess/convert schema objects, then validate and remediate application SQL and semantic differences.

---

## 2. Core Components and Flow

```text
Source DB ── source endpoint ──┐
                               v
                      DMS replication
                      instance/serverless
                               |
                 full load + optional CDC
                               v
Target DB ── target endpoint ──> validation / cutover
```

- **Endpoints:** connection definitions for source and target engines, credentials, certificates, and network settings.
- **Replication resources:** provisioned replication instance or DMS Serverless capacity, depending on the use case and feature support.
- **Task:** defines migration type, table mappings, transformations, logging, and target handling.
- **Full load:** copies existing rows. **CDC:** reads source change logs and applies ongoing changes to the target.
- **DMS Fleet Advisor:** helps discover database infrastructure and plan candidates. It is an assessment aid, not the replication engine.
- **Schema Conversion:** assesses/converts supported schema objects for heterogeneous migrations. Automated conversion is not a substitute for application and data validation.

---

## 3. Choosing a Migration Pattern

| Need | Pattern |
|---|---|
| Same engine, one-time move with acceptable outage | Full load, stop writes, validate, switch application |
| Same engine, low-downtime move | Full load plus CDC; monitor lag; freeze writes briefly for final catch-up and cutover |
| Different engines | Assess and convert schema/code; remediate incompatibilities; use DMS for full load/CDC where source/target pair and feature support allow |
| Database modernization with major semantic differences | Benchmark target and application, redesign schema/query behavior as needed; do not treat engine conversion as a lift-and-shift |
| Database files or server disks must move unchanged | Evaluate a server/storage migration approach, but maintain database-level consistency and supportability requirements |

---

## 4. Cutover Architecture and Consistency

1. Discover schemas, extensions, stored procedures, jobs, users, encodings, collation, time zones, and application connection behavior.
2. Confirm source log configuration and retention support CDC for the entire migration window.
3. Establish private network reachability, security groups, routes, DNS, certificates, and least-privilege database accounts.
4. Convert and apply schema where required; compare conversion output with engine-specific features and application SQL.
5. Run full load; monitor table state, errors, throughput, and target capacity.
6. Start/continue CDC; track source-to-target latency and validate row counts/checksums or DMS validation results where supported.
7. At cutover, pause or constrain writes, allow CDC to catch up, validate critical data and application paths, switch connections, and retain a rollback path.

Do not promise zero data loss merely because CDC is enabled. Correctness depends on supported source logs, task configuration, transaction semantics, target behavior, and a controlled write transition.

---

## 5. Security and Networking

- Put replication resources where they can reach both endpoints over the intended network path. For hybrid sources, plan Direct Connect/VPN bandwidth, routing, DNS, firewall policy, and failover.
- Use dedicated least-privilege database users; protect endpoint credentials and certificates; rotate credentials according to policy.
- Use TLS for endpoint connections when supported and configure trust certificates correctly. Encrypt supported DMS storage with KMS; check key policies and cross-account access.
- Use private subnets/VPC endpoints where applicable, and restrict security groups to the actual source, target, and DMS ports.
- For cross-account migrations, authorize role and KMS access explicitly; avoid broad resource policies.

---

## 6. Performance, Reliability, and Cost

- Size replication resources for source change rate, full-load concurrency, row/LOB characteristics, network bandwidth, target write capacity, and indexes/constraints.
- Too much parallelism can overload a source, target, or network. Benchmark representative tables and monitor CPU, memory, storage, network, CDC latency, and task errors.
- CDC backlog grows if target apply capacity is lower than source change volume. Resolve bottlenecks before the cutover window.
- DMS managed options reduce infrastructure operations but do not remove the need to operate tasks, endpoints, database engines, and migration validation.
- Cost includes replication capacity/runtime, storage/logging, data transfer, target resources, and parallel-running environments. Stop temporary resources after acceptance and retention requirements are met.
- Check current engine/version support, task limitations, table/LOB behavior, regional availability, and quotas in the DMS documentation.

---

## 7. Common Failure Modes

| Symptom | Likely checks |
|---|---|
| Endpoint test fails | DNS, routes, port, security groups/NACLs, credentials, TLS certificate, database listener, and source allowlists |
| Full load is slow | Replication sizing, table selection, indexes/constraints, source read pressure, target write IOPS, LOB settings, and network bandwidth |
| CDC does not start or stops | Source log prerequisites/retention, privileges, unsupported operations, task settings, and source failover behavior |
| CDC latency increases | Change rate versus apply rate, target locks/indexes, replication CPU/storage, network congestion, and long-running transactions |
| Data differs after migration | Unsupported types/objects, transformations, character set/time-zone behavior, ongoing writes, validation coverage, and application-side writes |

---

## 8. Exam Scenarios

**Scenario:** A company is moving a busy self-managed relational database to a different engine on Amazon RDS. The application can tolerate a short maintenance window but not a multi-day outage.

**Recommended approach:** Use schema assessment/conversion first, remediate incompatible objects and SQL, load historical data with DMS, and use CDC while the application remains on the source. Test representative transactions and performance. For cutover, stop source writes, wait for CDC to catch up, validate critical data, switch application endpoints, and monitor. Keep a rollback decision point before allowing new target-only writes.

**Why not just copy database files?** Physical file formats and engine semantics are not interchangeable across engines, and file copying does not provide the needed logical conversion or controlled CDC.

---

## 9. Exam Tips

- DMS full load copies existing data; CDC applies ongoing source changes.
- Heterogeneous migration requires schema/code assessment and application testing; DMS alone does not magically translate every object.
- Migration Hub tracks progress; it does not execute DMS replication.
- No-downtime claims need explicit source-write, final synchronization, and rollback design.
- Choose DMS when the database is the migration unit; MGN when the server is the unit; DataSync for files/objects.

---

## 10. 🎯 Architecture Decision Notes

### When DMS is the right tool

Choose DMS when the requirement is *database* migration with minimal
downtime: full load plus ongoing change data capture, then a short cutover.
Size the replication instance for the workload and test with
production-like change rates — DMS performance is not automatic.

Check the source/target combination in the supported matrix first; DMS is
not universal. For homogeneous engine moves where native tools exist
(e.g., Aurora clones, RDS snapshots), the native path is often simpler.
DMS shines on heterogeneous migrations and on keeping the source writable
until cutover.

---

## 11. Official References

- [AWS DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [AWS DMS Schema Conversion](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SchemaConversion.html)
- [DMS Serverless](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Serverless.html)
- [AWS DMS pricing](https://aws.amazon.com/dms/pricing/)
