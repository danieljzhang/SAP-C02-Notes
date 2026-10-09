# AWS Global Accelerator — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). Know when to use Global Accelerator vs CloudFront — a common exam trap.

---

## 1. Overview

AWS Global Accelerator improves availability and performance of applications by routing traffic through the AWS global network using **anycast static IP addresses**. It is not a CDN — it does not cache content.

---

## 2. How It Works

```
User → Nearest AWS Edge Location (anycast IP)
            └── AWS Global Network (private backbone)
                        └── Optimal endpoint (ALB, NLB, EC2, EIP)
```

- Two **static anycast IP addresses** assigned per accelerator (global, never change)
- Traffic enters the AWS network at the nearest edge location
- Routed over AWS private backbone — avoids public internet congestion
- Endpoints can be in multiple regions — Global Accelerator routes to the healthiest, lowest-latency endpoint

---

## 3. Global Accelerator vs CloudFront

| Feature | Global Accelerator | CloudFront |
|---|---|---|
| Content caching | No | Yes |
| Protocol | TCP, UDP | HTTP/HTTPS |
| Static IPs | Yes (2 anycast IPs) | No |
| Use case | Non-HTTP (gaming, IoT, VoIP), static IPs, TCP/UDP | HTTP/HTTPS content delivery, caching |
| Health checks | Yes (endpoint-level) | Via origin health |
| Failover | Automatic, ~30 seconds | Via Route 53 health checks |
| DDoS protection | Shield Standard | Shield Standard |
| Edge caching | No | Yes |

**Key rule:** If the question mentions **static IPs**, **TCP/UDP (non-HTTP)**, or **instant failover** → Global Accelerator. If it mentions **caching** or **HTTP content delivery** → CloudFront.

---

## 4. Traffic Dials and Endpoint Weights

- **Traffic dial:** percentage of traffic sent to an endpoint group (per region) — 0–100%
- **Endpoint weight:** relative weight within an endpoint group
- Use traffic dials for **blue/green deployments** or **gradual regional traffic shifts**

```
Accelerator
├── Endpoint Group: us-east-1 (traffic dial: 90%)
│   ├── ALB-1 (weight: 50)
│   └── ALB-2 (weight: 50)
└── Endpoint Group: eu-west-1 (traffic dial: 10%)
    └── ALB-3 (weight: 100)
```

---

## 5. Health Checks and Failover

- Global Accelerator performs health checks on endpoints every 10 or 30 seconds
- Unhealthy endpoints are automatically removed from routing
- Failover to healthy endpoint in same or different region in ~30 seconds
- No DNS TTL dependency — uses anycast IPs, so failover is near-instant

---

## 6. Exam Scenarios

**Scenario 1:** A gaming company needs low-latency UDP traffic routing to game servers in multiple regions with static IPs for firewall whitelisting. What is the solution?

> Use **Global Accelerator**. It supports UDP traffic, provides two static anycast IPs for whitelisting, and routes traffic over the AWS private backbone to the nearest healthy game server endpoint.

**Scenario 2:** A company wants to perform a blue/green deployment across two regions, gradually shifting traffic from us-east-1 to eu-west-1. What is the mechanism?

> Use **Global Accelerator traffic dials**. Set us-east-1 endpoint group to 100% and eu-west-1 to 0%. Gradually increase eu-west-1 dial while decreasing us-east-1. This shifts traffic without DNS changes or TTL delays.

**Scenario 3:** A company needs to serve static website content globally with caching. Should they use Global Accelerator or CloudFront?

> **CloudFront**. Global Accelerator does not cache content. CloudFront caches static content at edge locations, reducing origin load and improving latency for HTTP/HTTPS content.

---

## 7. Exam Tips

- Global Accelerator = **static IPs + TCP/UDP + non-HTTP + instant failover**
- CloudFront = **HTTP/HTTPS + caching + edge compute**
- Global Accelerator does **not cache** — it only routes
- **Anycast IPs** mean failover does not depend on DNS TTL — much faster than Route 53 failover
- Traffic dials = **regional traffic shifting**; endpoint weights = **within-region load distribution**
- Both Global Accelerator and CloudFront include **Shield Standard** DDoS protection

---

## 8. Official References

- [Global Accelerator Developer Guide](https://docs.aws.amazon.com/global-accelerator/latest/dg/)
- [Global Accelerator vs CloudFront](https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-how-it-works-client-ip.html)
- [Traffic Dials](https://docs.aws.amazon.com/global-accelerator/latest/dg/about-endpoint-groups-traffic-dial.html)
