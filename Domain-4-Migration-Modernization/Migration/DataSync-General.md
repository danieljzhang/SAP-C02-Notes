# AWS DataSync — SAP-C02 Study Notes

## 1. Purpose and Exam Relevance

AWS DataSync is a managed, accelerated service for transferring file and object data to, from, and between supported storage systems. Common patterns include moving on-premises NAS data to Amazon S3, EFS, or FSx; recurring replication; and moving cold data to an archival S3 storage class. DataSync handles transfer orchestration, integrity checks, encryption, scheduling, and monitoring.

It is a file/object data-movement service, not a block-level server replication tool, database CDC engine, or general-purpose backup catalog.

## 2. Architecture

```text
On-premises NFS / SMB / HDFS / object store
                 |
       DataSync agent (where required)
                 | encrypted transfer
                 v
       AWS DataSync service / task
                 |
        +--------+---------+
        v                  v
       S3             EFS / supported FSx
```

- **Location:** describes a source or destination storage system and access parameters.
- **Agent:** deployed in the source environment for supported self-managed storage transfers; it reads data locally and communicates securely with DataSync.
- **Task:** pairs source and destination locations and defines transfer options, filters, schedule, logging, and verification behavior.
- **Task execution:** one run of a task; monitor status, bytes/files transferred, errors, and duration.
- **AWS-to-AWS:** some supported transfers do not require a customer-managed agent. Verify agent requirements for the exact location pair.

## 3. Migration and Replication Patterns

| Requirement | Design |
|---|---|
| One-time NAS migration | Seed data with DataSync, validate task results, then perform a final delta run during cutover |
| Recurring file replication | Schedule executions, define ownership/metadata behavior, monitor failures, and document conflict/write policy |
| Archive cold file data | Copy to an appropriate S3 storage class; validate retrieval latency, minimum storage duration, and lifecycle policy economics |
| Data lake ingest | Transfer to S3 landing prefixes, then use event-driven validation/scanning/cataloging before consumers access data |
| Hybrid processing | Use DataSync for staged transfers between on-premises storage and AWS compute/storage; plan bandwidth and avoid coupling job completion to user-facing latency |

For cutover, stop or constrain source writes, run the final delta, verify the destination and permissions, update clients, and keep the source read-only until acceptance. A recurring copy task needs a clear authority/conflict model; it is not automatically a bidirectional file synchronization system.

## 4. Network, Security, and Governance

- Place agents close to source storage and ensure reachability to the source protocols, DataSync service endpoints, and target location.
- Plan Direct Connect/VPN capacity, routes, DNS, firewall rules, and data-transfer costs. VPC endpoints/private connectivity may be available for supported paths; check the exact pattern.
- DataSync encrypts transfers and validates data integrity. Use IAM roles for AWS location access and scope S3 bucket policies, KMS key policies, EFS access points, and FSx permissions to the task.
- For cross-account destinations, explicitly configure role trust, bucket/file-system policy, and KMS permissions. Keep transfer operators separate from destination data administrators.
- Restrict task logs and reports because paths, object names, and error details can be sensitive. Send operational logs to CloudWatch Logs with suitable retention.
- Validate metadata, ownership, ACL/permission mapping, timestamps, symbolic links, and special-file handling for the source/destination pair. Not every file-system attribute maps identically to object storage.

## 5. Performance, Reliability, and Cost

- Throughput depends on the source system, agent resources, network, task options, file sizes/count, destination limits, and concurrent workload. Many small files can behave differently from a few large files.
- Size bandwidth and schedule windows using representative tests; avoid saturating production WAN or source storage.
- Use filters to control scope and reduce unnecessary transfers. Avoid overlapping tasks that compete for the same files or destination prefixes.
- Monitor task execution status and errors; design retries and an operator process for files that fail consistently.
- Cost can include DataSync usage, agent/compute where applicable, network transfer, destination storage/requests, logs, and retrieval charges. Compare archival total cost of ownership rather than storage price alone.
- Confirm current task/location limits, supported storage classes and protocols, endpoint requirements, and pricing in the AWS guide.

## 6. Common Failure Modes

| Symptom | Checks |
|---|---|
| Agent offline or location test fails | Agent activation/health, source credentials, DNS, routes, firewall, protocol ports, and source export/share configuration |
| Low throughput | WAN capacity/latency, source disk/network, agent sizing, file-size distribution, task options, and destination throttling |
| Permission errors | Source identity, NFS export/SMB credentials, IAM role, bucket/file-system policy, KMS key policy, and ownership mapping |
| Files differ or are skipped | Include/exclude filters, timestamp/metadata options, verification mode, source changes during transfer, and unsupported file attributes |
| Unexpected cost | Repeated full scans, overlapping schedules, S3 requests, storage class/retrieval costs, logs, and data-transfer path |

## 7. When Another Service Fits Better

- **MGN:** continuous block-level replication of server disks to EC2.
- **DMS:** database full load and CDC between supported database engines.
- **AWS Transfer Family:** managed partner-facing SFTP/FTPS/FTP/AS2 or browser-based file interfaces to S3/EFS, not bulk NAS synchronization.
- **AWS Storage Gateway:** ongoing hybrid access to AWS-backed storage through local protocols, rather than a planned bulk copy task.
- **AWS Snowball Edge:** existing eligible customers may use supported offline data-transfer workflows, but AWS documentation states Snowball Edge is unavailable to new customers; new designs should check current Data Transfer Terminal/partner options.

## 8. Original Scenario

**Scenario:** A media company needs to move 700 TB of video files from an on-premises NFS system to S3 over a private WAN. New files continue to arrive during the migration; production cannot lose files.

**Recommended approach:** Estimate transfer time from measured usable throughput, deploy and test DataSync against the NFS export and S3 destination, use a repeatable task for the initial bulk transfer and deltas, and monitor execution/error reports. At final cutover, freeze or queue writes, run a final delta, validate file counts/integrity and permissions, then redirect ingest. Keep the old share available read-only until sign-off.

**Why not DMS or MGN?** The payload is a file dataset, not a relational database or server image.

## 9. Exam Tips

- DataSync = managed online file/object transfer with task-based scheduling and verification.
- Snow devices solve offline transfer constraints, but check current availability and alternatives; do not recommend Snowball Edge to a new customer without validating eligibility.
- Transfer Family preserves file-transfer protocols for users/partners; DataSync moves datasets between storage locations.
- Decide whether the goal is migration, recurring replication, archival, or ongoing hybrid access before choosing the service.

---

## 🎯 Architecture Decision Notes

### Storage Gateway — pick the gateway type by what the data is for, not where it lives

Storage Gateway is ongoing hybrid storage (local protocols, AWS-backed), not a one-time copy tool — that's DataSync's job. Three gateway types, three different answers:

- **File Gateway** (NFS/SMB → S3): for active file workloads. Files written locally land in S3 in near-real time, which makes the data immediately usable by AWS services. This is the gateway you want when on-premises files need *processing* in AWS — analytics, ML, media pipelines.
- **Tape Gateway** (virtual tapes → S3 Glacier): for backup/archive replacing physical tape libraries. Retrieval requires a Glacier restore (minutes to hours) — it is *not* readable storage for applications.
- **Volume Gateway** (iSCSI block): for on-premises block volumes with cloud-backed snapshots. **Cached volumes** keep hot data local with the full dataset in AWS (up to ~1 PB per volume); **stored volumes** keep the primary copy local (up to ~512 TB) with async backup to AWS. iSCSI initiator connections are authenticated with **CHAP** (per-target credentials, replay-attack protection).

The exam's favorite trap: a scenario archives video to a Tape Gateway, then asks how to run real-time analysis on it. The answer is *not* to read from the Tape Gateway — it's to add a File Gateway so new files land in S3 where services like Rekognition can consume them directly. Tape is for retention; File is for access.

Mini-example: a retailer archives surveillance footage via Tape Gateway, then wants face detection on new footage. Keep the Tape Gateway for the archive; deploy a File Gateway on the camera network writing to S3, and trigger Rekognition from S3 events. Trying to "read the tapes faster" fights the architecture.

## 10. Official References

- [AWS DataSync User Guide](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html)
- [DataSync locations](https://docs.aws.amazon.com/datasync/latest/userguide/working-with-locations.html)
- [DataSync task options](https://docs.aws.amazon.com/datasync/latest/userguide/configure-metadata.html)
- [AWS DataSync pricing](https://aws.amazon.com/datasync/pricing/)
- [AWS Data Transfer Terminal](https://aws.amazon.com/data-transfer-terminal/)
