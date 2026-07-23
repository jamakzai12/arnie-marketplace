# Arnie — Claude Code plugin

The Arnie GTM workspace for Claude Code: build tables, run workflows, save ingredients,
and automate go-to-market over the Arnie remote MCP server (`mcp__arnie__copilot_*`). The
plugin ships four skills (`arnie`, `arnie-tables`, `arnie-workflows`, `arnie-ingredients`)
and the `.mcp.json` that wires the `arnie` HTTP MCP server.

## Install

**Dev install (from this repo):**

```
/plugin marketplace add ./packages/arnie-claude-plugin
/plugin install arnie@arnie
```

**Public install (marketplace-only users):**

```
/plugin marketplace add jamakzai12/arnie-marketplace
/plugin install arnie@arnie
```

The public marketplace repo lives at github.com/jamakzai12/arnie-marketplace.

## Key

The `arnie` MCP server authenticates with `ARNIE_MCP_KEY`. Get a **scoped** key from
**Settings → MCP Keys** with scopes `discovery,workspace,recipes,ingredient_builder` (the
desktop scope set) — do **not** use the full-scope default.

**`ARNIE_MCP_KEY` must be set in the environment where `claude` actually runs.** The plugin's
`.mcp.json` expands `${ARNIE_MCP_KEY}` from that environment:

```
export ARNIE_MCP_KEY=<your-scoped-key>
```

Put the export in the shell/profile that launches `claude` — a launch with no
`ARNIE_MCP_KEY` gets a silent 401 on every Arnie tool call. Inside the Arnie desktop app the
key is injected automatically from `~/.arnie/agent.env`, so no manual export is needed there.
