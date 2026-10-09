# AWS Auto Scaling — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). Auto Scaling is fundamental to every HA architecture. Know scaling policies, warm pools, lifecycle hooks, and multi-service scaling.

---

## 1. Overview

AWS Auto Scaling automatically adjusts capacity for EC2, ECS, DynamoDB, Aurora, and other services based on demand. It maintains availability and optimizes cost.

**Services covered by AWS Auto Scaling:**
- EC2 Auto Scaling Groups (ASG)
- ECS Service Auto Scaling
- DynamoDB table/index capacity
- Aurora read replicas
- Spot Fleet

---

## 2. EC2 Auto Scaling Group (ASG)

### Key Settings

| Setting | Description |
|---|---|
| Min capacity | Minimum number of instances always running |
| Max capacity | Maximum instances allowed |
| Desired capacity | Target number of instances |
| Launch template | Defines instance configuration (AMI, type, SG, IAM role) |
| AZ distribution | Spread instances across AZs for HA |

### Scaling Policies

| Policy | Trigger | Use Case |
|---|---|---|
| Target Tracking | Maintain a metric at a target value | CPU at 60%, ALB requests per target |
| Step Scaling | CloudWatch alarm thresholds with step adjustments | Fine-grained control |
| Simple Scaling | Single CloudWatch alarm | Legacy, not recommended |
| Scheduled Scaling | Time-based | Known traffic patterns |
| Predictive Scaling | ML-based forecast | Proactive scaling before demand |

**Target Tracking** is the recommended default — simplest to configure and most responsive.

### Cooldown Period

- Prevents rapid scale-in/scale-out oscillation
- Default: 300 seconds after a scaling activity
- Reduce for faster response; increase for stability

---

## 3. Lifecycle Hooks

Pause instance launch or termination to perform custom actions:

```
Instance Launching:
  Pending → Pending:Wait (hook fires) → Pending:Proceed → InService
                    ↓
              Custom action (install software, register with config mgmt)

Instance Terminating:
  Terminating → Terminating:Wait (hook fires) → Terminating:Proceed → Terminated
                        ↓
                  Custom action (drain connections, backup data, deregister)
```

- Hook timeout: default 1 hour (max 48 hours)
- Complete hook with `complete-lifecycle-action` (CONTINUE or ABANDON)
- Trigger: EventBridge, SNS, SQS

---

## 4. Warm Pools

Pre-initialized instances that are ready to serve traffic immediately:

```
Warm Pool (stopped or running instances)
        └── Scale-out event → instance moves from warm pool to ASG
                └── No initialization delay — already bootstrapped
```

- Reduces scale-out latency for applications with long initialization times
- Warm pool instances can be in **Stopped** (cheaper) or **Running** state
- Supports lifecycle hooks for warm pool transitions

---

## 5. Mixed Instances Policy

Combine multiple instance types and purchase options in one ASG:

```yaml
MixedInstancesPolicy:
  InstancesDistribution:
    OnDemandBaseCapacity: 2          # Always keep 2 On-Demand
    OnDemandPercentageAboveBase: 20  # 20% On-Demand, 80% Spot above base
    SpotAllocationStrategy: capacity-optimized
  LaunchTemplate:
    Overrides:
      - InstanceType: m5.large
      - InstanceType: m5a.large
      - InstanceType: m4.large
```

- **capacity-optimized** Spot strategy: picks the pool with most available capacity (fewer interruptions)
- **price-capacity-optimized**: balances price and capacity availability (recommended)

---

## 6. Instance Refresh

Rolling replacement of instances with a new launch template version:

- Define minimum healthy percentage (e.g., 90%) — keeps most instances running during refresh
- Supports checkpoints — pause at a percentage for validation
- Automatically replaces instances that don't match the current launch template

---

## 7. Health Checks

| Type | Checks |
|---|---|
| EC2 health check | Instance status checks (default) |
| ELB health check | ALB/NLB target health (recommended) |
| Custom health check | Application-level via API |

Always use **ELB health checks** with load-balanced ASGs — EC2 checks only detect hardware failures, not application failures.

---

## 8. Exam Scenarios

**Scenario 1:** A web application experiences predictable traffic spikes every weekday at 9 AM. How should Auto Scaling be configured?

> Use **Scheduled Scaling** to increase desired capacity before 9 AM. Combine with **Target Tracking** for dynamic scaling during the day. Use **Predictive Scaling** if the pattern is consistent — it proactively scales before demand arrives.

**Scenario 2:** An ASG is scaling out too aggressively, causing cost spikes. How can this be controlled?

> Reduce the scale-out step size in the scaling policy. Increase the cooldown period to prevent rapid successive scale-outs. Use **Target Tracking** instead of Step Scaling for smoother behavior. Set a reasonable max capacity limit.

**Scenario 3:** A company needs instances to register with a configuration management tool before serving traffic. How?

> Add a **Lifecycle Hook** on the `autoscaling:EC2_INSTANCE_LAUNCHING` transition. The hook pauses the instance in `Pending:Wait` state. An EventBridge rule triggers a Lambda function that runs the registration. Lambda calls `complete-lifecycle-action` with CONTINUE when done.

**Scenario 4:** A company wants to minimize cost for a stateless web tier while maintaining availability. What is the configuration?

> Use a **Mixed Instances Policy** with `OnDemandBaseCapacity: 2` (minimum On-Demand for stability) and `OnDemandPercentageAboveBase: 0` (all additional capacity from Spot). Use `price-capacity-optimized` Spot strategy with multiple instance types to reduce interruption risk.

---

## 9. Exam Tips

- **Target Tracking** is the recommended default scaling policy — simplest and most responsive
- **Predictive Scaling** proactively scales before demand — combine with reactive policies
- **Warm Pools** solve slow initialization — not the same as pre-warming (which is manual)
- **Lifecycle Hooks** are the correct answer for "do something before instance serves traffic"
- Always use **ELB health checks** for load-balanced ASGs — EC2 checks miss application failures
- **Instance Refresh** = rolling update of launch template — not the same as a deployment
- Mixed instances with **capacity-optimized** Spot = fewer interruptions; **lowest-price** = cheapest but more interruptions

---

## 10. Official References

- [EC2 Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)
- [Warm Pools](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html)
- [Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)
- [Mixed Instances Policy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html)
