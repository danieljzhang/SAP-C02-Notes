# AWS Systems Manager — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3. Systems Manager is the operational backbone for EC2 and hybrid environments. Know Parameter Store, Session Manager, Run Command, Patch Manager, and Automation.

---

## 1. Overview

AWS Systems Manager (SSM) provides operational management capabilities for EC2 instances, on-premises servers, and other AWS resources. It replaces SSH/RDP access and manual operational tasks with secure, auditable automation.

**Key capabilities:**
- Secure remote access without SSH/RDP (Session Manager)
- Run scripts across fleets of instances (Run Command)
- Automated patching (Patch Manager)
- Configuration and secrets storage (Parameter Store)
- Automated operational runbooks (Automation)
- Inventory and compliance (Inventory, State Manager)

---

## 2. SSM Agent

- Required on EC2 instances and on-premises servers
- Pre-installed on most AWS-provided AMIs (Amazon Linux 2, Windows Server 2016+)
- Requires IAM instance profile with `AmazonSSMManagedInstanceCore` policy
- Communicates with SSM service over HTTPS (port 443) — no inbound ports needed

---

## 3. Session Manager

Secure, browser-based or CLI shell access to instances without SSH/RDP:

```
User → IAM authentication → SSM Session Manager → SSM Agent → Instance
```

**Benefits:**
- No SSH keys, no bastion hosts, no open inbound ports
- All session activity logged to CloudTrail and optionally S3/CloudWatch Logs
- Works for EC2 in private subnets (via VPC endpoint or NAT)
- Supports port forwarding for database access

```bash
# Start a session
aws ssm start-session --target i-1234567890abcdef0

# Port forwarding to RDS
aws ssm start-session \
  --target i-1234567890abcdef0 \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters '{"host":["mydb.cluster.us-east-1.rds.amazonaws.com"],"portNumber":["5432"],"localPortNumber":["5432"]}'
```

---

## 4. Parameter Store

Secure, hierarchical storage for configuration data and secrets:

| Tier | Max Size | Cost | Features |
|---|---|---|---|
| Standard | 4 KB | Free | Basic parameters |
| Advanced | 8 KB | $0.05/parameter/month | Parameter policies, higher throughput |

### Parameter Types

| Type | Use Case |
|---|---|
| String | Plain text configuration values |
| StringList | Comma-separated list of values |
| SecureString | Encrypted with KMS — for secrets |

### Hierarchy

```
/myapp/prod/database/host
/myapp/prod/database/password  (SecureString)
/myapp/dev/database/host
```

- Retrieve all parameters under a path: `GetParametersByPath`
- Reference in CloudFormation: `{{resolve:ssm:/myapp/prod/database/host}}`
- Reference in ECS task definitions: inject as environment variables

### Parameter Store vs Secrets Manager

| Feature | Parameter Store | Secrets Manager |
|---|---|---|
| Cost | Free (standard) | $0.40/secret/month |
| Automatic rotation | No (use Lambda) | Yes (built-in for RDS, Redshift, DocumentDB) |
| Cross-account | Limited | Yes |
| Secret versioning | Yes | Yes |
| Use case | Config + simple secrets | Database credentials, API keys needing rotation |

---

## 5. Run Command

Execute scripts or commands across multiple instances simultaneously:

```bash
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=tag:Environment,Values=prod" \
  --parameters '{"commands":["yum update -y"]}'
```

- Target by instance ID, tag, or resource group
- Output to S3 or CloudWatch Logs
- Rate control: limit concurrent executions to avoid overwhelming instances
- Error threshold: stop execution if N% of instances fail

---

## 6. Patch Manager

Automate OS and application patching:

- **Patch baseline:** defines approved/rejected patches (OS, severity, classification)
- **Patch group:** tag-based grouping of instances (e.g., `Patch Group: prod-web`)
- **Maintenance window:** scheduled time for patching operations
- **Compliance reporting:** view patch compliance status in SSM console and Config

```
Maintenance Window (Sunday 2 AM)
  └── Run Task: AWS-RunPatchBaseline
        └── Patch Group: prod-web
              └── Patch Baseline: AmazonLinux2-DefaultPatchBaseline
```

---

## 7. Automation

Run operational runbooks (SSM Documents) for complex multi-step tasks:

- Pre-built runbooks: `AWS-RestartEC2Instance`, `AWS-CreateImage`, `AWS-EnableS3BucketEncryption`
- Custom runbooks: YAML/JSON documents with steps
- Triggered by: manual, EventBridge, Config remediation, CloudWatch alarm
- Supports approval steps for human-in-the-loop workflows

---

## 8. State Manager

Maintain consistent configuration state on instances:

- Associate instances with SSM Documents that define desired state
- Runs on a schedule to enforce configuration
- Example: ensure CloudWatch Agent is always installed and running

---

## 9. Inventory

Collect metadata about instances and installed software:

- Collects: applications, AWS components, network config, Windows updates, custom inventory
- Store in S3 and query with Athena
- View in Systems Manager Inventory dashboard

---

## 10. Exam Scenarios

**Scenario 1:** A company needs to access EC2 instances in private subnets for troubleshooting without opening SSH ports or deploying bastion hosts. What is the solution?

> Use **Session Manager**. Ensure instances have the SSM Agent installed and an IAM instance profile with `AmazonSSMManagedInstanceCore`. Create a VPC endpoint for SSM if instances don't have internet access. All sessions are logged to CloudTrail.

**Scenario 2:** A company stores database passwords in EC2 user data (plaintext). How should this be improved?

> Store database passwords in **Parameter Store as SecureString** parameters encrypted with KMS. Update the application to retrieve the parameter at runtime using the SSM SDK. Remove credentials from user data and AMIs. Rotate the KMS key regularly.

**Scenario 3:** A company needs to apply security patches to 500 EC2 instances every Sunday at 2 AM with no more than 20% of instances patching simultaneously. What is the solution?

> Use **Patch Manager** with a **Maintenance Window** scheduled for Sunday 2 AM. Configure the Run Task with `MaxConcurrency: 20%` and `MaxErrors: 5%`. Tag instances with the appropriate Patch Group. Define a Patch Baseline with the required patch approvals.

**Scenario 4:** A Config rule detects an S3 bucket without encryption. The company wants automatic remediation. What is the solution?

> Configure **Config automatic remediation** using an **SSM Automation** document (`AWS-EnableS3BucketEncryption`). When Config detects a non-compliant bucket, it triggers the SSM Automation document to enable encryption automatically.

---

## 11. Exam Tips

- **Session Manager** = no SSH, no bastion, no open ports — always recommend over SSH
- **Parameter Store SecureString** = encrypted config; **Secrets Manager** = auto-rotation
- **Run Command** = ad-hoc scripts across fleets; **State Manager** = ongoing configuration enforcement
- **Patch Manager** = automated patching with maintenance windows and compliance reporting
- SSM Agent requires **IAM instance profile** — not just the agent installed
- For private instances: use **VPC endpoints** for SSM, SSMMessages, and EC2Messages
- **Automation** documents = the answer for Config remediation and complex operational tasks

---

---

## 🎯 Architecture Decision Notes

### Secrets rotation — how it actually works

Secrets Manager rotation is not magic — it is a Lambda function you configure (or use the AWS-provided templates for RDS, Redshift, DocumentDB):

1. **Staging labels:** rotation creates a new secret version labeled `AWSPENDING`, tests it, then promotes it to `AWSCURRENT`. The old version keeps `AWSPREVIOUS` until the next rotation. Applications should always read `AWSCURRENT` — never a hardcoded version.
2. **Rotation strategies:** single-user (one credential, brief downtime during swap) vs alternating-users (two credentials, zero downtime — the database flips between them). Choose alternating-users when the application cannot tolerate a failed connection during rotation.
3. **Rotation schedule:** automatic on a schedule (e.g., every 30 days) or on-demand. The Lambda function must have network access to the database (VPC configuration matters).
4. **Cross-account:** secrets can be shared via resource policies — but the rotation Lambda must still reach the target database.

**When Parameter Store is enough:** static configuration, non-rotating values, cost-sensitive fleets. **When Secrets Manager is required:** automatic rotation, RDS-native integration, cross-account secret sharing.

**Fresh example:** A payment service uses RDS with 30-day rotation. The alternating-users strategy means the app never sees a failed connection: while `AWSPENDING` is tested, the app keeps using `AWSCURRENT`; only after the new credential validates does the label flip. The rotation Lambda runs in the same VPC as the database — a Lambda in the wrong subnets is the most common rotation failure.

**Trap:** Assuming rotation is instant and atomic. There is a window where two credentials are valid; applications must handle the transition, not assume a single credential.

## 12. Official References

- [Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/)
- [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Patch Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html)
