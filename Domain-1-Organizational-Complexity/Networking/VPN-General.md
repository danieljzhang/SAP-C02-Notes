# AWS VPN — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). VPN is the cost-effective hybrid connectivity option and the standard DX backup. Know Site-to-Site VPN, Client VPN, and accelerated VPN.

---

## 1. Overview

AWS offers two VPN services:
- **AWS Site-to-Site VPN** — connects on-premises networks to AWS VPCs over IPsec tunnels
- **AWS Client VPN** — allows individual users to connect to AWS or on-premises resources over OpenVPN

---

## 2. Site-to-Site VPN

### Components

| Component | Description |
|---|---|
| Virtual Private Gateway (VGW) | AWS-side VPN endpoint, attached to a VPC |
| Transit Gateway (TGW) | AWS-side VPN endpoint for multi-VPC connectivity |
| Customer Gateway (CGW) | Represents the on-premises VPN device in AWS |
| VPN Connection | IPsec tunnel between CGW and VGW/TGW |

### Tunnel Details

- Each VPN connection has **2 tunnels** for redundancy (different AWS endpoints)
- Both tunnels should be configured on the customer device
- Supports **static routing** or **dynamic routing (BGP)**
- BGP is recommended — enables automatic failover and route propagation

### Bandwidth and Limits

| Limit | Value |
|---|---|
| Max throughput per tunnel | ~1.25 Gbps |
| Max throughput per VPN connection (2 tunnels) | ~2.5 Gbps with ECMP |
| Max VPN connections per VGW | 10 |
| Max VPN connections per TGW | 5,000 |

### ECMP with Transit Gateway

- Attach multiple VPN connections to a TGW
- Enable ECMP on TGW
- Traffic is distributed across all active tunnels
- Effectively aggregates bandwidth beyond single-tunnel limits

```
On-Premises ──── VPN Connection 1 (2 tunnels) ──┐
On-Premises ──── VPN Connection 2 (2 tunnels) ──┤── TGW (ECMP) ──── VPCs
On-Premises ──── VPN Connection 3 (2 tunnels) ──┘
```

---

## 3. Accelerated Site-to-Site VPN

- Routes VPN traffic over **AWS Global Accelerator** instead of the public internet
- Reduces latency and improves consistency for geographically distant sites
- Requires TGW attachment (not VGW)
- Additional cost for Global Accelerator usage

```
On-Premises ──── Internet ──── AWS Global Accelerator ──── TGW ──── VPCs
                               (nearest AWS edge location)
```

---

## 4. Client VPN

### Overview

AWS Client VPN is a managed OpenVPN service for individual users to connect to AWS or on-premises resources.

### Components

| Component | Description |
|---|---|
| Client VPN endpoint | AWS-managed OpenVPN server, associated with a VPC subnet |
| Target network | VPC subnet associated with the endpoint |
| Authorization rules | Control which users can access which networks |
| Routes | Define what networks clients can reach |

### Authentication Options

| Method | Use Case |
|---|---|
| Active Directory (AWS Managed AD) | Enterprise users with existing AD |
| SAML-based IdP (Okta, Azure AD) | Federated identity |
| Certificate-based (mutual TLS) | Device authentication, no IdP |

### Split Tunneling

- **Disabled (default):** all client traffic goes through VPN (full tunnel)
- **Enabled:** only traffic destined for VPN routes goes through VPN; internet traffic goes directly

Split tunneling reduces VPN bandwidth usage and improves performance for internet-bound traffic.

---

## 5. VPN vs Direct Connect

| Feature | Site-to-Site VPN | Direct Connect |
|---|---|---|
| Setup time | Minutes to hours | Weeks |
| Cost | Low | High |
| Bandwidth | Up to ~2.5 Gbps (ECMP) | Up to 100 Gbps |
| Latency | Variable (internet) | Consistent, low |
| Encryption | Yes (IPsec) | No (requires MACsec or VPN over DX) |
| Reliability | Internet-dependent | Dedicated circuit |
| Use case | Backup, small sites, quick setup | Production, high bandwidth, compliance |

---

## 6. Hybrid Connectivity Patterns

### VPN as DX Backup

```
On-Premises ──── Direct Connect (primary, BGP preferred) ──── TGW
On-Premises ──── Site-to-Site VPN (backup, BGP less preferred) ──── TGW
```

Configure BGP AS_PATH prepending on VPN to make it less preferred. Automatic failover when DX goes down.

### VPN for Remote Sites

```
Branch Office 1 ──── VPN ──┐
Branch Office 2 ──── VPN ──┤── TGW ──── VPCs ──── HQ (via DX)
Branch Office 3 ──── VPN ──┘
```

Small branch offices use VPN; headquarters uses Direct Connect.

---

## 7. Exam Scenarios

**Scenario 1:** A company needs to connect 20 branch offices to AWS. Each office has low bandwidth requirements. What is the most cost-effective solution?

> Use Site-to-Site VPN for each branch office, all connecting to a single Transit Gateway. This is cost-effective and scalable. TGW supports up to 5,000 VPN connections.

**Scenario 2:** A company's Direct Connect connection fails. They need automatic failover to VPN. What must be configured?

> Create a Site-to-Site VPN attached to the same TGW as the DX connection. Configure BGP on both connections. Set the DX BGP route to be more preferred (lower AS_PATH). When DX fails, BGP automatically fails over to VPN routes.

**Scenario 3:** Remote employees need to access resources in a VPC and on-premises. What is the solution?

> Deploy AWS Client VPN. Associate it with a VPC subnet. Add routes for both the VPC CIDR and the on-premises CIDR. Configure authorization rules for the user groups. Enable split tunneling if internet traffic should not go through VPN.

**Scenario 4:** A company needs more than 1.25 Gbps of VPN throughput. What is the solution?

> Use multiple Site-to-Site VPN connections attached to a Transit Gateway with ECMP enabled. Each VPN connection adds up to 2.5 Gbps (2 tunnels). ECMP distributes traffic across all connections.

---

## 8. Exam Tips

- VPN tunnels are **always encrypted** (IPsec) — unlike Direct Connect
- Each VPN connection has **2 tunnels** — configure both on the customer device for redundancy
- **ECMP** requires TGW (not VGW) and BGP routing
- **Accelerated VPN** requires TGW — not available with VGW
- Client VPN supports **split tunneling** — reduces bandwidth and improves performance
- VPN over DX = IPsec tunnel over a public VIF — provides encryption for DX traffic
- For automatic DX failover, use **BGP** — static routing requires manual intervention

---

## 9. 🎯 Architecture Decision Notes

### Site-to-Site vs Client VPN — different problems

| | Site-to-Site VPN | Client VPN |
|---|---|---|
| Connects | Network to network (on-premises ↔ VPC) | Individual users → VPC resources |
| Protocol | IPsec tunnels | OpenVPN-based client |
| Use when | Branch office / data center connectivity | Remote workforce, contractors |

Do not answer a "remote employees need VPC access" requirement with
Site-to-Site VPN, and do not answer a "connect the data center" requirement
with Client VPN. For encrypted hybrid connectivity with higher throughput
and consistency, evaluate Direct Connect — optionally with a VPN overlay
when encryption in transit is required.

## 10. Official References

- [Site-to-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/)
- [Client VPN User Guide](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/)
- [Accelerated VPN](https://docs.aws.amazon.com/vpn/latest/s2svpn/accelerated-vpn.html)
- [VPN + DX Failover](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-direct-connect-site-to-site-vpn.html)
