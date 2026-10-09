# AWS Migration Acceleration Program (MAP) — SAP-C02 Study Notes

## 1. Purpose and Exam Relevance

The AWS Migration Acceleration Program (MAP) is a structured program to help organizations assess, mobilize, migrate, and modernize workloads on AWS. It combines methodology, migration expertise, tools, and (for eligible engagements) financial incentives or credits subject to current program terms. MAP is not a data-transfer service and does not replace architecture, migration execution, or operational ownership.

For exam scenarios, MAP is relevant when an organization needs a repeatable enterprise migration approach, skills and partner support, business-case development, and a governed transformation plan.

## 2. Program Lifecycle (Conceptual)

```text
Assess                 Mobilize                   Migrate / Modernize
Inventory + business -> landing zone + people -> waves + workload moves
case + strategy        + operating model           + optimization
```

The exact phases, deliverables, tools, eligibility, and funding rules depend on current AWS program guidance and the specific engagement. Treat this as a planning model, not a guaranteed funding entitlement.

## 3. Architecture Workstreams MAP Does Not Replace

| Workstream | Example implementation |
|---|---|
| Portfolio discovery and business case | Inventory, application mapping, dependency and TCO analysis |
| Landing-zone readiness | Organizations, accounts, identity, network, security, logging, and governance |
| Migration factory | Standard runbooks, automation, wave planning, test/cutover/rollback process |
| Workload execution | MGN for supported server rehosting, DMS for databases, DataSync for files, Transfer Family for partner file interfaces |
| Modernization | Container, serverless, managed database, event-driven, or refactor patterns selected per workload |
| Operating model | Skills, service ownership, observability, backup, incident response, FinOps, and continuous improvement |

## 4. Decision and Governance Considerations

- Establish executive sponsorship, business outcomes, workload owners, scope, and measurable acceptance criteria.
- Classify applications by business criticality, dependency, technical debt, compliance, data residency, and modernization potential.
- Build a secure, governed landing zone before large-scale migration; avoid moving workloads into unowned accounts or ad hoc networks.
- Sequence waves around dependencies and risk. Pilot a representative workload, measure throughput and test assumptions, then scale the factory.
- Separate program incentives from architecture decisions. A workload should not be moved or modernized solely to qualify for a program benefit.
- Confirm current eligibility, partner participation, funding milestones, evidence requirements, and expiration dates directly with AWS or an authorized partner.

## 5. Risks and Failure Modes

| Risk | Mitigation |
|---|---|
| Business case based on unverified inventory | Reconcile discovery with application owners and actual utilization/cost data |
| Migration starts before landing zone is ready | Gate waves on identity, networking, security, logging, backup, and operations readiness |
| “Migration complete” mistaken for modernization | Define target operating model and explicit post-move optimization/refactoring outcomes |
| Incentive assumed or misapplied | Verify current written program terms; model economics without assuming unconfirmed credits |
| Factory optimizes for volume over business outcomes | Use risk-based waves, service acceptance criteria, and workload-specific exceptions |

## 6. Original Scenario

**Scenario:** A company must migrate hundreds of applications over two years, has limited cloud skills, and needs a business case for leadership. Some systems are candidates for rehosting; others should be retired or replaced.

**Recommended approach:** Use a MAP-style structured assessment and mobilization to build the portfolio, business case, landing zone, migration factory, skills plan, and governance. Classify every workload independently, pilot the tools and operating model, then execute waves. Verify eligibility for any program benefits and choose each technical service based on the workload.

**Trap:** MAP is not the tool that replicates a server or guarantees credits for every workload.

## 7. Exam Tips

- MAP = migration program/methodology and eligible support; MGN/DMS/DataSync = technical workload movement.
- Assess before committing to a migration pattern; some workloads should be retired, retained, or repurchased rather than rehosted.
- Keep the landing zone, people/process readiness, and post-migration operations in scope.
- Program incentives are conditional and version-sensitive; do not make unsupported cost claims.

---

## 🎯 Architecture Decision Notes

### Choose the migration path from the target outcome

The "6 Rs" are not a ladder — the right answer depends on business goal,
app constraints, and acceptable change:

| Path | When |
|---|---|
| Rehost (lift-and-shift) | Minimal change, speed matters, app cannot be redesigned now |
| Replatform | Targeted improvements (managed DB, containers) without full redesign |
| Refactor / re-architect | Current design cannot meet the goals at all |
| Repurchase | A SaaS product replaces the workload better than any migration |
| Retain | Keep as-is temporarily with a revisit date |
| Retire | Decommission — often the highest-ROI "migration" decision |

A plan that treats every server as lift-and-shift is not a plan. Discovery
(dependencies, licensing, data gravity), cutover/rollback criteria, and
post-migration validation are the plan; the tools execute it.

**Tool roles:** Application Discovery Service *collects* environment data;
Migration Hub *tracks* progress; Application Migration Service *rehosts*
servers; DMS *moves databases*; VM Import/Export handles VM images. Match
the tool to its job — none of them assess dependencies or licensing for you.

---

## 🎯 Architecture Decision Notes

### Discovery before migration — match the collection method to the estate

Migration fails most often before any workload moves: the inventory is
wrong. Application Discovery Service collects server configuration and
utilization data so the business case and wave plan rest on facts. Choose
the collection method by what the estate allows:

| Estate | Method |
|---|---|
| VMware environment where you can deploy an appliance | **Agentless** discovery via the OVA connector through vCenter — no per-VM installs |
| Physical servers, non-VMware hypervisors, or cloud VMs | **Agent-based** — install the discovery agent on each server |

Both feed Migration Hub, which is the *tracking* console — it does not
discover or move anything itself. Keep the roles straight: Discovery
*collects*, Migration Hub *tracks*, MGN *rehosts*, DMS *moves databases*.

Before committing to waves, run a readiness self-assessment (the Cloud
Adoption Readiness Tool's structured questionnaire across business,
people, process, platform, operations, and security perspectives). It
surfaces the non-technical blockers — skills, operating model, security
posture — that derail migrations more reliably than any tooling gap.

Trap: discovery data goes stale fast. Reconcile it with application
owners and real utilization before sizing targets; an inventory built
once and trusted for a year produces the wrong landing zone.

**Fresh example:** A manufacturer with 400 VMware VMs and 60 physical
servers deploys the agentless OVA connector for the virtual estate and
agents on the physical hosts, groups everything into applications in
Migration Hub, and runs the readiness survey — which reveals the ops
team has no runbook for cutover weekends. They fix the process gap
before scheduling wave one, instead of discovering it at 2 a.m. on
cutover night.

## 8. Official References

- [AWS Migration Acceleration Program](https://aws.amazon.com/migration-acceleration-program/)
- [AWS Cloud Adoption Framework](https://aws.amazon.com/cloud-adoption-framework/)
- [AWS Migration Hub](https://aws.amazon.com/migration-hub/)
- [AWS Transform](https://aws.amazon.com/aws-transform/)
