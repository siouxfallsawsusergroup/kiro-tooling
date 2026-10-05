---
inclusion: auto
name: doc-standards
description: README, ADR, and API documentation expectations. Use when writing or updating project docs.
tags:
  - type/steering
  - domain/documentation
---

# Documentation standards

Write the doc you would want on the day you return to this project. Keep it next to the code it describes.

## What to document

- Public functions, APIs, and configuration knobs, with one real example.
- A README that can get a new person to a running state.
- An architecture decision record when you pick a library, a service, or a security approach you might later question.
- An API description (OpenAPI or the equivalent) when other people call the interface.

## README

Cover, in this order when they apply:

1. What it is, and who it is for.
2. Prerequisites.
3. Install and first run.
4. A short usage example.
5. Configuration, including which environment variables are required.
6. How to contribute.

Add a license section when the project has a license. Troubleshooting, a changelog, and a roadmap are useful once the project has users, and optional before that.

## Architecture decision records

```markdown
# ADR-001: [Decision title]

## Status

[Proposed | Accepted | Deprecated | Superseded]

## Context

What problem forced a choice?

## Decision

What did we choose?

## Consequences

What gets easier, and what gets harder?
```

Write an ADR for stack choices, architecture patterns, a third-party service, or a security approach. A few sentences is enough. Link the ADR from the README when later readers would not find it otherwise.

## APIs

- Every endpoint has a purpose, parameters, and an example.
- Error responses and status codes are listed.
- Authentication is named (which header, which token, who issues it).
- Defaults are explicit.

## Keeping docs current

Update docs in the same change as the behavior they describe. When you notice a broken step, fix it or file an issue in the same session. A quarterly pass for links is plenty for a small project.

## Related

- [Code review](../development/dev-code-review.md)
- [Git workflow](../development/dev-gitflow.md)
