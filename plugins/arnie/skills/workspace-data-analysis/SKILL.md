---
name: workspace-data-analysis
description: Use the Arnie CLI to analyze existing workspace/table data, past campaign results, cross-table performance, or outcome metrics before doing new work — answer from workspace truth first.
---

# Workspace Data Analysis

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Use this to answer from existing workspace truth before doing new work. Core loop: `Map -> Prove -> Move`.

## Map

Do not answer from the first plausible table. Map the question and the workspace graph:

- **business question and metric:** replies, positive replies, booked calls, revenue, sent count, outcome rate, or freshness.
- **likely homes:** tables, columns, saved views, schedules, automations, old plans, and external campaign/list ids.
- **links:** campaign id, offer id, lead id, email, domain, LinkedIn URL, table link, or scheduler.
- **missing pieces:** denominator, date range, reply cache, normalized status, join key, source table, or outcome tracker.

Use SQL workspace metadata as the main map when the question is about existing tables, old results, campaign performance, outcome metrics, or where data lives. `copilot_workflow_workbench(action:"query")` can read `workspace_tables` (id, name, description, tags, row count, status, dates, recipe id, session id), `workspace_columns` (id/name/type/source/tool/formula/prompt/condition/semantic type/hidden/lookup config), and `workspace_links` (linked table relationships). Use `copilot_search_workspace` when you need fuzzy asset search, exact table/column details by id, or provider/action/capability discovery — do not force it before SQL when a broad metadata query is the simpler way to find the right tables.

## Prove

Use `copilot_workflow_workbench(action:"query")` for read-only proof. Use surgical, correct SQL — each query answers a named missing internal fact: candidate tables, useful columns, old outcomes, join keys, date coverage, duplicate risk, or result quality. Query metadata first when the right table/column/join key is unclear. Query old `workspace_rows`/`workspace_cells` only with literal `table_id` filters (see the **workbench-query** skill for the exact SQL shape and limits). When SQL returns app links, use `__APP_URL__/app?tab=tables&tableId=<table id>` in the returned string — never hard-code localhost or a prod host, and use `tableId` exactly.

Omit `show_visible_card` by default. Set `show_visible_card: true` only when the user explicitly asks to inspect/show/look up specific data and the result should appear as a card; never for background checks, validation, diagnostics, or next-step queries.

Check proof quality before answering: right tables and columns; strong join key, not name alone; date range and freshness; coverage and denominator; duplicate rows only when they affect the analysis; normalized statuses or clear raw values; proof rows, not only aggregate counts. If proof is weak, narrow the map, say what is missing, or ask for the missing business definition.

## Move

Choose the smallest honest next move:

- answer in chat when the user only needs understanding.
- use `copilot_workflow_workbench(action:"query", show_visible_card: true)` only when the user explicitly asked to see specific data as a visible result card.
- hand off: **workbench-query** for a saved/reusable view, **workflow-execution** for cached reuse/dedupe, **outcome-tracking** for a result tracker.
- run fresh research or enrichment only after existing workspace truth cannot answer the question.

Query is for proof, not production. A workspace lookup saves old facts onto current rows.

## Common mistakes

Finding one matching table and stopping early; using fuzzy search when SQL metadata can inspect all candidate tables at once; using stale results without checking freshness; ranking campaigns without sent counts or a clear conversion metric; joining people by name alone; ignoring the daily scheduler or fresh reply table; failing to recommend a cache, saved view, outcome tracker, or automation when tracking is missing.
