---
name: account-hygiene
description: Check IAM and account changes for long-lived keys, admin policies, and missing guardrails. Use when editing IAM, SCPs, or account structure.
---

# Account hygiene

Use this when the conversation is about IAM, access keys, SCPs, or which account a workload should live in.

## Before recommending a change

1. Say which account and principal the change applies to. If that is unknown, ask once.
2. Prefer this order for human access: IAM Identity Center permission set, then an IAM role a person assumes, then nothing else. Do not create an IAM user access key.
3. For a workload, prefer a role the service can assume (task role, instance profile, Lambda role). The policy names the actions and the resource ARNs you know. Where the resource is not known yet, narrow the action list and mark the resource as a follow-up, rather than using `*` on `*`.
4. SCPs are maximum permissions on member accounts. They do not grant access. Attach a deny at the OU when every account there should share it.
5. If the AWS MCP Server from this power is connected, confirm the service's IAM action names there before inventing them. The first connection needs `uvx` and working AWS credentials (`aws login` or a profile). See the `agent-toolkit-for-aws` skill for OAuth versus SigV4.

## What to flag

- `aws_iam_access_key` or any step that tells a person to run `aws configure` with a static key.
- `AdministratorAccess`, `PowerUserAccess` on production, or a policy with `Action` `*` and `Resource` `*`.
- A trust policy whose principal is `"AWS": "*"` or an account the user did not name.
- Production workloads proposed for the Organizations management account.

## What to suggest instead

- A permission set for the person, scoped to the account they need that day.
- A customer-managed policy with the actions the API call actually uses.
- An SCP on the workloads OU that denies disabling CloudTrail and denies the root user, once the user has an organization. Do not pretend a team without Organizations already has those OUs.

Keep the answer short enough to paste into a pull request.
