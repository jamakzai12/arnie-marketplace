---
name: list-quality-scorecard
description: Use over remote MCP to grade a prospect/lead table before outreach — duplicates, missing fields, risky emails/domains, title/ICP fit — and gate the send with concrete fixes.
---

# List Quality Scorecard

A bad list burns sending reputation and wastes enrichment spend. Grade the list first, fix what's cheap to fix, and block the send if the grade is too low. This is a gate, not a step you skip because the table "looks fine". Run it on the live prospect table after list building + enrichment land, before outreach copy or any export to a sending tool.

Use the **workflow-execution** rules before changing columns, **formulas-conditions** for deterministic metric/gate columns, **workbench-query** for rollups, **ai-columns** only for the fuzzy title/ICP judgment, and the **cost-approval** gate before widening a paid AI column.

## Method order (deterministic first, AI last)

Most of the scorecard is countable — do it with `formula`/`condition` columns and `copilot_workflow_workbench(action:"query")`, NOT AI. Reserve one cheap AI column for the only fuzzy metric (ICP fit / bad-title judgment). Sample first: compute the metrics on a read of the rows you already have; do not add columns you don't need. Widen only the one proven metric column at a time.

## The seven metrics

Score each 0–100, then weight into a grade. Compute against the rows actually in the table.

1. **Duplicate rate** — duplicate `email` (exact), then duplicate `domain` + person. Formula / `query` `GROUP BY`. High dupes = scrape artifact.
2. **Missing critical fields** — share of rows missing any of email / first name / company / domain. Deterministic `condition` columns; no AI.
3. **Email-verification coverage** — share with a verified/valid status from the verification enrichment. Unverified sends bounce; bounces wreck deliverability.
4. **Catch-all / risky domain density** — share on catch-all or role/disposable domains (from the verification result, or a formula over the domain).
5. **Bad / generic title patterns** — share with empty title, role inbox (`info@`, `sales@`, `support@`), or off-ICP seniority. Formula for the obvious string patterns; a cheap AI label only for the genuinely fuzzy ones.
6. **Title diversity** — too many identical titles signals a one-filter scrape. `query` `GROUP BY title`; flag if the top title is a large share.
7. **ICP fit** — the only real judgment call. One `ai_generated` **tier** column (`high` / `medium` / `low` / `unknown`), prompt carrying ≥1 positive and ≥1 negative example tied to the client's ICP. Default the cheap OpenAI mini model (`gpt-4o-mini`) — simple per-row scoring, not reasoning; reserve `gpt-5.4` only if the cheap one mislabels the sample. Prefer tier over boolean (see **ai-columns**).

## Roll-up to a grade

Weighted average (deliverability metrics matter most — they protect the domain): verification coverage + catch-all density + duplicate rate → ~60%; missing fields + bad titles + title diversity → ~25%; ICP fit → ~15%. Map to a letter: A ≥ 90, B ≥ 80, C ≥ 70, D ≥ 60, F < 60. Report the grade, the per-metric scores, and a short ordered **fix list** (cheapest, highest-impact first — usually: drop unverified/catch-all rows, dedupe, drop role inboxes, re-pull the thin segment).

## The gate

- **D or F → do not send.** Name the two metrics dragging it down and the fix.
- **C → send only the filtered subset** (verified + non-catch-all + ICP ≥ medium); show the row count that survives.
- **A/B → clear to send.**

Deduping, dropping unverified rows, and dropping catch-alls are non-destructive table filters — apply them as new gate columns or a filtered view, don't delete the source rows. Re-grade after fixes so the user sees the lift.
