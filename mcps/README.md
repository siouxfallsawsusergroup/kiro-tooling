# MCP examples

These files are snippets for Kiro's MCP config. They are not loaded from this folder.

Kiro reads:

- `.kiro/settings/mcp.json` for the current project
- `~/.kiro/settings/mcp.json` for every project

Merge one server entry into that file. If both files define `aws-mcp`, the project file wins.

| File | When to use |
| --- | --- |
| `aws-mcp-oauth.json` | Kiro IDE, browser sign-in, one AWS account at a time. The `?oauth=initialize` query is what the AWS docs specify for Kiro IDE. |
| `aws-mcp-sigv4.json` | Kiro IDE or Kiro CLI when you use `aws login` or named profiles and want to change accounts. Requires `uvx` and the AWS CLI. |

Kiro CLI 2.11 or later can register the OAuth endpoint without editing JSON:

```bash
kiro-cli mcp add --name aws-mcp --url https://aws-mcp.us-east-1.api.aws/mcp
```

Change `us-east-1` in the hostname to another [supported MCP Region](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html) (`us-west-2`, `eu-west-1`, and others). In the SigV4 file, `AWS_REGION` is the default Region for API calls and can differ from the endpoint Region.

Remove `aws-api-mcp-server` and `aws-knowledge-mcp-server` if they are still present. Do not commit credentials in `env` or `headers`. The `agent-toolkit-for-aws` skill has the IAM policy, prerequisites, and a connection test.

Kiro Crew configures MCP from its Integrations panel. Paste the SigV4 `command` and `args` there, or add the remote URL if the panel accepts one. Crew does not auto-load this directory.
