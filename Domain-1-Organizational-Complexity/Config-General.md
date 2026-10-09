# AWS Config — SAP-C02 Study Notes

> Exam relevance: Domains 1, 3. Config is the detective compliance engine. Know rules, conformance packs, aggregators, and remediation patterns.

---

## 1. Overview

AWS Config continuously records AWS resource configurations and evaluates them against desired rules. It provides a configuration history, change notifications, and compliance dashboards.

**Problems it solves:**
- No visibility into what changed, when, and by whom
- Manual compliance auditing across accounts
- Inability to detect configuration drift from security baselines

---

## 2. Core Concepts

| Concept | Description |
|---|---|
| Configuration recorder | Records resource configurations and changes |
| Delivery channel | Sends snapshots and change notifications to S3 and SNS |
| Config rule | Evaluates resource configurations against desired state |
| Conformance pack | Collection of Config rules deployed as a unit |
| Aggregator | Collects Config data from multiple accounts/regions |
| Remediation action | SSM Automation document triggered on non-compliance |

---

## 3. Config Rules

### Managed Rules (AWS-provided)

Common exam-relevant managed rules:

| Rule | Checks |
|---|---|
| `s3-bucket-public-read-prohibited` | S3 buckets not publicly readable |
| `encrypted-volumes` | EBS volumes are encrypted |
| `iam-password-policy` | IAM password policy meets requirements |
| `restricted-ssh` | Security groups do not allow unrestricted SSH |
| `rds-storage-encrypted` | RDS instances have storage encryption enabled |
| `cloudtrail-enabled` | CloudTrail is enabled |
| `mfa-enabled-for-iam-console-access` | IAM users with console access have MFA |
| `vpc-flow-logs-enabled` | VPC flow logs are enabled |

### Custom Rules

- Written as Lambda functions
- Triggered on configuration changes or periodically
- Can evaluate any resource attribute

### Evaluation Triggers

| Trigger | When |
|---|---|
| Configuration change | When a resource is created, modified, or deleted |
| Periodic | On a schedule (1h, 3h, 6h, 12h, 24h) |
| Hybrid | Both change and periodic |

---

## 4. Conformance Packs

- Bundle of Config rules + remediation actions deployed as a single unit
- AWS provides pre-built packs: CIS AWS Foundations, PCI DSS, HIPAA, NIST
- Deploy to a single account or organization-wide via the management account
- Results aggregated in the conformance pack compliance dashboard

```bash
# Deploy conformance pack
aws configservice put-conformance-pack \
  --conformance-pack-name "CIS-Benchmark" \
  --template-s3-uri s3://my-bucket/cis-conformance-pack.yaml
```

---

## 5. Multi-Account Aggregator

Collects Config compliance data from multiple accounts and regions into a single view:

```
Aggregator Account (e.g., Audit Account)
└── Config Aggregator
        ├── Account A (us-east-1)
        ├── Account B (eu-west-1)
        └── Account C (ap-southeast-1)
```

- Requires authorization from source accounts (or use Organizations for automatic authorization)
- Provides organization-wide compliance dashboard
- Does not deploy rules — only aggregates results

---

## 6. Remediation

### Automatic Remediation

```
Config Rule detects non-compliance
        └── Triggers SSM Automation document
                └── Fixes the resource (e.g., enables encryption, removes public access)
```

- Configure remediation action on the Config rule
- Set retry attempts and retry interval
- Use `AWS-EnableS3BucketEncryption`, `AWS-DisablePublicAccessForSecurityGroup`, etc.

### Manual Remediation

- Config marks resource as non-compliant
- Operator reviews and manually triggers remediation
- Or uses EventBridge rule → Lambda for custom remediation logic

---

## 7. Exam Scenarios

**Scenario 1:** A company needs to detect when any EC2 security group allows unrestricted SSH access (0.0.0.0/0 on port 22) across all accounts. What is the solution?

> Enable the `restricted-ssh` Config managed rule organization-wide via a conformance pack. Deploy the conformance pack from the management account or delegated admin account. Use an aggregator in the Audit account to view compliance across all accounts.

**Scenario 2:** A company wants non-compliant S3 buckets (public access enabled) to be automatically remediated. What is the architecture?

> Create a Config rule for `s3-bucket-public-read-prohibited`. Configure automatic remediation using the `AWS-DisableS3BucketPublicReadWrite` SSM Automation document. Set retry attempts to handle transient failures.

**Scenario 3:** A company needs a compliance report showing which accounts have unencrypted EBS volumes. What is the solution?

> Deploy the `encrypted-volumes` Config rule organization-wide. Set up a Config aggregator in the Audit account scoped to the entire organization. Query the aggregator for non-compliant resources across all accounts and regions.

---

## 8. Exam Tips

- Config is **detective** — it detects non-compliance but does not prevent it (use SCPs for prevention)
- Config rules evaluate **existing** resources — they do not block resource creation
- **Conformance packs** are the scalable way to deploy multiple rules at once
- **Aggregators** provide the organization-wide compliance view — deploy in the Audit account
- Remediation uses **SSM Automation** — know common automation documents
- Config must be enabled in **every region** you want to monitor — it is regional
- Organization-wide deployment requires **trusted access** enabled for Config in Organizations

---

## 9. 🎯 Architecture Decision Notes

### Companion services: certificates and parameters

**ACM — verify support before designing around it:**
- ACM manages public and private TLS certificates and automates renewal
  for *eligible* integrations. Confirm the target service and certificate
  type are supported first — not every service consumes ACM certificates,
  and private key exportability varies by workflow.

**Parameter Store vs secrets — choose by rotation need:**
- **Systems Manager Parameter Store** holds configuration values with
  IAM-controlled access. CloudFormation typed parameters can reference
  Parameter Store values, and dynamic references resolve them without
  embedding values in templates.
- When the value needs **rotation, cross-account sharing, or RDS-native
  integration**, use a secrets-management service (Secrets Manager)
  instead. Protect access to the underlying parameter/secret and consider
  *when* the value resolves (deploy time vs runtime).

**Config aggregator nuance:** a multi-account, multi-Region aggregator
gives a consolidated compliance view — but it does not itself enable
recording in every account and Region. Recording must be on where you
need it.

## 10. Official References

- [AWS Config Developer Guide](https://docs.aws.amazon.com/config/latest/developerguide/)
- [Managed Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [Conformance Packs](https://docs.aws.amazon.com/config/latest/developerguide/conformance-packs.html)
- [Multi-Account Aggregator](https://docs.aws.amazon.com/config/latest/developerguide/aggregate-data.html)
