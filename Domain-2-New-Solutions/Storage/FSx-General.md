# Amazon FSx — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). FSx covers four file system types. Know which to use for Windows, HPC, NetApp, and OpenZFS workloads.

---

## 1. FSx File System Types

| Type | Protocol | Use Case |
|---|---|---|
| FSx for Windows File Server | SMB | Windows applications, Active Directory, home directories |
| FSx for Lustre | Lustre | HPC, ML training, financial modeling, media processing |
| FSx for NetApp ONTAP | NFS, SMB, iSCSI | Enterprise storage, multi-protocol, data management |
| FSx for OpenZFS | NFS | Linux workloads needing ZFS features, migration from on-prem ZFS |

---

## 2. FSx for Windows File Server

- Fully managed Windows file system built on Windows Server
- **SMB protocol** — compatible with all Windows applications
- **Active Directory integration** — user authentication via AWS Managed AD or self-managed AD
- **DFS Namespaces:** aggregate multiple file shares under a single namespace
- **Multi-AZ:** active-standby configuration for HA; automatic failover
- **Single-AZ:** lower cost, no automatic failover
- Supports: shadow copies (VSS), data deduplication, encryption at rest (KMS)

### Use Cases

- Lift-and-shift Windows file servers
- Home directories for Windows users
- SharePoint, SQL Server (user databases), .NET applications

---

## 3. FSx for Lustre

- High-performance parallel file system for compute-intensive workloads
- Sub-millisecond latency, hundreds of GB/s throughput, millions of IOPS
- **S3 integration:** link to S3 bucket; lazy-load data on first access; write results back to S3
- **Deployment types:**
  - **Scratch:** temporary, no replication, highest performance — for short-term processing
  - **Persistent:** replicated within AZ, long-term storage — for ongoing workloads

### Use Cases

- ML training (SageMaker integration)
- HPC simulations
- Media rendering and transcoding
- Financial risk modeling
- Genomics processing

---

## 4. FSx for NetApp ONTAP

- Fully managed NetApp ONTAP file system
- Supports **NFS, SMB, and iSCSI** simultaneously (multi-protocol)
- **SnapMirror:** replicate to on-premises NetApp or another FSx ONTAP
- **FlexClone:** instant zero-cost clones of volumes (dev/test environments)
- **Tiering:** automatically tier cold data to S3
- Multi-AZ and Single-AZ deployment options

### Use Cases

- Lift-and-shift NetApp workloads
- Multi-protocol shared storage
- Dev/test with instant clones
- Hybrid cloud with on-premises NetApp replication

---

## 5. FSx for OpenZFS

- Fully managed OpenZFS file system
- **NFS protocol** (v3, v4, v4.1, v4.2)
- ZFS features: snapshots, clones, compression, data integrity checksums
- Up to 1 million IOPS, sub-millisecond latency
- Single-AZ only

### Use Cases

- Migrate on-premises ZFS workloads to AWS
- Linux workloads needing ZFS data management features
- Dev/test with instant snapshots and clones

---

## 6. Comparison Table

| Feature | Windows FS | Lustre | ONTAP | OpenZFS |
|---|---|---|---|---|
| Protocol | SMB | Lustre | NFS/SMB/iSCSI | NFS |
| OS | Windows | Linux | Both | Linux |
| AD integration | Yes | No | Yes | No |
| S3 integration | No | Yes | No | No |
| Multi-AZ | Yes | No | Yes | No |
| Multi-protocol | No | No | Yes | No |
| HPC performance | No | Yes | No | No |

---

## 7. Exam Scenarios

**Scenario 1:** A company is migrating Windows file servers to AWS. Users authenticate with Active Directory. What is the solution?

> Use **FSx for Windows File Server** with Active Directory integration. Join the file system to the existing AD domain (AWS Managed AD or AD Connector to on-premises AD). Users access shares using their existing AD credentials via SMB.

**Scenario 2:** A machine learning team needs a high-performance file system for training data stored in S3. What is the solution?

> Use **FSx for Lustre** linked to the S3 bucket. Data is lazy-loaded from S3 on first access. Training jobs read data at sub-millisecond latency with hundreds of GB/s throughput. Results can be exported back to S3.

**Scenario 3:** A company has on-premises NetApp storage and wants to replicate to AWS for DR while keeping the same protocols. What is the solution?

> Use **FSx for NetApp ONTAP**. Use **SnapMirror** to replicate from on-premises NetApp to FSx ONTAP. The same NFS/SMB/iSCSI protocols work on both sides. Failover to FSx ONTAP in a DR event.

---

## 8. Exam Tips

- **Windows workloads + AD** → FSx for Windows File Server
- **HPC + S3 integration** → FSx for Lustre
- **Multi-protocol + NetApp migration** → FSx for NetApp ONTAP
- **ZFS migration + Linux** → FSx for OpenZFS
- FSx for Lustre **Scratch** = temporary/no replication; **Persistent** = long-term/replicated
- Only **FSx for Windows** and **FSx for ONTAP** support Multi-AZ
- **EFS** = Linux NFS, auto-scaling; **FSx for Windows** = Windows SMB, AD integration

---

---

## 🎯 Architecture Decision Notes

### Choosing the FSx variant

| Variant | When |
|---|---|
| FSx for Windows File Server | SMB shares, Active Directory integration, Windows workloads |
| FSx for Lustre | HPC, ML training — S3-backed, massive parallel throughput |
| FSx for OpenZFS | NFS with ZFS features (snapshots, cloning, compression) |
| FSx for NetApp ONTAP | ONTAP features, multi-protocol (NFS/SMB/iSCSI) |

Match the variant to the protocol and the feature the workload actually
needs — "file storage" alone does not decide it. For a lift-and-shift
Windows file server, FSx for Windows with AD integration is usually the
least-change answer.

## 9. Official References

- [FSx for Windows](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/)
- [FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/)
- [FSx for NetApp ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/)
- [FSx for OpenZFS](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/)
