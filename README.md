# Glorya for AI agents

Make on-brand image ads, video ads, landing pages and websites, and launch ad campaigns, from Claude Code, Cursor, Codex or any MCP app. Glorya already knows your brand. You say what you want; it does the work and hands you the links.

## What you get

- **Image ads** for your brand, ready for feed and story placements.
- **Video ads** in the styles Glorya offers, from script to final cut.
- **Landing pages** that convert, with a preview link before anything goes live.
- **Websites** built and published when you say so.

You need a Glorya account with an active plan. Jobs use your Glorya credits, the same as in the platform.

## Set up in 2 minutes

**Claude Code: connector and skill in one step**

```
/plugin marketplace add Glorya-AI/skills
/plugin install glorya@glorya
```

Then run `/mcp`, pick **glorya**, choose **Authenticate**, sign in with your Glorya account and pick your default brand.

**Any agent (Claude Code, Cursor, Codex and more): the skill**

```
npx skills add Glorya-AI/skills
```

Also listed on [skills.sh](https://skills.sh/glorya-ai/skills/glorya).

Then connect the Glorya MCP server in your app: `https://platform.glorya.ai/mcp`.

## Use it

Ask in plain words:

- "Make an image ad for our spring offer."
- "Create a 20-second video ad for the new product."
- "Build a landing page that collects leads for the free consultation."
- "Make a UGC video ad, and let me review the script."

Glorya finishes on its own. Say "let me review" when you want to approve the concept, script or frames along the way. If Glorya needs a decision, your agent shows you its question with the exact choices. When the job is done you get the results and a link to open them in Glorya.

Glorya never launches or changes ad campaigns from here, and it only publishes a page or website when you ask.

## Docs

Full guide, tool reference and troubleshooting: [glorya.ai/docs/mcp](https://glorya.ai/docs/mcp).

## What's in this repo

| Path | What it is |
|---|---|
| `skills/glorya/SKILL.md` | The Glorya skill: how your agent runs Glorya jobs end to end |
| `skills/glorya/references/tools.md` | Every tool, input, status, error and limit, for any MCP client |
| `.mcp.json` | The Glorya MCP server, used by the Claude Code plugin |
| `.claude-plugin/` | Claude Code plugin and marketplace manifests |

Privacy: [glorya.ai/datenschutz](https://glorya.ai/datenschutz). License: MIT.
