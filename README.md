# Arnie — GTM automation for Claude Code

**[arnies.ai](https://www.arnies.ai)** · Official Claude Code plugin

Arnie is a go-to-market workspace where **tables are workflows**: every row is a company,
lead, or task, and every column is a live, re-runnable step — enrichment from data
providers, AI research, formulas, conditions, scheduled re-runs, webhooks, and outbound
actions. You describe the outcome; Arnie's agent builds the machine that produces it and
keeps it running.

This plugin connects **your own Claude Code** to your Arnie workspace over Arnie's remote
MCP server, so Claude can build and operate your GTM workflows directly from the terminal.

## What you can do

- **Prospect and build lists** — source companies and people from data providers, qualify
  them against your ICP, and keep the list growing on a schedule.
- **Enrich anything** — emails, phones, LinkedIn profiles, funding, hiring signals,
  technographics — with per-cell cost transparency before anything paid runs at scale.
- **Build reusable workflows** — proven column chains become saved *functions* you (and
  your team) reuse across tables instead of rebuilding.
- **Bring your own providers** — save any HTTP API as a reusable *ingredient* with your
  own key (BYOK runs are free of Arnie credits), or use Arnie's built-in catalog.
- **Automate the loop** — schedules re-run proven columns, webhooks insert event rows,
  and connected tables cascade new data through the whole lineage automatically.

Everything Claude does through this plugin lands in your real Arnie workspace — the same
tables, workflows, and history you see at [app.arnies.ai](https://www.arnies.ai), in the
web app and the Arnie desktop app.

## How it works

The plugin is a thin client: a handful of skills that teach Claude how Arnie thinks
(tables-as-workflows, sample-before-scale, spend approval) plus an `.mcp.json` that wires
the `arnie` remote MCP server (`mcp__arnie__copilot_*` tools). All execution, data, and
billing run on Arnie's servers behind your personal, scoped API key — no product code
ships in this repo, and no key or secret is ever stored here.

**Safety defaults built in:** Claude samples 1–3 rows and shows real output before
widening any paid step, lays out the full workflow plan before creating one, and paid
runs at scale require your explicit go-ahead. Keys are scoped so the plugin can only do
what you granted.

## Install

```
/plugin marketplace add jamakzai12/arnie-marketplace
/plugin install arnie@arnie
```

(Working from a checkout of this repo instead: `/plugin marketplace add .` then the same
install command.)

## Get your key

1. Sign up / sign in at [arnies.ai](https://www.arnies.ai).
2. Open **Settings → MCP Keys** and create a **scoped** key with
   `discovery,workspace,recipes,ingredient_builder` (the desktop scope set) — avoid the
   full-scope default.
3. Export it in the shell that launches `claude`:

```
export ARNIE_MCP_KEY=<your-scoped-key>
```

A launch without `ARNIE_MCP_KEY` gets a silent 401 on every Arnie tool call, so put the
export in your shell profile. Inside the **Arnie desktop app** the key is provisioned and
injected automatically — no manual export needed there, and the app walks you through
this whole setup with a live checklist.

## The desktop app

The easiest way to run Arnie with Claude is the Arnie desktop app: your workspace and a
Claude terminal side by side, automatic device registration and key rotation, and a
guided setup checklist. Get it at [arnies.ai](https://www.arnies.ai).

## Skills in this plugin

**Core operating skills:**

| Skill | What it teaches Claude |
|---|---|
| `arnie` | The operating model: tables as workflows, the core loop, spend and safety rules |
| `arnie-tables` | Building tables, columns, formulas, conditions, and live data |
| `arnie-workflows` | Planning workflows, reusable functions, schedules, and webhooks |
| `arnie-ingredients` | Saving and reusing your own API integrations (BYOK) |
| `workflow-execution` | Durable table runs, async/polling columns, validation loops |
| `recipe-creation` | Manual workflow structure: seeds, pagination, child tables, dedupe |
| `formulas-conditions` | Formula syntax, condition gating, sentinel handling |
| `cost-approval` | The spend gate: prove tiny, price the run, cap it, ask once |

**GTM motion skills:**

| Skill | What it teaches Claude |
|---|---|
| `prospecting` | The general company-first prospecting motion with qualification |
| `build-tam` | Building a full account universe with count-first sizing |
| `account-orgchart` | Org charts, buying committees, and reporting-line truth |
| `find-qualified-titles` | Finding real role holders from company title rosters |
| `portfolio-prospecting` | Prospecting from funds, accelerators, and member lists |
| `small-business-prospecting` | Local businesses with identity-anchored websites |
| `linkedin-url-lookup` | Resolving LinkedIn profiles with identity validation |
| `niche-signal-discovery` | Mining closed-won/lost accounts for niche ICP signals |
| `personalization-loop` | Converging on truthful outbound copy at scale |

New skills ship with plugin updates — run `/plugin marketplace update arnie` to get the latest.

## Links

- **Product**: [arnies.ai](https://www.arnies.ai)
- **This marketplace**: [github.com/jamakzai12/arnie-marketplace](https://github.com/jamakzai12/arnie-marketplace)
- **Support**: hello@arnies.ai
