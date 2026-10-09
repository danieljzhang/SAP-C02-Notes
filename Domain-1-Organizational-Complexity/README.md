# Domain 1: Design Solutions for Organizational Complexity (26%)

> Highest-weight domain after Domain 2. Focuses on multi-account governance, enterprise networking, and cross-account identity. Expect 17 questions.

---

## What This Domain Tests

- Designing multi-account AWS environments using Organizations and Control Tower
- Implementing governance guardrails with SCPs and Resource Control Policies
- Architecting enterprise-scale networking with Transit Gateway, Direct Connect, and VPN
- Managing cross-account identity with IAM Identity Center and IAM roles
- Sharing resources across accounts with RAM
- Enforcing compliance with Config and Security Hub

---

## Subdomain Breakdown

| Subdomain | Key Services |
|---|---|
| 1.1 Architect network connectivity strategies | Transit Gateway, Direct Connect, VPN, VPC, Route 53 |
| 1.2 Prescribe security controls | IAM, SCPs, RCPs, Config, Security Hub, GuardDuty |
| 1.3 Design reliable and resilient architectures | Multi-AZ, multi-region, Route 53 failover |
| 1.4 Design a multi-account AWS environment | Organizations, Control Tower, RAM, IAM Identity Center |

---

## Files in This Domain

### Organizations/
- [Organizations-General.md](./Organizations/Organizations-General.md) — AWS Organizations, OUs, account management
- [SCPs-General.md](./Organizations/SCPs-General.md) — Service Control Policies, guardrails, deny strategies

### ControlTower/
- [ControlTower-General.md](./ControlTower/ControlTower-General.md) — Landing zones, account factory, guardrails
- [LandingZone-General.md](./ControlTower/LandingZone-General.md) — Landing zone design, customizations, AFT

### IAM/
- [IAM-General.md](./IAM/IAM-General.md) — IAM roles, policies, cross-account trust, federation
- [IAM-IdentityCenter-General.md](./IAM/IAM-IdentityCenter-General.md) — SSO, permission sets, multi-account access
- [ResourceControlPolicies-General.md](./IAM/ResourceControlPolicies-General.md) — RCPs, data perimeter, vs SCPs

### Networking/
- [TransitGateway-General.md](./Networking/TransitGateway-General.md) — Hub-and-spoke, inter-region peering, route tables
- [DirectConnect-General.md](./Networking/DirectConnect-General.md) — Dedicated/hosted connections, VIFs, resilience
- [VPN-General.md](./Networking/VPN-General.md) — Site-to-Site VPN, Client VPN, accelerated VPN
- [VPC-Advanced-General.md](./Networking/VPC-Advanced-General.md) — VPC sharing, PrivateLink, peering, endpoints

### Standalone
- [RAM-General.md](./RAM-General.md) — Resource Access Manager, cross-account sharing
- [Config-General.md](./Config-General.md) — Config rules, conformance packs, aggregators
- [SecurityHub-General.md](./SecurityHub-General.md) — Findings aggregation, standards, automation
- [Route53-General.md](./Route53-General.md) — Routing policies, health checks, hybrid DNS, Resolver

---

## Key Architecture Patterns

### Multi-Account Landing Zone
```
Management Account
└── AWS Organizations
    ├── Root
    ├── Security OU
    │   ├── Log Archive Account
    │   └── Audit Account
    ├── Infrastructure OU
    │   └── Network Hub Account (Transit Gateway)
    ├── Sandbox OU
    └── Workloads OU
        ├── Dev Account
        ├── Staging Account
        └── Prod Account
```

### Hub-and-Spoke Networking
```
On-Premises
    └── Direct Connect / VPN
            └── Transit Gateway (Network Hub Account)
                    ├── VPC Spoke 1 (Dev)
                    ├── VPC Spoke 2 (Prod)
                    └── VPC Spoke 3 (Shared Services)
```

---

## Exam Tips

- SCPs **deny** — they never grant permissions. An explicit deny in an SCP overrides any IAM allow
- RCPs restrict what **resources** accept — SCPs restrict what **principals** can do
- Control Tower uses SCPs + Config rules as guardrails — know the difference between preventive and detective
- Transit Gateway supports inter-region peering — VPC peering does not scale to hundreds of VPCs
- Direct Connect does **not** encrypt by default — use MACsec or VPN over DX for encryption
- IAM Identity Center is the preferred solution for multi-account human access — not individual IAM users
- RAM shares resources within an Organization without needing cross-account IAM roles for every resource

---

## Common Exam Traps

- Confusing SCP scope (applies to all principals in the OU **except** the management account)
- Using VPC peering instead of Transit Gateway for large-scale hub-and-spoke
- Forgetting that Direct Connect failover requires a second connection or VPN backup
- Mixing up IAM Identity Center permission sets with IAM roles (they provision IAM roles behind the scenes)
- Assuming Config rules are preventive — they are detective only (use SCPs or IAM for prevention)
