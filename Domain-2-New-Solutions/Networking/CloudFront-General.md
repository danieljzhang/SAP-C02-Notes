# Amazon CloudFront — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). CloudFront is more than a CDN — it's a security layer, edge compute platform, and global distribution mechanism. Know origins, cache behaviors, OAC, and WAF integration.

---

## 1. Overview

Amazon CloudFront is a global content delivery network (CDN) with 400+ edge locations. It caches content close to users, reduces origin load, and integrates with WAF, Shield, and Lambda@Edge for security and edge compute.

---

## 2. Core Components

### Distributions

- **Web distribution:** HTTP/HTTPS content (static files, APIs, streaming)
- One distribution can have multiple origins and cache behaviors

### Origins

| Origin Type | Use Case |
|---|---|
| S3 bucket | Static websites, file downloads |
| ALB / NLB | Dynamic content, APIs |
| EC2 / custom HTTP | Any HTTP server |
| API Gateway | Serverless APIs |
| MediaStore / MediaPackage | Video streaming |

### Cache Behaviors

- Rules that map URL path patterns to origins and cache settings
- Default behavior (`*`) catches all unmatched paths
- More specific paths take precedence (e.g., `/api/*` before `*`)

```
Distribution
├── /api/*  → ALB origin (no caching, forward all headers)
├── /static/* → S3 origin (cache 1 year, compress)
└── *       → ALB origin (default behavior)
```

---

## 3. Origin Access Control (OAC)

Restricts S3 bucket access to CloudFront only — prevents direct S3 access:

```
User → CloudFront → S3 (OAC signs requests with SigV4)
User → S3 directly → 403 Forbidden (bucket policy denies direct access)
```

OAC replaces the older Origin Access Identity (OAI). Use OAC for new distributions.

S3 bucket policy:
```json
{
  "Principal": {"Service": "cloudfront.amazonaws.com"},
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/DIST_ID"
    }
  }
}
```

---

## 4. Caching

### Cache Key

Determines what makes a unique cached object. By default: URL path only.

Customize with **Cache Policies:**
- Include query strings, headers, cookies in cache key
- More cache key components = lower cache hit rate

### TTL

- **Default TTL:** 86,400 seconds (24 hours)
- **Min TTL / Max TTL:** bounds for origin Cache-Control headers
- **Cache-Control: no-cache** → CloudFront still caches but validates with origin on every request
- **Cache-Control: no-store** → CloudFront does not cache

### Invalidation

- Remove cached objects before TTL expires
- `/*` invalidates all objects (costs money per path after first 1,000/month)
- Use versioned file names (e.g., `app.v2.js`) instead of invalidation for better cache management

---

## 5. Security

### HTTPS

- **Viewer protocol policy:** HTTP only, HTTPS only, or redirect HTTP to HTTPS
- **Origin protocol policy:** HTTP only, HTTPS only, or match viewer
- Custom SSL certificate via ACM (must be in **us-east-1** for CloudFront)

### AWS WAF

- Attach a WAF Web ACL to a CloudFront distribution
- Rules evaluated at edge before reaching origin
- Protects against: SQL injection, XSS, rate limiting, geo-blocking, IP reputation

### AWS Shield

- **Shield Standard:** automatic DDoS protection for all CloudFront distributions (free)
- **Shield Advanced:** enhanced DDoS protection, cost protection, 24/7 DRT support

### Geo Restriction

- Allow or block requests from specific countries
- Implemented at CloudFront level — not origin level

### Signed URLs and Signed Cookies

| Feature | Use Case |
|---|---|
| Signed URL | Restrict access to a single file |
| Signed Cookie | Restrict access to multiple files (e.g., premium content) |

Both use a CloudFront key pair to sign the URL/cookie with an expiry time.

---

## 6. Lambda@Edge and CloudFront Functions

| Feature | Lambda@Edge | CloudFront Functions |
|---|---|---|
| Triggers | Viewer request/response, origin request/response | Viewer request/response only |
| Max execution time | 5s (viewer) / 30s (origin) | 1ms |
| Use cases | Auth, A/B testing, dynamic routing | URL rewrites, header manipulation |
| Cost | Higher | Lower (1/6th the cost) |

Use **CloudFront Functions** for simple, high-volume transformations. Use **Lambda@Edge** for complex logic.

---

## 7. Exam Scenarios

**Scenario 1:** A company hosts a static website on S3 and wants to prevent direct S3 access while serving content via CloudFront. What is the solution?

> Create a CloudFront distribution with the S3 bucket as origin. Configure **Origin Access Control (OAC)**. Update the S3 bucket policy to allow only the CloudFront service principal with the distribution ARN. Block all public S3 access.

**Scenario 2:** A company wants to serve different content to users in different countries and block access from sanctioned countries. What is the solution?

> Use CloudFront **Geo Restriction** to block requests from specific countries. For serving different content by country, use **Lambda@Edge** on the viewer request trigger to inspect the `CloudFront-Viewer-Country` header and redirect or rewrite the request.

**Scenario 3:** A company's API is being hit with a DDoS attack. They want to rate-limit requests at the edge. What is the solution?

> Attach an **AWS WAF Web ACL** to the CloudFront distribution with a rate-based rule. Set the rate limit (e.g., 2,000 requests per 5 minutes per IP). Requests exceeding the limit are blocked at the edge before reaching the origin.

**Scenario 4:** A media company wants to restrict video content to paying subscribers. What is the solution?

> Use **CloudFront Signed Cookies**. When a user authenticates and pays, issue a signed cookie with an expiry time. CloudFront validates the cookie on each request. Signed cookies work for multiple files (entire video library) unlike signed URLs (single file).

---

## 8. Exam Tips

- **OAC** replaces OAI — use OAC for new S3 origins
- ACM certificate for CloudFront **must be in us-east-1** — regardless of distribution region
- **CloudFront Functions** for simple/fast logic; **Lambda@Edge** for complex logic
- WAF on CloudFront = protection at the **edge** — before traffic reaches your origin
- **Signed URLs** = single file; **Signed Cookies** = multiple files
- Cache behaviors are matched **most specific first** — order matters
- CloudFront is **global** — you cannot restrict it to a single region

---

---

## 🎯 Architecture Decision Notes

### TLS has two legs — secure both

A CloudFront distribution terminates two separate TLS connections:
viewer→CloudFront and CloudFront→origin. A padlock in the browser proves
only the first leg. Configure certificates and the origin protocol policy
for the second leg explicitly, or "HTTPS everywhere" is fiction past the edge.

- ACM certificates for CloudFront must live in **us-east-1**; regional
  services need the certificate in their own Region.
- A CloudFront **504** means the origin failed to respond in time — check
  origin health, DNS, security rules, timeouts, and origin capacity before
  blaming the edge.
- **Lambda@Edge vs CloudFront Functions:** different runtimes, events, and
  limits — pick the lightest that satisfies the requirement and verify
  current restrictions.

---

## 🎯 Architecture Decision Notes

### Origin groups — fail over before the user notices

A CloudFront 504 means the edge waited too long for your origin — the fix isn't always "make the origin faster." An **origin group** pairs a primary origin with a secondary origin, and CloudFront automatically retries the secondary when the primary returns configured error codes (500, 502, 503, 504 by default). Failover happens inside the CDN: no DNS change, no client retry logic.

Decision rules:

- **Use an origin group** when the primary origin can degrade (overloaded ALB, deploy windows, regional impairment) and a degraded-but-working fallback exists — typically a static version of the site in S3, or a secondary region's ALB.
- **Design the secondary for graceful degradation:** it doesn't have to serve the full app. A static "we're having trouble, try again shortly" page or a cached catalog served from S3 beats a 504 every time.
- **Failover is per-request, not a mode switch:** CloudFront keeps sending new requests to the primary and only diverts the ones that fail. Recovery is automatic when the primary heals.
- Traps: the secondary origin must be able to serve the *same paths* (or your error pages) — a fallback that 404s on every path is worse than useless. And don't confuse this with multi-region active-active: origin groups are a resilience pattern for one distribution, not a traffic-splitting design.

**Fresh example:** A retail site runs its storefront on an ALB in us-east-1. During a flash sale the backend saturates and starts returning 504s. With an origin group configured (ALB primary, S3 bucket with a static "sale is hot, we're scaling — retry in a minute" page as secondary, failover on 502/503/504), shoppers see a branded holding page instead of a browser error, and full service resumes automatically as the backend recovers.

---

## 🎯 Architecture Decision Notes

### Field-level encryption — protect the payload, not just the pipe

TLS encrypts data in transit, but once CloudFront terminates TLS, the request body is plaintext inside your stack — visible to every hop, log, and service that touches it. **Field-level encryption** encrypts specific sensitive fields of POST requests *at the edge*, using a public key you configure on the distribution. The values stay encrypted through your entire pipeline; only the application holding the private key can decrypt them.

Decision rules:

- **Use it** when end users submit data that must stay confidential from intermediate systems — payment details, national IDs, health information — especially when the request passes through services you don't fully control (third-party processors, shared logging pipelines).
- **Scope tightly:** you nominate up to 10 fields per distribution. Encrypt the sensitive fields (card number, SSN), not the whole form — over-encryption makes debugging miserable and gains nothing.
- **It's a complement to, not a replacement for, HTTPS:** you still need TLS on both legs (viewer→edge, edge→origin). Field-level encryption answers "who inside my stack may see this field," TLS answers "who on the wire may see anything."
- Trap: key management is yours — the public key lives on the distribution, the private key must be distributed only to the decrypting applications, with rotation planned. Losing the private key means losing the data.

**Fresh example:** A loan-application form collects income and tax-ID numbers, then posts through CloudFront to an ALB fronting a shared application platform whose access logs are visible to a broad ops team. With field-level encryption on the two sensitive fields, the ops team sees ciphertext in every log and the loan-processing service alone decrypts with its private key — compliance is satisfied without re-architecting the shared platform.

## 9. Official References

- [CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/)
- [Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [CloudFront Functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html)
- [WAF with CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-awswaf.html)
