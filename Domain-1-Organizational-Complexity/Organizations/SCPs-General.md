# Service Control Policies (SCPs) — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). SCPs appear in almost every multi-account governance scenario. Know deny strategies, inheritance, and interaction with IAM policies cold.

---

## 1. Overview

SCPs are a type of Organizations policy that set the maximum permissions available to IAM principals (users, roles) in member accounts. They do not grant permissions — they define the ceiling.

**Effective permissions = IAM policy permissions ∩ SCP permissions**

If an action is denied by an SCP, no IAM policy can allow it. If an action is not in the SCP allow list (allow-list strategy), it cannot be performed regardless of IAM.

---

## 2. How SCPs Work

### Evaluation Logic

```
Request from principal in member account
        │
        ▼
Is there an explicit DENY in any SCP in the hierarchy?
        │ YES → DENY (final)
        │ NO
        ▼
Is the action within the SCP ALLOW scope?
        │ NO → DENY (final)
        │ YES
        ▼
Does the IAM policy allow the action?
        │ NO → DENY
        │ YES → ALLOW
```

### Key Rules

- SCPs apply to **all principals** in the account (IAM users, roles, root user of member account)
- SCPs do **not** apply to the **management account** — ever
- SCPs do **not** apply to **service-linked roles**
- SCPs affect the **account root user** of member accounts (unlike IAM policies)
- An explicit deny in an SCP cannot be overridden by any IAM policy

---

## 3. Inheritance

```
Root SCP: Deny s3:DeleteBucket
    └── Workloads OU SCP: Allow ec2:*, s3:*
            └── Prod Account SCP: Allow ec2:*, s3:*
                    └── IAM Role: Allow s3:DeleteBucket
```

Result: `s3:DeleteBucket` is **denied** — the root SCP deny propagates down regardless of allows below it.

SCPs are **inherited** from parent to child. A child OU or account cannot escape a deny set at a parent level.

---

## 4. Strategies

### Deny-List Strategy (AWS Default)

- `FullAWSAccess` SCP is attached at root by default — allows everything
- Add specific deny SCPs to restrict prohibited actions
- Easier to start with; lower operational overhead
- Risk: new services are allowed by default unless explicitly denied

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyLeaveOrganization",
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```

### Allow-List Strategy

- Remove `FullAWSAccess` from root
- Attach SCPs that explicitly allow only needed services
- More restrictive; requires careful maintenance as teams need new services
- Better for high-security environments

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "rds:*",
        "cloudwatch:*",
        "iam:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 5. Common SCP Patterns

### Prevent Disabling Security Services

```json
{
  "Sid": "DenyDisableSecurityServices",
  "Effect": "Deny",
  "Action": [
    "cloudtrail:DeleteTrail",
    "cloudtrail:StopLogging",
    "cloudtrail:UpdateTrail",
    "guardduty:DeleteDetector",
    "guardduty:DisassociateFromMasterAccount",
    "config:DeleteConfigurationRecorder",
    "config:StopConfigurationRecorder",
    "securityhub:DisableSecurityHub"
  ],
  "Resource": "*"
}
```

### Restrict to Approved Regions

```json
{
  "Sid": "DenyNonApprovedRegions",
  "Effect": "Deny",
  "NotAction": [
    "iam:*",
    "organizations:*",
    "support:*",
    "sts:*",
    "cloudfront:*",
    "route53:*",
    "waf:*"
  ],
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
    }
  }
}
```

> Note: Use `NotAction` for global services (IAM, Route 53, CloudFront) that don't have a region.

### Require MFA for Sensitive Actions

```json
{
  "Sid": "DenyWithoutMFA",
  "Effect": "Deny",
  "Action": ["iam:DeleteVirtualMFADevice", "iam:DeactivateMFADevice"],
  "Resource": "*",
  "Condition": {
    "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
  }
}
```

### Prevent Root Account Usage

```json
{
  "Sid": "DenyRootAccountActions",
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringLike": {"aws:PrincipalArn": "arn:aws:iam::*:root"}
  }
}
```

---

## 6. SCPs vs Other Policy Types

| Feature | SCP | IAM Policy | Permission Boundary | RCP |
|---|---|---|---|---|
| Grants permissions | No | Yes | No | No |
| Restricts permissions | Yes | Yes (deny) | Yes | Yes |
| Applies to | Account/OU | Principal | Principal | Resource |
| Affects management account | No | Yes | Yes | No |
| Affects service-linked roles | No | N/A | No | Yes |
| Scope | Organization-wide | Single account | Single principal | Organization-wide |

---

## 7. Quotas

| Limit | Value |
|---|---|
| Max SCPs per root/OU/account | 5 |
| Max SCP document size | 5,120 characters (whitespace counts) |
| Max SCPs per organization | 1,000 |

---

## 8. Exam Scenarios

**Scenario 1:** A company wants to ensure no member account can create IAM users with console access. What is the most effective control?

> Attach an SCP that denies `iam:CreateLoginProfile` and `iam:UpdateLoginProfile` at the root OU. This prevents all member accounts from creating password-based IAM users, enforcing federated access via IAM Identity Center.

**Scenario 2:** A security team needs to allow the network team to manage VPCs but prevent them from deleting VPC flow logs. The control must apply even if the network team has AdministratorAccess.

> Attach an SCP to the network team's account OU that denies `ec2:DeleteFlowLogs`. SCPs override IAM policies including AdministratorAccess.

**Scenario 3:** An SCP is attached to the Prod OU denying `ec2:TerminateInstances`. A developer in the Prod account has an IAM policy that allows `ec2:TerminateInstances`. Can they terminate instances?

> No. The SCP deny takes precedence over the IAM allow. Effective permissions are the intersection of SCP and IAM.

**Scenario 4:** A company wants to restrict all accounts to us-east-1 and eu-west-1 only, but IAM and Route 53 must still work globally.

> Use a region-restriction SCP with `NotAction` covering global services (IAM, STS, Route 53, CloudFront, WAF, Support, Organizations) and a `StringNotEquals` condition on `aws:RequestedRegion`.

**Scenario 5:** The management account needs to be exempt from all SCPs. Is this automatic?

> Yes. SCPs never apply to the management account by design. This is why workloads should not run in the management account — it has no SCP guardrails.

---

## 9. Troubleshooting

| Symptom | Likely Cause |
|---|---|
| Action denied despite IAM allow | SCP deny in hierarchy — check root, parent OUs, account-level SCPs |
| SCP not taking effect | "All Features" not enabled in Organizations (consolidated billing only) |
| Service-linked role blocked | SCPs do not apply to service-linked roles — check IAM policy instead |
| Management account not restricted | Expected — SCPs never apply to management account |
| SCP too large | Refactor into multiple SCPs (max 5 per target, 5,120 chars each) |

---

## 10. Exam Tips

- SCPs **never grant** — they only restrict. Always pair with IAM policies
- The management account is **always exempt** — never rely on SCPs to protect it
- `FullAWSAccess` is attached by default — removing it blocks everything unless you add explicit allows
- Use `NotAction` + region condition for region restriction to avoid blocking global services
- SCP inheritance is **additive for denies** — a deny at any level in the hierarchy sticks
- Max 5 SCPs per target — design SCPs to be modular and reusable across OUs

---

## 11. 🎯 Architecture Decision Notes

### Turning a soft budget alert into a hard spend guardrail

AWS Budgets notifies; it does not stop launches. When a team routinely
blows past its monthly budget and notifications are ignored, you need an
enforcement mechanism — and SCPs are the right one for two structural
reasons: an SCP deny binds even users with administrative rights, and a
budget owned by the management account cannot be edited or deleted by
the spenders.

The pattern, decision by decision:

1. **Detect centrally.** Create the budget in the management account, one
   per member account, alerting at 100% of the monthly threshold. Member
   account admins cannot alter a management-account budget.
2. **Enforce with an SCP, not IAM.** On breach, publish to SNS and have a
   Lambda attach a deny-all-infrastructure SCP to the over-budget
   account. IAM policies cannot do this job — an account admin could
   simply remove an IAM-based restriction; they cannot remove or override
   an SCP.
3. **Release automatically.** Schedule a second Lambda (EventBridge
   schedule at month start) to detach the SCP, so the guardrail resets
   without manual toil.

Traps: test the deny SCP in a sandbox OU first — a malformed deny can
freeze legitimate automation, and because SCPs bind admins too, recovery
requires the management account. Prefer narrowly-scoped denies (block
launching new compute/storage) over a blanket deny-all if the account
must keep running existing workloads. And treat this as a backstop, not
a substitute for tag policies, budget actions, and cost anomaly alerts
that catch the trend earlier.

**Fresh example:** Ten product teams each own a dev account and all
exceed budgets. Central FinOps creates per-account budgets in the
management account. When Team Aurora hits 100%, the breach Lambda
attaches a deny-launch SCP to their account within minutes — running
workloads continue, new launches stop, and on the first of the month the
scheduled Lambda lifts the restriction. The teams feel the consequence
exactly where the spending happened, and no one can click around it.

## 12. Official References

- [SCP Overview](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [SCP Syntax](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html)
- [Example SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)
- [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
