---
name: arnie-tables
description: Use the Arnie CLI to read, query, sample, expand, reconcile, or reason about Arnie workflow tables — row samples, table SQL, expand_array child tables, column config, and safe reruns.
---

# Arnie Tables

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

A table is a running machine, not a finished sheet. Read its live state before you change it, prove changes on a sample, and keep every derived table connected to its source.

## Reading state

- `copilot_read_rows` — small visible row samples for one user-owned table. Read only the columns the next decision needs. Use it to see real values, not to bulk-export.
- `copilot_workflow_workbench(action:"query")` — read-only SQL over `table_rows` / `table_cells` for filtering, counts, joins, sorting, and aggregates. Query is evidence, not a write path. Pass the explicit `tableId` whenever the table is known. In research mode, `query` is the only workbench action available — read-only analysis, no writes.
- Table status reads are point-in-time. When a query reads `table_cells.status`, treat it as "observed at that moment": pending/claimed cell counts mean "unfinished when observed", not "wedged now". Only diagnose a stuck run after a fresh read proves no active run, no future wake, and runnable cells left unexecuted. Do not sleep in a loop waiting — continue independent ready work, or say background execution is running.

## Shaping structured payloads

Do not use AI to parse JSON. Shape structured fields with formula columns, `read_rows`, or `copilot_expand_array` first; AI reasons over plain scalars, not raw payloads.

**Object-first:** do not explode one structured payload into `first_name`, `last_name`, `company`, `title`, `score` helper columns just because the paths exist. Keep the object whole in its enrichment/formula column when one downstream formula, condition, or enrichment can read it directly. Add a visible extracted column only for final/user-facing values, values reused by multiple downstream steps, filter/gate/dedupe/audit values, or an array that must be expanded.

## Column config is the evidence graph

Before modifying, wiring from, or building on existing columns, call `copilot_workflow_workbench(action:"get_column_config")` — pass `columnIds` to read several in one call — then read rows. A column's prompt, condition, formula, type, and params are the graph you reason over.

**Install-then-Reconcile (mandatory after `call_function`).** Installed columns are the *source* table's snapshot: prompts, conditions, formula/request literals, declared types, and lookup scopes carry over verbatim — only `{{column}}` refs were rewritten. Read every installed column in ONE `get_column_config` call and fix each mismatch against the current intent *before* any rerun. A carried literal destination id can write your rows to the source's campaign or list. Sampling an unreconciled function column is as serious as widening an unproven one.

## Sample before widening

After every `add_column`, `update_column`, `call_function`, expansion, or row append: rerun 2–3 rows of exactly what changed with `copilot_workflow_workbench(action:"rerun_columns")` (pass `rowIds`/`rowIndexes`), read the real values, and fix what is wrong. Do not widen, queue the whole table, or `expand_array` the output while the column is unproven. A failed or empty sample means fix and re-sample, not widen. Once proven, execute all remaining rows in that same step — nothing in the background fills unexecuted cells for you.

Spend responsibly: on a large table, never rerun `cellStatus:"all"` or a whole column by default. Rerun the exact rows or `cellStatus:"failed"` cells the step needs; a whole-column rerun is a spend decision — confirm the tradeoff with the user first if it repeats expensive provider calls at scale.

## Growing per-item tables with expand_array

When a column holds an array of items that should become records, make a visible array column first, then `copilot_expand_array` into a child table.

- Precondition: `sourceColumnId` must already hold a real array — read the column first and verify each `extractPaths.from` against one real item. Point at the array itself, not an enrichment wrapper like `{data:[...]}`.
- Omit `targetTableId` to create a new child; pass it to merge into an existing child. Reuse the same child table for the same item type.
- Set a stable plain-input `deduplicationKey` on the child before calling expansion done, so the standing edge does not duplicate rows on re-sync. `expand_array` is the ONLY way data leaves a table — the child IS the linked/derived table.
- Expand every ready source array across all relevant parent rows/pages into the same matching child table before going downstream in that child. Do not stop at a sample expansion unless the user asked for sample-only proof.

## Setting or repairing a dedupe key

Before repeated `add_rows`, set or repair the table's `deduplicationKey` first with `copilot_workflow_workbench(action:"update_settings")`. If the table already has rows or visible duplicates when you set/repair the key, pass `dedupeExisting:"actively_dedupe"` in the same `update_settings` call instead of creating a new table. Do not delete and rebuild a table to fix duplicates — repair in place.

## Row provenance and insert order

- **Row Provenance Rule:** `add_rows` carries only control/seed state — user-chosen values, static config, approved fixtures. It never carries result data (parsed entities, provider output, scraped items, or anything derived from existing table data). The moment bulk `add_rows` of real data looks like the next step, the structure is wrong: that data must arrive through runtime columns + the `expand_array` cascade. A `deduplicationKey` does not make bulk `add_rows` of real data acceptable.
- Respect table `insert_order`: rows keep the order they were inserted. Row-owner segments must preserve `insert_order`, and ready fast rows should stay ahead of slower provider work — do not reorder or block early rows behind a slow enrichment.
- Native row-enrichment claims are segment-scoped: a claim covers the row segment it named, not the whole table. Keep segment boundaries explicit so two runs never fight over the same rows.

## Repair at the root

When sampled values are junk, repair the producing column or its inputs — never add a downstream filter/cleanup/null-out column to mask bad upstream output. A cleanup column is legitimate only when the upstream value is already correct and just needs reformatting. Do not add duplicate same-purpose columns.
