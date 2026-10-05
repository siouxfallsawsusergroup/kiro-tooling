---
inclusion: auto
name: dev-code-review
description: Code review checklist for authors and reviewers. Use when reviewing a change or preparing one for review.
tags:
  - type/steering
  - domain/development
---

# Code review

## Before you ask for review

- The change follows the surrounding style.
- Commented-out blocks and debug logging are gone.
- Errors are handled, and messages do not leak secrets or stack traces to users.
- New behavior has a test, or you wrote down the manual check you ran.
- Public behavior and setup steps are updated when they changed.

## What reviewers look at

1. **Correctness** — does it do what the description says, including the empty and failure cases?
2. **Security** — input handling, authz, secrets, and overly broad IAM.
3. **Operability** — logs, timeouts, and what happens when a dependency is down.
4. **Clarity** — can the next person change this without a guided tour?

Priority:

- **Blocking** — security issues, bugs, broken behavior, or a change that cannot be rolled back safely.
- **Important** — performance, missing tests on a risky path, or a design that will be expensive to undo.
- **Optional** — naming, small cleanups, and style nits that do not change behavior.

Say what you observed and why it matters. Point at a concrete alternative when you have one.

## When to look harder

Slow down for authentication and authorization, data storage, third-party calls, infrastructure, and dependency upgrades.

- Input is validated.
- Queries are parameterized.
- Browser output is encoded where it meets untrusted data.
- No secrets in source, logs, or fixtures.
- Errors shown to users stay generic; detail stays in logs you control.

## Approval

- Everyday changes: one approval, and automated checks green when they exist.
- Auth, IAM, production data, or hard-to-reverse infrastructure: a second reviewer when someone is available.
- Urgent fixes: one reviewer who understands the area, then a short follow-up after deploy. Write down what happened if users were affected.

## Related

- [Git workflow](dev-gitflow.md)
- [Terraform security scanning](../terraform/tf-security-scanning.md)
- [CloudFormation testing and drift](../cloudformation/cfn-testing-and-drift.md)
