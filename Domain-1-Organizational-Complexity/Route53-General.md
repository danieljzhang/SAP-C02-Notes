# Amazon Route 53 — SAP-C02 Study Notes

> Exam relevance: Domains 1, 2, 3. Route 53 is tested heavily for routing policies, health checks, hybrid DNS, and multi-region failover. Know every routing policy and when to use it.

---

## 1. Overview

Amazon Route 53 is a highly available, scalable DNS service that also provides domain registration, health checking, and traffic routing. At the SAP-C02 level, focus on routing policies, health checks, hybrid DNS with Resolver, and multi-region architectures.

---

## 2. Routing Policies

| Policy | Use Case | Key Behavior |
|---|---|---|
| Simple | Single resource, no health checks | Returns all values; client picks randomly |
| Weighted | A/B testing, gradual traffic shifts | Distributes traffic by weight (0–255) |
| Latency | Route to lowest-latency region | Based on AWS latency measurements, not geography |
| Failover | Active-passive DR | Routes to secondary when primary health check fails |
| Geolocation | Route by user's geographic location | Country, continent, or default |
| Geoproximity | Route by proximity with bias | Requires Traffic Flow; can shift traffic with bias value |
| Multivalue Answer | Simple load balancing with health checks | Returns up to 8 healthy records |
| IP-based | Route by client IP CIDR | Route specific IP ranges to specific endpoints |

### Weighted Routing — Traffic Shifting

```
Record A: weight 90 → us-east-1 (production)
Record B: weight 10 → us-west-2 (canary)
```

Set weight to 0 to stop sending traffic to a record without deleting it.

### Failover Routing

```
Primary record: us-east-1 ALB (health check attached)
Secondary record: us-west-2 ALB (failover target)

If primary health check fails → Route 53 returns secondary record
```

Requires health checks on the primary record. Secondary can be another region, S3 static site, or any endpoint.

---

## 3. Health Checks

| Type | Checks |
|---|---|
| Endpoint health check | HTTP/HTTPS/TCP to an IP or domain |
| Calculated health check | Combines multiple health checks (AND/OR logic) |
| CloudWatch alarm health check | Based on a CloudWatch alarm state |

- Health checkers are distributed globally — endpoint must be publicly accessible (or use CloudWatch alarm for private resources)
- For **private resources**: create a CloudWatch alarm on the resource metric → create a health check based on the alarm
- Health check interval: 30 seconds (standard) or 10 seconds (fast, higher cost)
- Failure threshold: 3 consecutive failures (default)

---

## 4. Private Hosted Zones

- DNS zones for internal resources within VPCs
- Not accessible from the internet
- Associate with one or more VPCs (same or different accounts)
- Cross-account: use CLI/SDK to associate a VPC from another account with a private hosted zone

```bash
# Associate VPC from another account with private hosted zone
aws route53 create-vpc-association-authorization \
  --hosted-zone-id Z1234 \
  --vpc VPCRegion=us-east-1,VPCId=vpc-xxxx

# From the other account
aws route53 associate-vpc-with-hosted-zone \
  --hosted-zone-id Z1234 \
  --vpc VPCRegion=us-east-1,VPCId=vpc-xxxx
```

---

## 5. Route 53 Resolver (Hybrid DNS)

### Inbound Endpoints

- Allow on-premises DNS servers to resolve AWS private hosted zone records
- DNS queries from on-premises → Inbound Endpoint IP → Route 53 Resolver → private hosted zone

### Outbound Endpoints

- Allow VPC resources to resolve on-premises DNS records
- DNS queries from VPC → Outbound Endpoint → Forwarding Rule → on-premises DNS server

### Resolver Rules

- **Forwarding rules:** forward specific domains to on-premises DNS
- **System rules:** Route 53 handles (AWS domains, private hosted zones)
- Share rules via RAM to all accounts in the organization

```
VPC (10.0.0.0/16)
  └── EC2 queries corp.example.com
        └── Route 53 Resolver
                └── Forwarding Rule: corp.example.com → 192.168.1.10 (on-prem DNS)
                        └── On-premises DNS resolves and returns IP
```

---

## 6. Multi-Region Architecture Patterns

### Active-Passive Failover

```
Route 53 Failover Policy
├── Primary: us-east-1 ALB (health check)
└── Secondary: us-west-2 ALB (failover)
```

### Active-Active Multi-Region

```
Route 53 Latency Policy
├── us-east-1 ALB (latency record)
└── eu-west-1 ALB (latency record)
```

Users routed to lowest-latency region. Both regions serve traffic simultaneously.

### Weighted Canary Deployment

```
Route 53 Weighted Policy
├── Weight 95: current version ALB
└── Weight 5: new version ALB
```

Gradually shift traffic to new version. Roll back by setting new version weight to 0.

---

## 7. Exam Scenarios

**Scenario 1:** A company has applications in us-east-1 and eu-west-1. Users should be routed to the region with the lowest latency. What routing policy?

> Use **Latency routing policy**. Create latency records for both regions. Route 53 routes each user to the region with the lowest measured latency. Combine with health checks to avoid routing to an unhealthy region.

**Scenario 2:** A company needs DNS resolution between on-premises and AWS VPCs. On-premises servers need to resolve records in a private hosted zone. What is the solution?

> Create Route 53 Resolver **inbound endpoints** in the VPC. Configure on-premises DNS servers to forward the AWS domain to the inbound endpoint IPs. Route 53 Resolver resolves the private hosted zone records and returns the answer.

**Scenario 3:** A company wants to route users in Germany to a Frankfurt endpoint and all other users to us-east-1. What routing policy?

> Use **Geolocation routing**. Create a record for Germany pointing to the Frankfurt endpoint. Create a default record pointing to us-east-1. Users in Germany get Frankfurt; all others get us-east-1.

**Scenario 4:** A company wants to test a new application version with 5% of traffic before full rollout. What routing policy?

> Use **Weighted routing**. Set the new version record weight to 5 and the current version to 95. Monitor error rates. Gradually increase the new version weight. Set to 0 to instantly roll back.

**Scenario 5:** A company needs health checks for a private ALB that is not publicly accessible. How?

> Create a **CloudWatch alarm** on the ALB's HealthyHostCount metric. Create a Route 53 **health check based on the CloudWatch alarm**. Route 53 uses the alarm state to determine endpoint health without needing direct access to the private ALB.

---

## 8. Exam Tips

- **Latency** routing ≠ geographically closest — it is based on measured network latency
- **Geolocation** routing requires a **default record** — requests from unmapped locations fail without it
- **Failover** routing requires health checks on the primary record
- **Multivalue Answer** is not a load balancer — it returns multiple healthy IPs for client-side selection
- For private resource health checks: use **CloudWatch alarm health check**
- Route 53 Resolver rules shared via **RAM** = centralized hybrid DNS for the whole organization
- **Geoproximity** requires Route 53 Traffic Flow (visual editor) — not available in standard console

---

---

## 🎯 Architecture Decision Notes

### DNS routing is not failover — respect the cache

Route 53 routing policies (weighted, latency, geolocation/geoproximity,
failover, multivalue) steer *name resolution*, not connections. Clients and
resolvers cache answers, so a DNS change is not an instant traffic shift.
For fast API failover, pair DNS with Global Accelerator or an ALB-level
mechanism; do not rely on short TTLs alone — not every resolver honors them.

- **Alias vs CNAME:** an Alias record targets AWS resources and works at the
  zone apex; a CNAME cannot live at the apex. Use A/AAAA for literal IPs.
- **Hybrid DNS:** Resolver inbound endpoints let on-premises query AWS names;
  outbound endpoints with forwarding rules let VPC workloads query external
  DNS selectively. Sharing a private hosted zone across accounts needs the
  VPC association plus RAM sharing or cross-account authorization.
- **DNSSEC** proves data integrity/authenticity; it does not encrypt queries.

## 9. Official References

- [Route 53 Developer Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/)
- [Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [Route 53 Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)
- [Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)
