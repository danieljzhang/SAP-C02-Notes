# AWS Control Tower — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). Control Tower is the AWS-recommended way to set up and govern multi-account environments. Know how it relates to Organizations, SCPs, and Config.

---

## 1. Overview

AWS Control Tower automates the setup of a secure, multi-account AWS environment (landing zone) based on AWS best practices. It builds on top of AWS Organizations, AWS Config, CloudTrail, IAM Identity Center, and Service Catalog.

**Problems it solves:**
- Manual, error-prone multi-account setup
- Inconsistent security baselines across accounts
- No automated guardrails for new accounts
- Complex compliance tracking across accounts

---

## 2. Core Components

### Landing Zone

The overall multi-account environment set up by Control Tower. Includes:
- Management account
- Log Archive account (centralized S3 logging)
- Audit account (read-only access for security teams, Security Hub/GuardDuty delegated admin)
- Default OUs: Security OU, Sandbox OU

### Account Factory

- Automated account provisioning via Service Catalog
- Creates new accounts with pre-configured baselines (CloudTrail, Config, IAM Identity Center)
- Supports customization via Account Factory for Terraform (AFT) or Account Factory Customizations (AFC)
- Accounts are enrolled into the landing zone automatically

### Guardrails (Controls)

Controls are governance rules applied to OUs. Two types:

| Type | Mechanism | Behavior |
|---|---|---|
| Preventive | SCP | Blocks non-compliant actions before they happen |
| Detective | AWS Config rule | Detects non-compliant resources after the fact |
| Proactive | CloudFormation hooks | Checks resources before CloudFormation deploys them |

Guardrail guidance levels:

| Guidance | Meaning |
|---|---|
| Mandatory | Always enabled, cannot be disabled |
| Strongly Recommended | AWS best practice, optional |
| Elective | Optional, for specific use cases |

### Organizational Units

Control Tower manages OUs within Organizations. You can register existing OUs or create new ones. Guardrails are applied at the OU level.

---

## 3. Architecture

```
Management Account (Control Tower)
├── Organizations
│   ├── Security OU (mandatory)
│   │   ├── Log Archive Account
│   │   └── Audit Account
│   ├── Sandbox OU
│   └── Custom OUs (registered with Control Tower)
│       ├── Dev OU
│       └── Prod OU
├── IAM Identity Center (SSO for all accounts)
├── CloudTrail (organization trail → Log Archive)
└── AWS Config (aggregated → Audit Account)
```

---

## 4. Account Factory for Terraform (AFT)

AFT extends Account Factory to use Terraform for account provisioning and customization:

- Accounts are requested via a Terraform module
- AFT pipeline runs: account creation → baseline → customizations → account-level customizations
- Supports global customizations (applied to all accounts) and account-specific customizations
- Stored in CodeCommit/GitHub repositories

```
AFT Request (Terraform)
    └── AFT Pipeline (CodePipeline)
            ├── Account Provisioning (Service Catalog)
            ├── AFT Account Baseline
            ├── Global Customizations
            └── Account Customizations
```

---

## 5. Security Considerations

- **Log Archive account** — S3 bucket with SCPs preventing deletion of logs; only Control Tower can write
- **Audit account** — read-only cross-account access to all member accounts for security review
- **Mandatory guardrails** include: disallow changes to CloudTrail, disallow deletion of Log Archive bucket
- **IAM Identity Center** is configured automatically — use it for all human access
- Control Tower does not manage the management account's IAM — keep it locked down separately

---

## 6. Integration with Other Services

| Service | Role |
|---|---|
| AWS Organizations | Underlying account/OU structure |
| IAM Identity Center | SSO and permission sets for all accounts |
| AWS Config | Detective guardrails and compliance dashboard |
| CloudTrail | Organization trail for all accounts |
| Service Catalog | Account Factory product for account provisioning |
| SNS | Notifications for guardrail violations and drift |
| CloudFormation | StackSets for baseline deployment across accounts |

---

## 7. Drift Detection and Remediation

**Drift** occurs when changes are made outside Control Tower (e.g., manually modifying SCPs, deleting Config rules).

- Control Tower detects drift automatically
- Remediate drift by re-running the landing zone setup or re-enrolling the affected account/OU
- Drift types: SCP drift, OU drift, account drift, landing zone drift

---

## 8. Exam Scenarios

**Scenario 1:** A company wants to provision new AWS accounts with a security baseline automatically. Each account must have CloudTrail enabled and a specific VPC configuration. What is the recommended approach?

> Use Control Tower Account Factory with Account Factory Customizations (AFC) or AFT. Define the VPC configuration as a customization. Control Tower automatically enables CloudTrail via the organization trail.

**Scenario 2:** A company already has 20 AWS accounts in Organizations. They want to adopt Control Tower without disrupting existing accounts. What should they do?

> Set up Control Tower in the management account. Register existing OUs with Control Tower. Enroll existing accounts into the landing zone. Existing resources are preserved; guardrails are applied going forward.

**Scenario 3:** A security team needs to prevent developers from disabling AWS Config in any account. What Control Tower feature handles this?

> Enable the mandatory preventive guardrail "Disallow changes to AWS Config rules set up by AWS Control Tower." This is implemented as an SCP and cannot be disabled.

**Scenario 4:** A company needs to detect when any S3 bucket in any account becomes publicly accessible. What is the most scalable solution?

> Enable the detective guardrail "Detect whether public read access to Amazon S3 buckets is allowed" in Control Tower. This deploys a Config rule to all enrolled accounts in the OU.

**Scenario 5:** An account enrolled in Control Tower has drifted because an admin manually deleted a Config rule. How should this be resolved?

> Re-enroll the account in Control Tower from the Control Tower console. This re-applies the baseline and restores the deleted Config rule.

---

## 9. Exam Tips

- Control Tower is built **on top of** Organizations — it does not replace it
- Guardrails are applied at the **OU level**, not the account level directly
- **Mandatory guardrails cannot be disabled** — they are always enforced
- Control Tower does not manage the **management account** with guardrails
- AFT is the recommended approach for **large-scale, Terraform-based** account provisioning
- Drift must be **manually remediated** — Control Tower detects but does not auto-fix drift
- The **Audit account** has read-only access to all accounts — it is not a security enforcement account

---

## 10. Official References

- [AWS Control Tower User Guide](https://docs.aws.amazon.com/controltower/latest/userguide/)
- [Account Factory for Terraform](https://docs.aws.amazon.com/controltower/latest/userguide/aft-overview.html)
- [Control Tower Guardrails](https://docs.aws.amazon.com/controltower/latest/userguide/controls.html)
- [Landing Zone Setup](https://docs.aws.amazon.com/controltower/latest/userguide/setting-up.html)
