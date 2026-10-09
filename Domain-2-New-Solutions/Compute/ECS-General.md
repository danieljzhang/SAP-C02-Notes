# Amazon ECS — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). ECS is the AWS-native container orchestration service. Know Fargate vs EC2 launch types, task definitions, service discovery, and deployment strategies.

---

## 1. Overview

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service. It runs Docker containers on either AWS-managed infrastructure (Fargate) or EC2 instances you manage.

---

## 2. Launch Types

| Feature | Fargate | EC2 |
|---|---|---|
| Infrastructure management | AWS manages | You manage EC2 instances |
| Scaling | Task-level | Instance + task level |
| Cost model | Per vCPU/memory per second | EC2 instance cost + task overhead |
| Use case | Simplicity, variable workloads | GPU, custom AMI, cost optimization at scale |
| Spot support | Fargate Spot | EC2 Spot instances |
| Visibility | Limited OS access | Full OS access |

---

## 3. Core Concepts

### Task Definition

- Blueprint for a container (image, CPU, memory, ports, env vars, IAM role, logging)
- Versioned — new deployments create new revisions
- Supports multiple containers per task (sidecar pattern)

### Service

- Maintains desired number of running tasks
- Integrates with ALB/NLB for load balancing
- Supports rolling updates, blue/green deployments (via CodeDeploy)
- Auto Scaling based on CPU, memory, or custom CloudWatch metrics

### Cluster

- Logical grouping of tasks and services
- For EC2 launch type: also manages the EC2 instances (via ECS Container Agent)

---

## 4. IAM Roles

| Role | Purpose |
|---|---|
| Task Role | Permissions for the application running in the container (e.g., S3 access) |
| Task Execution Role | Permissions for ECS agent (pull ECR image, write CloudWatch logs, read Secrets Manager) |

Both roles are required. Task role = application permissions. Execution role = ECS infrastructure permissions.

---

## 5. Networking Modes

| Mode | Description | Use Case |
|---|---|---|
| awsvpc | Each task gets its own ENI and private IP | Fargate (required), EC2 (recommended) |
| bridge | Docker bridge network, port mapping | Legacy EC2 workloads |
| host | Container uses host network stack | High performance, no port mapping overhead |
| none | No network access | Batch jobs with no network needs |

**awsvpc** is required for Fargate and recommended for EC2 — enables security group assignment per task.

---

## 6. Service Discovery

- **AWS Cloud Map:** register tasks as DNS records; other services resolve by name
- **ECS Service Connect:** built-in service mesh with traffic routing and observability (newer, preferred)
- **ALB:** path-based or host-based routing to multiple ECS services

---

## 7. Deployment Strategies

| Strategy | Mechanism | Use Case |
|---|---|---|
| Rolling update | Replace tasks gradually | Default, simple deployments |
| Blue/Green (CodeDeploy) | Two target groups, traffic shift | Zero-downtime, instant rollback |
| External | Custom deployment controller | Advanced custom logic |

### Blue/Green with CodeDeploy

```
ALB
├── Listener Rule → Blue Target Group (current)
└── CodeDeploy shifts traffic → Green Target Group (new)
        ├── Canary: 10% → wait → 100%
        ├── Linear: 10% every 1 min
        └── All-at-once: 100% immediately
```

---

## 8. Auto Scaling

- **Target Tracking:** maintain CPU or memory at target percentage
- **Step Scaling:** scale based on CloudWatch alarm thresholds
- **Scheduled Scaling:** scale at known times
- Combine with **Fargate Spot** for cost-optimized variable workloads

---

## 9. Logging and Observability

- **awslogs driver:** send container logs to CloudWatch Logs
- **FireLens:** route logs to multiple destinations (S3, Kinesis, third-party) using Fluent Bit sidecar
- **Container Insights:** CloudWatch metrics for CPU, memory, network per task/service
- **X-Ray:** add X-Ray daemon as sidecar container for distributed tracing

---

## 10. Exam Scenarios

**Scenario 1:** A company wants to run containers without managing EC2 instances. They need automatic scaling and pay only for what they use. What is the solution?

> Use **ECS with Fargate** launch type. Fargate manages the underlying infrastructure. Configure ECS Service Auto Scaling with target tracking on CPU utilization. Use Fargate Spot for non-critical tasks to reduce costs.

**Scenario 2:** A company needs zero-downtime deployments for an ECS service with instant rollback capability. What is the approach?

> Use **ECS blue/green deployment with AWS CodeDeploy**. CodeDeploy creates a new (green) task set, shifts traffic from the ALB blue target group to the green target group, and keeps the blue environment available for rollback during the deployment window.

**Scenario 3:** A containerized application needs to access Secrets Manager to retrieve database credentials. What IAM configuration is required?

> Attach a **Task Role** to the task definition with `secretsmanager:GetSecretValue` permission. Also ensure the **Task Execution Role** has `secretsmanager:GetSecretValue` if the secret is injected as an environment variable at task launch (ECS can inject secrets directly from Secrets Manager).

**Scenario 4:** A company runs ECS on EC2 and wants to reduce costs for batch workloads that can tolerate interruption. What is the solution?

> Use an EC2 Auto Scaling group with **Spot Instances** as the ECS cluster capacity provider. Configure a mixed instances policy with multiple instance types. Use ECS capacity provider strategies to prefer Spot with On-Demand as fallback.

---

## 11. Exam Tips

- **Fargate** = no EC2 management; **EC2 launch type** = more control, better for GPU/custom AMI
- **Task Role** = app permissions; **Task Execution Role** = ECS agent permissions — both are needed
- **awsvpc** mode = security groups per task — always use for Fargate
- **ECS Service Connect** is the modern service discovery — know it over Cloud Map for new designs
- Blue/green deployments require **CodeDeploy** + **ALB** with two target groups
- **FireLens** = flexible log routing; **awslogs** = simple CloudWatch logging
- ECS integrates with **App Mesh** for advanced service mesh (verify current recommendation vs Service Connect)

---

## 12. 🎯 Architecture Decision Notes

### Choosing the compute operating model

Start from how much of the runtime the team wants to operate — this one
decision eliminates most options:

| Control needed | Fit |
|---|---|
| Full OS and runtime control, custom AMIs | EC2 (you own patching, scaling, capacity) |
| Managed app environment, some AWS resource access | Elastic Beanstalk |
| Container image or code to HTTPS, minimal platform work | App Runner |
| Container orchestration with placement/network control | ECS (EC2 capacity model) or ECS on Fargate (AWS manages hosts) |

With Fargate tasks using `awsvpc`, each task gets its own network interface
and security group. A task in a private subnet needs a route to every
dependency: ECR image pulls can use ECR interface endpoints, but image
layers, logs, secrets, and other integrations may each need their own
endpoint or an egress path. Create only the endpoints the task needs and
verify current service-specific requirements.

**Fresh example:** A small team shipping an HTTPS API from a container with
no host-management appetite picks App Runner. A platform team needing
placement control, cooperating sidecars, or custom networking picks ECS —
and chooses Fargate when host management adds no value.

---

### App Runner's operating model — when the simplest container platform wins

App Runner is the lowest-operational-overhead way to run an HTTP workload on AWS: you hand it source code (from a repo) or a container image, and it returns a running HTTPS endpoint with load balancing and request-driven horizontal scaling — no clusters, task definitions, or service discovery to manage. Think of it as Elastic Beanstalk's simpler successor for plain web apps and APIs.

Decision rules:

- **Pick App Runner** when the workload is a stateless HTTP service, the team wants zero container-orchestration work, and the app doesn't need fine-grained VPC control. Monolith-to-serverless replatforming with minimal refactoring is the textbook case.
- **Don't pick App Runner** for long-running background workers, scheduled batch processing, or event-driven pipelines — it scales on HTTP traffic, not queues or schedules. Those belong on Fargate or Batch.
- **App Runner vs Fargate:** if you need placement control, sidecars, service mesh, or private-VPC-only networking, you need ECS/Fargate; if you just need "my container, reachable over HTTPS," App Runner removes a layer of machinery.
- **App Runner vs Lambda:** Lambda is event-driven with 15-minute execution limits and per-request billing; App Runner is always-on HTTP with container packaging. A REST API backed by a containerized framework (Spring, Django) fits App Runner; a webhook handler that runs 200 ms fits Lambda.

Traps: App Runner's VPC support is partial compared to ECS — verify private-resource access (private ECR, RDS in a VPC) against current connector support before committing. And "scales down" does not mean serverless-to-zero in every configuration; model cost on sustained baseline traffic, not on the assumption of zero idle cost.

**Fresh example:** A two-person team has a Java Spring Boot service currently on EC2 behind an ALB and wants to stop patching hosts. They point App Runner at the GitHub repo, attach a custom domain with the built-in TLS certificate, and connect it to an RDS database. The hourly report job that used to run on the same EC2 box moves to Fargate as a scheduled task — because App Runner would have no trigger for it.

---

### AWS Batch — the managed scheduler for parallel batch work

AWS Batch is the answer when the workload is a large number of independent, long-running compute jobs: it takes job definitions (container images + resource requirements), queues them, provisions the right compute (EC2, Spot, or Fargate), runs them, and tears the compute down. You pay only for the underlying compute — there is no Batch charge.

Core model to internalize:

- **Jobs** are the unit of work (a container run). **Job queues** hold jobs until capacity is free and map to one or more **compute environments**.
- **Compute environments** come in two flavors: *managed* (AWS provisions and scales instances for you — the default choice) and *unmanaged* (you bring your own cluster). Managed environments can mix On-Demand and Spot.
- **Scheduling features that matter:** array jobs (submit thousands of indexed copies of one job), job dependencies (B before A finishes), and retry strategies.

Decision rules:

- **Batch over Lambda:** any job that can exceed 15 minutes, needs gigabytes of memory/disk, or requires dependency ordering and retries.
- **Batch over Fargate-alone:** when you have hundreds-to-thousands of jobs needing queueing, prioritization, and dependency graphs — Fargate runs containers, Batch *schedules* them.
- **Batch over hand-rolled SQS + Auto Scaling:** the queue-consumer pattern works but you own deduplication, scaling logic, and dependency handling. Batch gives you all three, plus Spot-aware provisioning.
- **Cost lever:** run fault-tolerant, checkpointed jobs on Spot within a managed compute environment; keep a small On-Demand allocation for jobs that can't tolerate interruption.

**Fresh example:** A genomics team runs 20,000 independent sequence-alignment jobs nightly, each taking 1–3 hours. They define one Batch job definition, submit the work as an array job to a queue backed by a managed compute environment (Spot-first, On-Demand fallback), with input data in S3 and results written back to S3. No cluster to babysit, no queue consumer fleet to scale — and the Spot mix cuts the compute bill versus their old fixed cluster.

---

### ECS Anywhere — one orchestrator for cloud and on-premises capacity

ECS Anywhere registers servers that live outside AWS (your data center,
edge sites, even a lab machine) as container instances in an ECS cluster.
Registration and the ongoing control channel run over the SSM agent, so
the cloud control plane schedules tasks onto machines it does not own.

Decide from the constraint, not the novelty:

| Situation | Verdict |
|---|---|
| Regulation or data residency keeps the workload on premises, but the team wants ECS task definitions, services, and IAM roles | Fit — one set of task definitions works in both places |
| Staged migration: run the same container on premises today, move it to Fargate or EC2 later without re-platforming | Fit — launch type becomes a deployment detail, not a rewrite |
| Pure greenfield with no on-prem footprint | No fit — use Fargate and skip the operational overhead |

Traps: ECS Anywhere is not a migration service — it does not move images,
data, or state for you. On-prem tasks never run on Fargate; the Fargate
launch type only exists inside AWS. And the cluster's scheduling brain
stays in the cloud, so the registration and SSM channel must be reliable
even when the workload itself is meant to be autonomous.

**Fresh example:** A regional bank must keep its transaction service in
its own data center for compliance but wants the same deployment pipeline
as its cloud services. It registers the on-prem hosts with ECS Anywhere,
ships identical task definitions to both, and later moves non-regulated
services to Fargate without changing the task definitions — only the
launch target changes.

### Fargate pulling ECR images from private subnets — pick the egress path deliberately

A Fargate task in a private subnet fails image pulls (`CannotPullContainer`
style errors) when it has no route to ECR. Do not reach for a public IP
on the task — that contradicts the private-subnet design. Decide:

| Requirement | Choice |
|---|---|
| Tasks need only AWS services (ECR, CloudWatch Logs, Secrets Manager) | **Interface VPC endpoints**: `ecr.api` for API calls, `ecr.dkr` for image layers, `logs` for log delivery. Cheaper and stays on the AWS network. |
| Tasks also call the public internet (third-party APIs, package registries) | **NAT gateway** in the public subnet; task route table sends 0.0.0.0/0 to the NAT. Higher cost, adds a hop. |
| Tasks sit in a public subnet | Auto-assign public IP on the task; the subnet's route to the internet gateway handles the rest. |

The classic trap: creating a **gateway** VPC endpoint for ECR. Gateway
endpoints serve only S3 and DynamoDB — ECR requires **interface**
endpoints. Route-table edits are unnecessary for interface endpoints;
they work through ENIs and security groups, so check that the endpoint
security group allows the task's HTTPS traffic.

**Fresh example:** A payment API runs on Fargate in private subnets with
no internet access allowed by policy. The team creates interface endpoints
for `ecr.api`, `ecr.dkr`, and `logs`, keeps auto-assign public IP
disabled, and verifies the task execution role can call ECR — image pulls
and log delivery work with zero public exposure. When the same API later
needs a fraud-scoring SaaS over HTTPS, they add a NAT gateway rather than
punching public IPs onto tasks.

## 13. Official References

- [ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [ECS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [ECS Blue/Green Deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html)
- [ECS Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)
