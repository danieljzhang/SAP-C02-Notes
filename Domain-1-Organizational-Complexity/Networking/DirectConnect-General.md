# AWS Direct Connect — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). Direct Connect is the standard answer for dedicated, low-latency hybrid connectivity. Know connection types, VIFs, resilience patterns, and encryption.

---

## 1. Overview

AWS Direct Connect (DX) provides a dedicated private network connection from on-premises to AWS, bypassing the public internet. It offers consistent bandwidth, lower latency, and reduced data transfer costs compared to internet-based VPN.

**Problems it solves:**
- Inconsistent internet-based connectivity to AWS
- High data transfer costs over the internet
- Latency-sensitive workloads requiring predictable performance
- Compliance requirements for private connectivity

---

## 2. Connection Types

| Type | Speed | How |
|---|---|---|
| Dedicated connection | 1, 10, or 100 Gbps | Physical port at DX location, ordered directly from AWS |
| Hosted connection | 50 Mbps – 10 Gbps | Ordered through an APN partner; shares partner's port |

- **Dedicated:** longer lead time (weeks), full port, higher cost
- **Hosted:** faster provisioning, flexible bandwidth, shared infrastructure

---

## 3. Virtual Interfaces (VIFs)

VIFs are logical connections over a DX connection:

| VIF Type | Connects To | Use Case |
|---|---|---|
| Private VIF | VPC via Virtual Private Gateway (VGW) or DX Gateway | Access private resources in a VPC |
| Public VIF | AWS public endpoints (S3, DynamoDB, etc.) | Access AWS public services without internet |
| Transit VIF | Transit Gateway via DX Gateway | Access multiple VPCs via TGW |

### Direct Connect Gateway (DXGW)

- Connects a DX connection to multiple VPCs across **multiple regions** and **multiple accounts**
- One DXGW can connect to up to 10 VGWs or 3 TGWs
- Enables single DX connection to reach VPCs in any region

```
On-Premises
    └── DX Connection
            └── DX Gateway
                    ├── TGW us-east-1 (Transit VIF)
                    └── TGW eu-west-1 (Transit VIF)
```

---

## 4. Resilience Patterns

### Single Connection (No Resilience)

```
On-Premises ──── DX ──── AWS
```
Single point of failure. Not recommended for production.

### High Resilience (Recommended)

```
On-Premises ──── DX Location 1 ──── AWS
On-Premises ──── DX Location 2 ──── AWS
```
Two connections at two different DX locations. Survives single location failure.

### Maximum Resilience

```
On-Premises Site A ──── DX Location 1 ──── AWS
On-Premises Site A ──── DX Location 2 ──── AWS
On-Premises Site B ──── DX Location 1 ──── AWS
On-Premises Site B ──── DX Location 2 ──── AWS
```
Four connections across two sites and two DX locations. Highest availability.

### DX + VPN Backup

```
On-Premises ──── DX (primary) ──── AWS
On-Premises ──── VPN (backup) ──── AWS
```
Cost-effective resilience. VPN activates automatically if DX fails. Use BGP routing with higher preference for DX.

---

## 5. Encryption

**Direct Connect does NOT encrypt traffic by default.**

Options for encryption:
- **MACsec (802.1AE):** Layer 2 encryption on dedicated connections (1G, 10G, 100G). Encrypts at the physical layer between on-premises router and DX location.
- **VPN over DX:** IPsec VPN tunnel over a public VIF. Encrypts at Layer 3. Lower throughput than MACsec but works with hosted connections.

| Option | Layer | Throughput | Use Case |
|---|---|---|---|
| MACsec | L2 | Line rate | High-throughput, compliance-driven encryption |
| VPN over DX | L3 | Limited by VPN (~1.25 Gbps per tunnel) | Existing VPN infrastructure, hosted connections |

---

## 6. Bandwidth and BGP

- DX uses **BGP** for dynamic routing
- Use **BGP communities** to control route preference and traffic engineering
- **AS_PATH prepending:** make one path less preferred for failover
- **Local Preference:** set in on-premises router to prefer DX over VPN
- Multiple DX connections support **ECMP** for bandwidth aggregation (via TGW)

---

## 7. Monitoring

- **CloudWatch metrics:** ConnectionState, ConnectionBpsIngress, ConnectionBpsEgress, ConnectionPpsIngress
- **CloudTrail:** DX API calls (CreateConnection, DeleteVirtualInterface, etc.)
- **DX Connection State:** Available, Down, Deleted
- Set CloudWatch alarms on `ConnectionState` to detect DX failures

---

## 8. Exam Scenarios

**Scenario 1:** A company needs dedicated 10 Gbps connectivity from on-premises to AWS with no single point of failure. What is the recommended architecture?

> Order two dedicated 10 Gbps DX connections at two different DX locations. Connect both to a DX Gateway. Attach the DXGW to a Transit Gateway. This provides high resilience — a single DX location failure does not impact connectivity.

**Scenario 2:** A company has a DX connection but needs to encrypt all traffic for compliance. The connection is a hosted 1 Gbps connection. What is the encryption option?

> MACsec requires dedicated connections. For a hosted connection, use **VPN over DX** — establish an IPsec VPN tunnel over a public VIF. This encrypts traffic at Layer 3 while using the DX connection for transport.

**Scenario 3:** A company wants to use a single DX connection to access VPCs in us-east-1 and eu-west-1. What is the architecture?

> Create a DX Gateway. Create a Transit VIF from the DX connection to the DXGW. Attach TGWs in both us-east-1 and eu-west-1 to the DXGW. The single DX connection can now reach VPCs in both regions.

**Scenario 4:** A company's DX connection goes down. They need automatic failover to VPN. How?

> Create a Site-to-Site VPN attached to the same TGW or VGW. Configure BGP so DX routes have a lower AS_PATH (more preferred). When DX fails, BGP withdraws DX routes and VPN routes take over automatically.

---

## 9. Exam Tips

- DX does **not** encrypt by default — always ask about encryption requirements
- **MACsec** = dedicated connections only; **VPN over DX** = works with hosted connections
- DXGW enables **multi-region, multi-account** access from a single DX connection
- Transit VIF → DXGW → TGW is the modern pattern for large-scale connectivity
- Private VIF → VGW is the legacy pattern (one VPC per VIF)
- DX failover to VPN requires **BGP** — static routes do not support automatic failover
- **Hosted connections** are faster to provision but share partner infrastructure

---

---

## 🎯 Architecture Decision Notes

### DX is not encrypted by default — design for it

A Direct Connect connection is a private path, not an encrypted one. When
the requirement says "encrypted in transit," layer IPsec where supported:

- **Site-to-Site VPN over DX** adds IPsec encryption on the private path.
- **MACsec** is available on supported dedicated connections for
  line-rate encryption — verify current availability per location.
- A **Direct Connect gateway** associates the connection with virtual
  private/transit gateways; it is not a general router between everything
  attached to it.
- For availability: redundant connections on independent failure paths.
  One DX connection is a single point of failure no matter how big the pipe.

Virtual interface choice follows the destination: private VIF for VPC-bound
private traffic, public VIF for AWS public endpoints, transit VIF via a DX
gateway to Transit Gateway.

## 10. Official References

- [Direct Connect User Guide](https://docs.aws.amazon.com/directconnect/latest/UserGuide/)
- [DX Resilience Recommendations](https://docs.aws.amazon.com/directconnect/latest/UserGuide/resilency_toolkit.html)
- [MACsec on Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/MACsec.html)
- [DX Gateway](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-gateways.html)
