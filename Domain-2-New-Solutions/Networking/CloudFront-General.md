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

## 9. Official References

- [CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/)
- [Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [CloudFront Functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html)
- [WAF with CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-awswaf.html)
