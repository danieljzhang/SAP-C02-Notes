# Resource Control Policies (RCPs) — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). RCPs are a newer Organizations policy type (2024). Know how they differ from SCPs and how they form the data perimeter.

---

## 1. Overview

Resource Control Policies (RCPs) are an Organizations policy type that set the maximum permissions available to **resources** in member accounts. While SCPs restrict what **principals** can do, RCPs restrict what **resources** can accept.

**RCPs enable the resource side of a data perimeter** — ensuring that even if a resource policy is misconfigured, resources cannot be accessed by untrusted principals or from untrusted networks.

---

## 2. RCPs vs SCPs

| Feature | SCP | RCP |
|---|---|---|
| Restricts | Principal actions | Resource access |
| Applied to | Account/OU (affects principals) | Account/OU (affects resources) |
| Affects management account | No | No |
| Affects service-linked roles | No | Yes |
| Affects resource-based policies | No | Yes |
| Use case | Prevent principals from doing things | Prevent resources from accepting untrusted access |
| Example | Deny ec2:TerminateInstances | Deny S3 access from outside the organization |

---

## 3. Data Perimeter Concept

A data perimeter has three dimensions:

| Dimension | Control Mechanism |
|---|---|
| Identity perimeter | Only trusted identities access your resources | SCP + IAM |
| Resource perimeter | Your identities only access trusted resources | SCP |
| Network perimeter | Access only from expected networks | VPC endpoints + resource policies |

RCPs enforce the **identity perimeter from the resource side** — resources reject requests from untrusted principals regardless of the resource policy.

```
External Principal (outside org)
        │
        ▼
Attempts to access S3 bucket in member account
        │
        ▼
RCP evaluation: Is principal from trusted org? No → DENY
        │
        ▼
Never reaches bucket policy evaluation
```

---

## 4. Common RCP Patterns

### Deny S3 Access from Outside the Organization

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyExternalS3Access",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": "*",
    "Condition": {
      "StringNotEqualsIfExists": {
        "aws:PrincipalOrgID": "o-xxxxxxxxxxxx"
      },
      "BoolIfExists": {
        "aws:PrincipalIsAWSService": "false"
      }
    }
  }]
}
```

### Require Encryption in Transit

```json
{
  "Sid": "DenyNonTLSAccess",
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "Bool": {"aws:SecureTransport": "false"}
  }
}
```

### Restrict KMS Key Usage to Organization

```json
{
  "Sid": "DenyKMSOutsideOrg",
  "Effect": "Deny",
  "Principal": "*",
  "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:PrincipalOrgID": "o-xxxxxxxxxxxx"
    }
  }
}
```

---

## 5. Supported Resources

RCPs currently support a subset of AWS services. Verify the current list at AWS documentation. Common supported services include:
- Amazon S3
- AWS KMS
- AWS STS
- Amazon SQS
- AWS Secrets Manager

> Always verify the current supported service list — this expands over time.

---

## 6. Exam Scenarios

**Scenario 1:** A company wants to ensure that S3 buckets in all member accounts can never be accessed by principals outside the organization, even if a bucket policy accidentally allows it. What is the solution?

> Attach an RCP at the root OU that denies S3 access when `aws:PrincipalOrgID` does not match the organization ID. This applies to all member accounts and overrides any permissive bucket policies.

**Scenario 2:** What is the difference between using an SCP and an RCP to protect S3 data?

> An SCP prevents principals **in the account** from performing S3 actions (e.g., making a bucket public). An RCP prevents the S3 **resource** from accepting requests from untrusted principals (e.g., external accounts). Both are needed for a complete data perimeter — SCPs protect from the inside out, RCPs protect from the outside in.

**Scenario 3:** A company needs to ensure KMS keys in all accounts can only be used by principals within the organization. What is the most scalable approach?

> Attach an RCP at the root OU that denies KMS Decrypt and GenerateDataKey when `aws:PrincipalOrgID` does not match the organization ID. This applies to all KMS keys in all member accounts without modifying individual key policies.

---

## 7. Exam Tips

- RCPs are the **resource-side complement** to SCPs — together they form a data perimeter
- RCPs do **not** apply to the management account (same as SCPs)
- RCPs **do** affect service-linked roles (unlike SCPs)
- RCPs work alongside resource-based policies — they add an additional layer of restriction
- Use `aws:PrincipalOrgID` condition to scope access to your organization
- RCPs are relatively new (2024) — verify supported services before the exam

---

## 8. Official References

- [Resource Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html)
- [Data Perimeter on AWS](https://aws.amazon.com/blogs/security/establishing-a-data-perimeter-on-aws/)
- [AWS Organizations Policy Types](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies.html)
