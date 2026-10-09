# Amazon EC2 Advanced — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). At the professional level, focus on purchasing options, placement groups, instance types for specific workloads, and Spot integration patterns.

---

## 1. Purchasing Options

| Option | Use Case | Commitment | Discount |
|---|---|---|---|
| On-Demand | Unpredictable, short-term | None | None |
| Reserved Instances (RI) | Steady-state, predictable | 1 or 3 years | Up to 72% |
| Savings Plans | Flexible steady-state | 1 or 3 years | Up to 66% |
| Spot Instances | Fault-tolerant, flexible | None | Up to 90% |
| Dedicated Hosts | Licensing, compliance | On-demand or reserved | Varies |
| Dedicated Instances | Isolation requirement | On-demand or reserved | ~10% |
| Capacity Reservations | Guaranteed capacity in AZ | None (pay on-demand rate) | None |

### Reserved Instances vs Savings Plans

| Feature | Reserved Instances | Savings Plans |
|---|---|---|
| Flexibility | Locked to instance type/region | Flexible across instance family, region, OS |
| Types | Standard, Convertible, Scheduled | Compute, EC2 Instance, SageMaker |
| Convertible RIs | Can exchange for different type | N/A |
| Compute Savings Plans | N/A | Most flexible — covers EC2, Lambda, Fargate |

### Spot Instances

- AWS can reclaim with 2-minute warning
- Best for: batch processing, CI/CD workers, stateless web servers, HPC
- **Spot Fleet:** mix of Spot + On-Demand across instance types and AZs
- **EC2 Auto Scaling with Spot:** use mixed instances policy for resilience
- **Spot Blocks (deprecated):** verify current availability

---

## 2. Placement Groups

| Type | Behavior | Use Case |
|---|---|---|
| Cluster | Same rack, same AZ | Low latency, high throughput (HPC, ML training) |
| Spread | Different racks, different AZs | Critical instances, max fault isolation (max 7 per AZ) |
| Partition | Groups of instances on separate racks | Distributed systems (Hadoop, Kafka, Cassandra) |

- **Cluster:** highest network performance (up to 100 Gbps with enhanced networking) but single point of failure at rack level
- **Spread:** max 7 instances per AZ per group — not for large clusters
- **Partition:** up to 7 partitions per AZ, hundreds of instances per partition; each partition is isolated from others

---

## 3. Instance Types for Specific Workloads

| Family | Optimized For | Examples |
|---|---|---|
| C (Compute) | CPU-intensive: web servers, batch, ML inference | c6i, c7g |
| M (General) | Balanced: app servers, small databases | m6i, m7g |
| R (Memory) | In-memory databases, caches, big data | r6i, r7g |
| I (Storage) | High IOPS local NVMe: NoSQL, data warehousing | i3, i4i |
| G/P (GPU) | ML training, graphics rendering | g5, p4d |
| Inf (Inferentia) | ML inference (AWS custom chip) | inf2 |
| Trn (Trainium) | ML training (AWS custom chip) | trn1 |
| X (Extra memory) | SAP HANA, large in-memory databases | x2idn |
| T (Burstable) | Variable workloads with CPU credits | t3, t4g |

### Graviton (ARM-based)

- AWS-designed ARM processors (g suffix: m7g, c7g, r7g)
- Up to 40% better price/performance than x86 equivalents
- Requires ARM-compatible software/containers
- Supported by most Linux distributions and managed services

---

## 4. Enhanced Networking and ENA

- **Elastic Network Adapter (ENA):** up to 100 Gbps, lower latency, lower CPU overhead
- **Elastic Fabric Adapter (EFA):** ENA + OS-bypass for HPC/ML (MPI workloads)
- Required for cluster placement groups to achieve maximum throughput
- Enable with: `aws ec2 modify-instance-attribute --ena-support`

---

## 5. Storage Options

| Storage | Type | Use Case |
|---|---|---|
| EBS gp3 | SSD, network-attached | General purpose, boot volumes |
| EBS io2 Block Express | SSD, high IOPS | Databases requiring >64K IOPS |
| EBS st1 | HDD, throughput-optimized | Sequential reads: log processing, data warehouse |
| EBS sc1 | HDD, cold | Infrequently accessed large data |
| Instance Store | NVMe, ephemeral | Temp data, buffers, caches — lost on stop/terminate |

- **EBS Multi-Attach:** io1/io2 volumes can attach to multiple instances in same AZ (cluster-aware file systems only)
- **EBS Snapshots:** incremental, stored in S3, can copy cross-region for DR

---

## 6. Auto Scaling Integration

- Always deploy across **multiple AZs** with Auto Scaling
- Use **mixed instances policy** for Spot + On-Demand resilience
- **Warm pools:** pre-initialized instances reduce scale-out latency
- **Instance refresh:** rolling replacement of instances with new launch template
- **Lifecycle hooks:** pause instance launch/termination for custom actions (e.g., drain connections, run scripts)

---

## 7. Exam Scenarios

**Scenario 1:** A company runs a tightly coupled HPC workload requiring maximum network throughput between instances. What is the recommended configuration?

> Use a **Cluster placement group** with instances that support EFA (e.g., c5n, p4d). EFA provides OS-bypass networking for MPI workloads. Cluster placement groups place instances on the same rack for lowest latency and highest throughput.

**Scenario 2:** A company needs 10 critical EC2 instances that must survive any single hardware failure. What placement strategy?

> Use a **Spread placement group**. Each instance is placed on a separate rack with independent power and networking. Maximum fault isolation — a single hardware failure affects at most one instance.

**Scenario 3:** A company runs batch processing jobs that can be interrupted. They want to minimize cost. What purchasing option?

> Use **Spot Instances** with a Spot Fleet or Auto Scaling mixed instances policy. Configure checkpointing so jobs can resume after interruption. Use multiple instance types and AZs to reduce interruption probability.

**Scenario 4:** A company has steady-state EC2 usage for 3 years but wants flexibility to change instance types. What purchasing option?

> Use **Compute Savings Plans**. They provide up to 66% discount and apply flexibly across EC2 instance families, regions, and operating systems — more flexible than Standard Reserved Instances.

**Scenario 5:** A company needs guaranteed EC2 capacity in a specific AZ for a planned event. What is the solution?

> Create an **On-Demand Capacity Reservation** in the target AZ for the required instance type. This reserves capacity without a commitment term. Combine with Reserved Instances or Savings Plans for cost optimization.

---

## 8. Exam Tips

- **Cluster PG** = performance; **Spread PG** = availability; **Partition PG** = distributed systems
- **Spot** = cheapest but interruptible; always design for interruption with checkpointing or stateless architecture
- **Savings Plans** are more flexible than RIs — prefer Compute Savings Plans for mixed workloads
- **Dedicated Hosts** = bring-your-own-license (BYOL) for Windows Server, SQL Server, Oracle
- **Dedicated Instances** = hardware isolation but no license benefit
- **EFA** is for HPC/MPI — not just enhanced networking
- **Graviton** = best price/performance for Linux workloads — recommend it when cost optimization is a requirement
- **Warm pools** solve slow scale-out — pre-initialized instances are ready immediately

---

---

## 🎯 Architecture Decision Notes

### Designing for Spot interruption

Spot's discount is only a good deal when the workload tolerates reclamation.
Before choosing Spot, verify the workload can do all of these:

- **Checkpoint** progress so an interrupted job resumes rather than restarts.
- **Handle the 2-minute warning** — drain connections, stop accepting work,
  persist state.
- **Run across diverse capacity** — Spot Fleet or Auto Scaling mixed-instances
  policies across instance types and AZs, so one reclamation event does not
  take the whole fleet.
- **Fall back** — keep On-Demand capacity (or a second Spot pool) for the
  portion that must stay available.

For ECS tasks on Spot: watch termination notices, stop new placements on
draining instances, let tasks finish or restart elsewhere, and keep enough
alternative capacity for service availability. Savings Plans and Reserved
Instances are billing commitments for steady usage — they are not a
resilience architecture and do not substitute for the design above.

## 9. Official References

- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [EC2 Purchasing Options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
- [Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
- [Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)
- [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/)
