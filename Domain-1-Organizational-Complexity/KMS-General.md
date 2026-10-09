# AWS KMS — SAP-C02 Study Notes

> Exam relevance: Domains 1 and 2. KMS underpins encryption across S3, EBS, RDS, and more. At the Professional level, focus on key types, the key policy vs grant distinction, and cross-account patterns — not just "KMS encrypts things."

---

## 1. Key Types

| Type | Control | Use when |
|---|---|---|
| **AWS-managed key** (`aws/<service>`) | AWS creates, manages, rotates | Default; no key policy control |
| **Customer-managed key (CMK)** | You define key policy, rotation, aliases | Cross-account access, audit requirements, custom rotation |
| **AWS-owned key** | Fully AWS-managed, invisible | Default S3-SSE, no configuration |

**Decision rule:** Use AWS-managed keys unless the scenario requires cross-account sharing, custom key policies, or CloudTrail audit of key usage — then it is a CMK. The exam often hides the requirement in "the security team must audit key usage" or "shared with another account."

---

## 2. Key Policies vs Grants

Both control key usage, but they solve different problems:

- **Key policy** (attached to the key): the primary authorization document. Defines who may use/manage the key. **Cross-account access REQUIRES the key policy to allow the external account** — IAM policies alone are insufficient.
- **Grants**: temporary, fine-grained delegations (e.g., an EC2 instance needs decrypt for 1 hour). Grants do not appear in the key policy and are ideal for dynamic, short-lived access.

**The cross-account rule:** For Account B to use Account A's CMK, both of the following are required:
1. Account A's **key policy** allows Account B (or B's role).
2. Account B's **IAM policy** allows the kms: actions.

Miss either one and the call fails. This is the exam's favorite KMS trap.

---

## 3. Envelope Encryption

KMS uses envelope encryption: a **data key** encrypts the data; the **data key** is encrypted by the KMS key. This means:
- The plaintext data key exists only in the caller's memory (`GenerateDataKey` returns it, but KMS never retains it) — only the encrypted data key travels/stores with the data.
- `GenerateDataKey` returns both plaintext (for immediate use) and ciphertext (for storage) versions.
- Rotation creates a new key version for future encryption operations; existing ciphertexts (and their data keys) stay encrypted under the old version, which KMS retains for decryption. Nothing is re-encrypted — that is why rotation is cheap.

---

## 4. Rotation and Deletion

- **Automatic rotation** (yearly) is available for CMKs using AWS-managed key material. Enable it unless compliance forbids.
- **Key deletion** has a 7–30 day waiting period — a safety net, not an instant action. During the wait, the key can be cancelled.
- **Imported key material**: you control rotation manually; useful for hybrid requirements but operationally heavier.

---

## 5. 🎯 Architecture Decision Notes

### The KMS questions the exam asks

1. **"Who can use this key across accounts?"** → Check the key policy FIRST. IAM policies are necessary but not sufficient.
2. **"How do we audit key usage?"** → CMK + CloudTrail (key usage is logged). AWS-managed keys offer less control (no key-policy edits, no manual rotation/disable/delete); usage still logs to CloudTrail — only AWS-owned keys are invisible in your account.
3. **"Temporary access for a service?"** → Grants, not key policy edits.
4. **"KMS vs CloudHSM?"** → KMS is multi-tenant managed; CloudHSM is single-tenant dedicated hardware for compliance regimes requiring exclusive key custody (and you manage the HSM cluster).

**Fresh example:** A SaaS platform stores each tenant's data with a separate CMK for billing isolation. Tenant onboarding creates a CMK with a key policy allowing the platform's role; offboarding schedules key deletion (30-day wait as a safety net). Cross-account analytics access requires BOTH the key policy update AND the analytics role's IAM policy — the runbook checks both.

---

### KMS vs CloudHSM — the custody question decides (detail)

Both manage encryption keys. The decider is **who holds the hardware**:

| | AWS KMS | AWS CloudHSM |
|---|---|---|
| Hardware | Multi-tenant, AWS-managed HSMs | **Single-tenant, dedicated** HSMs in your VPC |
| Management | AWS handles everything | You manage users, partitions, backups |
| Integration | Native: every AWS service, key policies, grants, automatic rotation | No KMS API — your apps use PKCS#11 / JCE / CNG |
| Compliance fit | Standard regulatory needs | "Dedicated hardware, not shared with other tenants" |
| Cost/ops | Pay per key + API calls, near-zero ops | Cluster hourly charge + real operational burden |

**Fresh example:** A bank's regulator requires encryption keys on
hardware "dedicated to the customer and not shared with any other tenant."
KMS — despite being the default right answer for 95% of key questions — is
wrong here. CloudHSM is the only fit, and the application team must budget
for HSM user management and client-side crypto integration.

### Trap callouts

- **"Dedicated" or "single-tenant"** in the requirement → CloudHSM,
  full stop. Everything else → KMS.
- **KMS key types matter:** symmetric (KMS does the crypto) vs asymmetric
  (sign/verify, TLS) vs HMAC. Match the key type to the operation; the exam
  loves a mismatched key type as a distractor.
- **Envelope encryption** is the pattern KMS enables: data key encrypts
  data, KMS key encrypts the data key. CloudHSM users build this
  themselves.
- **Key policies vs grants vs IAM policies:** the key policy is the
  primary authorization document for a KMS key — cross-account access
  needs the key policy to allow it, not just the caller's IAM policy.

---

## 6. Exam Tips

- Cross-account KMS = key policy + IAM policy. Both are required.
- Grants are for temporary/delegated access; key policies are the durable control.
- Envelope encryption means rotation does not re-encrypt all data.
- Deletion waiting period (7–30 days) is the safety net — the exam tests whether you know deletion is not instant.

---

## 7. Official References

- [AWS KMS key types](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html)
- [Key policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)
- [Grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html)
- [Cross-account access](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying-external-accounts.html)
