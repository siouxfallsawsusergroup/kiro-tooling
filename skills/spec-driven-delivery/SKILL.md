---
name: spec-driven-delivery
description: Drive a change with a Kiro spec and tests before implementation. Use when starting a feature, planning with Kiro Crew, writing requirements, or deciding between a spec, a workflow, and a single chat.
metadata:
  author: Sioux Falls AWS User Group
---

# Spec-driven delivery

Use a spec when the change is large enough that you would regret skipping the plan: more than one file with behavior, anything touching auth or data, or work you might hand to another session. A one-line fix can stay in the chat.

Kiro stores feature specs in the project only:

```text
.kiro/specs/<feature-name>/
├── requirements.md    # or bugfix.md for a bug
├── design.md
└── tasks.md
```

Docs: [Specs](https://kiro.dev/docs/specs/), [Feature specs](https://kiro.dev/docs/specs/feature-specs/), [Quick Spec](https://kiro.dev/docs/specs/quick-spec/).

## Pick a path

| Situation | Use |
| --- | --- |
| New behavior, you want a gate between phases | IDE spec workflow, Requirements → Design → Tasks. Approve each file before the next. |
| You already know the shape and want the three files quickly | Quick Spec. You still read the files before anyone implements them. |
| A bug with a known symptom | Bugfix spec (`bugfix.md` for current behavior, expected behavior, and what must not change). |
| The work should continue in the background with separate review | A workflow recipe in `.kiro/workflows/` (IDE, CLI, and Web). See `workflows/spec-review.yaml` in this pack. |
| Kiro Crew, one autonomous run from a written spec | Crew Task Runner. |
| Kiro Crew, the same stages every time across many agents | A Crew workflow (a Python script in Crew's workflow library). That is a different feature from `.kiro/workflows/` recipes. |

Crew overview: [Kiro Crew](https://kiro.dev/crew/). Crew workflows: [Crew workflows](https://kiro.dev/docs/crew/features/workflows/).

## Requirements before code

Write requirements a test can fail. Prefer EARS-style statements, which is what Kiro feature specs use:

- When the caller is unauthenticated, the API returns 401.
- While the shopping cart is empty, checkout is disabled.

Each requirement gets a name and an acceptance check. If you cannot say how you would notice the requirement is wrong, it is still a wish.

Ask for an analysis pass before design: contradictions, missing error cases, and words like "fast" or "secure" that have no threshold.

## Design, then tasks

`design.md` names the components, the data, and the failure behavior. It does not paste the whole implementation.

`tasks.md` is a list of small outcomes. A task is done when its check passes, not when a file exists. Keep tasks ordered by dependency. Mark which ones can run in parallel.

When requirements change, update `requirements.md`, then design, then sync tasks. Do not let `tasks.md` become the only copy of the plan.

## Tests

For each acceptance criterion, name the test or command that proves it before writing the production code. A useful order:

1. The failing check (unit, integration, or a documented manual script).
2. The smallest code that makes that check pass.
3. The next criterion.

If there is no harness yet, the first task is the harness, not the feature.

## Review without the author's reasoning

A second session should see the artifact (the requirements file, the diff, the test output), not the chat that produced it. That is why a workflow step runs in its own session. In Crew, spawn a reviewer subagent with the file path and the acceptance criteria, and do not paste the implementation transcript into the review prompt.

Keep these decisions with a person: whether to ship, a new account boundary, a data model that is expensive to migrate, and anything a customer would notice as a policy change. Leave local structure, naming, and test layout to the agent when the spec already decided the behavior.

## In this repository

- Copy this skill to `.kiro/skills/spec-driven-delivery/` or `~/.kiro/skills/spec-driven-delivery/`.
- The `well-architected-reviewer` agent references it with `skill://`.
- Invoke it in chat with `/spec-driven-delivery` when you want the steps even if the description did not match.
