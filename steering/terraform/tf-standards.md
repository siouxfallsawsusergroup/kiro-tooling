---
inclusion: fileMatch
fileMatchPattern: ["**/*.tf", "**/*.tfvars", "**/*.hcl"]
name: tf-standards
description: Terraform naming, version pinning, tagging, and validation. Use when editing Terraform.
tags:
  - type/steering
  - tool/terraform
  - aws/governance
---

# Terraform standards

These rules apply to `*.tf`, `*.tfvars`, and `*.hcl` files.

## Naming

- Resources, variables, and outputs use `snake_case`.
- The local name describes the role, not the type: `aws_vpc.main`, not `aws_vpc.vpc`.
- Split files by concern: `versions.tf`, `providers.tf`, `variables.tf`, `outputs.tf`, `main.tf`, `data.tf`.
- Put the environment in tags and in the state key, not in every resource name.

## Provider and version pinning

Set `required_version` and pin providers with `~>`. A bare `>=` drifts.

Commit `.terraform.lock.hcl`. Refresh the lock for the platforms CI uses with `terraform providers lock`.

```hcl
terraform {
  required_version = "~> 1.9"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.60"
    }
  }
}
```

## Variables

Every variable is typed and described. Constrain enums with `validation`. Mark secrets `sensitive = true` and do not give them defaults.

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment."
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be dev, staging, or prod."
  }
}
```

## Outputs

Describe every output. Mark sensitive outputs `sensitive = true`. Export what the next module needs, not the whole resource object.

## Tags

Apply a small, consistent tag set once with `default_tags`. Do not repeat it on every resource.

```hcl
provider "aws" {
  region = var.region
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project
      Owner       = var.owner
      CostCenter  = var.cost_center
      ManagedBy   = "terraform"
    }
  }
}
```

`Owner` should be a team mailbox or a shared channel, not one person's personal inbox, so the tag survives vacations.

## Secrets

Do not commit passwords, tokens, keys, or live ARNs that contain account-specific secrets. Read secrets at apply time from SSM Parameter Store or Secrets Manager.

```hcl
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "example/app/db"
}
```

## Checks

`terraform fmt -recursive` and `terraform validate` pass before review. `tflint` should pass when it is installed. Scanning is covered in [Terraform security scanning](tf-security-scanning.md).

## Related

- [State management](tf-state-management.md)
- [Module design](tf-module-design.md)
- [Security scanning](tf-security-scanning.md)
- [CloudFormation design](../cloudformation/cfn-design-best-practices.md)
