# Bots

Kiro does not load a `bots/` directory. Nothing you put here is picked up by Kiro IDE, Kiro CLI, or Kiro Crew.

Use these instead:

| You want | Where it actually lives |
| --- | --- |
| A specialist the IDE or CLI can switch to | [`agents/`](../agents/). Project path `.kiro/agents/`, or global `~/.kiro/agents/`. |
| The same idea in Kiro Crew | Crew reads agent JSON from `~/.kiro/agents/`. The dashboard path is Agent Capabilities → Agents. |
| Slack, Discord, Teams, or another chat channel | Crew dashboard → Messaging Channels. Channels are outbound from the Crew gateway. They are not files in this repo. |
| The name shown on the Crew dashboard | `kirocrew config set dashboard.bot_name "Your name"` in `~/.kiro/crew/config.json`. That key labels the dashboard. It does not define a bot. |

Crew install and the gateway are documented at [kiro.dev/docs/crew](https://kiro.dev/docs/crew/installation/).

This folder stays so the pack can say, in the place people look, that chat bots are a Crew channel setting and custom behavior is an agent.
