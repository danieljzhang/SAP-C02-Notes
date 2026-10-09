# AWS Security Hub — SAP-C02 Study Notes

> Exam relevance: Domains 1, 3. Security Hub is the central security findings aggregator. Know standards, integrations, and automated response patterns.

---

## 1. Overview

AWS Security Hub provides a centralized view of security findings across AWS accounts, aggregating results from AWS security services and third-party tools. It evaluates your environment against security standards and best practices.

**Problems it solves:**
- Security findings scattered across GuardDuty, Inspector, Macie, Config, etc.
- No single compliance dashboard across accounts
- Manual security review processes

---

## 2. Core Concepts

### Security Standards

Pre-built compliance frameworks evaluated automatically:

| Standard | Description |
|---|---|
| AWS Foundational Security Best Practices (FSBP) | AWS-curated controls across 30+ services |
| CIS AWS Foundations Benchmark | CIS Level 1 and Level 2 controls |
| PCI DSS | Payment Card Industry controls |
| NIST SP 800-53 | US federal security controls |
| SOC 2 | Service Organization Controls |

### Findings

- Normalized using **AWS Security Finding Format (ASFF)**
- Aggregated from: GuardDuty, Inspector, Macie, Config, IAM Access Analyzer, Firewall Manager, and 50+ third-party integrations
- Severity: Critical, High, Medium, Low, Informational

### Controls

- Individual checks within a security standard
- Each control maps to a Config rule or Security Hub-native check
- Status: PASSED, FAILED, NOT_AVAILABLE, NO_DATA

---

## 3. Multi-Account Architecture

```
Management Account
  └── Delegates Security Hub admin to Audit Account

Audit Account (delegated admin)
  └── Security Hub administrator
        ├── Aggregates findings from all member accounts
        ├── Manages standards and controls centrally
        └── Cross-region aggregation (optional)

Member Accounts
  └── Auto-enrolled via Organizations
  └── Findings sent to Audit Account
```

- Use **delegated administrator** pattern — do not manage from management account
- Enable **cross-region aggregation** to see all findings in one region

---

## 4. Automated Response

### EventBridge Integration

```
Security Hub Finding (CRITICAL severity)
        └── EventBridge Rule
                └── Lambda function
                        └── Remediation action (e.g., isolate EC2, revoke IAM key)
```

### Security Hub Automations (native)

- Define rules that automatically update finding status or trigger actions
- No Lambda required for simple actions (suppress, update severity, send notification)

### AWS Systems Manager Automation

- Security Hub → EventBridge → SSM Automation
- Pre-built runbooks for common remediations

---

## 5. Exam Scenarios

**Scenario 1:** A company wants a single dashboard showing security compliance across 40 AWS accounts and 3 regions. What is the solution?

> Enable Security Hub with Organizations integration. Delegate administration to the Audit account. Enable cross-region aggregation in the Audit account. All findings from all accounts and regions appear in the Audit account's Security Hub dashboard.

**Scenario 2:** A company wants to automatically isolate an EC2 instance when GuardDuty detects a cryptocurrency mining finding. What is the architecture?

> GuardDuty sends the finding to Security Hub. Create an EventBridge rule that matches the finding type. Trigger a Lambda function or SSM Automation that modifies the EC2 instance's security group to block all traffic (isolation).

**Scenario 3:** A company needs to demonstrate PCI DSS compliance to auditors. What AWS service provides the most relevant compliance dashboard?

> Enable the PCI DSS security standard in Security Hub. The dashboard shows which controls pass and fail, with evidence for each control. Export findings to S3 for auditor review.

---

## 6. Exam Tips

- Security Hub **aggregates** findings — it does not generate them for most services (GuardDuty, Inspector, Macie generate their own)
- **FSBP** is the most comprehensive AWS-native standard — enable it first
- Use **delegated admin** in the Audit account — not the management account
- Security Hub findings use **ASFF** — normalized format enables cross-service correlation
- **Cross-region aggregation** is required to see all findings in one place
- Security Hub does not **remediate** — use EventBridge + Lambda/SSM for automated response
- Suppressed findings still count in compliance scores — use with caution

---

## 7. Official References

- [Security Hub User Guide](https://docs.aws.amazon.com/securityhub/latest/userguide/)
- [AWS Security Finding Format](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-format.html)
- [Automated Response](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cloudwatch-events.html)
- [Security Hub with Organizations](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-accounts.html)
