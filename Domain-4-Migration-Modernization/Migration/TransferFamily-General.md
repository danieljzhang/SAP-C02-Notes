# AWS Transfer Family — SAP-C02 Study Notes

## 1. Purpose and Exam Relevance

AWS Transfer Family provides managed endpoints and interfaces for moving files into and out of AWS storage while preserving familiar file-transfer workflows. It supports SFTP, FTPS, FTP, AS2, and browser-based experiences for supported use cases, with Amazon S3 and Amazon EFS as storage backends. Use it when employees, customers, or business partners need a managed protocol-compatible interface; it is not a bulk NAS migration engine.

## 2. Core Patterns

```text
Partner / employee client
        | SFTP / FTPS / FTP / AS2 / web app
        v
AWS Transfer Family endpoint
        | identity + scoped access
        +-------------------+
        v                   v
      Amazon S3           Amazon EFS
        |
  EventBridge / Lambda / managed workflows
  -> validate, scan, transform, notify, archive
```

- **Server endpoint:** managed protocol endpoint associated with a selected storage backend and identity provider.
- **Identity:** service-managed users, directory integration, or a custom identity provider pattern, depending on protocol and configuration.
- **Logical directories and policies:** expose a controlled view of S3/EFS paths and restrict each user to the intended data boundary.
- **Managed workflows:** automate supported post-upload file-processing steps and provide workflow visibility.
- **Connectors:** support managed transfers to/from remote SFTP servers for supported workflows; validate exact connector capabilities separately from inbound server endpoints.
- **Web apps:** provide a managed browser-based interface for supported S3 access patterns.

## 3. When to Use It

| Requirement | Fit |
|---|---|
| Replace self-managed partner-facing SFTP servers | Transfer Family SFTP endpoint with scoped identity and storage access |
| B2B EDI over AS2 | Transfer Family AS2 capabilities, with certificates, agreements, and message processing designed explicitly |
| Internal file exchange over FTPS/FTP | Transfer Family where the legacy protocol requirement is mandatory; prefer secure protocols and retire plaintext FTP where possible |
| Bulk migration of a large NFS share | DataSync, not individual client sessions through Transfer Family |
| Shared file system for Linux/Windows applications | EFS/FSx directly or an appropriate hybrid service; Transfer Family is a file-transfer interface |
| Replace a database replication path | DMS, not Transfer Family |

## 4. Security and Network Design

- Select a public or VPC-hosted endpoint pattern based on partner reachability, private connectivity, address controls, and security requirements. Confirm current endpoint options before deployment.
- Use least-privilege identity mapping and scope each user to a home directory or logical path. Avoid granting broad bucket or file-system access through a shared role.
- Protect SSH keys, passwords, AS2 certificates, and custom identity-provider secrets. Rotate credentials and certificates; define expiry and partner offboarding.
- Use S3 bucket policies, access points where suitable, KMS key policies, EFS access points, and POSIX identities consistently with the selected backend.
- Enforce encryption in transit, encryption at rest, audit logging, and appropriate data retention. Record who uploaded/downloaded what and when; protect logs as sensitive metadata.
- For AS2, define partner identifiers, certificates, message signing/encryption expectations, receipts, retries, and duplicate-message handling.
- Consider malware scanning and content validation before downstream processing; a successful transfer does not mean the file is safe or well-formed.

## 5. Reliability, Operations, and Cost

- Managed endpoint fleets reduce server patching and scaling work, but identity integration, DNS, certificates, storage permissions, partner configuration, and file-processing workflows remain customer responsibilities.
- Design idempotent downstream processing: uploads may be retried, re-sent, or delivered more than once. Use object versions/keys, event deduplication, and quarantine paths where appropriate.
- Monitor server health, authentication failures, transfer errors, workflow outcomes, and storage capacity/requests. Alert on certificate expiry and failed partner deliveries.
- Use multiple Availability Zones where offered by the chosen configuration; verify endpoint type, Region, and resilience characteristics in current docs.
- Costs include endpoint hours, protocol/connector usage where applicable, data transfer, S3/EFS storage and requests, logs, and downstream processing. Compare with the operational cost of self-managed file servers.

## 6. Common Failure Modes

| Symptom | Checks |
|---|---|
| Partner cannot connect | DNS/host key, protocol/port, endpoint exposure, partner IP allowlists, firewall, and client cipher compatibility |
| Authentication succeeds but file access fails | Identity mapping, home directory, IAM role, bucket/file-system policy, KMS permissions, and POSIX permissions |
| File arrives but is not processed | Event configuration, workflow execution, permissions, file type/size constraints, and idempotency logic |
| AS2 messages fail | Certificates, partner IDs, signing/encryption agreement, message format, receipt path, and clock/certificate validity |
| Partner sees another tenant's data | Logical-directory mapping, policy scope, role session permissions, and test with negative authorization cases |

## 7. Original Scenario

**Scenario:** A manufacturer must keep existing suppliers' SFTP clients unchanged while replacing internet-facing file servers. Each supplier may access only its own inbound/outbound folders, and uploaded files must be scanned and routed to an ERP import process.

**Recommended approach:** Use Transfer Family SFTP endpoints backed by S3, a managed or custom identity provider, per-user logical directory mappings and least-privilege storage access. Route uploads through a quarantine/validation process, scan before release, then trigger idempotent processing and audit events. Test supplier isolation explicitly.

**Why not DataSync?** DataSync is optimized for managed dataset transfers between storage locations, not interactive partner sessions with per-user authentication and protocol compatibility.

## 8. Exam Tips

- Transfer Family = managed file-transfer protocols and interfaces to S3/EFS; DataSync = storage-to-storage migration/synchronization.
- Protocol compatibility can avoid changing partner clients, but it does not eliminate identity, network, certificate, or data-isolation design.
- Do not treat successful upload as validation, malware scanning, or business acceptance.
- FTP is not encrypted by itself; use a secure protocol or explicit compensating controls when a legacy requirement cannot be removed.

---

## 🎯 Architecture Decision Notes

### Transfer Family placement decisions

Decide the endpoint's network posture first: publicly reachable vs VPC-private.
A private endpoint keeps transfers off the internet but needs the VPC,
subnets, and security groups designed around the transfer clients. Then lock
down identities — the protocol (SFTP/FTPS/FTP/AS2) determines the auth
options, and the S3/EFS backend determines the IAM story. Measure the
throughput need against the protocol choice; managed does not mean
unlimited.

## 9. Official References

- [What is AWS Transfer Family?](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html)
- [Transfer Family identity providers](https://docs.aws.amazon.com/transfer/latest/userguide/authenticating-users.html)
- [Transfer Family managed workflows](https://docs.aws.amazon.com/transfer/latest/userguide/transfer-workflows.html)
- [Transfer Family pricing](https://aws.amazon.com/aws-transfer-family/pricing/)
