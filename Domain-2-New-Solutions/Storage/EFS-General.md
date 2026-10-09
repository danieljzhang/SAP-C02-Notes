# Amazon EFS — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). EFS is the shared POSIX file system for Linux workloads. Know performance modes, throughput modes, and when to use EFS vs EBS vs FSx.

---

## 1. Overview

Amazon Elastic File System (EFS) is a fully managed, scalable NFS file system for Linux workloads. Multiple EC2 instances, ECS tasks, Lambda functions, and EKS pods can mount the same EFS file system simultaneously.

---

## 2. Performance Modes

| Mode | Use Case | Latency |
|---|---|---|
| General Purpose (default) | Web serving, CMS, home directories | Low (sub-ms) |
| Max I/O | Big data, media processing, parallel workloads | Higher latency, higher aggregate throughput |

Set at creation time — cannot change after creation.

---

## 3. Throughput Modes

| Mode | Behavior | Use Case |
|---|---|---|
| Elastic (recommended) | Automatically scales throughput up/down | Variable, unpredictable workloads |
| Bursting | Throughput scales with storage size; burst credits | Workloads that burst occasionally |
| Provisioned | Fixed throughput regardless of storage size | Consistently high throughput needs |

**Elastic throughput** is the recommended default — no capacity planning needed.

---

## 4. Storage Classes

| Class | Use Case |
|---|---|
| EFS Standard | Frequently accessed files |
| EFS Standard-IA | Infrequently accessed files (lower cost, retrieval fee) |
| EFS One Zone | Single AZ, frequently accessed |
| EFS One Zone-IA | Single AZ, infrequently accessed |

Use **EFS Lifecycle Management** to automatically move files to IA after 7, 14, 30, 60, or 90 days of inactivity.

---

## 5. Access and Security

- **Mount targets:** one per AZ in the VPC — mount via DNS name
- **Security groups:** control NFS access (port 2049) to mount targets
- **IAM authorization:** control who can mount and what operations they can perform
- **EFS Access Points:** application-specific entry points with enforced user identity and root directory
- **Encryption:** at rest (KMS) and in transit (TLS) — enable both for compliance

---

## 6. Replication

- EFS Replication: replicate to another region or AZ for DR
- RPO: minutes; RTO: depends on failover process
- Replicated file system is read-only until failover

---

## 7. EFS vs EBS vs FSx

| Feature | EFS | EBS | FSx |
|---|---|---|---|
| Protocol | NFS | Block | SMB, Lustre, NFS, iSCSI |
| Shared access | Yes (multi-instance) | No (single instance, except io2 Multi-Attach) | Yes (SMB, Lustre) |
| OS | Linux only | Linux and Windows | Windows (FSx for Windows), Linux (Lustre) |
| Scaling | Automatic | Manual (resize) | Depends on type |
| Use case | Shared Linux storage | Single-instance block storage | Windows shares, HPC |

---

## 8. Exam Scenarios

**Scenario 1:** Multiple EC2 instances running a web application need to share uploaded files. What is the storage solution?

> Use **Amazon EFS**. Mount the EFS file system on all EC2 instances. Files written by one instance are immediately visible to all others. Use General Purpose performance mode and Elastic throughput for a web workload.

**Scenario 2:** A company needs shared storage for a Linux-based HPC cluster with high aggregate throughput. What is the configuration?

> Use **EFS with Max I/O performance mode** and **Provisioned or Elastic throughput**. Max I/O is optimized for parallel access from many instances. Alternatively, consider **FSx for Lustre** for HPC workloads requiring even higher performance.

---

## 9. Exam Tips

- EFS = **Linux only**, **NFS**, **shared multi-instance** — not for Windows
- **Elastic throughput** is the recommended default — no capacity planning
- **Max I/O** = higher throughput but higher latency — for parallel HPC, not web serving
- EFS Access Points = enforce user identity and root directory per application
- For Windows shared storage → **FSx for Windows File Server**
- For HPC → **FSx for Lustre**

---

## 10. Official References

- [EFS User Guide](https://docs.aws.amazon.com/efs/latest/ug/)
- [EFS Performance](https://docs.aws.amazon.com/efs/latest/ug/performance.html)
- [EFS Storage Classes](https://docs.aws.amazon.com/efs/latest/ug/storage-classes.html)
