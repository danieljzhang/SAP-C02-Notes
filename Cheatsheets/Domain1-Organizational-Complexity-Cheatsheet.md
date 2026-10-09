# Domain 1 Cheatsheet: Design Solutions for Organizational Complexity

> Focus: governance, identity, multi-account operations, hybrid connectivity, and shared services. Use the current exam guide for exact objective wording.

## Governance and Identity

| Need | Prefer | Remember |
|---|---|---|
| Account vending and governed multi-account baseline | AWS Organizations + Control Tower | Landing zone, OUs, account factory, centralized logging/security; plan customization and drift |
| Set maximum permissions for accounts/OUs | SCP | Does not grant permissions; IAM still must allow |
| Central workforce sign-in to many AWS accounts | IAM Identity Center | Permission sets provision roles; integrate identity source and lifecycle |
| Limit access to supported resources across an organization | RCP | Resource-side maximum guardrail; does not grant access |
| Share supported resources across accounts | AWS RAM | Share only needed resource types/scopes; understand owner/consumer responsibilities |
| Delegate security administration | Service delegated admin + Organizations | Keep management account use limited; centralize findings and response |
| Access AWS APIs from workloads/users | IAM role + temporary credentials | Prefer federation/roles over long-lived access keys |

- **Policy evaluation:** explicit deny wins. SCP/RCP set ceilings; identity/resource policies must still allow access.
- **Organization design:** separate accounts by environment, workload, and ownership; use OUs for policy inheritance without making OU depth unmanageable.
- **Break glass:** tightly controlled emergency access, tested, monitored, and independent of ordinary federation failure modes.

## Network and Hybrid Patterns

| Requirement | Pattern | Watch for |
|---|---|---|
| Many VPCs and on-premises sites | Transit Gateway hub-and-spoke | Route tables, propagation/association, segmentation, attachment scale, cross-Region design |
| Dedicated private connectivity | Direct Connect | Not encrypted by default; use VPN/IPsec over DX when encryption is required |
| Quick encrypted site connectivity | Site-to-Site VPN | Internet path can vary; design redundant tunnels/endpoints and routing |
| Centralized outbound/inbound inspection | Network Firewall / inspection VPC via TGW | Symmetric routing, stateful inspection, AZ failure behavior, capacity |
| Private access to AWS services | VPC endpoints | Gateway endpoints for S3/DynamoDB; interface endpoints use PrivateLink and have cost/DNS considerations |
| Hybrid DNS | Route 53 Resolver inbound/outbound endpoints | Resolver rules, forwarding direction, on-premises DNS, endpoint security groups |
| Global DNS failover | Route 53 health checks and routing policy | DNS TTL/caching means failover is not instant |

## Cross-Account Design Checklist

- Use IAM roles with explicit trust, external IDs where appropriate, and scoped session permissions.
- Centralize CloudTrail, Config, security findings, and log retention in dedicated accounts.
- Share networks/resources through approved mechanisms; avoid broad peering meshes.
- Design account bootstrap, identity, quota, and regional enablement before migrating workloads.
- Apply encryption/key policies across account boundaries and test the consumer role path end to end.

## Common Traps

- SCPs and RCPs **do not grant** access.
- Direct Connect is private connectivity, not automatically encrypted or redundant.
- VPC peering is non-transitive; do not use it as a scalable hub-and-spoke substitute.
- Control Tower guardrails do not replace workload-specific security architecture.
- A shared-services account is not a reason to centralize every workload permission or failure domain.
- Route 53 health checks and low TTLs cannot eliminate resolver/client caching delays.

## 30-Second Review

1. Identify account boundary and who owns the resource.
2. Separate authentication, authorization, and organization guardrails.
3. Pick hub-and-spoke or private service access based on traffic flow and segmentation.
4. Add resilience, encryption, centralized logging, and tested break-glass access.
5. Verify current policy behavior, service support, and quotas in official docs.

## Official References

- [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
- [Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html)
- [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
- [Direct Connect encryption](https://docs.aws.amazon.com/directconnect/latest/UserGuide/encryption-in-transit.html)
