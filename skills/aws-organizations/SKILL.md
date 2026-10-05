---
name: aws-organizations
description: Explain and review AWS Organizations multi-account layout, OUs, SCPs, and IAM Identity Center. Use when designing accounts, guardrails, or a landing zone for a small team.
metadata:
  author: Sioux Falls AWS User Group
---

# AWS Organizations, multi-account basics

Use this when someone is splitting one AWS account into a few accounts, or reviewing a layout that already has a management account. This is the starter pattern the Sioux Falls AWS User Group wanted available for early meetups. It is not a full landing-zone build.

Prefer the AWS MCP Server (see the `agent-toolkit-for-aws` skill) when you need live account facts. Do not guess account ids, OU ids, or SCP contents that you have not read.

## When more than one account helps

A second account is worth it when a mistake, a bill, or a set of permissions should not touch production. Typical split for a small team:

| Account | What lives there |
| --- | --- |
| Management | Organizations, billing, IAM Identity Center. Almost no workloads. |
| Security / log archive | Organization CloudTrail, Config snapshots if you use them. Tight access. |
| Workloads — prod | Production only. |
| Workloads — nonprod | Dev and test. |
| Sandbox | Experiments. SCPs still apply. Expect the account to be noisy and cheap to rebuild. |

One OU level is enough at the start:

```text
Root
├── Security
├── Workloads
│   ├── Prod
│   └── NonProd
└── Sandbox
```

Put the SCP on the OU when every account in that OU should share the guardrail. Put it on one account only when the exception is real.

## SCPs do not grant access

A service control policy sets the maximum permissions in a member account. It does not give anyone permission. Identity-based policies and permission boundaries still have to allow the action. The management account is not affected by SCPs the way member accounts are, so keep workloads out of it.

Starter guardrails to discuss before writing JSON:

- Deny the root user as a principal in member accounts (leave password recovery to a break-glass process you have actually tested).
- Deny leaving the organization.
- Deny disabling CloudTrail or deleting the organization trail.
- Optionally deny Regions you will not use. Add this after you know which Region your services need, including global services that call `us-east-1`.

Show the SCP JSON and say which OU it attaches to. Call out anything the deny would break (for example a Region deny that also blocks IAM Identity Center's home Region).

## People

Use IAM Identity Center in the management account (or a delegated admin) for human access. Assign a permission set to an account. Do not create IAM users with access keys in each account so that agents or laptops can "just log in."

Break-glass access is one role, MFA, and a written reason to use it. It is not a shared password in a steering file.

## Trail and cost

- An organization trail in the log archive account records management events from member accounts. Data events are optional and can be a large CloudWatch or S3 bill. Turn them on for the buckets that matter.
- Tag resources with `Environment`, `Project`, `Owner`, and `CostCenter` so Cost Explorer can group the bill. Activate those tag keys in the billing console or they will not show up.
- Name the expensive defaults before you recommend them: a NAT gateway per AZ, interface VPC endpoints, CloudTrail data events, and AWS Config recording everything.

## Review checklist

When asked to review a layout, answer these in order:

1. Is production in its own account, and is the management account free of workloads?
2. Where would an SCP attach, and what does it deny? Does it accidentally deny the security account from logging?
3. How do humans sign in, and is there any long-lived access key?
4. Where do CloudTrail logs land, and who can delete them?
5. What will this cost in an idle month?

If the user has no organization yet, describe the first three accounts (management, nonprod, prod) and stop. Do not generate a 40-account design for a team of two.
