---
inclusion: auto
name: cfn-security-standards
description: IAM, tagging, encryption, and detective-control expectations for AWS accounts. Use when changing IAM, encryption, backups, or security services.
tags:
  - type/steering
  - aws/security
  - tool/cloudformation
---

# AWS security standards

Practical defaults for an account you share with other people. Tighten them for production data. A sandbox can be smaller, and it should still not use the root user or long-lived access keys.

## IAM

- People do not use the account root user. Root has MFA and sits unused.
- People sign in through IAM Identity Center (or your existing SSO) and assume roles. Do not hand out long-lived IAM user access keys.
- Review who can do what a few times a year, or when someone leaves.
- A role's trust policy names the account or service that may assume it. Cross-account access uses an external id when a third party is on the other side, and a session length that matches the job.
- Delete roles nobody has used.
- Write a small customer-managed policy for the task. An AWS managed policy is fine when it matches the job (for example a read-only policy). Do not attach `AdministratorAccess` to get unblocked.
- Avoid `"Action": "*"` on `"Resource": "*"`. Resource-level permissions are preferred when the service supports them.
- Inline policies are harder to reuse and review. A named customer-managed policy is usually clearer.

## Tags

Use the same keys in Terraform and CloudFormation so cost and ownership reports agree.

```yaml
Environment: [dev|staging|prod]
Project: <project-name>
Owner: <team-email-or-shared-list>
CostCenter: <cost-center-id>
DataClassification: [public|internal|confidential|restricted]
```

Add `Backup`, `Monitoring`, or `PatchGroup` when you operate those processes. Skip tags nobody will query.

## Network

- Application and data tiers sit in private subnets.
- Public subnets are for load balancers and NAT.
- VPC Flow Logs are on.
- Security groups have descriptions. Inbound `0.0.0.0/0` is limited to the public listener (usually 443).
- Details and examples: [Network security baseline](../networking/net-security-baseline.md).

## Data

- Encrypt EBS, S3, and RDS.
- Secrets live in Secrets Manager or SSM Parameter Store, not in templates or task definitions.
- Clients use TLS. Prefer TLS 1.2 or newer on public endpoints.
- Backups have a retention period, and someone has restored one. Critical data that you cannot recreate gets a copy in another Region only after you accept that Region's cost.

## Detection

Turn these on in accounts that hold real data. A brand-new sandbox can start with CloudTrail and add the rest before production:

- CloudTrail, including a trail that is not limited to one Region.
- GuardDuty.
- Security Hub, if you will look at the findings.
- A few AWS Config rules for the controls you care about (public buckets, unrestricted SSH, missing encryption), not every managed rule on day one.

Useful alarms once you have a baseline: spend anomalies, and the security findings you have agreed to answer. A CPU alarm with no owner is noise.

## Related

- [CloudFormation design](cfn-design-best-practices.md)
- [CloudFormation testing and drift](cfn-testing-and-drift.md)
- [Network security baseline](../networking/net-security-baseline.md)
