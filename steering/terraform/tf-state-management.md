---
inclusion: fileMatch
fileMatchPattern: ["**/*.tf", "**/*.tfvars", "**/*.hcl"]
name: tf-state-management
description: Remote Terraform state, locking, and per-environment isolation. Use when configuring backends or moving state.
tags:
  - type/steering
  - tool/terraform
  - aws/security
  - aws/governance
---

# Terraform state

State is the map of what Terraform believes it owns. Keep it remote, encrypted, locked, and separate per environment.

## Rules

- Do not commit `terraform.tfstate` or store the working copy only on a laptop.
- Use an S3 backend with encryption on.
- Lock the state. On Terraform 1.10 or newer, S3 native locking (`use_lockfile = true`) is enough. Older versions use a DynamoDB lock table. Do not apply with no lock.
- One state per environment. Separate keys (or separate buckets). Do not share one state file across dev, staging, and prod.
- Version the state bucket so a bad write can be rolled back.
- Limit who can read the bucket. State often contains resource ids and sometimes sensitive attributes. A customer-managed KMS key is appropriate when the bucket holds production state.

Account IDs and key ARNs below are placeholders.

## S3 backend with a DynamoDB lock

```hcl
terraform {
  backend "s3" {
    bucket         = "example-tfstate-prod"
    key            = "network/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:111122223333:key/00000000-0000-0000-0000-000000000000"
    dynamodb_table = "example-tfstate-locks"
  }
}
```

## S3 native locking

```hcl
terraform {
  backend "s3" {
    bucket       = "example-tfstate-prod"
    key          = "network/prod/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    kms_key_id   = "arn:aws:kms:us-east-1:111122223333:key/00000000-0000-0000-0000-000000000000"
    use_lockfile = true
  }
}
```

## Separate keys per environment

```text
network/dev/terraform.tfstate
network/staging/terraform.tfstate
network/prod/terraform.tfstate
```

A mistake in dev then cannot rewrite prod state. Distinct backend keys are easier to reason about than many Terraform workspaces in one file.

## Hygiene

- Do not hand-edit state. `terraform state mv` and `terraform import` belong in a reviewed change, not in an unattended script.
- Do not commit state, or `.tfvars` files that hold secrets.
- Take a moment before state surgery. Bucket versioning is the safety net.

## Related

- [Terraform standards](tf-standards.md)
- [Module design](tf-module-design.md)
- [Security scanning](tf-security-scanning.md)
- [AWS security standards](../cloudformation/cfn-security-standards.md)
