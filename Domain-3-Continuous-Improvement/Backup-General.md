# AWS Backup — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3. AWS Backup is the centralized backup service. Know backup plans, vaults, cross-region/cross-account backup, and supported services.

---

## 1. Overview

AWS Backup is a fully managed, centralized backup service that automates and consolidates backup tasks across AWS services and accounts. It replaces service-specific backup configurations with a single policy-driven approach.

---

## 2. Supported Services

| Category | Services |
|---|---|
| Compute | EC2 instances (EBS volumes) |
| Storage | EFS, FSx (all types), S3 |
| Databases | RDS, Aurora, DynamoDB, DocumentDB, Neptune |
| Hybrid | VMware workloads (on-premises via AWS Backup Gateway) |
| Other | Storage Gateway volumes |

---

## 3. Core Concepts

### Backup Plan

- Policy that defines: backup frequency, retention period, lifecycle rules, vault destination
- Assign resources by tag, resource ID, or resource type

```yaml
BackupPlan:
  Rules:
    - RuleName: DailyBackup
      TargetBackupVault: MyVault
      ScheduleExpression: "cron(0 2 * * ? *)"  # 2 AM daily
      StartWindowMinutes: 60
      CompletionWindowMinutes: 120
      Lifecycle:
        DeleteAfterDays: 35
    - RuleName: MonthlyBackup
      TargetBackupVault: MyVault
      ScheduleExpression: "cron(0 2 1 * ? *)"  # 1st of month
      Lifecycle:
        MoveToColdStorageAfterDays: 90
        DeleteAfterDays: 365
```

### Backup Vault

- Encrypted container for backup recovery points
- **Vault Lock:** WORM protection — backups cannot be deleted before retention period
  - **Governance mode:** admins with special permissions can override
  - **Compliance mode:** no one can delete — even root
- Default vault uses AWS-managed KMS key; create custom vaults with customer-managed keys

### Recovery Points

- Individual backup of a resource
- Stored in the backup vault
- Restore from recovery point to original or new resource

---

## 4. Cross-Region Backup

- Copy backups to another region for DR
- Configure in backup plan as a copy action
- Destination vault must exist in the target region

```yaml
CopyActions:
  - DestinationBackupVaultArn: arn:aws:backup:eu-west-1:123456789012:backup-vault:DR-Vault
    Lifecycle:
      DeleteAfterDays: 90
```

---

## 5. Cross-Account Backup

- Copy backups to another AWS account for additional protection
- Requires: backup vault in destination account with resource policy allowing source account
- Use Organizations integration for simplified cross-account backup policies
- Protects against accidental deletion or ransomware in the source account

---

## 6. AWS Backup for Organizations

- Deploy backup policies (Backup Plans) across all accounts in an OU or organization
- Centrally manage and monitor backup compliance
- **Backup compliance reports:** identify resources not covered by backup plans
- Delegated admin: manage from a dedicated backup account

---

## 7. Exam Scenarios

**Scenario 1:** A company needs to ensure all RDS databases, DynamoDB tables, and EFS file systems are backed up daily with 35-day retention, and monthly backups retained for 1 year. What is the solution?

> Create an **AWS Backup plan** with two rules: daily backup (35-day retention) and monthly backup (1-year retention). Assign resources by tag (e.g., `Backup: required`). Tag all RDS, DynamoDB, and EFS resources with the tag. AWS Backup handles all services centrally.

**Scenario 2:** A company needs backups to be protected from deletion even by administrators, to meet compliance requirements. What is the solution?

> Enable **Backup Vault Lock** in **Compliance mode** on the backup vault. In Compliance mode, no user — including root — can delete recovery points before the retention period expires. This satisfies WORM compliance requirements.

**Scenario 3:** A company wants to protect backups from ransomware by storing copies in a separate AWS account. What is the solution?

> Configure **cross-account backup** in the backup plan. Create a backup vault in the DR account with a resource policy allowing the source account to copy backups. Add a copy action to the backup plan targeting the DR account vault.

**Scenario 4:** A company needs to enforce backup policies across 50 AWS accounts without configuring each account individually. What is the solution?

> Use **AWS Backup with Organizations**. Create backup policies in the management account or delegated admin account. Apply policies to OUs or the entire organization. All member accounts automatically have the backup plans applied.

---

## 8. Exam Tips

- AWS Backup = **centralized, policy-driven** backup across multiple services and accounts
- **Vault Lock Compliance mode** = no one can delete — even root — strongest protection
- **Cross-account backup** = protection against ransomware and accidental deletion
- **Organizations integration** = deploy backup policies to all accounts at scale
- AWS Backup does not replace service-native snapshots — it orchestrates them centrally
- **Backup compliance reports** = identify unprotected resources — use for audit evidence
- Assign resources by **tag** for scalable, automatic backup coverage

---

---

## 🎯 Architecture Decision Notes

### Replication is not backup

Replicas, read replicas, and cross-Region replication copy *everything* —
including corruption, ransomware encryption, and accidental deletes. A
recovery plan needs **isolated recovery points** (snapshots, vault-locked
backups) with separate retention, separate credentials, and ideally a
separate account.

- **AWS Backup Vault Lock (Compliance mode):** immutable for the retention
  period — not even the backup account's root can delete early. This is the
  control that survives a compromised administrator.
- **S3 Object Lock (Compliance mode):** the object-level equivalent;
  versioning alone is not immutability since versions can be deleted.
- Document retention, cross-account/Region copies, KMS key access, restore
  permissions, and validation of recovered data — then test the restore
  quarterly. An untested backup is a hope.

---

## 🎯 Architecture Decision Notes

### Design the backup cadence from the RPO, not the other way around

RPO (Recovery Point Objective) is the maximum data loss you can tolerate, measured in time. The design rule is simple: **your most frequent recovery point must be at least as frequent as the RPO**. An hourly full backup alone gives you an RPO of one hour — no better. To promise a 15-minute RPO, you need something captured every 15 minutes or less: transaction logs shipped every 5 minutes, continuous replication, or storage-level snapshots on a 15-minute schedule.

Decision rules:

- **Decompose the cadence:** full backups (cheap to reason about, slow to take) plus frequent incremental/log capture (cheap to take, gives the fine-grained recovery points). Restore = latest full + replay of everything after it.
- **Put the pieces in different failure domains:** logs in a separate bucket (or account/region) from the full backups, so one bucket issue doesn't take both.
- **RPO is about data loss; RTO is about time to restore.** A 5-minute RPO with a 6-hour restore is a valid design only if the business accepts the downtime — confirm both numbers, not just one.
- **Test the replay path.** Log-shipped recovery that has never been rehearsed is a hope, not a plan. The restore drill is what turns "RPO 15 minutes" from a slide into a capability.

**Fresh example:** A fintech ledger needs RPO ≤ 15 minutes. The design: nightly full database snapshots to S3, plus transaction logs streamed to a second S3 bucket every 5 minutes. Worst case, they lose at most 5 minutes of transactions — inside the objective. Quarterly, they restore the full snapshot plus logs into an isolated account and verify the ledger balances, proving the RPO is real.

---

## 🎯 Architecture Decision Notes

### Classify what's in the backups — Macie on the backup buckets

Backups concentrate your most sensitive data in one place, often with broader access than production (restores, audits, cross-account copies). Amazon Macie continuously scans S3 buckets and identifies sensitive data — PII, credentials, financial records — so you know *what* your backup buckets actually contain and can prove it to auditors.

Decision rules:

- **Run Macie on backup/log buckets**, not just production data lakes. Backup buckets are where forgotten sensitive fields accumulate, and they're the buckets most likely to be shared cross-account for DR.
- **Use findings to scope controls:** a bucket Macie flags as holding national IDs gets tighter bucket policies, KMS encryption with a restricted key, and access logging — while the bucket holding public assets stays simple. Classification lets you spend security effort where the data warrants it.
- **Feed it into the compliance story:** for regulated data, "we scan backups for sensitive data types and alert on unexpected findings" is a concrete control, stronger than "backups are encrypted" alone.
- Trap: Macie classifies and alerts — it doesn't remediate. Pair findings with bucket policies and lifecycle rules; and remember scanning has a cost, so scope it to buckets that can plausibly hold sensitive data rather than every bucket in the account.

**Fresh example:** A healthcare SaaS ships database transaction logs to S3 every 5 minutes for RPO purposes. Macie is enabled on the log bucket and flags that patient email addresses appear in a log stream the team assumed was de-identified. The team tightens the bucket policy, rotates to a dedicated KMS key, and fixes the logging pipeline — catching the exposure in the backup copy before an auditor (or worse) does.


---

## 🎯 Architecture Decision Notes

### Continuous backups — when snapshot schedules can't meet the RPO

AWS Backup offers two backup modes, and the choice is driven by the recovery point objective:

- **Snapshot backups** (scheduled): recovery points at the backup frequency you choose (e.g., daily). RPO = the interval between snapshots.
- **Continuous backups**: AWS Backup takes an initial full backup, then continuously captures transaction logs. You can rewind the resource to any second within the retention window (up to 35 days) — this is point-in-time recovery (PITR) through the centralized backup console.

Use continuous backups when the RPO is measured in seconds-to-minutes rather than hours — e.g., "lose no more than 5 minutes of database writes." The tradeoff: continuous backup keeps log data flowing constantly, so model the storage cost of the log stream against the RPO it buys. For RDS specifically, native automated backups already provide PITR via 5-minute transaction-log capture to S3; choose AWS Backup continuous mode when you want that same PITR capability managed centrally alongside cross-region and cross-account copies in one backup plan.

Mini-example: a payments database requires 10-minute RPO and 2-hour RTO. A daily snapshot alone misses the RPO — but an AWS Backup plan with continuous backups enabled gives second-granularity rewind within the window, satisfying the RPO while the plan's cross-region copy handles the DR side of the RTO.

## 9. Official References

- [AWS Backup Developer Guide](https://docs.aws.amazon.com/aws-backup/latest/devguide/)
- [Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [Cross-Account Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/create-cross-account-backup.html)
- [Backup for Organizations](https://docs.aws.amazon.com/aws-backup/latest/devguide/manage-cross-account.html)
