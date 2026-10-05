---
name: agent-toolkit-for-aws
description: Enable the Agent Toolkit for AWS on Kiro IDE or Kiro CLI. Use when installing AWS skills, configuring the AWS MCP Server, choosing OAuth or SigV4, or running aws configure agent-toolkit.
compatibility: Kiro IDE or Kiro CLI. AWS CLI 2.35.0+ for the setup wizard. uv for the SigV4 proxy.
metadata:
  author: Sioux Falls AWS User Group
---

# Enable the Agent Toolkit for AWS

The toolkit has two independent parts. Skills are local instruction files. The AWS MCP Server is a remote tool endpoint for AWS docs and API calls. Either one works without the other. Install both for the usual setup.

Official references:

- Setup wizard: `aws configure agent-toolkit` ([AWS CLI command reference](https://docs.aws.amazon.com/cli/latest/reference/configure/agent-toolkit.html))
- Skills: `npx skills add aws/agent-toolkit-for-aws/skills`
- MCP: [Setting up the AWS MCP Server](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html)
- Repo: [aws/agent-toolkit-for-aws](https://github.com/aws/agent-toolkit-for-aws)

Do not put access keys, tokens, or passwords in `mcp.json`, skills, or the chat.

## Fast path

Requires AWS CLI `2.35.0` or later. The wizard looks for a Kiro config directory (`~/.kiro`). Create that directory first if Kiro is installed but the folder is missing (open Kiro once, or `mkdir -p ~/.kiro`).

```bash
aws configure agent-toolkit
```

The wizard detects Kiro, installs the default AWS skills into `~/.kiro/skills`, and writes the AWS MCP Server entry into `~/.kiro/settings/mcp.json`. Skills land in the global config, not in a project.

Skip the prompts only when the defaults are what you want:

```bash
aws configure agent-toolkit --yes
```

Add one skill later:

```bash
aws agent-toolkit add-skill --skill-name aws-cdk --agent kiro
aws agent-toolkit search-skills --search-query iam
```

Restart Kiro after the config file changes if the server does not reconnect on its own.

## Install skills without the wizard

```bash
npx skills add aws/agent-toolkit-for-aws/skills
```

That command installs into `~/.kiro/skills/` (global) or `.kiro/skills/` (this project), depending on how the skills CLI is scoped. Each skill is a directory with a `SKILL.md`. Kiro reads the name and description at startup and loads the body when a task matches.

## Configure the AWS MCP Server

Remove older servers first if they are still configured, so tools do not collide:

- `aws-api-mcp-server`
- `aws-knowledge-mcp-server`

Edit either file:

- This project: `.kiro/settings/mcp.json`
- All projects: `~/.kiro/settings/mcp.json`

Workspace entries override a global server with the same name. Copy-paste snippets live in `mcps/` in this repo. Pick one authentication mode.

The MCP endpoint Region is the host you connect to. `AWS_REGION` in the SigV4 metadata is the default Region for AWS API calls, and it can differ. Endpoints exist for `us-east-1`, `us-west-2`, `ap-southeast-1`, `ap-southeast-2`, `ap-northeast-1`, `eu-central-1`, `eu-west-1`, and `eu-west-2`. The examples use `us-east-1`. Swap the Region in the hostname to use another endpoint.

### OAuth (simpler, one account at a time)

The IAM principal needs `AWSMCPSignInOAuthAccessPolicy` (`signin:AuthorizeOAuth2Access` and `signin:CreateOAuth2Token`):

```bash
aws iam attach-role-policy \
  --role-name MyRole \
  --policy-arn arn:aws:iam::aws:policy/AWSMCPSignInOAuthAccessPolicy
```

Attach it to the user instead of the role when that is who signs in. A 400 page after sign-in usually means this policy is missing.

Kiro IDE — remote server, with the query string the IDE docs require:

```json
{
  "mcpServers": {
    "aws-mcp": {
      "url": "https://aws-mcp.us-east-1.api.aws/mcp?oauth=initialize"
    }
  }
}
```

Kiro CLI 2.11 or later:

```bash
kiro-cli mcp add --name aws-mcp --url https://aws-mcp.us-east-1.api.aws/mcp
```

The first tool call opens a browser. Access tokens last about an hour and refresh for up to 12 hours. OAuth does not switch AWS accounts inside one session. Use SigV4 for that.

### SigV4 (IDE and CLI, multi-account)

Use this when you change accounts or profiles often, want a default Region, or want the proxy in front of the remote server.

1. AWS CLI `2.32.0` or later.
2. `aws login` (credentials rotate about every 15 minutes, for a session up to 12 hours). Confirm with `aws sts get-caller-identity`.
3. [uv](https://docs.astral.sh/uv/) so `uvx` can start the proxy.

```json
{
  "mcpServers": {
    "aws-mcp": {
      "command": "uvx",
      "timeout": 100000,
      "transport": "stdio",
      "args": [
        "mcp-proxy-for-aws-cli@latest",
        "https://aws-mcp.us-east-1.api.aws/mcp",
        "--metadata",
        "AWS_REGION=us-east-1"
      ]
    }
  }
}
```

Change `AWS_REGION` to the Region you want API calls to default to. The first `uvx` run can take a few minutes while the package downloads.

Read-only mode and named-profile switching are SigV4 features. See the AWS MCP Server user guide for the current flags. Do not invent a second copy of those flags here.

## Check that it worked

Ask: "What AWS Regions are available?"

In Kiro CLI, `/mcp` lists servers and `/tools` lists tools. Expect names such as `aws___search_documentation` and `aws___retrieve_skill`.

| Symptom | What to do |
| --- | --- |
| `ExpiredTokenException` | `aws login` again, or `aws sso login --profile <name>`, then restart the MCP client. |
| 400 after the OAuth browser step | Attach `AWSMCPSignInOAuthAccessPolicy`. |
| No credentials | `aws sts get-caller-identity` fails. Sign in before starting the proxy. |
| Tools never appear | Confirm the server is not `disabled`, and that an older AWS MCP entry was removed. |

## Kiro Crew

Crew does not read `.kiro/settings/mcp.json` the same way the IDE does. Add the AWS MCP Server from Crew's Integrations (MCP) panel, or point that panel at the same command and arguments as the SigV4 block above. Skills copied to `~/.kiro/skills/` are what Crew's skills browser discovers. The `aws configure agent-toolkit` wizard targets Kiro IDE and Kiro CLI config paths; confirm the files landed before you expect Crew to see them.
