# Deployment Strategies and CI/CD — SAP-C02 Study Notes

> Official Domain 2 task 2.1 covers deployment strategy, change management, rollback, IaC, configuration management, and managed services. Choose a rollout pattern from business risk and recovery requirements.

## 1. Service Roles

| Service | Role | Not a substitute for |
|---|---|---|
| AWS CodePipeline | Orchestrates source, build, test, approval, and deploy stages | Build execution or application deployment engine by itself |
| AWS CodeBuild | Runs managed build/test commands in build environments | Release orchestration or production rollback policy |
| AWS CodeDeploy | Coordinates supported in-place/blue-green deployments for EC2, on-premises, Lambda, and ECS targets | General-purpose source/build pipeline |
| AWS CloudFormation / CDK | Provisions and updates infrastructure as code | Application testing or safe data migration |
| AWS Systems Manager | Configuration, automation, patching, and operational actions | Application release strategy on its own |

A typical pipeline uses version control, immutable build artifacts, automated tests, approval gates where required, infrastructure deployment, application rollout, and post-deployment validation. Use least-privilege service roles for each stage.

## 2. Rollout Pattern Selection

| Pattern | Fit | Trade-off |
|---|---|---|
| All at once | Low-risk change, short deploy window, easy rollback | Largest immediate blast radius |
| Rolling | Incremental replacement of instances/tasks | Mixed versions coexist; backward compatibility required |
| Blue/green | Keep old and new environments during cutover | Higher temporary capacity/cost; fast traffic rollback if data/schema permits |
| Canary / linear traffic shift | Validate with a small or gradually increasing traffic share | Requires health alarms, representative traffic, and automated stop/rollback criteria |
| Immutable replacement | Replace rather than mutate servers | Repeatability improves; capacity and state migration must be planned |

Deployment pattern names and availability differ by service. For example, CodeDeploy supports specific deployment configurations for supported compute targets; ECS blue/green requires the appropriate load balancer and task/service setup.

## 3. Safe Pipeline Design

```text
Commit -> build/test -> scan/artifact -> deploy infrastructure
                                          |
                          deploy to test -> health + integration checks
                                          |
                        canary / blue-green / rolling rollout
                                          |
                     alarms + business checks -> promote or rollback
```

- Build immutable, versioned artifacts and promote the same artifact through environments.
- Separate application and infrastructure changes where rollback semantics differ; use CloudFormation change sets and review destructive replacements.
- Make database changes backward-compatible (expand/migrate/contract). Rolling application versions may run concurrently.
- Define automated health checks, alarms, success thresholds, deployment windows, approvers, and rollback ownership before release.
- Store secrets in a managed secret store; do not place credentials in pipeline definitions or build logs.
- Restrict pipeline roles and artifact-bucket access. Separate deployment accounts and production approvals where governance requires.
- Use feature flags for behavior rollout when code deployment and customer exposure need independent control.

## 4. Failure Modes and Recovery

| Failure | Design response |
|---|---|
| Build passes but application is unhealthy | Integration/smoke tests and service health alarms; halt traffic promotion |
| Rollback restores code but not schema/data | Backward-compatible migrations, backups, and explicit compensation/reconciliation |
| Pipeline role is over-privileged | Per-stage least privilege, scoped resource ARNs, approval controls, CloudTrail audit |
| Blue/green cutover overloads shared database | Capacity/load test both fleets and throttle traffic shift |
| Artifact differs between test and production | Immutable artifact promotion with digest/version verification |

## 5. Original Scenario

**Scenario:** A critical API is deployed to EC2 Auto Scaling groups behind an ALB. Releases must minimize downtime and provide rapid rollback; the database is shared by old and new application versions.

**Recommended approach:** Use a pipeline to build/test once, deploy an immutable artifact, and use a supported blue/green or canary strategy with ALB health checks and CloudWatch alarms. Keep the prior fleet until application and business checks pass. Make schema changes backward-compatible before shifting traffic; define an automated rollback threshold and verify that database writes remain compatible.

**Trap:** Traffic rollback cannot undo incompatible database writes. Test stateful rollback separately from compute rollback.

## 6. Exam Tips

- CodePipeline orchestrates; CodeBuild builds/tests; CodeDeploy deploys supported targets; CloudFormation/CDK manages infrastructure.
- Blue/green enables fast traffic reversal only when state, schema, and external side effects permit it.
- Canary reduces blast radius but requires meaningful telemetry and a promotion/abort policy.
- Use managed deployment services when they meet the target/runtime constraints; verify supported deployment types and current features.

---

## 🎯 Architecture Decision Notes

### Elastic Beanstalk rollout controls

For supported Linux platforms, Beanstalk platform hooks run deployment
scripts at defined lifecycle points: `prebuild` (before platform setup),
`predeploy` (after setup, before the new version lands), `postdeploy`
(after deployment). Use hooks for deployment-time customization — not as a
substitute for an infrastructure-as-code plan.

For blue/green: run two Beanstalk environments, validate the candidate,
then swap the environment CNAMEs so the stable application name points at
the new environment. Keep the old environment for rollback. Note that a
CNAME swap is not instantaneous for every client — account for DNS caching
in the cutover plan.

---

## 🎯 Architecture Decision Notes — Legacy Awareness

### OpsWorks: know it exists, do not choose it new

AWS OpsWorks appears in older study material as a configuration-management
and lifecycle-automation service. For the exam:

- Know it well enough to **distinguish** configuration management (desired
  state on hosts) from application deployment and from infrastructure
  provisioning — the exam tests the distinction, not the service.
- For new designs, select currently supported IaC, configuration, and
  deployment tooling (CloudFormation/CDK, Systems Manager, CodeDeploy).
  Verify any OpsWorks reference against current AWS service lifecycle
  before treating it as a live recommendation.

## 7. Official References

- [AWS CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [AWS CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [AWS CodeDeploy](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
- [AWS CloudFormation change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [SAP-C02 Domain 2 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain2.html)
