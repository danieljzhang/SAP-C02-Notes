# AWS IoT (Core & Greengrass) — SAP-C02 Study Notes

> Exam relevance: Domain 2 (29%). IoT appears in event-driven and edge scenarios. Know the ingestion paths, the Greengrass vs Outposts distinction, and offline behavior.

---

## 1. Overview

AWS IoT connects devices to AWS services for ingestion, processing, and management. At the Professional level, the exam tests *architectural* choices: how device data enters AWS, what happens during intermittent connectivity, and what runs where.

---

## 2. IoT Core Ingestion Paths

| Path | Behavior |
|---|---|
| Message broker (MQTT pub/sub) | Standard path; devices publish, rules engine and subscribers consume |
| **Basic Ingest** | Routes device messages **directly to the rules engine**, skipping the broker pub/sub path |

Choose Basic Ingest when device telemetry flows onward for processing (rules engine → Lambda/Kinesis/S3) and nothing needs to consume it via broker subscriptions. It removes cost and hops for the pure-ingestion pattern.

### Rules Engine

- SQL-like queries route messages to 15+ AWS services (Lambda, Kinesis, S3, DynamoDB, SNS, Step Functions, etc.)
- Republish to reserved topics (e.g., `$aws/rules/...`) is restricted — design topic namespaces accordingly.

---

## 3. Greengrass — Edge Software, Not AWS Hardware

AWS IoT Greengrass deploys components and local Lambda execution to **customer-managed edge devices**, including during intermittent connectivity. Key distinctions:

| | Greengrass | Outposts |
|---|---|---|
| What it is | **Software** on your edge devices | **AWS infrastructure** in your facility |
| Hardware | Customer-managed devices | AWS-managed racks |
| Connectivity | Designed for intermittent/offline operation | Requires connectivity to its parent Region |
| Use when | Local processing with unreliable links | Local processing, data residency, low latency with AWS APIs on-site |

The exam's classic trap: describing Greengrass as "extending the AWS Region to the edge like Outposts." Greengrass extends *compute to your devices*; Outposts extends *AWS infrastructure to your building*. Different ownership, different connectivity assumptions.

---

## 4. Offline and Connectivity Design

- Greengrass components can run locally during outages; sync state when connectivity returns. Design for **conflict resolution** on reconnect, not just buffering.
- Device Defender audits and monitors device security posture — use it when the scenario mentions fleet security at scale.
- For bidirectional command: jobs (deploy operations to fleets) vs shadow (desired/reported state sync). Shadows suit state reconciliation; jobs suit fleet-wide operations.

---

## 5. Exam Scenarios

**Scenario 1:** Thousands of sensors stream telemetry to Kinesis for analytics. No device needs to subscribe to other devices' messages. Minimize cost.

> Use IoT Core **Basic Ingest** to route directly to the rules engine → Kinesis. Skipping the broker removes the pub/sub cost for a pure-ingestion flow.

**Scenario 2:** Factory gateways must run anomaly detection locally and keep working through WAN outages, syncing results when the link returns.

> Deploy the detection as a Greengrass component on the gateways. Local execution survives the outage; results sync on reconnect. This is not an Outposts case — no AWS hardware is being installed.

---

## 6. Exam Tips

- Basic Ingest = cost optimization for telemetry-only flows; know when the broker path is unnecessary.
- Greengrass vs Outposts is a favorite confusion pair — anchor on **who owns the hardware**.
- Device shadows solve state sync; jobs solve fleet operations. Do not mix them up.

---

## 7. 🎯 Architecture Decision Notes

### The three edge answers — pick by ownership

- **Greengrass:** your devices, your hardware, intermittent links OK.
- **Outposts:** AWS hardware in your building, needs Region connectivity, for residency/latency with AWS APIs.
- **Snowball Edge:** offline data transfer or temporary edge compute where network transfer is impractical — a device you ship, not a deployment model (existing customers only; closed to new customers Nov 2025).

When a scenario says "factory floor, unreliable WAN, local ML inference," the answer is Greengrass — and the wrong answers will be Outposts ("install AWS racks") or pure cloud ingestion ("stream everything to Kinesis" fails during the outage).

---

## 8. Official References

- [AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html)
- [IoT Core Basic Ingest](https://docs.aws.amazon.com/iot/latest/developerguide/iot-basic-ingest.html)
- [AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html)
- [AWS Outposts](https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html)
