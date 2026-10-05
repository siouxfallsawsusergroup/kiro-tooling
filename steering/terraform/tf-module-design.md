---
inclusion: fileMatch
fileMatchPattern: ["**/*.tf", "**/*.tfvars", "**/*.hcl"]
name: tf-module-design
description: How to structure, version, and compose Terraform modules. Use when creating or consuming a module.
tags:
  - type/steering
  - tool/terraform
  - aws/governance
---

# Terraform module design

Write a module for one job, pin the version you consume, and compose modules at the root.

## Layout

```text
modules/vpc/
├── versions.tf     # required_version + required_providers (no provider block)
├── variables.tf    # typed, described, validated inputs
├── main.tf         # resources
├── outputs.tf      # described outputs
├── README.md       # purpose, inputs, outputs, example
└── examples/       # a runnable example
```

## Rules

- A module declares `required_providers` and does not configure the provider. The root module owns provider configuration, aliases, and `default_tags`.
- Every input is typed, described, and validated. Every output is described.
- One purpose per module (a VPC, a database). Compose them in the root. A module that creates "the whole platform" is hard to reuse and hard to review.
- No environment names baked into the module. Pass them in.

## Pin what you call

Pin a version. Do not track a floating branch.

```hcl
module "vpc" {
  source  = "example.com/platform/vpc/aws"
  version = "3.2.1"

  environment = var.environment
  vpc_cidr    = var.vpc_cidr
}
```

A git source pins a tag:

```hcl
module "vpc" {
  source = "git::https://example.com/tf-modules.git//vpc?ref=v3.2.1"
}
```

Tag releases with SemVer. A breaking input or output change bumps the major version.

## Composition

Prefer a flat root that calls small modules over modules nested several layers deep. Use `for_each` instead of `count` when the set of resources can be removed from the middle.

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"
}

module "app_sg" {
  for_each = var.services
  source   = "./modules/security-group"
  name     = "${local.name_prefix}-${each.key}"
  vpc_id   = module.vpc.vpc_id
}
```

## Related

- [Terraform standards](tf-standards.md)
- [State management](tf-state-management.md)
- [Security scanning](tf-security-scanning.md)
- [CloudFormation design](../cloudformation/cfn-design-best-practices.md)
