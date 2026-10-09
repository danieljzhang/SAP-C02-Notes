# AWS Transform MGN — SAP-C02 Study Notes

> AWS documentation currently refers to Application Migration Service as **AWS Transform MGN** (often shortened to MGN). Product names and supported source/target configurations can change; confirm them in the linked guide before implementation.

## 1. Purpose and Exam Relevance

MGN is a rehosting service for moving physical, virtual, and cloud servers to Amazon EC2 with limited application changes. It continuously replicates source-server block storage to a staging area in the target Region, then launches converted EC2 test and cutover instances. It is useful when the priority is to move server workloads quickly and defer redesign.

MGN does not redesign an application, convert a database schema, or move arbitrary shared-file datasets. Pair it with DMS for database replication, DataSync for file/object data, and landing-zone/network services for the target environment.

## 2. Migration Flow

```text
Source servers                 AWS target Region
+------------------+           +---------------------------+
| Physical / VM /  | agent --->| Replication servers and   |
| cloud workloads  |  blocks   | staging EBS volumes       |
+------------------+           +-------------+-------------+
                                             |
                                  test launch | cutover launch
                                             v
                                  +---------------------------+
                                  | EC2 instances in target   |
                                  | VPC / subnets / accounts  |
                                  +---------------------------+
```

1. Assess source inventory, application dependencies, network paths, and target account/Region readiness.
2. Configure replication and launch settings; install the replication agent where required.
3. Replicate changed blocks to the staging area and monitor lag/health.
4. Launch test instances, validate application behavior, networking, security, and performance.
5. Schedule cutover, quiesce writes where appropriate, wait for replication to catch up, then launch cutover instances.
6. Validate production, update traffic/DNS and integrations, and retain a rollback plan until acceptance.
7. Decommission sources only after business owners confirm data and service integrity.

## 3. Architecture and Design Decisions

- **Rehost versus modernize:** MGN is optimized for rehosting. Treat it as a migration wave enabler; modernize after the workload is stable unless there is a compelling reason to change the design before cutover.
- **Staging versus target network:** keep replication infrastructure isolated and sized for throughput; place launched test/cutover servers in the intended target VPC and subnets.
- **Application grouping:** group interdependent servers into applications/waves; coordinate database, shared storage, identity, DNS, and firewall changes as one release.
- **Test launches:** validate boot, drivers, licensing, hostnames, static IP assumptions, domain membership, monitoring, backup, and downstream connectivity. A successful launch is not proof of application correctness.
- **Cutover:** establish an explicit write-freeze or consistency strategy for stateful systems. Define go/no-go criteria and rollback ownership before starting.

## 4. Security, Connectivity, and Governance

- Use a dedicated migration account or controlled target accounts with least-privilege cross-account roles. Apply Organizations guardrails and centralized logging consistently.
- Restrict agent and replication-server egress to documented service endpoints and required source/target paths. Use private connectivity where supported and required by policy; validate DNS, routes, firewalls, proxies, and TLS inspection.
- Protect credentials, agent installation packages, and replication data. Use KMS-backed encryption for supported stored data and TLS for network traffic; verify the exact encryption controls and key ownership for the selected setup.
- Separate replication, test, and production permissions. Control who can launch, modify, or terminate migrated instances.
- Validate data residency, regulated workload requirements, and supported Regions before selecting a target.

## 5. Reliability, Operations, and Cost

- Monitor replication health, lag, agent connectivity, staging capacity, launch failures, and test/cutover status. Alert on stalled replication well before a migration window.
- Replication is not an application-consistent backup. Use workload-aware quiescence and independent backups for recoverability.
- Rehearse at least one representative test launch per server pattern and verify rollback; test failures often reveal undocumented dependencies.
- Cost drivers include replication compute/storage, test instances, target EC2/EBS, data transfer, and parallel-running source/target environments. Right-size after baseline performance is measured, not before functional acceptance.
- Confirm current service quotas, supported operating systems, Regions, and licensing requirements in AWS documentation. Avoid treating a supported OS list as proof every application is compatible.

## 6. Common Failure Modes

| Symptom | Checks |
|---|---|
| Replication agent cannot register | Credentials/role, outbound connectivity, DNS, proxy, time synchronization, and supported source OS |
| High or increasing lag | Source change rate, available bandwidth, replication resources, staging volume throughput/capacity, and competing traffic |
| Test instance fails to boot | Boot mode, drivers, disk layout, kernel/initramfs, encryption, and source-specific configuration |
| Instance boots but application fails | Dependency map, DNS, firewall rules, IP allowlists, secrets, licenses, identity, and shared storage |
| Cutover data is inconsistent | Source write activity, replication completion, database consistency procedures, and timing of dependent-system changes |

## 7. When Another Service Fits Better

| Requirement | Better fit / companion |
|---|---|
| Heterogeneous database engine conversion | AWS DMS with DMS Schema Conversion or AWS Schema Conversion Tool assessment |
| Continuous database CDC and low-downtime cutover | AWS DMS; MGN alone does not perform logical database migration |
| Large file-system or object transfer with verification | AWS DataSync |
| Move a complete virtual machine with VMware-specific orchestration | Evaluate AWS migration options for the current VMware environment; do not assume MGN is a hypervisor-level lift-and-shift tool |
| Redesign into containers/serverless | Replatform/refactor workstream; MGN may be a temporary landing step |

## 8. Original Scenario

**Scenario:** A retailer must move 180 Windows and Linux application servers from a data center before a facility lease ends. The applications are poorly documented, downtime must be brief, and the company plans to modernize later.

**Recommended approach:** Inventory and group servers by dependency, use MGN to continuously replicate supported servers, test representative applications in the target account/VPC, and cut over in coordinated waves. Use DMS for databases that need independent replication and DataSync for shared file data. Keep source systems available until acceptance and a defined rollback window expires.

**Why not refactor first?** A broad redesign increases schedule and scope risk before the lease deadline. Rehosting is a deliberate first step, not a claim that the resulting architecture is optimized.

## 9. Exam Tips

- MGN = server rehost through continuous block-level replication and EC2 launch; DMS = database migration/CDC; DataSync = file/object movement.
- A test launch validates the target but is not a production cutover or an application-consistency guarantee.
- The replication staging network and the final workload network have different purposes.
- Migration Hub can track waves; it does not itself replicate servers or databases.
- Check current naming, Region/OS support, quotas, ports, and setup steps before implementation.

## 10. Official References

- [AWS Transform MGN User Guide](https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html)
- [AWS Transform MGN](https://aws.amazon.com/application-migration-service/)
- [AWS Migration Hub User Guide](https://docs.aws.amazon.com/migrationhub/latest/ug/whatishub.html)
- [AWS DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
