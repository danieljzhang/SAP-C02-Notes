# AWS X-Ray — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3. X-Ray is the distributed tracing service. Know service maps, sampling, annotations, and integration with Lambda, ECS, and API Gateway.

---

## 1. Overview

AWS X-Ray provides distributed tracing for applications. It tracks requests as they travel through microservices, Lambda functions, databases, and external APIs — helping identify latency bottlenecks and errors.

**Problems it solves:**
- Latency is high but you don't know which service is slow
- Errors occur but you can't trace them across services
- No visibility into downstream dependency performance

---

## 2. Core Concepts

| Concept | Description |
|---|---|
| Trace | End-to-end record of a request across all services |
| Segment | Data from a single service (one segment per service) |
| Subsegment | Granular data within a segment (DB call, HTTP call, custom) |
| Service map | Visual graph of services and their connections |
| Annotation | Key-value pair indexed for filtering (searchable) |
| Metadata | Key-value pair not indexed (for debugging, not filtering) |
| Sampling | Percentage of requests traced (reduces cost and overhead) |

---

## 3. Service Map

- Visual representation of your application's architecture
- Shows: services, connections, latency, error rates, throttle rates
- Color-coded: green (healthy), yellow (slow), red (errors)
- Drill down into any node to see traces

---

## 4. Sampling

- By default: first request per second + 5% of additional requests
- Custom sampling rules: define rate by service, URL, method, host
- Higher sampling = more visibility, higher cost
- Use **reservoir** (fixed number per second) + **rate** (percentage above reservoir)

```json
{
  "version": 2,
  "rules": [{
    "description": "High-value transactions",
    "host": "*",
    "http_method": "POST",
    "url_path": "/api/payment*",
    "fixed_target": 10,
    "rate": 0.5
  }]
}
```

---

## 5. Integration with AWS Services

| Service | Integration |
|---|---|
| Lambda | Built-in — enable Active Tracing in function config |
| API Gateway | Enable X-Ray tracing on stage |
| ECS | Add X-Ray daemon as sidecar container |
| EC2 | Install X-Ray daemon, instrument SDK |
| Elastic Beanstalk | Enable via configuration option |
| App Mesh | Native integration |
| SNS / SQS | Trace propagation via message attributes |

### Lambda Active Tracing

```yaml
# SAM template
Properties:
  Tracing: Active  # Enables X-Ray for Lambda
```

Lambda automatically creates a segment. Add subsegments for downstream calls using the X-Ray SDK.

---

## 6. X-Ray SDK

Instrument application code to create custom segments and subsegments:

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()  # Auto-instrument AWS SDK calls, HTTP requests

@xray_recorder.capture('process_order')
def process_order(order_id):
    # Creates a subsegment named 'process_order'
    xray_recorder.put_annotation('orderId', order_id)
    xray_recorder.put_metadata('orderDetails', {...})
    # ... business logic
```

---

## 7. X-Ray Groups

- Filter traces by expression (e.g., `service("payment-service") AND responsetime > 2`)
- Create CloudWatch alarms on group metrics
- Use for: SLO monitoring, alerting on specific service degradation

---

## 8. CloudWatch ServiceLens

- Unified view combining X-Ray traces, CloudWatch metrics, and CloudWatch Logs
- Service map with health indicators
- Drill from service map → traces → logs in one workflow
- Recommended interface for production observability

---

## 9. Exam Scenarios

**Scenario 1:** A microservices application has high latency but the team cannot identify which service is causing it. What is the solution?

> Enable **AWS X-Ray** across all services. Use the **service map** to visualize the request flow and identify which service has the highest response time. Drill into traces for that service to find the specific operation causing latency.

**Scenario 2:** A company wants to trace requests through API Gateway → Lambda → DynamoDB and identify slow DynamoDB queries. What is required?

> Enable X-Ray tracing on the **API Gateway stage**. Enable **Active Tracing** on the Lambda function. The AWS SDK for DynamoDB is automatically instrumented when using `patch_all()` in the X-Ray SDK. X-Ray creates subsegments for each DynamoDB call showing latency.

**Scenario 3:** A high-traffic application cannot afford to trace 100% of requests. How should sampling be configured?

> Configure **custom sampling rules** in X-Ray. Set a reservoir of 5 requests per second (always trace) plus 1% of additional requests. For critical paths (e.g., payment), set a higher rate. This balances visibility with cost.

---

## 10. Exam Tips

- **Annotations** = indexed, searchable — use for filtering traces (orderId, userId, environment)
- **Metadata** = not indexed — use for debugging details not needed for filtering
- **Active Tracing** on Lambda = one config change, no code changes needed
- X-Ray **daemon** is required for EC2 and ECS — it buffers and sends trace data to X-Ray
- **ServiceLens** = unified view of metrics + traces + logs — the recommended production interface
- X-Ray does not replace CloudWatch — they complement each other (metrics vs traces)
- **Sampling** is critical for high-traffic applications — 100% tracing is expensive

---

## 11. Official References

- [X-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)
- [X-Ray Sampling](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html)
- [CloudWatch ServiceLens](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ServiceLens.html)
- [X-Ray with Lambda](https://docs.aws.amazon.com/lambda/latest/dg/services-xray.html)
