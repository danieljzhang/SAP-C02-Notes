# VPC Advanced — SAP-C02 Study Notes

> Exam relevance: Domain 1 (26%), Domain 2 (29%). Advanced VPC features are tested heavily at the professional level. Know VPC sharing, PrivateLink, endpoints, and peering limitations.

---

## 1. VPC Endpoints

### Gateway Endpoints

- Free; route-table based
- Supported services: **S3** and **DynamoDB** only
- Traffic stays within AWS network — does not leave to internet
- Add a route in the route table pointing to the endpoint

### Interface Endpoints (AWS PrivateLink)

- ENI with private IP in your VPC subnet
- Supports 100+ AWS services and custom services
- Billed per hour + per GB
- DNS resolves service hostname to private IP within VPC
- Supports endpoint policies to restrict access

```
VPC
├── Subnet
│   └── Interface Endpoint ENI (private IP: 10.0.1.50)
│           └── PrivateLink → AWS Service (e.g., Secrets Manager)
└── Route Table (no change needed — DNS-based)
```

### Gateway Load Balancer Endpoints

- Used for inline traffic inspection (third-party appliances)
- Integrates with Gateway Load Balancer for transparent bump-in-the-wire inspection

---

## 2. VPC Sharing (Resource Access Manager)

VPC sharing allows multiple accounts to use subnets from a **single VPC** owned by one account (the owner).

```
Network Account (VPC Owner)
└── VPC: 10.0.0.0/16
    ├── Subnet A (10.0.1.0/24) → shared with Account B
    └── Subnet B (10.0.2.0/24) → shared with Account C

Account B: launches EC2 in Subnet A (10.0.1.0/24)
Account C: launches RDS in Subnet B (10.0.2.0/24)
```

**Benefits:**
- Fewer VPCs to manage and peer
- Centralized network management
- Resources in shared subnets can communicate privately
- Reduces Transit Gateway attachment costs

**Limitations:**
- Participants cannot modify the VPC, subnets, route tables, or security groups owned by the owner
- Participants can create their own security groups in the shared VPC

---

## 3. VPC Peering

- Direct private connectivity between two VPCs (same or different accounts/regions)
- **Not transitive** — if A peers with B and B peers with C, A cannot reach C through B
- No bandwidth limit, no single point of failure
- Supports cross-account and cross-region peering
- Route tables must be updated in both VPCs

**When to use:** Small number of VPCs (< 10), simple connectivity requirements
**When not to use:** Large-scale hub-and-spoke (use Transit Gateway instead)

---

## 4. AWS PrivateLink (Custom Services)

PrivateLink allows you to expose a service in your VPC to other VPCs or accounts without VPC peering or internet exposure.

```
Service Provider VPC
└── NLB → EC2/ECS service
        └── VPC Endpoint Service (PrivateLink)

Consumer VPC (same or different account)
└── Interface Endpoint → connects to Endpoint Service
        └── Private IP in consumer subnet
```

**Use cases:**
- SaaS providers exposing services to customers
- Sharing internal microservices across accounts
- Replacing VPC peering for service-to-service communication

---

## 5. Security Groups and NACLs

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance (ENI) | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and Deny |
| Evaluation | All rules evaluated | Rules evaluated in order (lowest number first) |
| Default | Deny all inbound, allow all outbound | Allow all inbound and outbound |

**Key difference:** NACLs are stateless — return traffic must be explicitly allowed. Security groups are stateful — return traffic is automatically allowed.

---

## 6. VPC Flow Logs

- Capture IP traffic metadata for VPC, subnet, or ENI
- Destinations: CloudWatch Logs, S3, Kinesis Data Firehose
- Does not capture: DNS queries, DHCP, instance metadata, Windows license activation
- Use Athena to query flow logs stored in S3

```
Flow log record fields:
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```

---

## 7. DNS in VPC

- **enableDnsSupport:** enables DNS resolution via Route 53 Resolver (169.254.169.253)
- **enableDnsHostnames:** assigns public DNS hostnames to instances with public IPs
- Both must be enabled for interface endpoints to use private DNS names
- **Route 53 Resolver:** handles DNS for VPC, on-premises forwarding rules, and inbound/outbound endpoints

---

## 8. Exam Scenarios

**Scenario 1:** A company wants EC2 instances in a private subnet to access S3 without going through the internet or NAT Gateway. What is the solution?

> Create a **Gateway Endpoint** for S3 in the VPC. Add a route in the private subnet route table pointing to the endpoint. Traffic to S3 stays within the AWS network. Gateway endpoints are free.

**Scenario 2:** A company has 30 accounts and wants to minimize the number of VPCs while keeping network management centralized. What is the approach?

> Use **VPC sharing** via RAM. Create VPCs in a central Network account. Share subnets to workload accounts. Each workload account deploys resources into shared subnets without managing their own VPC.

**Scenario 3:** A SaaS company wants to expose their API to customer AWS accounts without requiring VPC peering or internet exposure. What is the solution?

> Use **AWS PrivateLink**. Deploy an NLB in front of the API. Create a VPC Endpoint Service. Customers create Interface Endpoints in their VPCs pointing to the Endpoint Service. Traffic never leaves the AWS network.

**Scenario 4:** A security team needs to inspect all traffic leaving a VPC before it reaches the internet. What is the architecture?

> Deploy **AWS Network Firewall** in an inspection VPC. Use a Gateway Load Balancer Endpoint to redirect egress traffic through the firewall. Route all outbound traffic from the VPC through the GWLB endpoint before reaching the NAT Gateway/internet gateway.

---

## 9. Exam Tips

- **Gateway endpoints** (S3, DynamoDB) are free; **interface endpoints** cost money
- VPC peering is **not transitive** — use TGW for hub-and-spoke
- VPC sharing reduces VPC sprawl — one VPC, multiple accounts
- PrivateLink does not require VPC peering, IGW, NAT, or VPN
- Both `enableDnsSupport` and `enableDnsHostnames` must be true for private DNS on interface endpoints
- NACLs are **stateless** — always configure both inbound and outbound rules
- Flow logs capture **metadata** not packet contents — use for troubleshooting, not DLP

---

---

## 🎯 Architecture Decision Notes

### Choosing the connection pattern — start from what must be reachable

| Pattern | Reachability model | Key constraint |
|---|---|---|
| VPC peering | Two VPCs routed directly | **Not transitive**; needs non-overlapping CIDRs |
| Transit Gateway | Regional hub for many VPCs/networks | Route tables need deliberate design; still needs non-overlapping CIDRs for routed traffic |
| PrivateLink | One *service* exposed privately, no VPC routing | Consumer never routes into provider VPC — **works with overlapping CIDRs** |
| Gateway endpoint | S3/DynamoDB via route-table entry | Only supported services |
| Interface endpoint | ENIs for many AWS/PrivateLink services | Per-AZ ENIs, security groups, endpoint policies |

The exam's favorite trap: overlapping CIDRs with a "connect the VPCs"
requirement. Peering and Transit Gateway cannot fix overlapping address
space for routed traffic — PrivateLink can, because no routes are exchanged.

**Fresh example:** Two acquired companies both use `10.0.0.0/16`; one needs
the other's catalog API. Publish the API as a PrivateLink endpoint service
(typically behind an NLB) instead of attempting a network merger. Retire the
link after re-addressing without touching the caller's endpoint pattern.

An endpoint is reachability, not permission: combine it with security
groups, endpoint policies, and the destination service's own authorization.

---

## 🎯 Architecture Decision Notes

### Forward proxy vs NAT Gateway — control versus plumbing

A NAT Gateway is dumb plumbing: it lets private-subnet resources reach the internet, with no say over *what* they reach. A **forward proxy** (e.g., a Squid fleet behind an internal load balancer) is a policy checkpoint for outbound web traffic — and the exam reaches for it whenever the requirement is about *which* destinations are allowed, not just *whether* the internet is reachable.

Choose a forward proxy when you need:

- **URL/domain allow-listing** — e.g., "EC2 instances may only reach approved vendor update sites." A NAT Gateway cannot express this; a proxy enforces it per request.
- **Centralized outbound logging** — one place recording every HTTP/HTTPS request for audit and compliance, far richer than flow logs.
- **Content filtering/caching** — block categories of sites, cache repeated downloads (OS patches, dependencies).

Tradeoffs: a proxy is usually HTTP/HTTPS-only (NAT handles all IP protocols), and unless you use a Marketplace appliance or firewall service, you're managing the fleet's HA and patching yourself. Also remember the client side: workloads must be configured to use the proxy (proxy env vars or transparent redirection) — traffic doesn't flow through it by default the way it does through a NAT route.

Mini-example: a compliance rule says factory-floor instances may download PLC firmware only from `updates.vendor.com` and all outbound web requests must be logged. Answer: forward proxy fleet in a public subnet with a domain allow-list and request logging, default routes removed so nothing bypasses it — NAT Gateway alone fails both the allow-list and the logging requirement.

## 10. Official References

- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
- [VPC Sharing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html)
- [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
