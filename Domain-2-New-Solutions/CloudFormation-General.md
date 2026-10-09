# AWS CloudFormation — SAP-C02 Study Notes

> Exam relevance: Domains 2, 3, 4. CloudFormation is the AWS-native IaC service. Know stacks, StackSets, nested stacks, drift detection, and change sets.

---

## 1. Overview

AWS CloudFormation provisions and manages AWS infrastructure as code using JSON or YAML templates. It ensures consistent, repeatable deployments and tracks resource state.

---

## 2. Template Anatomy

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "My stack"

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-12345678

Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-bucket-${Environment}"

Outputs:
  BucketName:
    Value: !Ref MyBucket
    Export:
      Name: !Sub "${AWS::StackName}-BucketName"
```

---

## 3. Stack Operations

| Operation | Description |
|---|---|
| Create | Provision all resources in template |
| Update | Modify existing stack — uses change sets |
| Delete | Remove all stack resources |
| Detect drift | Compare actual vs expected resource configuration |

### Change Sets

- Preview changes before applying an update
- Shows: resources to add, modify, replace, or delete
- **Replace** = resource will be deleted and recreated (potential downtime)
- Always review change sets before executing in production

### Stack Policies

- JSON policy that protects stack resources from unintended updates
- Deny updates to specific resources (e.g., production databases)

---

## 4. Nested Stacks

Break large templates into reusable components:

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/network.yaml

  AppStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/app.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
```

- Parent stack manages child stacks
- Child stack outputs can be referenced by parent or sibling stacks
- Use for: VPC, security groups, IAM roles as reusable modules

---

## 5. Cross-Stack References

Share outputs between independent stacks using exports:

```yaml
# Stack A exports
Outputs:
  VpcId:
    Export:
      Name: "NetworkStack-VpcId"

# Stack B imports
Resources:
  MyInstance:
    Properties:
      VpcId: !ImportValue "NetworkStack-VpcId"
```

- Exported values cannot be deleted while imported by another stack
- Use for: sharing VPC IDs, subnet IDs, security group IDs across stacks

---

## 6. CloudFormation StackSets

Deploy stacks to multiple accounts and regions from a single operation:

```
Management Account
  └── StackSet
        ├── Account A, us-east-1
        ├── Account A, eu-west-1
        ├── Account B, us-east-1
        └── Account B, eu-west-1
```

- **Service-managed:** uses Organizations for automatic deployment to new accounts in OUs
- **Self-managed:** specify target accounts and regions manually
- **Automatic deployment:** new accounts added to an OU automatically get the StackSet
- Use for: baseline resources (VPC, IAM roles, Config rules) across all accounts

---

## 7. Drift Detection

- Detects when actual resource configuration differs from the CloudFormation template
- Supported for most resource types
- Does not auto-remediate — reports drift status
- Remediate by updating the stack or importing the drifted resource

---

## 8. CloudFormation Guard (cfn-guard)

- Policy-as-code tool to validate CloudFormation templates before deployment
- Write rules in Guard DSL to enforce compliance
- Integrates with CI/CD pipelines
- Use for: prevent non-compliant resources from being deployed

---

## 9. Exam Scenarios

**Scenario 1:** A company wants to deploy a standard VPC configuration to all new AWS accounts automatically when they join the organization. What is the solution?

> Use **CloudFormation StackSets** with **service-managed** permissions and **automatic deployment** enabled for the target OU. When a new account joins the OU, CloudFormation automatically deploys the VPC stack to the account.

**Scenario 2:** A company wants to update a CloudFormation stack but needs to review the changes before applying them to production. What is the process?

> Create a **Change Set** for the stack update. Review the change set to identify resources being added, modified, or replaced. Execute the change set only after approval. Resources marked as "Replace" will be deleted and recreated.

**Scenario 3:** A CloudFormation stack update accidentally modified a production RDS instance. How can this be prevented in the future?

> Apply a **Stack Policy** that denies updates to the RDS resource. Stack policies use JSON to specify which resources can be updated and by whom. This prevents accidental modifications to critical resources.

---

## 10. Exam Tips

- **StackSets** = deploy to multiple accounts/regions — use for organization-wide baselines
- **Service-managed StackSets** = automatic deployment to new accounts in OUs
- **Change sets** = preview before apply — always use in production
- **Nested stacks** = reusable modules; **cross-stack references** = share between independent stacks
- **Drift detection** = detective only — does not auto-remediate
- **cfn-guard** = preventive policy-as-code for template validation
- CloudFormation uses **IAM roles** — the stack role must have permissions to create all resources

---

## 11. Official References

- [CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
- [StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html)
- [Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [CloudFormation Guard](https://docs.aws.amazon.com/cfn-guard/latest/ug/)
