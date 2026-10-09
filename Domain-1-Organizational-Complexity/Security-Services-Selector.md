# Security Service Selection — SAP-C02 Study Notes

> Security scenarios often require several controls. Select services by the control plane they provide, then design IAM, network, data, response, and governance integrations around them.

## 1. Which Service Answers Which Question?

| Requirement | Service / control | Role and boundary |
|---|---|---|
| Detect suspicious account/API/network behavior | Amazon GuardDuty | Threat detection from supported data sources; findings require triage/remediation |
| Find software vulnerabilities and unintended exposure | Amazon Inspector | Vulnerability management for supported compute, container, and code resources |
| Discover sensitive data in S3 | Amazon Macie | Sensitive-data discovery and classification for S3; scope findings and access carefully |
| Investigate relationships around security findings | Amazon Detective | Helps analyze and visualize security data; complements, does not replace, detection |
| Aggregate/normalize security findings and posture signals | AWS Security Hub CSPM | Central security posture/findings view; it does not generate every integrated service's finding |
| Detect configuration drift/noncompliance | AWS Config | Resource configuration history and rule evaluation; remediation needs automation/policy |
| Analyze unintended resource access | IAM Access Analyzer | Findings for public/cross-account access and policy validation features |
| Filter HTTP/S requests to web applications | AWS WAF | Web ACLs, managed/custom rules, rate controls; applies at supported associations |
| Mitigate common and large-scale DDoS risks | AWS Shield | Standard protections plus paid Advanced capabilities; use WAF/architecture controls as required |
| Inspect and filter VPC network traffic | AWS Network Firewall | Stateful/stateless network policy at VPC boundaries; design routing and symmetry |
| Centrally apply supported security policies across accounts/resources | AWS Firewall Manager | Organization-wide policy management when prerequisites and supported resource types fit |
| Manage encryption keys | AWS KMS / CloudHSM | Select managed key service or dedicated HSM control based on key custody/compliance requirements |
| Store/rotate application secrets | AWS Secrets Manager / Systems Manager Parameter Store | Choose secret rotation and lifecycle features versus parameter/configuration storage needs |

## 2. Layered Detection and Response Pattern

```text
GuardDuty / Inspector / Macie / Config
                 |
        Security Hub CSPM (aggregate)
                 |
      EventBridge finding rule / workflow
                 |
  Lambda or Systems Manager Automation
                 |
  ticket, isolate, revoke, notify, remediate
                 |
       CloudTrail + CloudWatch evidence
```

- Keep management, delegated administration, and workload accounts distinct; use Organizations/Control Tower for enrollment and guardrails.
- Configure least-privilege service roles, organization enablement, delegated administrators, Regions, and finding aggregation explicitly.
- Make remediation idempotent, auditable, and reversible where possible. A finding should not automatically trigger a destructive action without severity/context checks.
- Protect findings, logs, and investigation data with retention, encryption, and access controls.

## 3. Network and Web Application Controls

- Use security groups for stateful workload boundaries; NACLs for subnet-level stateless filtering; route tables and firewall appliances for traffic paths.
- Use AWS WAF for application-layer request inspection at supported endpoints. Shield addresses DDoS resilience; neither replaces application scaling, caching, origin protection, or incident plans.
- Use Network Firewall or a centralized inspection design for VPC network traffic. Validate return paths, AZ resilience, routing symmetry, and throughput.
- Use Firewall Manager when central policy administration across an Organization is required and its supported resource types match.

## 4. Original Scenario

**Scenario:** A regulated organization needs to find exposed S3 buckets, detect suspicious account behavior, identify vulnerable EC2 packages, and route high-severity findings to a security team across many accounts.

**Recommended approach:** Enable Macie for S3 sensitive-data discovery, GuardDuty for threat detection, and Inspector for supported vulnerability findings. Use Security Hub CSPM and delegated administration to aggregate findings across enrolled accounts/Regions; route selected findings through EventBridge to a ticketing or approved remediation workflow. Use IAM Access Analyzer for unintended access analysis and Config for configuration compliance.

**Trap:** Security Hub aggregates posture/findings; it is not a substitute for GuardDuty, Inspector, or Macie detection sources.

## 5. Exam Tips

- Identify whether the requirement is preventive policy, configuration compliance, threat detection, vulnerability management, sensitive-data discovery, investigation, or response.
- Security Hub is a central view; GuardDuty, Inspector, and Macie are distinct detection services.
- WAF is application-layer filtering; Network Firewall is VPC network inspection; Shield addresses DDoS protection.
- Control Tower/Organizations can govern accounts, but workload policies, service enrollment, IAM permissions, and remediation still need design.
- Check current Regions, plan entitlements, supported resources, and integrations in AWS documentation.

## 6. Official References

- [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
- [Amazon Inspector](https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html)
- [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)
- [AWS Security Hub CSPM](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
- [AWS Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/fms-chapter.html)
- [AWS Network Firewall](https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html)
- [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html)
- [AWS Shield](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-overview.html)
- [SAP-C02 Domain 1 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02-domain1.html)
