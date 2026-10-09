# AWS IAM Identity Center — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). IAM Identity Center is the AWS-recommended solution for workforce identity across multiple accounts. Know permission sets, identity sources, and attribute-based access control.

---

## 1. Overview

AWS IAM Identity Center (formerly AWS SSO) provides centralized access management for human users across all AWS accounts in an organization and business applications. It replaces the need for individual IAM users in every account.

**Problems it solves:**
- Managing IAM users in dozens of accounts
- Inconsistent access controls across accounts
- No single sign-on for AWS console and CLI
- Complex SAML federation setup per account

---

## 2. Core Concepts

### Identity Sources

| Source | Use Case |
|---|---|
| IAM Identity Center directory (built-in) | Small teams, no existing IdP |
| Active Directory (AWS Managed AD or AD Connector) | Enterprise with on-premises AD |
| External IdP (Okta, Azure AD, Ping, etc.) via SAML 2.0 / SCIM | Enterprise with existing cloud IdP |

Only **one identity source** can be active at a time. Changing it requires careful planning.

### Permission Sets

- Define the IAM policies that users/groups get when they access an account
- Deployed as IAM roles in each assigned account automatically
- Can reference AWS managed policies, inline policies, or customer managed policies
- Support permission boundaries

```
Permission Set: "DevReadOnly"
  └── AWS Managed Policy: ReadOnlyAccess
  └── Inline Policy: Allow CloudWatch:GetMetricData

Assigned to: Dev OU accounts → Group: "developers"
Result: IAM role "AWSReservedSSO_DevReadOnly_xxxx" created in each Dev account
```

### Assignments

- Map **users or groups** to **accounts** with a **permission set**
- Can assign at the account level or use attribute-based access control (ABAC)
- Changes propagate automatically to all assigned accounts

---

## 3. Architecture

```
Identity Source (AD / Okta / Built-in)
        │
        ▼
IAM Identity Center
        │
        ├── Permission Sets (define IAM policies)
        │
        └── Assignments (user/group → account → permission set)
                │
                ▼
        IAM Roles provisioned in each member account
                │
                ▼
        User authenticates → selects account → assumes role → accesses AWS
```

---

## 4. SCIM Provisioning

- Automatic user/group sync from external IdP to IAM Identity Center
- Supported by Okta, Azure AD, OneLogin, and others
- Eliminates manual user management — users created/deleted in IdP sync automatically
- Requires SCIM endpoint and bearer token from IAM Identity Center

---

## 5. Attribute-Based Access Control (ABAC)

Use user attributes (department, team, cost center) from the identity source to control access dynamically.

```json
{
  "Condition": {
    "StringEquals": {
      "aws:PrincipalTag/Department": "${aws:ResourceTag/Department}"
    }
  }
}
```

- Reduces the number of permission sets needed
- Users automatically get access to resources tagged with their department
- Attributes flow from IdP → IAM Identity Center → session tags → IAM conditions

---

## 6. CLI and Programmatic Access

IAM Identity Center supports temporary credentials for CLI access:

```bash
# Configure SSO profile
aws configure sso
# SSO start URL: https://your-domain.awsapps.com/start
# SSO region: us-east-1
# Account ID and role selected interactively

# Login
aws sso login --profile my-profile

# Use profile
aws s3 ls --profile my-profile
```

The AWS CLI v2 natively supports IAM Identity Center. Credentials are short-lived and automatically refreshed.

---

## 7. Integration with Other Services

| Service | Integration |
|---|---|
| AWS Organizations | Manages access across all organization accounts |
| AWS Managed AD / AD Connector | Uses on-premises AD as identity source |
| CloudTrail | All sign-in and access events logged |
| CloudWatch | Monitor authentication events |
| Lake Formation | ABAC with Identity Center attributes for data lake access |
| Amazon QuickSight | Federated access via Identity Center |

---

## 8. Security Considerations

- Enable **MFA** in IAM Identity Center for all users (supports TOTP, FIDO2/WebAuthn)
- Use **SCIM** to ensure users removed from IdP lose AWS access immediately
- Apply **permission boundaries** to permission sets for extra guardrails
- Use **ABAC** to reduce permission set sprawl
- Audit access via **CloudTrail** — look for `sso.amazonaws.com` events
- IAM Identity Center is a **regional service** but access works globally — deploy in your primary region

---

## 9. Exam Scenarios

**Scenario 1:** A company has 30 AWS accounts and uses Okta as their IdP. They want developers to access the AWS console with their Okta credentials without creating IAM users. What is the solution?

> Configure IAM Identity Center with Okta as the external IdP using SAML 2.0. Enable SCIM for automatic user/group provisioning. Create permission sets for developer access. Assign developer groups to the appropriate accounts with the developer permission set.

**Scenario 2:** A company wants to give each team access only to AWS resources tagged with their team name, without creating a separate permission set per team. What is the best approach?

> Use ABAC in IAM Identity Center. Configure team attributes in the identity source. Create a single permission set with IAM conditions that match `aws:PrincipalTag/Team` to `aws:ResourceTag/Team`. Assign all teams to the same permission set.

**Scenario 3:** A developer needs temporary CLI access to multiple AWS accounts. What is the recommended approach?

> Use IAM Identity Center with AWS CLI v2. Run `aws configure sso` to set up profiles for each account. Use `aws sso login` to authenticate. The CLI automatically uses short-lived credentials from IAM Identity Center.

**Scenario 4:** A company is migrating from individual IAM users in each account to IAM Identity Center. What is the migration path?

> (1) Set up IAM Identity Center with the appropriate identity source. (2) Create permission sets matching existing IAM user permissions. (3) Assign users/groups to accounts with permission sets. (4) Verify access. (5) Delete individual IAM users. Use IAM Access Analyzer to identify any remaining direct IAM user dependencies.

---

## 10. Exam Tips

- IAM Identity Center is the **preferred** solution for multi-account human access — not individual IAM users
- Permission sets are **deployed as IAM roles** in each assigned account — they are not IAM Identity Center-specific constructs
- Only **one identity source** at a time — plan the migration carefully
- SCIM ensures **just-in-time deprovisioning** — critical for security
- ABAC reduces permission set sprawl — one permission set can serve many teams
- IAM Identity Center works with **Organizations** — it can manage access to all accounts automatically
- For **machine/application** access, still use IAM roles — Identity Center is for humans

---

## 11. Official References

- [IAM Identity Center User Guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/)
- [SCIM Provisioning](https://docs.aws.amazon.com/singlesignon/latest/userguide/provision-automatically.html)
- [ABAC with IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/abac.html)
- [CLI v2 SSO Integration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
