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

## 9. Official References

- [AWS Backup Developer Guide](https://docs.aws.amazon.com/aws-backup/latest/devguide/)
- [Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [Cross-Account Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/create-cross-account-backup.html)
- [Backup for Organizations](https://docs.aws.amazon.com/aws-backup/latest/devguide/manage-cross-account.html)
