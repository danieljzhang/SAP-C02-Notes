# AWS Transit Gateway — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). Transit Gateway is the standard answer for hub-and-spoke networking at scale. Know route tables, inter-region peering, and multicast.

---

## 1. Overview

AWS Transit Gateway (TGW) is a regional network transit hub that connects VPCs and on-premises networks through a central gateway. It replaces complex VPC peering meshes with a scalable hub-and-spoke model.

**Problems it solves:**
- VPC peering does not scale (N*(N-1)/2 connections for N VPCs)
- Separate VPN connections per VPC
- No transitive routing between peered VPCs
- Complex on-premises connectivity to multiple VPCs

---

## 2. Core Concepts

### Attachments

| Attachment Type | Connects |
|---|---|
| VPC attachment | VPC to TGW |
| VPN attachment | Site-to-Site VPN to TGW |
| Direct Connect Gateway attachment | DX Gateway to TGW |
| TGW peering attachment | TGW in another region or account |
| Connect attachment | SD-WAN/third-party appliances via GRE |

### Route Tables

- TGW has one or more route tables
- Each attachment is associated with one route table (for inbound routing)
- Each attachment can propagate routes to one or more route tables
- Default: all attachments share one route table (full mesh)
- Custom: separate route tables per environment (Dev, Prod, Shared) for isolation

### Associations vs Propagations

- **Association:** which route table an attachment uses to look up routes (one per attachment)
- **Propagation:** which route tables an attachment advertises its CIDR into (can be multiple)

---

## 3. Architecture Patterns

### Basic Hub-and-Spoke

```
                    Transit Gateway
                   /       |        \
              VPC-Dev   VPC-Prod   VPC-Shared
                                      |
                                  On-Premises
                                (VPN or DX)
```

All VPCs route through TGW. On-premises connects once to TGW and reaches all VPCs.

### Isolated Routing (Security Segmentation)

```
TGW Route Table: Dev
  ← VPC-Dev (associated)
  ← VPC-Shared (propagated)

TGW Route Table: Prod
  ← VPC-Prod (associated)
  ← VPC-Shared (propagated)

TGW Route Table: Shared
  ← VPC-Shared (associated)
  ← VPC-Dev (propagated)
  ← VPC-Prod (propagated)
```

Dev and Prod cannot reach each other — they only see Shared Services. Shared Services can reach both.

### Centralized Egress (Cost Optimization)

```
Spoke VPCs → TGW → Egress VPC (NAT Gateway) → Internet
```

One NAT Gateway in the Egress VPC serves all spoke VPCs. Reduces NAT Gateway costs.

### Centralized Inspection (Security)

```
Spoke VPCs → TGW → Inspection VPC (AWS Network Firewall / third-party) → TGW → Destination
```

All east-west and north-south traffic inspected centrally. Use appliance mode on TGW attachment to keep flows symmetric.

### Inter-Region Peering

```
TGW us-east-1 ←──── TGW Peering ────→ TGW eu-west-1
      │                                       │
   VPCs (US)                              VPCs (EU)
```

- Static routes only (no dynamic route propagation across peering)
- Traffic stays on AWS backbone (not public internet)
- Supports cross-account peering

---

## 4. Key Features

| Feature | Detail |
|---|---|
| Max VPC attachments | 5,000 per TGW |
| Max route tables | 20 per TGW (default) |
| Bandwidth per VPC attachment | Up to 50 Gbps burst |
| Equal-cost multi-path (ECMP) | Supported for VPN (multiple tunnels) |
| Multicast | Supported — requires multicast domain |
| Appliance mode | Keeps flow symmetric for stateful appliances |
| Sharing via RAM | Share TGW across accounts in the organization |

---

## 5. Sharing with RAM

TGW can be shared across accounts using AWS Resource Access Manager (RAM):

```
Network Hub Account
  └── Creates TGW
  └── Shares TGW via RAM to organization or specific accounts

Spoke Accounts
  └── Accept RAM share
  └── Create VPC attachments to shared TGW
```

This avoids creating a TGW in every account and centralizes routing management.

---

## 6. VPN over Transit Gateway

- Attach Site-to-Site VPN to TGW instead of individual VGWs
- Supports ECMP across multiple VPN tunnels for higher bandwidth
- Each VPN connection has 2 tunnels (active/active with ECMP or active/passive)
- Accelerated VPN: routes VPN traffic over AWS Global Accelerator for lower latency

---

## 7. Monitoring

- **CloudWatch metrics:** BytesIn, BytesOut, PacketsIn, PacketsOut, PacketDropCountBlackhole
- **VPC Flow Logs:** enable on TGW attachments for traffic visibility
- **Transit Gateway Network Manager:** global view of TGW topology, on-premises connectivity, and health

---

## 8. Exam Scenarios

**Scenario 1:** A company has 50 VPCs across 3 regions and needs full connectivity between all VPCs and on-premises. VPC peering is too complex to manage. What is the solution?

> Deploy a TGW in each region. Connect VPCs to the regional TGW. Peer the TGWs across regions. Connect on-premises via Direct Connect or VPN to the TGW in the primary region. Use static routes for inter-region TGW peering.

**Scenario 2:** A company needs Dev and Prod VPCs to share a Shared Services VPC but must not communicate with each other. How?

> Create separate TGW route tables for Dev and Prod. Associate Dev VPC with the Dev route table and Prod VPC with the Prod route table. Propagate Shared Services VPC routes into both Dev and Prod route tables. Do not propagate Dev routes into Prod route table or vice versa.

**Scenario 3:** A company wants to inspect all traffic between VPCs using a third-party firewall appliance. How?

> Deploy the firewall appliance in an Inspection VPC. Create a TGW route table that routes all inter-VPC traffic through the Inspection VPC. Enable **appliance mode** on the Inspection VPC TGW attachment to ensure symmetric routing for stateful inspection.

**Scenario 4:** A company has multiple VPN connections to on-premises and wants to maximize bandwidth. What TGW feature helps?

> Enable **ECMP** on the TGW. Create multiple VPN attachments with multiple tunnels. TGW distributes traffic across all active tunnels using ECMP, effectively aggregating bandwidth.

---

## 9. Exam Tips

- TGW supports **transitive routing** — VPC peering does not
- **Appliance mode** is required for stateful inspection appliances — without it, asymmetric routing breaks stateful firewalls
- Inter-region TGW peering uses **static routes only** — no BGP propagation across regions
- Share TGW via **RAM** — do not create a TGW in every account
- **ECMP** requires BGP and equal-cost paths — works with VPN, not VPC attachments
- TGW does **not** support IPv6 inter-region peering (verify current status)
- Default route table = full mesh — use custom route tables for isolation

---

## 10. Official References

- [Transit Gateway User Guide](https://docs.aws.amazon.com/vpc/latest/tgw/)
- [TGW Routing](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html)
- [Centralized Inspection Architecture](https://aws.amazon.com/blogs/networking-and-content-delivery/centralized-inspection-architecture-with-aws-gateway-load-balancer-and-aws-transit-gateway/)
- [TGW Network Manager](https://docs.aws.amazon.com/network-manager/latest/tgwnm/)
