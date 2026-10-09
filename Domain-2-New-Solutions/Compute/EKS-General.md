# Amazon EKS — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). EKS is the managed Kubernetes service. Know node types, networking (VPC CNI), IAM integration (IRSA), and when to choose EKS over ECS.

---

## 1. Overview

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes control plane. AWS manages the Kubernetes API server and etcd. You manage the worker nodes (or use Fargate).

**EKS vs ECS:**

| Factor | EKS | ECS |
|---|---|---|
| Orchestrator | Kubernetes (open standard) | AWS proprietary |
| Portability | High — runs anywhere Kubernetes runs | AWS-only |
| Complexity | Higher — Kubernetes expertise required | Lower |
| Ecosystem | Large Kubernetes ecosystem | AWS-native integrations |
| Use case | Kubernetes workloads, multi-cloud, existing K8s | AWS-native, simpler container workloads |

---

## 2. Node Types

| Type | Description | Use Case |
|---|---|---|
| Managed Node Groups | AWS provisions and manages EC2 nodes | Standard workloads, automated updates |
| Self-managed nodes | You manage EC2 instances | Custom AMI, specific configurations |
| Fargate profiles | Serverless pods — no EC2 management | Variable workloads, simplicity |

### Fargate Profiles

- Define which pods run on Fargate using namespace and label selectors
- No node management, no capacity planning
- Does not support DaemonSets or privileged containers
- Each pod gets its own isolated compute environment

---

## 3. Networking — VPC CNI

EKS uses the **AWS VPC CNI plugin** by default:
- Each pod gets a real VPC IP address (from the node's ENI)
- Pods are directly routable within the VPC — no overlay network
- Security groups can be applied per pod (security groups for pods)
- Requires sufficient IP addresses in subnets — plan CIDR carefully

```
EC2 Node (m5.large)
├── Primary ENI: 10.0.1.10 (node IP)
├── Secondary ENI: 10.0.1.20, 10.0.1.21, 10.0.1.22 (pod IPs)
└── Pod A: 10.0.1.20, Pod B: 10.0.1.21, Pod C: 10.0.1.22
```

**IP exhaustion:** use `/19` or larger subnets for node groups; enable prefix delegation for more IPs per ENI.

---

## 4. IAM Integration — IRSA

**IAM Roles for Service Accounts (IRSA)** allows Kubernetes pods to assume IAM roles:

```
EKS OIDC Provider (per cluster)
        └── Kubernetes Service Account (annotated with IAM role ARN)
                └── Pod uses service account → assumes IAM role via OIDC
                        └── Gets temporary credentials for AWS API calls
```

```yaml
# Service account annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-app-role
```

IAM role trust policy:
```json
{
  "Principal": {
    "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/XXXX"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "oidc.eks.us-east-1.amazonaws.com/id/XXXX:sub": "system:serviceaccount:default:my-app"
    }
  }
}
```

---

## 5. EKS Add-ons

Managed add-ons maintained by AWS:
- **VPC CNI** — pod networking
- **CoreDNS** — cluster DNS
- **kube-proxy** — network rules
- **AWS Load Balancer Controller** — provisions ALB/NLB for Kubernetes Ingress/Service
- **EBS CSI Driver** — dynamic EBS volume provisioning
- **EFS CSI Driver** — shared EFS volumes for pods

---

## 6. Load Balancing

| Resource | Controller | AWS Service |
|---|---|---|
| Service (type: LoadBalancer) | AWS Load Balancer Controller | NLB |
| Ingress | AWS Load Balancer Controller | ALB |

- **ALB Ingress:** path-based and host-based routing, WAF integration, SSL termination
- **NLB Service:** Layer 4, static IPs, ultra-low latency

---

## 7. Cluster Autoscaler and KEDA

- **Cluster Autoscaler:** scales EC2 node groups based on pending pods
- **Karpenter:** AWS-native node provisioner — faster, more flexible than Cluster Autoscaler (preferred for new deployments)
- **KEDA:** event-driven pod autoscaling based on SQS depth, Kinesis, custom metrics

---

## 8. Security

- **EKS control plane logging:** API server, audit, authenticator, controller manager, scheduler → CloudWatch Logs
- **Network policies:** use Calico or VPC CNI network policies to restrict pod-to-pod traffic
- **Pod Security Standards:** replace deprecated PodSecurityPolicy
- **Secrets encryption:** encrypt Kubernetes secrets with KMS (envelope encryption)
- **Private cluster:** API server endpoint accessible only within VPC

---

## 9. Exam Scenarios

**Scenario 1:** A company has existing Kubernetes workloads on-premises and wants to migrate to AWS with minimal changes. What is the recommended service?

> Use **Amazon EKS**. It runs standard Kubernetes, so existing manifests, Helm charts, and tooling work without modification. ECS would require rewriting deployment configurations.

**Scenario 2:** A company wants pods to access DynamoDB without storing AWS credentials in the container. What is the solution?

> Use **IRSA**. Create an IAM role with DynamoDB permissions. Enable the OIDC provider on the EKS cluster. Annotate the Kubernetes service account with the IAM role ARN. Pods using that service account automatically get temporary credentials.

**Scenario 3:** A company wants to run Kubernetes pods without managing EC2 nodes. What is the approach?

> Use **EKS with Fargate profiles**. Define Fargate profiles that match the pod namespaces and labels. Pods matching the profile run on Fargate — no EC2 node management required.

**Scenario 4:** An EKS cluster is running out of pod IP addresses. What is the solution?

> Enable **prefix delegation** on the VPC CNI plugin. This assigns /28 prefixes to ENIs instead of individual IPs, increasing the number of available pod IPs per node significantly. Also consider using larger subnets.

---

## 10. Exam Tips

- **EKS** = Kubernetes compatibility and portability; **ECS** = AWS-native simplicity
- **IRSA** is the correct way to give pods AWS permissions — not instance profiles or environment variables
- **Fargate profiles** = no node management but no DaemonSets or privileged containers
- **Karpenter** is the modern node autoscaler — faster and more flexible than Cluster Autoscaler
- **VPC CNI** = pods get real VPC IPs — plan subnet sizing carefully
- EKS control plane is managed by AWS — you cannot SSH into the control plane nodes
- **AWS Load Balancer Controller** is required for ALB Ingress — not built into EKS by default

---

---

## 🎯 Architecture Decision Notes

### Cluster endpoint access — the exam's favorite kubectl trap

The EKS API server endpoint has three postures. Pick by who needs kubectl
and from where:

| Posture | Reachability | Use when |
|---|---|---|
| Public | Internet (optionally CIDR-restricted) | Developers/tools reach the cluster from anywhere approved |
| Private | VPC only | Cluster must not be internet-reachable |
| Private + public | Both, independently toggled | Mixed access during migration |

**Fresh mini-scenario:** Developers administer the cluster from the
corporate office connected via Direct Connect. Requirement: "the API server
must not be reachable from the internet." A public endpoint with the
office CIDR allow-listed is **wrong** — the endpoint is still
internet-reachable, and the requirement says it must not be. The answer is
the **private endpoint**, with kubectl traveling over DX/VPN.

**Trap:** "Restrict with security groups" does not make a public endpoint
private. Reachability model (public vs private) is the decision;
CIDR allow-lists are a refinement *within* the public model.

### Storage for pods — EBS CSI vs EFS CSI

| | EBS CSI driver | EFS CSI driver |
|---|---|---|
| Volume type | Block storage, one AZ | Shared file system, multi-AZ |
| Access mode | ReadWriteOnce (one node) | ReadWriteMany (many pods, many nodes) |
| Pick when | Database pods, single-writer stateful workloads | Shared config, media, multi-pod read/write |

**Fresh mini-scenario:** Three replicas of a CMS need shared access to the
same uploaded-files directory. EBS CSI fails — EBS volumes attach to one
node in one AZ. EFS CSI with a StorageClass is the answer; dynamic
provisioning creates the access point per claim.

**Trap:** "Persistent storage for pods" alone does not pick the driver —
*shared* vs *exclusive* access does. Also: EBS snapshots back up EBS
volumes; EFS needs AWS Backup or its own backup story.

### Cluster version upgrades — order matters

Kubernetes enforces version skew policy: upgrade the **control plane
first**, then nodes, then add-ons. Reversing the order (nodes before
control plane) violates skew policy and can break the cluster.

- AWS supports each minor version for ~14 months, then **extended
  support** pricing applies — the exam may frame "avoid extended support
  cost" as the upgrade driver.
- Managed node groups automate much of the node upgrade; self-managed
  nodes are your runbook.
- Verify add-on (VPC CNI, CoreDNS, kube-proxy) compatibility with the
  target version *before* upgrading — add-ons lag the control plane.

### EKS Anywhere vs EKS on Outposts — whose hardware?

| | EKS Anywhere | EKS on Outposts |
|---|---|---|
| Hardware | **Customer's** on-premises servers | **AWS** racks installed on-premises |
| Control | EKS console/CLI experience on your infra | Full AWS services locally |
| Pick when | "Kubernetes on-prem with EKS tooling, our hardware" | "AWS APIs on-prem for latency/residency" |

**Trap:** EKS Anywhere is not "EKS in the cloud" and not "AWS hardware
on-prem" — it is the EKS *software experience* on customer hardware.
"Run EKS on our existing VMware cluster" → EKS Anywhere. "Need DynamoDB
and RDS APIs in our data center" → Outposts.

## 11. Official References

- [EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/)
- [IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [VPC CNI](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html)
- [Karpenter](https://karpenter.sh/)
- [EKS Fargate](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html)
