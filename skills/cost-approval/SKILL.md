---
name: cost-approval
description: Use before scaling any paid, row-scale run over remote MCP — enrichment columns, provider/ingredient calls, or sends. Prove tiny, price the run, cap it, ask once in the conversation. Not for free formula columns or a single read.
---

# Cost Approval

One loop: **prove tiny → price → cap → ask → run.** There is no approval card over MCP — approval is the user's explicit go-ahead in this conversation.

## When this gate applies

A run is **paid** when it charges credits or spends the user's real account:

- an **enrichment column** with a `creditCost` — charges `rows × creditCost` (per cell, not per run).
- a **provider/ingredient call at row-scale** — per-call scrapes/searches/enrichment across rows.
- an **account or send action** (post/reply/DM/message/apply) — priced in account blast-radius, not credits.

Exempt: formula-only columns, conditions, a single one-off read, cached reruns.

## Price posture: cheapest equivalent first

- **Scope before provider — quote the smallest scope that serves the goal.** Fit-filtered rows, minimum results per row (one decision-maker unless the goal needs more). The lean scope is the default line of the quote; the wider scope is a listed upgrade with its own price, never the silent default.
- **Default/Arnie route first.** Prefer the connected/default route for simple work at or below the ~$0.02/cell baseline. When simple work is evidenced above that baseline, compare the exact Arnie/default route against up to two serious candidates before you lock — this compare-before-lock is not permission to switch a route that is already locked or committed.
- **When two providers produce the same output at similar quality, start with the cheaper one.** Step up only when live evidence shows the cheap route cannot deliver the required value or coverage.
- **Price is not a quality signal.** A provider charging ~10x for a similar, good-enough result is a red flag, not a premium tier — commit to it only with evidence of what the premium buys. Say the comparison in the quote when one exists: "X does this at $A/row; Y wants $B/row for similar output."
- When coverage is uncertain, prefer pay-on-success / cheap-probe providers — a `limit:1` probe often returns the total-match count for the price of one call; size with it before spending.

## The gate (strict order)

1. **Prove tiny.** 1–3 rows, or one safe `copilot_workflow_workbench(action:"external_call")` sample. Read the actual output AND the real per-cell cost. Fix and re-prove until clean. Never price a run you haven't proven.
2. **Price the spend.** Show two numbers when they exist: the current advertised unit price from the ingredient's exact-read description, and the actual charged proof cost from `billing.cost_usd`. Use the observed charge to estimate the full run. For platform-credit columns, calculate `chargeable rows × creditCost per chargeable column`, summed across chargeable columns. Count only rows the `condition` actually admits.
3. **Show the approval message** (template below). A missing section = not ready: run nothing paid.
4. **Cap the run.** `maxRowsPerRun` + a self-draining `condition` so a bug can't overspend. The cap is the ceiling; approval is the trigger.
5. **Run once, then narrate:** what ran, what it cost, what's ready.

There is **no tool to read the user's remaining balance** — price the SPEND; never invent a "credits remaining" number.

## The approval message — exact template

```text
Assumptions
- <intent assumption 1>
- <intent assumption 2>

CSV Preview (ASCII)
<paste verbatim ASCII output from the real one-row Arnie pilot>

Credits + Scope + Cap
- Provider: <name>
- Current advertised unit price: <price and basis, or unknown>
- Actual charged pilot cost: <billing.cost_usd total and per returned result/call when measurable, or unknown>
- Estimated credits: <value or range>
- Full-run scope: <rows/items>
- Spend cap: <cap>
- Pilot summary: <one short paragraph>

Approval Question
Approve full run?
```

The preview is real persisted pilot output, never guessed. Include the firing `condition` in scope/pilot summary when one applies. Keep it short and scannable; the four header names are exact. Move to the full run only after the user's explicit confirmation in this conversation.

## Spend rules

- **Adding rows is never a rerun.** `add_rows` on a table with committed columns auto-cascades every committed column across the connected lineage (linked/child tables included) — new rows' cells are already queued and charge exactly once. NEVER follow `add_rows` with `rerun_columns` on that data set: the cascade is already running and a rerun re-charges existing rows — duplicate spend. Price an `add_rows` batch as `new rows × per-row cost`. A rerun is a SEPARATE spend decision with its own justification (config changed for those rows, or `runScope:"cells:failed"`).
- **Stop after the pilot on bad signal.** Low usable coverage or wrong matches → change route before buying the same failure at scale. Don't approve-around a failed proof.
- **One approval = one run.** New scope, raised cap, or a different column needs a fresh gate.
- **Unknown cost degrades, never blocks.** No rate yet / BYOK without a published rate → the Cost line reads "cost unknown — approve anyway?" and the run may proceed on a yes. Show what IS known (row count, native credits).
- **Do not confuse advertised price with charged cost.** The ingredient description records the advertised rate; `billing.cost_usd` records what the proof actually charged. Show both.
- **A stop is a spoken handoff, not a silent halt.** When the gate can't be satisfied, say so and hand back one clear next step. Scheduled/autonomous runs must report the blocker, never vanish.

## Account and send actions

- **Never raise `maxRowsPerRun` on an account-acting column** unless the user explicitly asks. Copy-generation approval is not send approval; access to a sender does not authorize a send.
- Real sends, posts, DMs, and outbound need explicit approval and never auto-scale — a whole-column rerun of an action is a spend AND blast-radius decision.

## Gotchas

- A `condition` that admits more rows than you think = a bigger bill than you quoted. Count admitted rows.
- `creditCost` is per **cell** — 500 rows × 2 credits is 1,000, not 2.
- Re-running re-charges cells that recompute; cached/unchanged cells don't. Say which when you quote.
- "Only a few credits per row" is how a 10,000-row column quietly costs 20,000. Multiply out and show the total.
