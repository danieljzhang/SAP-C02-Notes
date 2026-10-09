> Exam relevance: Domain 4. Container migration and modernization patterns: packaging applications, choosing orchestration, and designing for production operations.

---

## 1. When Containers Fit

Containers package an application and its dependencies into an image, improving deployment consistency and portability. They are useful for applications that need controlled runtime environments, faster repeatable releases, service decomposition, or orchestration. Containerization alone does not make an application cloud-native: state, scaling, resilience, security, and operations still need design.

---

## 2. Migration Path

```text
Source application / VM
        |
Assess dependencies, state, ports, OS features, licenses
        |
Build and scan image -> registry (Amazon ECR)
        |
Choose runtime: ECS / EKS / App Runner / other fit
        |
Load balancer + service discovery + secrets + observability
        |
Test, shift traffic gradually, retain rollback path
```

- **Rehost first:** MGN can move eligible servers as EC2 instances; containerize later as a separate modernization step.
- **Replatform:** package a compatible application with limited code change and run it on a managed container platform.
- **Refactor:** split services, externalize state, redesign health/scale behavior, and adopt cloud-native integration.
- Inventory OS dependencies, privileged access, local disk, shared files, licensing, network rules, startup order, and operational scripts before packaging.

---

## 3. Runtime Selection

| Option | Best fit | Trade-offs to evaluate |
|---|---|---|
| Amazon ECS | AWS-native container orchestration with simpler operational model for many workloads | AWS-specific task/service model; choose EC2 or Fargate capacity based on control and operations |
| Amazon EKS | Kubernetes API/ecosystem, portability, or existing Kubernetes skills/tooling are important | More Kubernetes control-plane/data-plane concepts, cluster operations, upgrades, policy, and add-ons |
| AWS Fargate | Run supported ECS/EKS tasks without managing worker instances | Less host-level control; review workload compatibility, networking, storage, resource shape, and pricing |
| Amazon EC2-backed containers | Need host/instance control, specialized hardware, or economics at steady utilization | Own host patching, capacity, placement, scaling, and security hardening |
| AWS App Runner | Simpler web-service deployment when its supported model fits | Less low-level orchestration control; verify networking and workload requirements |

Select the orchestration and compute model based on team expertise, Kubernetes requirements, workload constraints, operational ownership, and cost—not on presumed performance differences.

---

## 4. Production Design Checklist

- Store immutable, versioned images in ECR; scan images and dependencies, sign/verify artifacts where required, and prevent unreviewed tag mutation.
- Run tasks with least-privilege task/pod roles; separate image-pull identity from application permissions; use Secrets Manager or Parameter Store for secrets.
- Put services in appropriate subnets, apply security groups, encrypt traffic, and use private registries/endpoints where required.
- Keep durable state outside ephemeral container filesystems: managed databases, S3, EFS, or supported persistent storage with tested backup/restore.
- Implement health/readiness checks, graceful shutdown, signal handling, resource requests/limits, autoscaling, and disruption budgets where supported.
- Use rolling, canary, or blue/green deployment with backward-compatible APIs and database changes. Define rollback for both images and data changes.
- Centralize logs, metrics, traces, audit events, deployment history, and cluster/task events. Set retention and alerting.
- Plan multi-AZ placement, service discovery, load balancing, capacity, quotas, and dependency failure behavior.

---

## 5. Cost, Reliability, and Common Pitfalls

Container costs include compute, load balancers, storage, networking, observability, image scanning, and idle/overprovisioned capacity. Fargate removes host management, not capacity or cost planning. EC2 may be efficient at predictable utilization but transfers host operations to the team.

| Pitfall | Why it fails |
|---|---|
| Containerize without removing local state | Rescheduling/replacement loses data or breaks affinity assumptions |
| Choose EKS only for “future flexibility” | Adds Kubernetes operations without a real requirement or capable team |
| Treat image as complete security boundary | Vulnerable dependencies and excessive runtime permissions remain risks |
| Use mutable latest tags | Rollback and reproducibility become ambiguous |
| Migrate all services in one cutover | Expands blast radius and makes troubleshooting difficult |
| Ignore external dependencies | DNS, identity, licensing, shared storage, and firewall assumptions still apply |

---

## 6. Exam Scenarios

**Scenario:** A Java application runs on several long-lived VMs. Releases are infrequent, but the team wants repeatable deployments and burst scaling. The team has AWS experience but no Kubernetes operations capability.

**Recommended approach:** First externalize session state and identify host-level dependencies. Package the application, store versioned images in ECR, and run it as an ECS service behind an ALB. Compare Fargate for reduced host operations with EC2 capacity if host control/economics require it. Migrate incrementally, test autoscaling and failure behavior, and keep a rollback path.

**Why not EKS by default?** No Kubernetes requirement offsets its additional operational surface for this team.

---

## 7. Exam Tips

- Containers change packaging and scheduling; stateful design and application dependencies still matter.
- ECS is often the simpler AWS-native choice; EKS is justified by Kubernetes requirements/skills/ecosystem.
- Fargate reduces server management; EC2-backed capacity gives host control. Compare against workload constraints.
- MGN rehosts servers; it does not automatically convert a VM into a well-designed container application.

---

## 8. Official References

- [Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [AWS Prescriptive Guidance: Container migration](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-phased-approach/welcome.html)
