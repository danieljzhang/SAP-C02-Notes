# Amazon EBS — SAP-C02 Study Notes

> Exam relevance: Domain 2 (30%). EBS is the block storage behind most EC2 workloads. At the Professional level, focus on volume type selection, snapshot mechanics, and encryption — not just "what is EBS."

---

## 1. Volume Type Selection

| Type | Backed by | Use when |
|---|---|---|
| **gp3** | SSD, baseline 3,000 IOPS + 125 MiB/s, independently scalable | Default choice — decouple IOPS/throughput from size |
| **gp2** | SSD, bursty IOPS tied to volume size | Legacy; gp3 is cheaper and more flexible for new workloads |
| **io2 / io2 Block Express** | SSD, provisioned IOPS, 99.999% durability | I/O-intensive databases needing sustained provisioned IOPS |
| **st1** | HDD, throughput-optimized | Big data, log processing — sequential access |
| **sc1** | HDD, cold | Infrequent access, lowest cost |

**Decision rule:** Start with gp3 (the current default). Move to io2 only when the workload needs provisioned IOPS beyond what gp3 offers or multi-attach. Use st1/sc1 only for throughput-oriented, non-latency-sensitive workloads — never for boot volumes or databases.

---

## 2. Snapshots — How They Actually Work

- Snapshots are **incremental**: the first captures the full volume; subsequent ones store only changed blocks.
- Snapshots live in **S3** (managed by AWS, not visible in your buckets) and are **regional** — copy across Regions/accounts for DR and sharing.
- **Fast Snapshot Restore (FSR)** pre-warms snapshots to avoid first-access latency on restored volumes. Without FSR, blocks are lazily loaded and the first reads are slow.
- You can create a volume from a snapshot **larger** than the original, but never smaller.
- **Recycle Bin for EBS snapshots** protects against accidental deletion — a soft-delete retention window, not a backup strategy by itself.

---

## 3. Encryption

- EBS encryption uses **KMS**. You can use the AWS-managed key or a customer-managed key (CMK) for control over rotation and key policy.
- Encryption is set **at volume creation** — you cannot encrypt an existing unencrypted volume in place. The migration path: snapshot → copy snapshot with encryption → create new encrypted volume.
- Encrypted snapshots share the source volume's key unless re-encrypted on copy. Cross-account snapshot sharing requires the CMK key policy to allow the target account.

---

## 4. Multi-Attach and Instance Store

- **io2 multi-attach** lets one volume attach to up to 16 Nitro instances in the same AZ — for clustered applications (e.g., active-standby database) that manage their own fencing. The application handles write coordination; EBS does not.
- **Instance store** is ephemeral NVMe physically attached to the host — highest performance, but data is lost on stop/termination/failure. Use for caches, temp files, and scratch data only. Never the sole copy of anything.

---

## 🎯 Architecture Decision Notes

### The EBS decision chain

1. **Durability need:** Instance store (ephemeral) vs EBS (persistent)? If the data must survive a stop/start, it is EBS — no exceptions.
2. **Performance profile:** General SSD (gp3) → provisioned IOPS (io2) → throughput HDD (st1/sc1). Do not pay for io2 when gp3's burstable baseline covers the workload.
3. **Snapshot strategy:** RPO determines snapshot frequency; cross-Region copy determines the DR story. FSR is for recovery-time-sensitive restores, not for routine backups.
4. **Encryption:** Decide at creation. Retrofitting encryption is a migration, not a toggle.

**Fresh example:** A database needs 20,000 sustained IOPS and multi-AZ failover within minutes. gp3 tops out below that IOPS need → io2 with provisioned IOPS. Snapshots every 6 hours to meet RPO, copied to a second Region. FSR enabled on the latest snapshot so a restore does not start cold. CMK encryption with the key policy shared to the DR account.

---

## 5. Exam Tips

- gp3 is the default answer unless the scenario demands provisioned IOPS (io2) or throughput HDD (st1).
- "Encrypt an existing volume" → snapshot → encrypted copy → new volume. There is no in-place conversion.
- Snapshot sharing across accounts requires **both** snapshot permissions **and** KMS key policy access.
- Multi-attach is io2-only, same-AZ, and the application owns write fencing.

---

## 6. Official References

- [Amazon EBS volume types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [EBS encryption](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-encryption.html)
