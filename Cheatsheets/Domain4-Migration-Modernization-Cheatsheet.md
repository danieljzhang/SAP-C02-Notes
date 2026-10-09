# Domain 4 Cheatsheet: Migration and Modernization

> Focus: select the right workload strategy, tool, target, and cutover plan. Validate service availability and current exam-guide objectives.

## The 7 Rs (Terminology Varies)

| Strategy | Cue |
|---|---|
| Retire | Decommission an unneeded workload after dependency/owner validation |
| Retain | Keep due to timing, compliance, technical constraints, or business need |
| Rehost | Move with minimal changes; MGN can replicate eligible servers to EC2 |
| Relocate | Move a supported platform/environment with limited redesign; confirm current support |
| Repurchase | Replace with SaaS or a different product |
| Replatform | Make limited cloud optimizations without a full redesign |
| Refactor / re-architect | Change architecture to achieve business outcomes unavailable through smaller changes |

## Tool Selection

| Work unit / requirement | Tool or pattern | Not this |
|---|---|---|
| Physical/virtual server rehost with block replication | AWS Transform MGN | DMS or Migration Hub |
| Database full load + ongoing CDC | AWS DMS; schema conversion assessed separately | MGN |
| Online file/object copy, repeatable tasks, integrity checks | DataSync | Transfer Family for bulk NAS migration |
| Partner SFTP/FTPS/AS2 interface to S3/EFS | Transfer Family | DataSync for per-user protocol access |
| Migration inventory and wave progress | Migration Hub / discovery tools | Data replication engine |
| Structured migration program, assessment, mobilization | AWS MAP | Technical migration service |
| Physical/offline transfer | Current eligible physical transfer option | Assume Snowball Edge is open to new customers |

## Cutover Checklist

- Inventory owners, dependencies, data classification, source/target regions, licenses, and network flows.
- Define RTO/RPO, allowed outage, freeze strategy, validation checks, go/no-go authority, and rollback.
- Verify agent/endpoints, routes, DNS, identity, encryption, security groups, quotas, target capacity, and service compatibility.
- Test representative workloads in target; confirm application behavior, not just infrastructure status.
- Monitor replication/CDC lag or transfer errors; set a measurable cutover threshold.
- Keep source systems/data recoverable until the acceptance window closes.

## Modernization Choices

- **Microservices:** independent business capabilities need separate scale/release/ownership; avoid distributed monoliths and shared writable databases.
- **Containers:** package compatible applications; choose ECS for AWS-native simplicity, EKS for Kubernetes needs; state must live in durable external services.
- **Serverless:** event-driven or variable workloads with managed operations; design for retries, idempotency, quotas, and downstream limits.
- **Database modernization:** assess engine semantics, schema conversion, CDC, application SQL, performance, and cutover consistency separately.

## Common Traps

- Migration Hub tracks; it does not copy data.
- DMS moves database data; it does not automatically convert every schema object or application query.
- MGN rehosts servers; it does not containerize/refactor them.
- Replication is not backup, nor proof of application consistency.
- Physical transfer may meet bandwidth constraints but has eligibility, shipment, import, and scheduling risks.
- “Zero downtime” requires a precise write and rollback strategy, not only a replication product.

## Official References

- [Domain 4 overview](../Domain-4-Migration-Modernization/README.md)
- [AWS Migration Hub](https://aws.amazon.com/transform/)
- [AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [AWS DataSync](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html)
- [AWS Transfer Family](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html)
