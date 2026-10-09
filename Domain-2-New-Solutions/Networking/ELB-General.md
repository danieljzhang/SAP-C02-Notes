# Elastic Load Balancing — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). Know when to use ALB vs NLB vs GWLB. This is a frequent decision point in architecture scenarios.

---

## 1. Load Balancer Types

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP/TLS) | 3 (IP) |
| Routing | Path, host, header, query, method | IP, port | Transparent bump-in-wire |
| Static IP | No (use Global Accelerator) | Yes (Elastic IPs) | Yes |
| WebSocket | Yes | Yes | N/A |
| gRPC | Yes | No | N/A |
| TLS termination | Yes | Yes (TLS passthrough or terminate) | No |
| Health checks | HTTP/HTTPS | TCP/HTTP/HTTPS | HTTP/HTTPS |
| Use case | Web apps, microservices, containers | TCP/UDP, static IP, extreme performance | Inline traffic inspection |
| Target types | Instance, IP, Lambda | Instance, IP, ALB | Instance, IP |

---

## 2. Application Load Balancer (ALB)

### Routing Rules

- **Path-based:** `/api/*` → service A, `/web/*` → service B
- **Host-based:** `api.example.com` → service A, `app.example.com` → service B
- **Header-based:** route by HTTP header value
- **Query string:** route by query parameter
- **HTTP method:** route by GET/POST/etc.
- **Source IP:** route by client IP CIDR

### Target Groups

- Targets: EC2 instances, ECS tasks, Lambda functions, IP addresses
- Health check: HTTP/HTTPS on configurable path and port
- Stickiness: session affinity via cookie (duration-based or application-based)

### Authentication

- Native OIDC and Cognito authentication at the ALB level
- Redirects unauthenticated users to IdP — no application code changes needed

### Connection Draining (Deregistration Delay)

- Allows in-flight requests to complete before deregistering a target
- Default: 300 seconds; set to 0 for immediate deregistration

---

## 3. Network Load Balancer (NLB)

- Operates at Layer 4 — routes based on IP and port
- Handles millions of requests per second with ultra-low latency
- **Static IPs:** one per AZ — useful for whitelisting in firewalls
- **Elastic IPs:** assign your own EIPs to NLB for fixed IPs
- **TLS termination:** NLB can terminate TLS or pass through to targets
- **Preserve source IP:** targets see the client's real IP (unlike ALB which uses X-Forwarded-For)
- Supports **PrivateLink** — expose services to other VPCs/accounts

### NLB + ALB

NLB can target an ALB — combines static IPs (NLB) with advanced routing (ALB):

```
Client → NLB (static IP, whitelisted) → ALB (path/host routing) → targets
```

---

## 4. Gateway Load Balancer (GWLB)

- Transparent Layer 3 load balancer for inline traffic inspection
- Integrates with third-party virtual appliances (firewalls, IDS/IPS)
- Uses **GENEVE protocol** (port 6081) to encapsulate traffic
- Works with **Gateway Load Balancer Endpoints** in spoke VPCs

```
Spoke VPC → GWLB Endpoint → GWLB → Firewall Appliances → GWLB → destination
```

Traffic is inspected transparently — source and destination IPs are preserved.

---

## 5. Cross-Zone Load Balancing

| LB Type | Default |
|---|---|
| ALB | Enabled (always on, no charge) |
| NLB | Disabled (enable per target group, charged) |
| GWLB | Disabled (enable per target group, charged) |

With cross-zone enabled: each LB node distributes traffic evenly across all targets in all AZs.
Without: each LB node distributes only to targets in its AZ.

---

## 6. Access Logs and Monitoring

- **ALB/NLB access logs:** stored in S3; contain request details, response codes, latency
- **CloudWatch metrics:** RequestCount, TargetResponseTime, HTTPCode_Target_5XX, HealthyHostCount
- **AWS WAF:** attach to ALB for Layer 7 protection (not NLB)

---

## 7. Exam Scenarios

**Scenario 1:** A company needs to route traffic to different microservices based on the URL path. What load balancer?

> Use **ALB** with path-based routing rules. `/api/users/*` routes to the users service target group, `/api/orders/*` routes to the orders service target group.

**Scenario 2:** A company needs a load balancer with static IP addresses that can be whitelisted in customer firewalls. What is the solution?

> Use **NLB** with Elastic IPs assigned to each AZ. NLB provides static IPs per AZ. Customers whitelist these IPs in their firewalls.

**Scenario 3:** A company wants to inspect all inbound traffic with a third-party firewall before it reaches their application. What is the architecture?

> Use **Gateway Load Balancer** with the firewall appliances as targets. Deploy GWLB Endpoints in the application VPC. Route inbound traffic through the GWLB endpoint → GWLB → firewall → back to application. Traffic is inspected transparently.

**Scenario 4:** A company wants to add authentication to their web application without modifying application code. What is the solution?

> Use **ALB with built-in authentication**. Configure an OIDC or Cognito authentication action on the ALB listener rule. Unauthenticated requests are redirected to the IdP. Authenticated requests include user identity headers forwarded to the application.

---

## 8. Exam Tips

- **ALB** = Layer 7, content-based routing, WAF, auth — for web apps and microservices
- **NLB** = Layer 4, static IPs, ultra-low latency, PrivateLink — for TCP/UDP and IP whitelisting
- **GWLB** = transparent inspection — always paired with third-party security appliances
- NLB preserves **source IP** natively; ALB uses **X-Forwarded-For** header
- **WAF** attaches to ALB (and CloudFront) — not NLB or GWLB
- Cross-zone load balancing is **free on ALB**, **charged on NLB/GWLB**
- NLB can target an ALB — use this pattern for static IP + advanced routing

---

## 9. 🎯 Architecture Decision Notes

### NLB as the PrivateLink front door

When publishing a service via PrivateLink, the endpoint service is commonly
fronted by a **Network Load Balancer** — the NLB's static IPs and
TCP/UDP handling suit the PrivateLink model. (Gateway Load Balancer
endpoint services serve the network-appliance pattern instead.)

Choose the load balancer by traffic shape: ALB for HTTP(S) routing
decisions, NLB for extreme throughput / static IPs / non-HTTP protocols,
GWLB for inserting virtual appliances. The exam often hides the real
requirement in one adjective — "static IP," "preserve source IP," or
"inspect traffic" each points to a different answer.

---

### ACM certificates are regional — plan one per region for regional endpoints

An ACM certificate lives in exactly one region, and a regional service can only use a certificate from its own region. This bites in multi-region designs: an Application Load Balancer in eu-west-1 cannot use the certificate you requested in us-east-1 — you must request a separate certificate for the same domain in each region where an ALB (or NLB, or API Gateway) terminates TLS. CloudFront is the sole exception: because it is a global service, its viewer-facing certificate must be issued in us-east-1 and is then distributed to all edge locations.

Decision rules:

- **Multi-region ALB/NLB/API Gateway:** request (or import) the certificate in every region you terminate TLS in. Automate this — DNS-validated ACM requests are scriptable and renew automatically.
- **CloudFront in front:** one certificate in us-east-1 covers the viewer leg globally; each regional origin behind it still needs its own regional certificate if you run HTTPS to the origin.
- **Imported certificates:** same regional rule applies, except you now also own renewal everywhere — prefer ACM-issued certs unless compliance mandates your own CA.
- Trap: "the certificate works in us-east-1, so I'll reuse it in eu-west-1" is the classic exam wrong answer. The console won't even list a foreign-region certificate on an ALB listener.

**Fresh example:** A company runs its checkout API on ALBs in us-east-1 and eu-west-1 behind a CloudFront distribution. They hold three ACM certificates for `shop.example.com`: one in us-east-1 attached to the CloudFront distribution (viewer leg), one in us-east-1 attached to the us-east-1 ALB, and one in eu-west-1 attached to the eu-west-1 ALB (origin legs). One domain, three certificates — and zero renewal toil because all are ACM-managed.

## 10. Official References

- [ELB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)
- [ALB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [NLB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)
- [GWLB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/)
