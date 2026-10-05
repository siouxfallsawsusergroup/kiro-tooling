---
inclusion: fileMatch
fileMatchPattern: ["**/*.tf", "**/*.tfvars", "**/*.hcl"]
name: tf-security-scanning
description: Static analysis and policy checks to run before terraform apply. Use when reviewing or scanning Terraform.
tags:
  - type/steering
  - tool/terraform
  - aws/security
---

# Terraform security scanning

Run static checks before a plan is applied. A failing check on a pull request should block merge once the repo has CI.

## Scanners

| Tool | Purpose |
| --- | --- |
| `tfsec` or `trivy config` | Misconfiguration: encryption, public exposure, logging. |
| `checkov` | Broad policy checks, including custom policies. |
| `terrascan` | Policy packs (CIS and others) over the config. |
| `tflint` | Provider-aware lint and deprecated syntax. |

## Run them

```bash
terraform fmt -check -recursive
tflint --recursive
tfsec .            # or: trivy config .
checkov -d . --quiet
terrascan scan -i terraform -d .
```

Install the tools you will actually maintain. Two scanners you read are better than five that nobody triages.

## Policy checks on the plan

OPA and Conftest can evaluate the plan JSON:

```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
conftest test tfplan.json --policy policy/
```

Do not commit `tfplan.binary` or `tfplan.json`. Plans can contain sensitive values.

If you use HCP Terraform or Terraform Enterprise, Sentinel (or a policy set) can enforce the same ideas at the run level: required tags, allowed Regions, no public buckets.

## Baseline to enforce

- Encryption at rest on S3, EBS, RDS, and DynamoDB.
- No `0.0.0.0/0` on SSH, RDP, or database ports.
- S3 Block Public Access on, and no public ACLs.
- Tags present: Environment, Project, Owner, CostCenter.
- No IAM policy that allows `*` on `*`.
- Logging you can search later: VPC Flow Logs, a CloudTrail trail, and access logs on public endpoints.

## In CI

```yaml
# .github/workflows/tf-scan.yml (excerpt)
- run: tflint --recursive
- run: tfsec . --soft-fail=false
- run: checkov -d . --quiet
- run: conftest test tfplan.json --policy policy/
```

A suppression needs an inline comment that says why, and a reviewer should read that comment. A suppression without a reason does not count.

## Related

- [Terraform standards](tf-standards.md)
- [State management](tf-state-management.md)
- [Module design](tf-module-design.md)
- [CloudFormation testing and drift](../cloudformation/cfn-testing-and-drift.md)
- [AWS security standards](../cloudformation/cfn-security-standards.md)
