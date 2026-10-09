# Identity and Governance

## Authentication is not authorization

Authentication establishes an identity; authorization decides what that identity may do. Workforce access and customer sign-in are different problems:

- **IAM Identity Center** centralizes workforce sign-in and permission sets across AWS accounts and supported applications. It can use an external identity provider.
- **Amazon Cognito user pools** manage or federate application users and issue tokens for application sign-in.
- **Cognito identity pools** exchange supported identity-provider tokens for temporary AWS credentials when an app needs to call AWS services directly.
- **SAML** is common for enterprise browser-based single sign-on. **OIDC/OAuth-based flows** are common for web and mobile applications. Select the protocol supported by the identity provider and client; do not assume every social provider speaks SAML.
- For AWS workloads, prefer temporary role credentials over long-lived access keys. An EC2 instance profile makes a role available to software on an instance without embedding credentials in the application.
- A **role trust policy** defines who or what may assume a role; its permissions policies define what the assumed role can do. A **service role** is configured for a workload or service to assume. A **service-linked role** is a service-specific role with permissions defined for that AWS service.
- When a third party assumes a role in your account, an **external ID** in the trust relationship can help distinguish the intended customer relationship and reduce confused-deputy risk. It is not a password or a replacement for restricting the trusted principal.

**Fresh example:** A field-service app needs employee login and must upload photos to a private S3 bucket. Use a user sign-in mechanism for the application, then grant short-lived, narrowly scoped access to the required resources. Keep the app's identity flow separate from the bucket's authorization policy.

## Reading AWS permissions

Treat policies as pieces of an authorization decision, not interchangeable permission grants:

- An **identity-based policy** grants actions to an IAM user, group, or role.
- A **resource-based policy** grants a principal access to a supported resource. Support varies by service; DynamoDB currently supports resource-based policies for tables, indexes, and streams.
- A **permissions boundary** limits what identity-based policies can grant to a user or role. It does not grant access by itself.
- A **session policy** further narrows a temporary session created with supported STS operations.
- An **Organizations service control policy (SCP)** sets the maximum permissions available to principals in member accounts. It does not grant permissions.
- A **resource control policy (RCP)** sets a maximum permissions boundary on supported resources in an organization. It also does not grant permissions.
- A **VPC endpoint policy** restricts requests that pass through that endpoint; it is an additional filter, not a replacement for the service's authorization checks.

An explicit deny overrides an allow. For cross-account access, check both the principal's permissions and the target resource or role's trust configuration, along with any organization, boundary, session, or endpoint restrictions that apply. Exact evaluation depends on the principal and service.

**Fresh example:** A platform team lets product teams create application roles but wants each role limited to a centrally approved set of permissions. A permissions boundary can cap each role; the application policy still needs to grant the operations the application actually uses.

IAM Access Analyzer's external-access analyzer evaluates supported resource-based policies and reports access granted outside its account or organization zone of trust; configure it in each Region where supported resources need coverage. Its policy-generation feature is different: that feature uses CloudTrail activity to suggest permissions based on observed use. Access Analyzer is a review and analysis tool, not an authorization grant. Validate the intended principal and actions, and prefer least privilege and temporary credentials.

## Organizations and shared resources

Use AWS Organizations to group accounts and apply governance consistently. Organizational units help scope policy changes to teams or environments. Start restrictive policies with a small test group and maintain a recovery path; an organization policy can restrict even root-user activity in member accounts.

AWS Resource Access Manager (RAM) shares supported resources, such as selected network resources, across accounts, OUs, or an organization. Sharing does not transfer ownership. Some integrations require trusted access between an AWS service and Organizations; enable it only when needed and understand which service is acting on the organization's behalf.

Tags support ownership, cost allocation, automation, and attribute-based access control. Define a small, governed tag vocabulary. IAM conditions can use tags for access decisions; preventive policies can restrict certain create or tag operations, while AWS Config can evaluate deployed resources for compliance. Prevention and audit solve different parts of the problem.

**Fresh example:** A central networking account shares a subnet with application accounts through RAM. The network team retains control of the shared resource, while each consumer deploys its own workloads and must still configure its own routing and security controls correctly.

## Configuration, certificates, and parameters

- **AWS Config** records supported resource configuration and evaluates it against rules. A multi-account, multi-Region aggregator provides a consolidated view; it does not itself turn on recording in every account and Region.
- **AWS Certificate Manager (ACM)** manages supported public and private TLS certificates and can automate renewal for eligible integrations. Confirm the target service and certificate type are supported before designing around ACM; a certificate's private key is not exportable in every certificate workflow.
- **Systems Manager Parameter Store** stores configuration values and parameters, with access controlled through IAM. CloudFormation can use typed parameters that refer to Parameter Store values. Keep secret values in a secrets-management service when its rotation and secret-handling features are required.
- CloudFormation template parameters make deployment inputs explicit; SSM parameter types can resolve parameter values during stack operations. Dynamic references can retrieve supported values without embedding them in a template, but protect access to the underlying parameter or secret and consider when the value is resolved.

## CloudFormation resource lifecycle

CloudFormation has separate controls for different removal events:

- `DeletionPolicy` governs what happens when a resource is removed from a stack or its template definition is removed. Options include retaining the physical resource or taking a snapshot for resource types that support snapshots.
- `UpdateReplacePolicy` governs the old physical resource when a stack update replaces it with a new physical resource.
- `RetainExceptOnCreate` is useful when a newly created resource should be cleaned up if the creating operation rolls back, but retained in later deletion cases.

CloudFormation has documented default-snapshot exceptions for an RDS DB cluster and a standalone RDS DB instance. Check the current attribute documentation and resource type before assuming that an omitted policy means deletion.

In practical terms, `Delete` removes the resource, `Retain` leaves the physical resource outside the stack, and `Snapshot` creates a snapshot before removal only for supported resource types. Retained resources and snapshots can continue to incur charges and need an owner and cleanup plan.

Choose policies per resource, and account for retained resources and snapshots that remain outside the stack and may continue to incur charges. A snapshot policy is not a substitute for a tested backup and restore plan. For a database that must survive both stack deletion and replacement, consider both lifecycle attributes and validate the resource type's documented behavior.

The shared responsibility model is another useful boundary: AWS secures the cloud infrastructure, while customers remain responsible for the security configuration and data controls assigned to them for each service. The exact division varies between managed and customer-managed services.

## References

- [IAM policy types and permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [AWS IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)
- [Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
- [DynamoDB resource-based policies](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-resource-based.html)
- [AWS Organizations SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [AWS RAM](https://docs.aws.amazon.com/ram/latest/userguide/what-is.html)
- [AWS Config aggregators](https://docs.aws.amazon.com/config/latest/developerguide/aggregate-data.html)
- [CloudFormation DeletionPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html)
- [CloudFormation UpdateReplacePolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatereplacepolicy.html)
