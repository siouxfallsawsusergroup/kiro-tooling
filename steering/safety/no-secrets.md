---
inclusion: always
name: no-secrets
description: Keep credentials and internal secrets out of prompts, steering, and commits.
tags:
  - type/steering
  - aws/security
---

# Safety

- Never put secrets, access keys, passwords, session tokens, or private keys in steering files, skills, prompts, or git.
- Prefer short-lived credentials (`aws login`, IAM Identity Center, or a role). Do not create long-lived IAM user access keys for people or for agents.
- Sample account IDs, client IDs, and ARNs in this pack are placeholders. Replace them; do not treat them as real resources.
- Before adding infrastructure, name the cost drivers you are about to turn on (NAT gateways, idle load balancers, CloudWatch log volume, cross-AZ traffic) and tag the resources so the bill can be traced.
