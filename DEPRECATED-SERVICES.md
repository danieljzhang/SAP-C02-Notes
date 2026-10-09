# Deprecated and Restricted Services — SAP-C02

> Single reference for retired, restricted, or exam-irrelevant AWS services.
> Do not create study files for services listed here. Cross-reference this file if a service appears in older study materials.

---

## Retired / End-of-Life Services

| Service | Status | Notes |
|---|---|---|
| AWS OpsWorks | Retired | End of life June 2024. Replaced by Systems Manager and CloudFormation. Do not study for SAP-C02. |
| AWS CodeStar (project service) | Retired | Shut down July 2024. Do not treat CodeCatalyst as a one-to-one replacement; select currently supported source-control, build, and deployment services for the requirements. |
| Amazon WorkMail (exam relevance) | Low | Rarely tested at professional level. Skip unless exam guide explicitly lists it. |
| AWS Managed Blockchain (Ethereum) | Limited | Ethereum network support ended. Hyperledger Fabric remains but is low exam priority. |

## Services in Maintenance Mode / Limited Investment

| Service | Status | Notes |
|---|---|---|
| AWS CodeCommit | No new customers (Oct 2023) | Existing customers can still use it. Exam may still reference it — verify against current exam guide. Prefer GitHub/GitLab integration patterns in new scenarios. |
| Amazon CodeCatalyst | No new customers | Existing customers can continue to use it; AWS provides migration guidance. Do not recommend it for a new customer without checking current service lifecycle guidance. |
| AWS Cloud9 | No new customers | IDE service restricted. Not a primary SAP-C02 topic. |

## Services Outside SAP-C02 Scope

| Service | Notes |
|---|---|
| Amazon Chime | Collaboration tool, not an architecture service. Not tested. |
| Amazon Connect (deep config) | High-level awareness only. Deep configuration not in scope. |
| AWS RoboMaker | Niche robotics service. Not in SAP-C02 exam guide. |

---

> Last updated: 2026-10-08 (CodeCatalyst lifecycle entry). Other lifecycle entries have not all been revalidated in this review.
> Verify every entry against current AWS documentation and the official exam guide before the exam.
> See [CHANGELOG.md](./CHANGELOG.md) for history of content decisions.
