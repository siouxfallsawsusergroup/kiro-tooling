---
inclusion: auto
name: cfn-design-best-practices
description: CloudFormation naming, section order, and tagging. Use when writing or editing a CloudFormation template.
tags:
  - type/steering
  - tool/cloudformation
---

# CloudFormation design

A predictable template is easier to review than a clever one. The prefixes below are a community convention, not an AWS requirement. Use them when a team wants every template to read the same way.

## Names

| Kind | Prefix | Example |
| --- | --- | --- |
| Parameter | `p` | `pVpcCidr`, `pEnvironment` |
| Resource | `r` | `rVpcMain`, `rS3Bucket` |
| Condition | `c` | `cIsProduction` |
| Output | `o` | `oVpcId` |
| Mapping | `m` | `mRegionToAmi` |
| Metadata | `meta` | `metaParameterGroups` |

## Section order

1. `AWSTemplateFormatVersion`
2. `Description`
3. `Metadata` (if used)
4. `Parameters`
5. `Mappings` (if used)
6. `Conditions` (if used)
7. `Resources`
8. `Outputs`

## Parameters, resources, outputs

- Parameters have a `Description`. Use `AllowedValues` when the set is small, and `ConstraintDescription` when a bad value needs a human sentence.
- Defaults belong on parameters that are safe in every environment. Do not default a production setting to the cheapest option by accident.
- Set `DeletionPolicy` (and `UpdateReplacePolicy` where it matters) on anything that holds data.
- Use `DependsOn` when the dependency is not already obvious from `Ref` or `GetAtt`.
- Outputs that another stack consumes get an `Export` name that includes the stack name, plus a description.

## Tags

```yaml
Tags:
  - Key: Environment
    Value: !Ref pEnvironment
  - Key: Project
    Value: !Ref pProject
  - Key: Owner
    Value: !Ref pOwner
  - Key: CostCenter
    Value: !Ref pCostCenter
```

Optional when you use them: `Application`, `Version`, `Backup`, `Monitoring`.

## Secrets and network

- Do not hardcode secrets. Reference Secrets Manager or SSM Parameter Store.
- Security group rules name the port and the peer. Avoid `0.0.0.0/0` except for the public listener that needs it (typically 443 on an ALB).
- See [Network security baseline](../networking/net-security-baseline.md) for the VPC rules.

## Before you deploy

- Names follow the prefixes above, or the repo documents a different convention.
- Required tags are parameters, not string literals copied into every resource.
- No account-specific ids buried in the template when a parameter or `AWS::AccountId` would do.
- IAM is as small as the feature allows. See [AWS security standards](cfn-security-standards.md).
- Data resources have a retain policy.

## Sketch

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Example template using the naming prefixes above

Parameters:
  pEnvironment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Description: Deployment environment

Conditions:
  cIsProduction: !Equals [!Ref pEnvironment, prod]

Resources:
  rVpcMain:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub "${pEnvironment}-vpc"

Outputs:
  oVpcId:
    Description: VPC ID
    Value: !Ref rVpcMain
    Export:
      Name: !Sub "${AWS::StackName}-VpcId"
```

## Related

- [AWS security standards](cfn-security-standards.md)
- [Testing and drift](cfn-testing-and-drift.md)
- [Terraform standards](../terraform/tf-standards.md)
