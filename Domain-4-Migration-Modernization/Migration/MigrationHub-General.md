# AWS Migration Hub — SAP-C02 Study Notes

## 1. Purpose and Exam Relevance

AWS Migration Hub provides a place to discover/organize migration information and track application migration progress across supported AWS and partner tools. It helps teams group servers into applications, coordinate waves, and report status across a portfolio. It is a planning and tracking layer, not the data plane that copies servers, databases, or files.

Service names and integrated capabilities evolve; confirm the current Migration Hub documentation and console experience. AWS Transform and individual migration services may provide additional planning or execution capabilities.

## 2. Portfolio Tracking Pattern

```text
Discovery / inventory sources
          |
          v
  Application mapping and waves
          |
          +--> MGN: server replication / launch
          +--> DMS: database load / CDC
          +--> DataSync: file/object transfer
          +--> Partner tools / manual workstreams
          |
          v
 Migration Hub status, dependencies, and reports
```

- **Applications:** group related servers/resources and capture dependencies/ownership.
- **Waves:** sequence applications by dependency, business criticality, risk, and target readiness.
- **Integrations:** connect supported migration tools and source data so status can be observed centrally.
- **Tracking:** use consistent application identifiers, owner, source/target, planned dates, status, blockers, and acceptance criteria.

## 3. Using Migration Hub Well

1. Discover and reconcile the inventory; remove duplicates and resolve missing owners.
2. Map technical dependencies to business applications. Verify automated discovery with application teams.
3. Choose migration strategies and execution tools per workload, not one tool for the entire portfolio.
4. Build waves around dependencies, network readiness, identity, shared databases, change windows, and rollback capability.
5. Integrate tools where supported; define a common status model for work that is not automatically reported.
6. Track blockers, test results, cutover, validation, and decommission approvals.
7. Use portfolio reports to identify stalled work and capacity constraints, not as proof of technical correctness.

## 4. Governance, Security, and Operations

- Decide which account/Region owns the migration workspace and how teams access it. Use IAM Identity Center or controlled IAM roles and least privilege.
- In multi-account migrations, align application IDs and ownership across the landing zone, migration accounts, and source systems.
- Protect discovery credentials and network access; gather only approved inventory data and follow data-residency requirements.
- Centralize migration logs/status with normal audit controls; status records can expose hostnames, owners, architecture, and planned dates.
- Assign an accountable owner for each application and gate production cutover on test evidence, backup, security review, and business sign-off.

## 5. Limitations and Pitfalls

- A centralized dashboard does not perform replication, schema conversion, file copy, testing, or traffic cutover.
- Inventory data may be stale, incomplete, or overinclusive; validate ownership and dependencies.
- A server can show as migrated while its application remains unusable because of DNS, identity, licensing, certificates, shared storage, or downstream integrations.
- Do not confuse migration progress tracking with AWS Application Discovery Service, MGN, DMS, or AWS Transform; consult current product documentation for their roles.
- Check current Regions, integrations, API behavior, and service lifecycle before automation.

## 6. Original Scenario

**Scenario:** A global company has 900 servers across multiple business units. Teams use MGN, DMS, and partner tools, but executives need a consolidated view and cutovers are blocked by undocumented dependencies.

**Recommended approach:** Use discovery and Migration Hub tracking to establish a common application inventory and wave/status model. Validate dependency maps with owners; integrate execution status from supported tools; assign blockers and acceptance gates. Continue using MGN/DMS/DataSync or partners to execute the actual workload moves.

**Trap:** Recommending Migration Hub as the mechanism that transfers all 900 servers confuses orchestration visibility with migration execution.

## 7. Exam Tips

- Migration Hub answers “where do we organize/track portfolio progress?” not “what copies the data?”
- Pair the tracking layer with workload-specific tools: MGN for servers, DMS for databases, DataSync for files/objects.
- Automated discovery informs a plan; application owners and testing confirm the plan.

## 8. Official References

- [AWS Migration Hub User Guide](https://docs.aws.amazon.com/migrationhub/latest/ug/whatishub.html)
- [AWS Application Discovery Service](https://docs.aws.amazon.com/application-discovery/latest/userguide/what-is-appdiscovery.html)
- [AWS Transform](https://docs.aws.amazon.com/transform/latest/userguide/what-is-aws-transform.html)
