# AWS Landing Zone Design — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). Landing zone design is a core SAP-C02 topic. Know the components, account structure, and customization options.

---

## 1. What Is a Landing Zone?

A landing zone is a pre-configured, secure, multi-account AWS environment that serves as a starting point for deploying workloads at scale. It enforces security baselines, centralized logging, and governance guardrails from day one.

AWS provides two paths:
- **AWS Control Tower** — managed landing zone with automated setup (recommended)
- **Custom landing zone** — built manually using Organizations, CloudFormation StackSets, and Config

---

## 2. Core Accounts

| Account | Purpose |
|---|---|
| Management account | Organizations root, billing, Control Tower management. No workloads. |
| Log Archive account | Centralized S3 bucket for CloudTrail logs, Config snapshots, VPC flow logs |
| Audit account | Security team read-only access, Security Hub/GuardDuty delegated admin |
| Network Hub account | Transit Gateway, Direct Connect, shared DNS (Route 53 Resolver) |
| Shared Services account | Shared AMIs, Service Catalog, common tooling |
| Workload accounts | Dev, Staging, Prod — isolated per environment or per application |

---

## 3. OU Design Principles

```
Root
├── Security OU (restricted SCPs — no workloads)
│   ├── Log Archive Account
│   └── Audit Account
├── Infrastructure OU
│   ├── Network Hub Account
│   └── Shared Services Account
├── Sandbox OU (loose SCPs — experimentation)
├── Workloads OU
│   ├── Dev OU
│   │   └── Dev Accounts (per team or per app)
│   ├── Staging OU
│   └── Prod OU
│       └── Prod Accounts
├── Suspended OU (closed/decommissioned accounts)
└── Exceptions OU (accounts needing SCP exemptions)
```

**Design principles:**
- Model OUs around **security and compliance boundaries**, not org chart
- Keep the **management account** empty of workloads
- Use **Suspended OU** for accounts being decommissioned — SCPs prevent resource creation
- Use **Exceptions OU** sparingly and with compensating controls

---

## 4. Baseline Services Applied to Every Account

| Service | What Gets Configured |
|---|---|
| CloudTrail | Organization trail writing to Log Archive S3 bucket |
| AWS Config | Configuration recorder + delivery channel to Log Archive |
| IAM Identity Center | Permission sets provisioned for account access |
| VPC | Default VPC deleted; custom VPC deployed via StackSet |
| GuardDuty | Enrolled as member of organization |
| Security Hub | Enrolled as member; standards enabled |
| Amazon Macie | Enrolled as member (if data classification required) |

---

## 5. Networking Baseline

```
Network Hub Account
├── Transit Gateway (shared via RAM)
├── Direct Connect Gateway
├── Route 53 Resolver (inbound/outbound endpoints)
└── Inspection VPC (optional — AWS Network Firewall)

Spoke Accounts
└── VPC → TGW attachment → routes to on-premises and other VPCs
```

- Spoke VPCs are shared or connected via Transit Gateway
- DNS resolution for on-premises hostnames via Route 53 Resolver forwarding rules shared via RAM
- Centralized egress via NAT Gateway in Network Hub account (cost optimization)

---

## 6. Customization Options

### Account Factory Customizations (AFC)

- Native Control Tower feature
- Define customizations as CloudFormation templates
- Applied automatically to new and existing accounts

### Account Factory for Terraform (AFT)

- Terraform-based account provisioning pipeline
- Supports global customizations (all accounts) and per-account customizations
- Integrates with existing Terraform workflows

### CloudFormation StackSets

- Deploy resources to multiple accounts/regions simultaneously
- Used for baseline VPC, IAM roles, Config rules, etc.
- Supports automatic deployment to new accounts in an OU

---

## 7. Security Controls in the Landing Zone

| Control | Implementation |
|---|---|
| Prevent log deletion | SCP denying S3 delete on Log Archive bucket |
| Enforce CloudTrail | SCP denying cloudtrail:StopLogging, DeleteTrail |
| Require encryption | SCP denying unencrypted S3 puts, EBS without encryption |
| Region restriction | SCP with NotAction + aws:RequestedRegion condition |
| Prevent leaving org | SCP denying organizations:LeaveOrganization |
| Enforce tagging | Tag Policies via Organizations |
| Enforce backup | Backup Policies via Organizations |

---

## 8. Exam Scenarios

**Scenario 1:** A company is setting up AWS for the first time with 10 teams. Each team needs isolated accounts with a consistent security baseline. What is the recommended approach?

> Use Control Tower to set up a landing zone. Use Account Factory to provision one account per team. Apply guardrails at the OU level. Use IAM Identity Center for cross-account access.

**Scenario 2:** A company needs centralized VPC flow logs and CloudTrail logs from all accounts stored in a single S3 bucket that no account can delete. How?

> Create a Log Archive account. Configure an organization CloudTrail trail writing to an S3 bucket in Log Archive. Apply an SCP denying S3 delete actions on the Log Archive account. Enable VPC flow logs in each account with the destination set to the Log Archive bucket.

**Scenario 3:** A company wants new accounts to automatically get a standard VPC configuration without manual steps. What is the best approach?

> Use Control Tower Account Factory with a CloudFormation customization (AFC) or AFT that deploys the standard VPC. The customization runs automatically when a new account is provisioned.

**Scenario 4:** A company has a Prod OU and a Dev OU. They want stricter controls in Prod (no public S3, no unencrypted EBS) but more flexibility in Dev. How?

> Attach restrictive SCPs to the Prod OU (deny public S3, deny unencrypted EBS). Attach less restrictive SCPs to the Dev OU. Both OUs inherit the root-level SCPs (e.g., no CloudTrail deletion).

---

## 9. Exam Tips

- Landing zone = the **overall environment** — Control Tower is the tool that builds it
- The **Log Archive** and **Audit** accounts are mandatory in a Control Tower landing zone
- **Delete the default VPC** in every account — it is a security risk and landing zone best practice
- **StackSets** are the mechanism for deploying baseline resources to all accounts
- **Suspended OU** is the correct place for decommissioned accounts — do not delete accounts immediately
- AFT is for teams already using Terraform — AFC is the native Control Tower option

---

## 10. Official References

- [Landing Zone Best Practices](https://docs.aws.amazon.com/controltower/latest/userguide/aws-multi-account-landing-zone.html)
- [AFT Overview](https://docs.aws.amazon.com/controltower/latest/userguide/aft-overview.html)
- [CloudFormation StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html)
- [AWS Security Reference Architecture](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html)
