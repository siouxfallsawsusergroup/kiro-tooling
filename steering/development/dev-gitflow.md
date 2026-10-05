---
inclusion: auto
name: dev-gitflow
description: Lightweight git branch, commit, and pull request conventions. Use when creating branches, writing commits, or opening a pull request.
tags:
  - type/steering
  - domain/development
---

# Git workflow

A small, consistent git habit is enough for most user-group and side projects. Use a ticket id in the branch name when your team already tracks work that way. Skip it when you do not have one.

## Branches

Prefer:

```text
<type>/<short-description>
```

Optional, when a tracker id exists:

```text
<type>/<ticket-id>/<short-description>
```

Types:

- `feature/` — new behavior
- `bugfix/` — a defect fix
- `hotfix/` — an urgent fix on the released line
- `chore/` — maintenance that does not change behavior
- `docs/` — documentation only

Examples:

- `feature/user-authentication`
- `bugfix/login-timeout`
- `feature/123/user-authentication` (ticket id included because one exists)

Keep the description short and lowercase, with hyphens.

## Commits

```text
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `security`.

```text
feat(auth): add authorization-code login

fix(api): stop retrying after the upstream times out

docs(readme): document the local setup steps
```

The subject says what changed. The body, when you need one, says why.

## Pull requests

- Title follows the same convention as a commit subject.
- The description says what changed and why.
- Link an issue or ticket when you have one. A missing ticket is not a reason to invent an id.
- Include a screenshot when the change is visual.
- Note how you checked it (tests run, or the manual path you tried).

Review:

- One reviewer is enough for most changes.
- Ask for a second look on authentication, IAM, production data, or infrastructure that is hard to undo.
- CI checks should pass before merge when the repo has them.
- Do not merge with unresolved conflicts.

## Protected branches

For a shared `main` branch, these settings are a good default:

- Changes land through a pull request.
- At least one review.
- Required status checks, when CI exists.
- The branch is up to date before merge.

Restricting direct pushes is worth it once more than one person commits. A release branch can use the same rules; an extra "release owner" approval is optional, not required.

## Related

- [Code review](code-review.md)
- [Documentation standards](../documentation/doc-standards.md)
