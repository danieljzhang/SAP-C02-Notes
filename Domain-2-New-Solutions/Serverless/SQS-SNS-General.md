# Amazon SQS and SNS — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). SQS and SNS are the decoupling backbone. Know Standard vs FIFO, fan-out patterns, DLQ, and when to use each.

---

## 1. Amazon SQS

### Queue Types

| Feature | Standard | FIFO |
|---|---|---|
| Throughput | Unlimited | 3,000 msg/s with batching, 300 without |
| Ordering | Best-effort | Strict (First-In-First-Out) |
| Delivery behavior | SQS standard queues are at-least-once; FIFO deduplicates sends within its deduplication interval and orders within a message group | Subscription delivery behavior depends on the protocol/endpoint; consumers should be idempotent |
| Deduplication | No | Yes (5-minute dedup window) |
| Use case | High throughput, order not critical | Order-critical, financial transactions |

### Key Settings

| Setting | Description |
|---|---|
| Visibility timeout | Time a message is hidden after being received (default 30s, max 12h) |
| Message retention | How long messages stay in queue (default 4 days, max 14 days) |
| Delivery delay | Delay before message is visible (default 0, max 15 min) |
| Max message size | 256 KB (use S3 + SQS Extended Client for larger) |
| Long polling | Wait up to 20s for messages — reduces empty responses and cost |

### Dead Letter Queue (DLQ)

- Messages that fail processing N times are moved to DLQ
- Set `maxReceiveCount` on the source queue's redrive policy
- DLQ must be same type as source (Standard → Standard DLQ, FIFO → FIFO DLQ)
- Use DLQ alarms to detect processing failures

### Lambda Integration

- Lambda polls SQS (event source mapping)
- Batch size: 1–10,000 messages
- Lambda scales up to 1,000 concurrent executions for Standard queues
- Failed batches: entire batch returned to queue (or sent to DLQ if configured)
- Use **bisect on error** to identify the failing message in a batch

---

## 2. Amazon SNS

### Overview

SNS is a pub/sub messaging service. Publishers send messages to **topics**; subscribers receive messages.

### Subscription Types

| Protocol | Use Case |
|---|---|
| SQS | Fan-out to queues for buffered processing |
| Lambda | Real-time processing |
| HTTP/HTTPS | Webhooks |
| Email | Notifications |
| SMS | Mobile alerts |
| Mobile push | iOS, Android, FireOS |
| Kinesis Firehose | Stream to S3, OpenSearch, Redshift |

### SNS FIFO

- Ordered delivery within a message group and send deduplication; consumers still need idempotent effects
- Only SQS FIFO queues can subscribe
- Lower throughput than Standard SNS

### Message Filtering

- Subscribers can filter messages using **filter policies** (JSON attribute matching)
- Only matching messages are delivered to that subscriber
- Reduces unnecessary processing

```json
{
  "eventType": ["order_placed", "order_cancelled"],
  "region": ["us-east-1"]
}
```

---

## 3. Fan-Out Pattern (SNS + SQS)

The standard pattern for delivering one event to multiple consumers:

```
Producer → SNS Topic
                ├── SQS Queue A → Lambda (order processing)
                ├── SQS Queue B → Lambda (inventory update)
                └── SQS Queue C → Lambda (notification service)
```

**Benefits:**
- Each consumer processes at its own pace (SQS buffers)
- Consumer failures don't affect other consumers
- Easy to add new consumers without changing the producer

---

## 4. SQS vs SNS vs EventBridge

| Feature | SQS | SNS | EventBridge |
|---|---|---|---|
| Pattern | Queue (pull) | Pub/Sub (push) | Event bus (push) |
| Consumers | One consumer per message | Multiple subscribers | Multiple targets per rule |
| Filtering | No | Message attributes | Rich event pattern matching |
| Ordering | FIFO option | FIFO option | No |
| Retention | Up to 14 days | No retention | No retention |
| Use case | Decoupling, buffering, work queues | Fan-out, notifications | Event routing, AWS service events |

---

## 5. Exam Scenarios

**Scenario 1:** An application processes orders. Each order must be processed exactly once and in the order received. What is the queue type?

> Use an **SQS FIFO queue** with a message group ID for the required ordering scope and a deduplication ID (or content-based deduplication) to suppress duplicate sends within the deduplication interval. Make the consumer idempotent and record completed order IDs with the business update: FIFO deduplication does not make arbitrary downstream side effects exactly-once.

**Scenario 2:** A company wants to send a single event to 5 different downstream services for processing. What is the pattern?

> Use **SNS fan-out**. Create an SNS topic. Subscribe 5 SQS queues to the topic. Each downstream service reads from its own SQS queue. This decouples producers from consumers and allows each service to process at its own rate.

**Scenario 3:** Messages in an SQS queue are failing to process. The company wants to preserve failed messages for analysis without losing them. What is the solution?

> Configure a **Dead Letter Queue (DLQ)**. Set `maxReceiveCount` to 3 on the source queue's redrive policy. After 3 failed processing attempts, messages are moved to the DLQ. Set a CloudWatch alarm on the DLQ's `ApproximateNumberOfMessagesVisible` metric.

**Scenario 4:** An SNS topic delivers to 10 SQS queues, but only 3 queues care about "payment" events. How to avoid unnecessary processing?

> Configure **SNS message filter policies** on the 3 payment-related SQS subscriptions. Only messages with the `eventType: payment` attribute are delivered to those queues. Other queues receive all messages or their own filtered subset.

---

## 6. Exam Tips

- **SQS** = pull-based, buffering, one consumer per message
- **SNS** = push-based, fan-out, multiple subscribers
- **FIFO** = ordered delivery within a message group plus send deduplication; make consumer effects idempotent because retries/redelivery can still occur
- **DLQ** = isolate repeatedly failing messages; configure redrive behavior to match replay and retention needs
- **Long polling** = reduce empty responses and cost for polling consumers; tune wait time to the workload
- **Visibility timeout** must be > Lambda function timeout to prevent duplicate processing
- Fan-out = SNS → multiple SQS queues — not SNS → multiple Lambda (Lambda can fail and lose messages)

---

---

## 🎯 Architecture Decision Notes

### DLQ design that actually helps

A dead-letter queue isolates poison messages for investigation — but only if
tuned deliberately:

- Set `maxReceiveCount` high enough to survive transient downstream outages.
  Too low, and recoverable messages get diverted on the first bad hour.
- A DLQ is not a fix; pair it with **alarms on DLQ depth**, a retention
  policy, and a runbook for the redrive operation that returns messages for
  reprocessing.
- Keep consumers **idempotent**: retries and redrives can deliver the same
  message twice, and at-least-once delivery makes "exactly once" the
  application's job, not the queue's.

Use SNS when the pattern is fan-out to many subscribers; use SQS when the
pattern is durable buffering for workers. A down SNS subscriber without a
durable subscription (e.g., an SQS endpoint) loses messages.

---

## 🎯 Architecture Decision Notes

### The five-way messaging decider — one pattern, one service

Every messaging question reduces to the consumption pattern. Learn this table
and most "which service" questions answer themselves:

| Pattern | Service | Decider word |
|---|---|---|
| Buffered work for workers to pull | **SQS** | *queue* — one message, one consumer group |
| One event to many subscribers | **SNS** | *fan-out* — pub/sub, no ordering guarantee |
| Events routed by pattern to many targets | **EventBridge** | *bus* — rules match events; producers never know consumers |
| Ordered stream, each consumer reads everything independently | **Kinesis Data Streams** | *stream* — consumer positions, replay |
| Kafka protocol / ecosystem | **MSK** | *Kafka* — lift-and-shift or Kafka tooling |

**Fresh mini-scenario:** An order service emits `OrderPlaced`. Inventory must
deduct exactly once in order; notifications go to email/SMS/push; a future
fraud service will need the event stream; analytics replays yesterday's
orders. The composition: EventBridge as the bus (new consumers attach via
rules, archive/replay for analytics), SQS FIFO for inventory (ordering +
dedup), SNS for notifications (fan-out). One service cannot do all four jobs
— the exam rewards the composition, not the single pick.

### Trap callouts

- **"Multiple consumers each need every message in order"** — SNS fan-out
  loses ordering and durability; SQS lets one consumer group eat the
  message. The answer is Kinesis (independent consumer positions) — or MSK
  if the word "Kafka" appears.
- **EventBridge vs SNS:** both fan out, but EventBridge routes by *event
  pattern* with archive/replay and 20+ AWS targets; SNS is simpler
  pub/sub. "Add consumers later without touching the producer" points to
  EventBridge.
- **Kinesis vs MSK:** "the team already runs Kafka" or "needs Kafka
  Connect/ksqlDB" → MSK. "AWS-native, no broker management" → Kinesis.
  Throughput alone rarely decides; ecosystem does.
- **FIFO everywhere?** FIFO costs throughput and exists only where ordering
  is a real requirement. Do not pay the FIFO tax on a notifications fan-out.

## 7. Official References

- [SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/)
- [SNS Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/)
- [SNS Message Filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [Fan-Out Pattern](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html)
