# AWS IAM — SAP-C02 Study Notes

> Exam relevance: Domains 1, 2, 3. IAM is the security foundation of every AWS architecture. At the professional level, focus on cross-account patterns, federation, permission boundaries, and policy evaluation.

---

## 1. Overview

AWS Identity and Access Management (IAM) controls who can do what on which AWS resources. At the SAP-C02 level, the focus is on:
- Cross-account role assumption
- Identity federation (SAML, OIDC, web identity)
- Permission boundaries and policy evaluation order
- Least-privilege design at scale
- Integration with Organizations, IAM Identity Center, and resource-based policies

---

## 2. Policy Types and Evaluation Order

| Policy Type | Attached To | Purpose |
|---|---|---|
| Identity-based policy | IAM user/role/group | Grant permissions to a principal |
| Resource-based policy | AWS resource (S3, KMS, etc.) | Grant access to a resource from any principal |
| Permission boundary | IAM user/role | Maximum permissions a principal can have |
| SCP | Organizations account/OU | Maximum permissions for all principals in an account |
| Session policy | AssumeRole call | Restrict permissions for a specific session |
| RCP | Organizations account/OU | Restrict what resources can accept |

### Evaluation Logic (simplified)

```
1. Explicit DENY anywhere → DENY (final)
2. SCP allows the action? No → DENY
3. RCP allows the action? No → DENY
4. Resource-based policy allows? → ALLOW (for cross-account, both identity + resource policy needed)
5. Permission boundary allows? No → DENY
6. Session policy allows? No → DENY
7. Identity-based policy allows? Yes → ALLOW
```

---

## 3. Cross-Account Role Assumption

The standard pattern for cross-account access:

```
Account A (trusting)          Account B (trusted)
IAM Role: cross-account-role  IAM User/Role: deployer
  Trust policy: allows         calls sts:AssumeRole
    Account B to assume         → gets temporary credentials
  Permission policy: S3:*       → accesses Account A resources
```

### Trust Policy (in Account A)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::ACCOUNT_B_ID:role/deployer"},
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {"sts:ExternalId": "unique-external-id"}
    }
  }]
}
```

### ExternalId

- Used to prevent the **confused deputy problem**
- Required when a third party (e.g., SaaS vendor) assumes a role in your account
- The third party provides the ExternalId; you set it in the trust policy condition

---

## 4. Identity Federation

### SAML 2.0 Federation

- Integrates with corporate IdPs (Active Directory, Okta, Ping)
- Users authenticate with IdP → IdP issues SAML assertion → AWS STS exchanges for temporary credentials
- Use `sts:AssumeRoleWithSAML`
- Supports console access via SSO endpoint

```
User → Corporate IdP (AD/Okta)
         └── SAML Assertion
                └── AWS STS AssumeRoleWithSAML
                       └── Temporary credentials → AWS Console/API
```

### OIDC / Web Identity Federation

- For applications (mobile, web) using Google, Amazon, Facebook, or any OIDC provider
- Use `sts:AssumeRoleWithWebIdentity`
- For EC2/ECS/Lambda: use IAM roles directly (instance profile, task role, execution role)

### IAM Identity Center (preferred for workforce)

- Replaces manual SAML federation for human users
- Centrally manages access across all Organization accounts
- See [IAM Identity Center](./iam/iam-identitycenter-general.md)

---

## 5. Permission Boundaries

A permission boundary sets the **maximum permissions** an IAM entity can have. It does not grant permissions on its own.

**Use case:** Allow developers to create IAM roles for their applications, but prevent them from creating roles with more permissions than they have.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:*", "dynamodb:*", "cloudwatch:*"],
    "Resource": "*"
  }]
}
```

If this is set as a permission boundary on a role, the role can only perform S3, DynamoDB, and CloudWatch actions — even if the identity policy grants more.

**Effective permissions = identity policy ∩ permission boundary**

---

## 6. IAM Roles for AWS Services

| Service | Role Type | Purpose |
|---|---|---|
| EC2 | Instance profile | Grants EC2 instance access to AWS services |
| Lambda | Execution role | Grants Lambda function access to AWS services |
| ECS | Task role | Grants container access to AWS services |
| ECS | Task execution role | Grants ECS agent access to ECR, CloudWatch |
| CodePipeline | Service role | Grants pipeline access to source, build, deploy |
| CloudFormation | Stack role | Grants CloudFormation access to create resources |

---

## 7. IAM Best Practices at Scale

- Use **IAM roles** everywhere — never long-term access keys for applications
- Use **IAM Identity Center** for human access — not individual IAM users
- Apply **permission boundaries** when delegating IAM management to developers
- Use **SCPs** for organization-wide guardrails
- Enable **IAM Access Analyzer** to identify unintended external access
- Use **AWS Organizations** tag policies to enforce resource tagging
- Rotate access keys regularly; prefer roles over keys
- Enable **MFA** for all human IAM users (or enforce via SCP)

---

## 8. IAM Access Analyzer

- Identifies resources shared with external principals (outside your account or organization)
- Analyzes: S3 buckets, IAM roles, KMS keys, Lambda functions, SQS queues, Secrets Manager secrets
- Can be scoped to the **organization** (findings only for truly external access)
- Use **policy validation** to check policies for errors and best practice violations
- Use **policy generation** to create least-privilege policies from CloudTrail activity

---

## 9. Exam Scenarios

**Scenario 1:** A Lambda function in Account A needs to read from an S3 bucket in Account B. What is required?

> Two things: (1) The Lambda execution role in Account A must have `s3:GetObject` permission for the bucket ARN in Account B. (2) The S3 bucket policy in Account B must allow the Lambda execution role ARN from Account A. Both identity-based and resource-based policies are required for cross-account access.

**Scenario 2:** A company wants developers to be able to create IAM roles for their Lambda functions, but not create roles with admin access. What is the solution?

> Attach a permission boundary to the developer's IAM user that limits the maximum permissions to the services they need. Require developers to attach this same permission boundary to any roles they create (enforce via SCP or IAM condition `iam:PermissionsBoundary`).

**Scenario 3:** A third-party SaaS vendor needs access to an S3 bucket in your account. How do you prevent the confused deputy problem?

> Create an IAM role with a trust policy that requires the vendor's AWS account ID and a unique ExternalId. Share the ExternalId with the vendor through a secure channel. The vendor must provide the ExternalId when calling AssumeRole.

**Scenario 4:** A company uses Active Directory for authentication. They want AD users to access the AWS console without creating IAM users. What is the solution?

> Configure SAML 2.0 federation between AD FS (or another SAML IdP) and AWS IAM. Create IAM roles with trust policies for the SAML provider. Users authenticate with AD, receive a SAML assertion, and are redirected to the AWS console via the SSO endpoint. Alternatively, use IAM Identity Center with AD as the identity source.

---

## 10. Exam Tips

- Cross-account access requires **both** identity policy (in the calling account) **and** trust policy (in the target account)
- Resource-based policies can grant cross-account access **without** AssumeRole for some services (S3, KMS, SQS)
- Permission boundaries **do not grant** — they only restrict the ceiling
- SCPs restrict the account ceiling; permission boundaries restrict the principal ceiling — they stack
- `sts:AssumeRole` requires the caller to have explicit allow in their identity policy
- IAM Access Analyzer uses **zone of trust** — set it to the organization for best coverage
- For EC2: use **instance profiles** (a container for an IAM role) — not access keys on the instance

---

## 11. 🎯 Architecture Decision Notes

### Workforce vs customer identity — decide first

Authentication (proving who you are) and authorization (what you may do) are
separate decisions, and workforce access vs customer sign-in are different
problems. Pick the wrong category and every subsequent choice is wrong:

| Need | Answer |
|---|---|
| Employees accessing AWS accounts | IAM Identity Center, optionally federated with a corporate IdP |
| Application end-users signing in | Cognito user pools (issues tokens the app accepts) |
| App calling AWS APIs directly | Cognito identity pools (exchanges IdP tokens for temporary AWS credentials) |

Choose the protocol both the identity provider and the client support: SAML
is common for enterprise browser SSO; OIDC/OAuth flows fit web and mobile
apps. Never assume a social provider speaks SAML.

For workloads, prefer temporary role credentials over long-lived access keys
— an EC2 instance profile puts a role on the instance without embedding
anything in the application.

### Reading an authorization decision

Treat each policy type as answering a different question, not as
interchangeable grants. When access is unexpectedly denied, walk the full
chain: identity policy → permission boundary → session policy → SCP → RCP →
resource-based policy → role trust policy → VPC endpoint policy. An explicit
deny at any layer is final; SCPs and RCPs set ceilings and never grant.

### Access Analyzer subtleties

- The external-access analyzer must be configured **in each Region** holding
  covered resources; findings do not automatically span Regions.
- Policy generation (suggesting least-privilege policies from CloudTrail
  activity) is a separate feature from external-access analysis.
- Analyzer output is a finding, not a fix — a human still validates the
  intended principal and actions against least privilege.

### Service role vs service-linked role — who owns the role decides the behavior

Both let an AWS service act on your behalf, but ownership and control
differ completely:

| | Service role | Service-linked role |
|---|---|---|
| Created by | You, manually | AWS, automatically when you enable the feature |
| Permissions | You define the policy (custom, least privilege) | AWS defines a fixed policy you cannot freely edit |
| Scope | Whatever your policy allows — often across your resources | Tightly bound to one service's documented needs |
| Lifecycle | You manage and delete it | Follows the service — deleting the feature/related resources cleans it up; you cannot delete it while the service still needs it |

Decide: when a service needs **custom access to your resources**
(an automation document that reboots instances, writes snapshots, and
logs to S3), create a **service role** with exactly those permissions.
When the service needs to manage **its own internal machinery** (the
service emitting its operational data to CloudWatch, managing ENIs for
your tasks), accept the **service-linked role** AWS provisions.

Exam traps: you cannot "tighten" a service-linked role's policy the way
you would your own — if its permissions look broad, the answer is usually
to constrain what the service can touch elsewhere, not to edit the role.
And if a question asks why an SCP isn't restricting something, check
whether the actor is a service-linked role — SCPs do not apply to them.

**Fresh example:** A team automates patching with Systems Manager. They
create a service role for the automation document granting only
`ec2:RebootInstances`, `ec2:CreateSnapshot`, log writing, and S3 puts —
nothing more. They leave the Systems Manager service-linked role alone,
knowing AWS manages it and that their SCPs won't touch it either.

## 12. Official References

- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [Cross-Account Role Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)
- [Permission Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
