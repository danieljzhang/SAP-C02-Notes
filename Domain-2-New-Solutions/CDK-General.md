# AWS CDK — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). CDK lets you define infrastructure in familiar programming languages. Know constructs, stacks, apps, and how CDK relates to CloudFormation.

---

## 1. Overview

AWS Cloud Development Kit (CDK) is an open-source framework for defining cloud infrastructure using familiar programming languages (TypeScript, Python, Java, C#, Go). CDK synthesizes to CloudFormation templates.

**CDK vs CloudFormation:**

| Feature | CDK | CloudFormation |
|---|---|---|
| Language | TypeScript, Python, Java, C#, Go | JSON / YAML |
| Abstraction | High-level constructs | Low-level resource definitions |
| Reuse | Libraries, npm/PyPI packages | Nested stacks, modules |
| IDE support | Full (autocomplete, type checking) | Limited |
| Output | Synthesizes to CloudFormation | Native |
| Learning curve | Requires programming knowledge | Lower for simple templates |

---

## 2. Core Concepts

### App

- Root of the CDK application
- Contains one or more stacks

### Stack

- Unit of deployment — maps to a CloudFormation stack
- Contains constructs (resources)
- Can be deployed to a specific account and region

### Constructs

Three levels of abstraction:

| Level | Description | Example |
|---|---|---|
| L1 (Cfn*) | Direct CloudFormation resource mapping | `CfnBucket` |
| L2 | Higher-level with sensible defaults | `Bucket` |
| L3 (Patterns) | Complete solutions (multiple resources) | `ApplicationLoadBalancedFargateService` |

```typescript
// L2 construct — sensible defaults, type-safe
const bucket = new s3.Bucket(this, 'MyBucket', {
  versioned: true,
  encryption: s3.BucketEncryption.KMS_MANAGED,
  removalPolicy: cdk.RemovalPolicy.RETAIN,
});

// L3 pattern — entire ALB + Fargate service in one construct
const service = new ecs_patterns.ApplicationLoadBalancedFargateService(this, 'Service', {
  cluster,
  memoryLimitMiB: 1024,
  cpu: 512,
  taskImageOptions: { image: ecs.ContainerImage.fromRegistry('nginx') },
});
```

---

## 3. CDK Workflow

```bash
# Initialize a new CDK app
cdk init app --language typescript

# Synthesize CloudFormation template
cdk synth

# Compare deployed stack with current state
cdk diff

# Deploy stack
cdk deploy

# Destroy stack
cdk destroy
```

---

## 4. CDK Constructs Library

- AWS Constructs Library: L2 and L3 constructs for all AWS services
- **Construct Hub:** community and third-party constructs
- Share constructs as npm/PyPI packages across teams

---

## 5. CDK Pipelines

Self-mutating CI/CD pipeline for CDK applications:

```
Source (CodeCommit/GitHub)
  └── CDK Pipeline (CodePipeline)
        ├── Synth stage (cdk synth)
        ├── Self-mutation (update pipeline itself)
        └── Deploy stages (dev → staging → prod)
```

- Pipeline updates itself when pipeline code changes
- Supports multiple stages with approval gates
- Cross-account and cross-region deployments

---

## 6. Exam Scenarios

**Scenario 1:** A development team wants to define AWS infrastructure using Python with IDE autocomplete and type checking, and reuse infrastructure patterns across multiple projects. What is the solution?

> Use **AWS CDK** with Python. Define infrastructure using L2/L3 constructs. Package reusable patterns as Python libraries published to a private PyPI repository. CDK synthesizes to CloudFormation for deployment.

**Scenario 2:** A company wants a CI/CD pipeline that automatically deploys CDK infrastructure changes through dev, staging, and prod environments with manual approval before prod. What is the solution?

> Use **CDK Pipelines**. Define a self-mutating pipeline with stages for each environment. Add a manual approval step before the prod stage. The pipeline updates itself when pipeline code changes, and deploys application stacks when application code changes.

---

## 7. Exam Tips

- CDK **synthesizes to CloudFormation** — all CloudFormation features apply
- **L3 constructs** (patterns) are the most powerful — entire architectures in a few lines
- CDK Pipelines = self-mutating — the pipeline updates itself when you change pipeline code
- CDK is not a replacement for CloudFormation — it generates CloudFormation templates
- Use **cdk diff** before deploying to preview changes (equivalent to CloudFormation change sets)
- CDK apps can deploy to **multiple accounts and regions** in a single `cdk deploy`

---

## 8. Official References

- [CDK Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/)
- [CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/)
- [CDK Pipelines](https://docs.aws.amazon.com/cdk/v2/guide/cdk-pipeline.html)
- [Construct Hub](https://constructs.dev/)
