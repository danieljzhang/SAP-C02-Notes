# Amazon API Gateway — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). API Gateway is the front door for serverless APIs. Know REST vs HTTP vs WebSocket, throttling, caching, and integration types.

---

## 1. API Types

| Type | Protocol | Use Case | Key Features |
|---|---|---|---|
| REST API | HTTP/HTTPS | Full-featured APIs | Caching, usage plans, request validation, API keys |
| HTTP API | HTTP/HTTPS | Low-latency, low-cost APIs | Simpler, cheaper, JWT auth, no caching |
| WebSocket API | WebSocket | Real-time bidirectional | Chat, gaming, live dashboards |

**REST API vs HTTP API:**

| Feature | REST API | HTTP API |
|---|---|---|
| Cost | Higher | ~70% cheaper |
| Latency | Higher | Lower |
| Caching | Yes | No |
| Usage plans / API keys | Yes | No |
| Request/response transformation | Yes (mapping templates) | Limited |
| Private integrations | Yes | Yes |
| JWT authorizer | Yes | Yes (native) |
| Lambda authorizer | Yes | Yes |

Use **HTTP API** for most new serverless APIs. Use **REST API** when you need caching, usage plans, or advanced request transformation.

---

## 2. Integration Types

| Type | Description |
|---|---|
| Lambda proxy | Passes entire request to Lambda; Lambda returns full response |
| Lambda custom | API Gateway transforms request/response using mapping templates |
| HTTP proxy | Passes request to HTTP endpoint as-is |
| HTTP custom | API Gateway transforms request/response |
| AWS service | Direct integration with AWS services (SQS, DynamoDB, SNS, etc.) |
| Mock | Returns a static response without backend |

**AWS service integration** (e.g., API Gateway → SQS directly) removes Lambda from the path — reduces cost and latency for simple operations.

---

## 3. Throttling

| Level | Default Limit |
|---|---|
| Account-level | 10,000 RPS, burst 5,000 |
| Stage-level | Configurable per stage |
| Method-level | Configurable per method |
| Usage plan | Per API key, per day/month/RPS |

- Requests exceeding limits return **429 Too Many Requests**
- Use **Usage Plans** + **API Keys** to throttle per customer/client
- Throttling is applied in order: usage plan → method → stage → account

---

## 4. Caching (REST API only)

- Cache responses at the stage level
- TTL: 0–3,600 seconds (default 300)
- Cache capacity: 0.5 GB to 237 GB
- Cache key: method + path + query strings + headers (configurable)
- Invalidate cache: `Cache-Control: max-age=0` header (requires IAM permission)

---

## 5. Authorization

| Method | Use Case |
|---|---|
| IAM authorization | AWS services, internal APIs (SigV4 signing) |
| Lambda authorizer (token) | JWT, OAuth, custom token validation |
| Lambda authorizer (request) | Header, query string, or context-based auth |
| Cognito User Pool authorizer | JWT from Cognito (REST API) |
| JWT authorizer | Any OIDC/OAuth2 provider (HTTP API native) |

**Lambda authorizer** returns an IAM policy that API Gateway caches (TTL configurable). Reduces Lambda invocations for repeated requests.

---

## 6. Deployment and Stages

- **Stage:** a named reference to a deployment (dev, staging, prod)
- **Stage variables:** key-value pairs used in integration URIs (e.g., different Lambda aliases per stage)
- **Canary deployments:** route a percentage of traffic to a new deployment before full rollout

```
Stage: prod
├── Canary: 10% → new deployment
└── 90% → current deployment
```

---

## 7. Private APIs

- Deploy API Gateway inside a VPC using a **VPC Endpoint (Interface Endpoint)**
- Only accessible from within the VPC or connected networks
- Resource policy controls which VPCs/accounts can access the private API

---

## 8. Exam Scenarios

**Scenario 1:** A company wants to expose a serverless API with per-customer rate limiting and monthly quotas. What is the solution?

> Use **REST API** with **Usage Plans** and **API Keys**. Create a usage plan with throttle (RPS) and quota (requests/month) settings. Associate API keys with the usage plan. Each customer uses their API key, and API Gateway enforces their individual limits.

**Scenario 2:** A company wants to reduce costs for a high-volume internal API that does not need caching or usage plans. What is the recommendation?

> Use **HTTP API** instead of REST API. HTTP API is ~70% cheaper, has lower latency, and supports Lambda and HTTP integrations. For internal APIs with JWT-based auth, HTTP API's native JWT authorizer is sufficient.

**Scenario 3:** A company wants API Gateway to write messages directly to SQS without invoking Lambda. What is the integration type?

> Use **AWS service integration** (non-proxy). Configure API Gateway to call the SQS `SendMessage` API directly using a mapping template to format the request. This removes Lambda from the path, reducing cost and latency.

**Scenario 4:** A company needs an API accessible only from within their VPC. What is the solution?

> Create a **Private REST API**. Create a VPC Interface Endpoint for API Gateway in the VPC. Attach a resource policy to the API that allows access only from the VPC endpoint. The API is not accessible from the internet.

---

## 9. Exam Tips

- **HTTP API** = cheaper, faster, simpler — use for most new APIs
- **REST API** = needed for caching, usage plans, API keys, request transformation
- **WebSocket API** = real-time bidirectional — chat, live updates
- **AWS service integration** removes Lambda — use for simple pass-through to SQS, DynamoDB, SNS
- **Lambda authorizer** caches the IAM policy — reduces repeated Lambda invocations
- **429** = throttled; **502** = bad gateway (Lambda error); **504** = timeout
- API Gateway timeout: **29 seconds** maximum — Lambda must respond within 29s

---

## 10. Official References

- [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/)
- [HTTP API vs REST API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Usage Plans](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html)
- [Private APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html)
