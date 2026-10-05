---
inclusion: auto
name: cfn-testing-and-drift
description: Lint, policy-check, change-set, and drift workflow for CloudFormation. Use when validating or deploying a template.
tags:
  - type/steering
  - tool/cloudformation
  - aws/governance
---

# CloudFormation testing and drift

Lint and scan a template before it becomes a stack. Deploy through a change set. Treat drift as a signal to redeploy the template, not to click around in the console until the resource matches.

## Tools

| Tool | Purpose |
| --- | --- |
| `cfn-lint` | Template shape against the resource spec and the Regions you target. |
| `cfn-nag` | Security scan: open ingress, broad IAM, missing encryption. |
| `cfn-guard` | Rules you write: tags, encryption, allowed types. |
| `taskcat` | Deploy a template in one or more Regions and tear it down. Use it when the template is meant to be reused, not on every one-line edit. |

```bash
cfn-lint template.yaml --regions us-east-1
cfn_nag_scan --input-path template.yaml
cfn-guard validate --data template.yaml --rules rules/governance.guard
```

## A small cfn-guard rule

```text
# rules/governance.guard
AWS::S3::Bucket {
  Properties.BucketEncryption exists
  Properties.PublicAccessBlockConfiguration.BlockPublicAcls == true
}

Resources.*.Properties.Tags exists
```

## Suppressing a finding

```yaml
rBucket:
  Type: AWS::S3::Bucket
  Metadata:
    cfn_nag:
      rules_to_suppress:
        - id: W35
          reason: "Access logging is sent to the central logging bucket."
```

A suppression without a reason should fail review.

## Change sets

```bash
aws cloudformation create-change-set \
  --stack-name example-prod \
  --change-set-name "cs-$(date +%s)" \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

aws cloudformation describe-change-set \
  --stack-name example-prod \
  --change-set-name "<name>" \
  --query 'Changes[].ResourceChange.{Action:Action,Resource:LogicalResourceId,Replace:Replacement}'
```

Read the change set before you execute it. `Replacement: True` on a database, bucket, or other stateful resource is a stop-and-think moment.

```bash
aws cloudformation execute-change-set \
  --stack-name example-prod \
  --change-set-name "<name>"
```

## Drift

```bash
aws cloudformation detect-stack-drift --stack-name example-prod
aws cloudformation describe-stack-resource-drifts \
  --stack-name example-prod \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

You can schedule detection with EventBridge or the Config rule `cloudformation-stack-drift-detection-check`. Fix drift by updating the template and deploying it. Hand-edits in the console come back as drift the next time someone looks.

## Rollback

- Leave failure behavior on rollback.
- Rollback triggers (a CloudWatch alarm) can revert an update that looks healthy to CloudFormation and unhealthy to users.
- Data resources use `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain`.
- Turn on termination protection for stacks you would hate to delete.

## Related

- [CloudFormation design](cfn-design-best-practices.md)
- [AWS security standards](cfn-security-standards.md)
- [Terraform security scanning](../terraform/tf-security-scanning.md)
