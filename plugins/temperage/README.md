# temperage

Claude Code and Cowork plugin. One skill: **webflow-template**. Turns a
design into an unpublished draft page in a Webflow site, built from a reusable
template family. Never publishes. The catalog lives in your team's Notion
(or a working folder) and is mirrored into Webflow Agent
Instructions.

## Installation

From the marketplace root:

```
/plugin marketplace add DayFlo/temperage
/plugin install temperage@temperage
```

## Usage

```
/temperage:webflow-template
```

Ask for a flow by name: onboard, build, maintain, sync, or resume. Onboard a
site before the first build.

Cowork: **Customize → Plugins → Add from a repository**, paste the repository
URL, install **temperage**, invoke the skill by name.

See the [repository README](../../README.md) for Codex and Claude.ai install,
the organization setup, the public-exposure table, and the full rulebook.

## Structure

```
temperage/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json                  # Webflow MCP server
└── skills/
    └── webflow-template/
        └── SKILL.md
```
