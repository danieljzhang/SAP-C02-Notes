# AWS Organizations — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). Foundation of every multi-account architecture question. Appears in Domains 3 and 4 as well.

---

## 1. Overview

AWS Organizations lets you centrally manage multiple AWS accounts under a single management account. It provides consolidated billing, policy-based governance via SCPs, and integration with nearly every AWS governance service.

**Problems it solves:**
- Sprawling, ungoverned AWS accounts
- Inconsistent security controls across teams
- Complex cross-account billing and cost allocation
- Manual account provisioning and configuration

---

## 2. Core Concepts

### Account Types

| Account Type | Role |
|---|---|
| Management account | Root of the organization. Creates and manages member accounts. SCPs do NOT apply to it. |
| Member account | Any account that joins the organization. Subject to SCPs. |
| Delegated administrator | Member account granted admin rights for a specific AWS service (e.g., GuardDuty, Security Hub). |

### Organizational Units (OUs)

- Hierarchical grouping of accounts under the root
- SCPs applied to an OU are inherited by all child OUs and accounts
- Best practice: model OUs around security/compliance boundaries, not team structure
- Maximum OU nesting depth: **5 levels**

### Root

- Top-level container for all OUs and accounts
- SCPs applied at root affect all accounts (except management account)

### Policies

| Policy Type | Purpose |
|---|---|
| [Service Control Policies (SCPs)](./organizations/scps-general.md) | Restrict what actions principals in member accounts can perform |
| [Resource Control Policies (RCPs)](./iam/resourcecontrolpolicies-general.md) | Restrict what resources in member accounts can accept (data perimeter) |
| Tag Policies | Enforce tag key/value standards across accounts |
| Backup Policies | Enforce AWS Backup plans across accounts |
| AI Services Opt-Out Policies | Control whether AWS AI services can use data for improvement |

---

## 3. Architecture Patterns

### Recommended OU Structure

```
Root
├── Security OU
│   ├── Log Archive Account      ← centralized CloudTrail, S3 access logs
│   └── Audit Account            ← Security Hub, GuardDuty delegated admin
├── Infrastructure OU
│   └── Network Hub Account      ← Transit Gateway, Direct Connect, DNS
├── Sandbox OU                   ← developer experimentation, loose SCPs
├── Workloads OU
│   ├── Dev OU
│   ├── Staging OU
│   └── Prod OU
└── Exceptions OU                ← accounts needing SCP exemptions
```

### Consolidated Billing

- All member account charges roll up to the management account
- Volume discounts (e.g., S3, EC2 Reserved Instances) are pooled across the organization
- Use Cost Allocation Tags + AWS Cost Explorer for per-account visibility

### Delegated Administration

```
Management Account
  └── enables GuardDuty
        └── delegates admin to Audit Account
              └── Audit Account manages GuardDuty for all member accounts
```

Supported services for delegated admin include: GuardDuty, Security Hub, Config, Firewall Manager, IAM Access Analyzer, Macie, Inspector, and more.

---

## 4. Security Considerations

- **Management account is exempt from SCPs** — minimize workloads in it; treat it as a break-glass account
- Enable **CloudTrail organization trail** from the management account — member accounts cannot disable it
- Use **IAM Identity Center** (not individual IAM users) for human access across accounts
- Enable **AWS Config** organization-wide via delegated admin for centralized compliance visibility
- Use **Tag Policies** to enforce cost allocation and resource identification standards

---

## 5. Integration with Other Services

| Service | Integration |
|---|---|
| Control Tower | Builds on Organizations; automates OU/account creation with guardrails |
| IAM Identity Center | Provides SSO across all organization accounts |
| AWS Config | Aggregates compliance data across all accounts via organization aggregator |
| Security Hub | Aggregates findings across all accounts; uses delegated admin |
| GuardDuty | Organization-wide threat detection via delegated admin |
| CloudTrail | Organization trail captures all account activity centrally |
| RAM | Shares resources (TGW, subnets, etc.) across accounts without cross-account roles |
| Backup | Enforces backup policies across all accounts via Backup Policies |
| Firewall Manager | Centrally manages WAF, Shield, Security Groups across accounts |

---

## 6. Quotas and Limits

| Limit | Value |
|---|---|
| Max accounts per organization | 10 (default; increasable via quota request) — verify current limit |
| Max OU nesting depth | 5 levels |
| Max SCPs per account/OU | 5 |
| Max SCP size | 5,120 characters |
| Max Tag Policies per account | 5 |

> Always verify current limits at https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_limits.html

---

## 7. Monitoring and Troubleshooting

- **CloudTrail** — all Organizations API calls are logged (CreateAccount, AttachPolicy, etc.)
- **AWS Config** — use `AWS::Organizations::Account` resource type to track account changes
- **Access Denied errors** — check SCP inheritance chain from root → OU → account
- **Account creation failures** — check service quotas and email uniqueness requirements
- **SCP not taking effect** — confirm "All Features" is enabled (not just consolidated billing)

---

## 8. Exam Scenarios

**Scenario 1:** A company has 50 AWS accounts. The security team needs to prevent any account from disabling CloudTrail. What is the most scalable solution?

> Attach an SCP at the root OU that denies `cloudtrail:DeleteTrail`, `cloudtrail:StopLogging`, and `cloudtrail:UpdateTrail`. This applies to all member accounts and cannot be overridden by account-level IAM policies.

**Scenario 2:** A developer account needs to use a service that is blocked by an SCP on the Workloads OU. The exception should not affect other accounts.

> Move the developer account to an Exceptions OU with a less restrictive SCP, or create a child OU under Workloads with an SCP that allows the specific service. Do not modify the parent OU SCP.

**Scenario 3:** A company wants centralized billing but does not want SCPs to restrict member accounts yet.

> Enable Organizations with **Consolidated Billing features only**. SCPs require "All Features" to be enabled. Note: migrating from consolidated billing to all features is a one-way operation.

**Scenario 4:** The security team needs to enforce S3 Block Public Access across all accounts without touching each account individually.

> Use an SCP that denies `s3:PutBucketPublicAccessBlock` with a condition that allows only the security team, combined with an AWS Config organization conformance pack to detect non-compliant buckets.

**Scenario 5:** A company acquires another company with an existing AWS account. How should they bring it into the organization?

> Send an invitation from the management account to the acquired account. The acquired account accepts the invitation and becomes a member account. Existing resources and IAM configurations are preserved. Apply appropriate SCPs after joining.

---

## 9. CLI Reference

```bash
# Create an organization
aws organizations create-organization --feature-set ALL

# List all accounts
aws organizations list-accounts

# Create an OU
aws organizations create-organizational-unit \
  --parent-id r-xxxx \
  --name "Workloads"

# Move account to OU
aws organizations move-account \
  --account-id 123456789012 \
  --source-parent-id r-xxxx \
  --destination-parent-id ou-xxxx-xxxxxxxx

# Attach SCP to OU
aws organizations attach-policy \
  --policy-id p-xxxxxxxxxxxx \
  --target-id ou-xxxx-xxxxxxxx

# Enable delegated administrator
aws organizations register-delegated-administrator \
  --account-id 123456789012 \
  --service-principal guardduty.amazonaws.com
```

---

## 10. Exam Tips

- SCPs are **permission boundaries for accounts** — they do not grant permissions, only restrict them
- The management account is **never** subject to SCPs — keep it clean and locked down
- "All Features" must be enabled for SCPs to work — consolidated billing only does not support SCPs
- OU inheritance: SCP on parent OU applies to all children — use allow-list or deny-list strategy consistently
- **Allow-list strategy:** default deny all, explicitly allow needed services (more restrictive, harder to maintain)
- **Deny-list strategy:** default allow all, explicitly deny prohibited services (AWS default, easier to start with)
- Delegated admin accounts can manage services organization-wide without using the management account

---

## 11. 🎯 Architecture Decision Notes

### Sharing without losing control — RAM and tags

**AWS RAM** shares supported resources (subnets, Transit Gateway, Route 53
rules, license configurations) across accounts, OUs, or the organization.
Key exam points:

- Sharing does **not** transfer ownership — the sharing account retains
  control; consumers use the resource within their own configuration.
- Some sharing integrations require **trusted access** between the AWS
  service and Organizations. Enable it only for the service that needs it,
  and understand which service acts on the organization's behalf.

**Fresh example:** A central networking account shares subnets with
application accounts via RAM. The network team keeps control of the IP
plan, while each consumer deploys its own workloads — but each consumer
must still configure its own routing and security groups correctly.
Sharing the subnet does not share the responsibility for what runs in it.

### Tag governance as a control plane

Tags drive cost allocation, automation, and attribute-based access control
(ABAC) — but only with a **small, governed vocabulary** enforced
consistently:

- **Preventive:** SCPs or IAM policies can deny create/tag operations that
  violate the vocabulary.
- **Detective:** AWS Config rules evaluate deployed resources for tag
  compliance.
- Prevention and audit solve different halves — use both, and do not
  confuse a Config rule (detective) with an SCP (preventive).

## 12. Official References

- [AWS Organizations User Guide](https://docs.aws.amazon.com/organizations/latest/userguide/)
- [SCP Syntax Reference](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html)
- [Delegated Administrator Services](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_integrate_services_list.html)
- [AWS Organizations Limits](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_limits.html)
- [SAP-C02 Exam Guide](https://aws.amazon.com/certification/certified-solutions-architect-professional/)
