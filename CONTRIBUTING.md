# Contributing

This repo is a public pack of Kiro examples for the Sioux Falls AWS User Group. A useful contribution is one file other members can copy into `.kiro/` or `~/.kiro/` and run.

## Before you open a PR

- Match the current Kiro docs for the file type you are adding ([kiro.dev/docs](https://kiro.dev/docs/)). A skill is a folder with `SKILL.md`. A hook is JSON with `"version": "v1"`. A power is an Agent Plugin (`plugin.json`) under `powers/`.
- Say, in the file or in the PR, whether it is for Kiro IDE, Kiro CLI, Kiro Crew, or more than one.
- Keep one topic per steering file. Set `inclusion` so it does not have to load on every chat. `always` is for short rules only.
- Use placeholders for account ids, client ids, and ARNs. No secrets, no real credentials, no internal-only system names.
- Do not add a license file in a drive-by change.

## How to try your copy

Clone the branch, copy the changed directory into a scratch project's `.kiro/` folder (powers go to `~/.kiro/powers/`), and start a new Kiro chat. In the PR, write the sentence you typed and what you expected to see.

Pull requests are the path for changes. Issues are welcome when you are not sure of the Kiro shape yet.
