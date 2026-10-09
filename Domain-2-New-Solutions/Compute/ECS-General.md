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

## 12. Official References

- [ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [ECS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [ECS Blue/Green Deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html)
- [ECS Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)
