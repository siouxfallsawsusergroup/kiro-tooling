# Workflows

`spec-review.yaml` is a [Kiro workflow recipe](https://kiro.dev/docs/workflows/). IDE, CLI, and Web load recipes from `.kiro/workflows/` in a project or from `~/.kiro/workflows/` globally. Copy the file to one of those directories. Workflows are off until you enable them, and a new chat is required after you do.

The recipe drafts requirements and asks a second agent to approve or return them. It stops when `review.json` contains `"verdict": "APPROVED"`, or after three passes. It does not write application code.

Launch it from the parent chat with a task and a `run_dir` inside the workspace, for example `.kiro/workflow-runs/health-check`.

Kiro Crew workflows are different. Crew uses a Python script in its own workflow library and the `/workflow` command inside Crew. Enable that surface with `kirocrew app enable workflows`. Use the `spec-driven-delivery` skill in this pack to choose between a spec, this recipe, and a Crew workflow.
