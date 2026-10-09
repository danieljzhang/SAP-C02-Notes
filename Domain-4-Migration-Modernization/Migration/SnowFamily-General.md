> Exam relevance: Domain 4. Snow Family covers the offline-transfer decision — physical shipment versus network transfer — and the operational, security, and chain-of-custody controls it requires.
>
> **Availability caveat:** AWS documentation states that Snowball Edge is no longer available to new customers. Existing eligible customers should follow current service guidance. For new projects, validate current regional options such as AWS Data Transfer Terminal or AWS Partner solutions; do not assume a Snow device can be ordered.

---

## 1. Purpose and Exam Relevance

AWS Snow Family refers to physical devices and related workflows for transporting data to or from AWS when network transfer is impractical, too slow, too costly, or unavailable. Historically, Snowball Edge also offered local compute/storage for edge environments with intermittent connectivity. The exam concept is the decision between online transfer and physical shipment, plus the operational, security, and chain-of-custody controls required.

Always check current product availability before recommending a specific Snow device. The existing Snowball Edge documentation describes device capabilities, but availability to new customers has changed.

---

## 2. Offline Transfer Decision

```text
Source data center                    AWS destination
+---------------------+                +------------------+
| Export data, encrypt| -> shipping -> | Import to S3 or  |
| and verify manifest |                | supported target |
+---------------------+                +------------------+
           |                                      |
           +----- track chain of custody --------+
```

Compare estimated network transfer time and cost with device/job lead time, shipping, handling, and import duration. Include the final delta-transfer window if the source changes while the bulk copy is in transit. Physical transfer is a data-movement method, not an application migration or database conversion strategy.

---

## 3. Design Considerations

- **Eligibility and ordering:** validate customer eligibility, Region, device/job type, supported interfaces, data size, and current lead time before basing a schedule on a device.
- **Data preparation:** define what is copied, encryption/key ownership, checksums/manifests, naming, retry/reconciliation, and treatment of failed or duplicated files.
- **Security:** use supported encryption mechanisms, control device access, protect job credentials and manifests, maintain chain-of-custody records, and confirm data sanitization procedures after job completion.
- **Network/offline constraints:** physical shipment is useful when network bandwidth or connectivity is the bottleneck; if a private network can complete the transfer within the required window, DataSync may be simpler.
- **Edge processing:** historical Snowball Edge configurations included local compute and storage, but do not assume every model or feature is orderable. For new edge designs evaluate AWS Outposts, local processing appliances, or partner solutions based on latency, autonomy, and service availability.
- **Data integrity:** preserve source copies until AWS import completion and validation. A device being shipped back is not proof the destination dataset is complete.

---

## 4. Reliability, Operations, and Cost

Total elapsed time includes job approval/provisioning, data loading, shipping both directions, AWS import, reconciliation, and final delta. Include buffers for weather, customs, remote-site logistics, and business cutover windows.

Cost can include job/device fees, shipping, data-transfer options, onsite labor, destination storage, and time spent operating parallel systems. Compare with DataSync network, Direct Connect/VPN, and transfer terminal/partner alternatives. Check current service terms and pricing rather than using old study-guide amounts.

---

## 5. Common Failure Modes

| Risk | Control |
|---|---|
| Device cannot be ordered for new customer | Verify current eligibility and availability at the start; select an online or currently supported physical option |
| Transfer misses a deadline | Measure network baseline, account for shipment/import lead time, and stage incremental deltas where supported |
| Incomplete or duplicated import | Use manifests/checksums, reconcile object/file counts, and retain a source copy until validation |
| Sensitive data exposed at a remote site | Encrypt data, restrict physical access, document custody, and use approved key-management processes |
| Local compute cannot reach required AWS dependencies | Design for disconnected operation, local dependency availability, and explicit re-synchronization behavior |

---

## 6. Exam Scenarios

**Scenario:** A research organization has a remote site producing several petabytes of data. The WAN link is unreliable, and a one-time transfer must finish before a facility closes.

**Decision:** First validate that a current physical-transfer offering is available for this customer, Region, data volume, and timeline. Compare it with staged DataSync over available links and a transfer-terminal or partner option. If an eligible offline service is selected, encrypt and inventory the dataset, maintain custody, plan time for AWS import, reconcile results, and keep source data until acceptance.

**Trap:** Recommending Snowball Edge by name solely because the dataset is large ignores its current new-customer availability caveat.

---

## 7. Exam Tips

- Choose physical transfer when network constraints dominate and the shipment/import schedule is acceptable.
- Choose DataSync for managed online file/object movement when the network path and transfer window are viable.
- Separate data transport from server rehosting (MGN), database migration (DMS), and application modernization.
- Availability and device capabilities are version-sensitive; check the current AWS Snow documentation and alternatives.

---

## 8. Official References

- [What is Snowball Edge?](https://docs.aws.amazon.com/snowball/latest/developer-guide/whatisedge.html)
- [AWS Snowball pricing and availability](https://aws.amazon.com/snowball/pricing/)
- [AWS Data Transfer Terminal](https://aws.amazon.com/data-transfer-terminal/)
- [AWS DataSync User Guide](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html)
