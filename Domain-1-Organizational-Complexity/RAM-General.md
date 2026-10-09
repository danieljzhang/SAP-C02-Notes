# AWS Resource Access Manager (RAM) — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%). RAM is the mechanism for sharing AWS resources across accounts without cross-account IAM roles. Know what can be shared and how it integrates with Organizations.

---

## 1. Overview

AWS Resource Access Manager (RAM) lets you share AWS resources with other AWS accounts, OUs, or your entire organization — without duplicating resources or creating complex cross-account IAM setups.

**Problems it solves:**
- Duplicating resources (e.g., Transit Gateway) in every account
- Complex cross-account IAM role chains for shared infrastructure
- Inconsistent shared resource configurations across accounts

---

## 2. Commonly Shared Resources

| Resource | Use Case |
|---|---|
| Transit Gateway | Central network hub shared to all spoke accounts |
| VPC Subnets | VPC sharing — workload accounts deploy into shared subnets |
| Route 53 Resolver Rules | Share DNS forwarding rules to all accounts |
| AWS License Manager configurations | Share license rules across accounts |
| Aurora DB clusters | Share read replicas across accounts |
| AWS Glue Data Catalog | Share data catalog across accounts |
| AWS Network Firewall policies | Share firewall policies |
| Resource Groups | Share resource groups |
| EC2 Capacity Reservations | Share reserved capacity across accounts |

> Verify the current full list at https://docs.aws.amazon.com/ram/latest/userguide/shareable.html

---

## 3. How RAM Works

```
Owner Account
  └── Creates resource (e.g., TGW)
  └── Creates Resource Share in RAM
        ├── Adds resource (TGW ARN)
        ├── Sets principals (account IDs, OU ARNs, or org ARN)
        └── Enables sharing

Consumer Account
  └── Receives invitation (if outside org) or auto-accepts (if within org)
  └── Uses shared resource (e.g., creates VPC attachment to shared TGW)
```

### Within an Organization

- Enable **trusted access** for RAM in Organizations
- Shares are **automatically accepted** by member accounts — no invitation needed
- Share to entire org, specific OUs, or specific accounts

### Outside an Organization

- Consumer account receives an invitation
- Must explicitly accept before using the resource

---

## 4. Key Behaviors

- The **owner** retains full control of the resource — consumers cannot modify or delete it
- Consumers can use the resource but cannot see other consumers
- Deleting a resource share does not delete the resource itself
- RAM shares are **regional** — resources must be in the same region as the share (except some global resources)

---

## 5. Exam Scenarios

**Scenario 1:** A company wants all 50 accounts to use a single Transit Gateway managed by the network team. How?

> The network team creates the TGW in the Network Hub account. They create a RAM resource share for the TGW, scoped to the entire organization. Member accounts auto-accept the share and create VPC attachments to the shared TGW.

**Scenario 2:** A company wants workload accounts to deploy EC2 instances into centrally managed subnets without managing their own VPCs. How?

> Use VPC sharing via RAM. The network account creates a VPC and shares specific subnets to workload accounts. Workload accounts deploy EC2 instances into the shared subnets. The network account retains control of the VPC, route tables, and internet gateways.

**Scenario 3:** A company wants all accounts to use the same Route 53 Resolver forwarding rules for on-premises DNS resolution. How?

> Create Route 53 Resolver rules in the Network Hub account. Share the rules via RAM to the entire organization. Member accounts associate the shared rules with their VPCs automatically.

---

## 6. Exam Tips

- RAM is the correct answer when the question asks about **sharing resources across accounts** without duplication
- Sharing within an **organization** = auto-accept; sharing with **external accounts** = requires invitation
- The **owner** controls the resource — consumers only use it
- RAM shares are **regional** — check region alignment
- VPC subnet sharing via RAM is different from VPC peering — subnets are shared, not VPCs
- Route 53 Resolver rule sharing via RAM is the standard pattern for centralized hybrid DNS

---

## 7. Official References

- [RAM User Guide](https://docs.aws.amazon.com/ram/latest/userguide/)
- [Shareable Resources](https://docs.aws.amazon.com/ram/latest/userguide/shareable.html)
- [VPC Sharing with RAM](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html)
