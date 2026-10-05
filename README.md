# Kiro tooling

Shared [Kiro](https://kiro.dev/) resources from the [Sioux Falls AWS User Group](https://www.siouxfallsawsuser.group/). Steering files, skills, an agent, a hook, a power, MCP snippets, and a workflow recipe you can copy into Kiro IDE, Kiro CLI, or Kiro Crew.

The examples are community guidance for mixed experience levels. They are not a company standard and not a substitute for your own review.

- User group: [siouxfallsawsuser.group](https://www.siouxfallsawsuser.group/)
- GitHub org: [siouxfallsawsusergroup](https://github.com/siouxfallsawsusergroup)
- Kiro: [kiro.dev/docs](https://kiro.dev/docs/)
- Crew: [kiro.dev/crew](https://kiro.dev/crew/)

## What to copy where

Kiro does not read this repository layout in place. Copy the directory you want into the project (`.kiro/`) or into your home directory (`~/.kiro/`). Project files win when the same name exists in both places.

| In this repo | Project | Global | Surfaces |
| --- | --- | --- | --- |
| `steering/` | `.kiro/steering/` | `~/.kiro/steering/` | IDE, CLI, Web. Crew loads steering from Agent Capabilities. |
| `skills/` | `.kiro/skills/` | `~/.kiro/skills/` | IDE, CLI, Web, Crew |
| `agents/` | `.kiro/agents/` | `~/.kiro/agents/` | IDE and CLI can select the agent. Web can delegate to a project agent. Crew reads `~/.kiro/agents/`. |
| `hooks/` | `.kiro/hooks/` | `~/.kiro/hooks/` | IDE, CLI, Web. Not mobile. |
| `workflows/` | `.kiro/workflows/` | `~/.kiro/workflows/` | IDE, CLI, Web recipes. Turn workflows on, then start a new chat. |
| `mcps/*.json` | merge into `.kiro/settings/mcp.json` | merge into `~/.kiro/settings/mcp.json` | IDE and CLI. Crew uses its Integrations panel instead. |
| `powers/aws-account-hygiene/` | not used (powers are global) | `~/.kiro/powers/aws-account-hygiene/` | IDE, and CLI v3. Import from the Powers panel if you prefer. |
| `bots/` | — | — | Not a Kiro feature. The README there explains what to use. |

Specs that Kiro generates stay in the project at `.kiro/specs/`. This pack does not ship a spec.

### IDE, CLI, and Crew

- **Kiro IDE** uses the paths above. Custom agents do not load skills or steering unless the agent lists them. The reviewer in `agents/` already references `skill://` and `file://.kiro/steering/**/*.md`.
- **Kiro CLI** uses the same files. CLI 3 supports the hook format in `hooks/`. Workflows are enabled under `/settings` → Features. OAuth for the AWS MCP Server can be added with `kiro-cli mcp add` (CLI 2.11 or later) instead of the IDE JSON snippet.
- **Kiro Crew** is a persistent workspace on top of Kiro CLI, with its own gateway under `~/.kiro/crew/`. It shares agent, skill, and steering files, and it configures MCP and chat channels in the dashboard. Crew workflows are Python scripts in Crew's library (`kirocrew app enable workflows`), not the YAML recipe in `workflows/`. See `bots/README.md` before looking for a bot config.

A short copy for a single project:

```bash
git clone https://github.com/siouxfallsawsusergroup/kiro-tooling.git
cd your-app
mkdir -p .kiro
cp -R /path/to/kiro-tooling/steering \
      /path/to/kiro-tooling/skills \
      /path/to/kiro-tooling/hooks \
      /path/to/kiro-tooling/agents \
      /path/to/kiro-tooling/workflows \
      .kiro/
```

Global copy is the same destination with `~/.kiro/` instead of `.kiro/`. Copy the power separately:

```bash
mkdir -p ~/.kiro/powers
cp -R powers/aws-account-hygiene ~/.kiro/powers/
```

Or, in the IDE: Powers panel → Add Custom Power → Import power from a folder, and select `powers/aws-account-hygiene`.

## Directory map

| Path | What it is |
| --- | --- |
| `steering/` | Conventions grouped by topic. Only `steering/safety/no-secrets.md` is `inclusion: always`. Terraform files load on `*.tf` / `*.hcl`. The Cognito note is manual (`#auth-cognito-authguard`). The rest use `inclusion: auto`. |
| `skills/agent-toolkit-for-aws/` | Install AWS skills and the AWS MCP Server on Kiro. |
| `skills/aws-organizations/` | Multi-account starter: accounts, OUs, SCPs, Identity Center. |
| `skills/spec-driven-delivery/` | Requirements, design, tasks, and when to use a workflow or Crew. |
| `mcps/` | OAuth and SigV4 snippets for the AWS MCP Server. Read `mcps/README.md` before copying. |
| `agents/well-architected-reviewer.json` | Read-only review agent. It does not write files. |
| `hooks/terraform-fmt.json` | `terraform fmt` after a `.tf` save, plus a reminder when IAM-related files change. |
| `powers/aws-account-hygiene/` | Power (Agent Plugin) that bundles the AWS MCP proxy and a short IAM hygiene skill. |
| `workflows/spec-review.yaml` | Background requirements draft and an independent review. No application code. |
| `bots/README.md` | Why this folder is not a config drop. |

## Agent Toolkit for AWS

The fastest setup on a machine that already has Kiro and AWS CLI `2.35.0` or later:

```bash
aws configure agent-toolkit
```

That installs default AWS skills into `~/.kiro/skills` and writes the MCP server into `~/.kiro/settings/mcp.json`.

To do the two parts yourself:

```bash
npx skills add aws/agent-toolkit-for-aws/skills
```

Then merge either `mcps/aws-mcp-oauth.json` or `mcps/aws-mcp-sigv4.json` into `.kiro/settings/mcp.json` or `~/.kiro/settings/mcp.json`. SigV4 needs `uvx` and `aws login`. OAuth on Kiro IDE uses the `?oauth=initialize` URL. Details, the IAM policy, and a connection check are in [`skills/agent-toolkit-for-aws/SKILL.md`](skills/agent-toolkit-for-aws/SKILL.md).

Docs: [AWS MCP Server](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html), [aws configure agent-toolkit](https://docs.aws.amazon.com/cli/latest/reference/configure/agent-toolkit.html), [agent-toolkit-for-aws](https://github.com/aws/agent-toolkit-for-aws).

## Try it

1. Copy `skills/spec-driven-delivery` and `skills/agent-toolkit-for-aws` into `.kiro/skills/` or `~/.kiro/skills/`.
2. Ask Kiro to enable the Agent Toolkit, or run `aws configure agent-toolkit` if the CLI is new enough.
3. Open a chat and ask for a review of a small design with the `well-architected-reviewer` agent (after copying `agents/`).
4. If workflows are enabled, run `spec-review` with a one-sentence task and a `run_dir` under the workspace. The steps are in [`workflows/README.md`](workflows/README.md).

In Crew, copy the skills and the agent into `~/.kiro/`, add the AWS MCP Server from Integrations, and use `/spec-driven-delivery` when you want a spec before a Task Runner plan.

## Safety

Steering and skills are loaded into the model context. Do not put secrets, account passwords, or access keys in them. The always-on note in `steering/safety/no-secrets.md` says the same thing after you copy it.

## Contributing

Open a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md).
