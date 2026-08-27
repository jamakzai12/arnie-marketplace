---
name: workbench-query
description: Use the Arnie CLI before copilot_workflow_workbench(action:"query") — read-only SQL over table/workspace snapshots, row/cell status, provider errors, JSON probes, and SQL repair.
---

# Workbench Query

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Use this before `copilot_workflow_workbench(action:"query")` when the hard part is SQL shape, table names, status/error fields, or what the query action can and cannot do.

`query` answers one read-only question. It does not create rows, edit recipes, run providers, or replace durable columns. For a reusable segment the user asks for, use `copilot_workflow_workbench(action:"save_view")` — a saved view, not a repeated ad-hoc query.

## Hard limits

- Only one read-only SELECT is allowed.
- No `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `ALTER`, `COPY`, `PRAGMA`, `SET`, `INSTALL`, `LOAD`, `EXPORT`, `IMPORT`, `VACUUM`, `SELECT INTO`, or locking reads.
- No external file/table functions (`read_csv`, `read_json`, `read_parquet`, `read_text`, `glob`, `load_extension`).
- No schema-qualified tables (`main.table_rows`, `information_schema.tables`).
- No CTE may shadow a snapshot table name such as `table_rows` or `workspace_cells`.
- Omit `show_visible_card` by default. Set `show_visible_card: true` only when the user explicitly asks to inspect/show/look up specific workspace or table data and the result should appear as a card. Never set it for background checks, progress/status checks, validation, diagnostics, internal planning, or next-step queries. If unsure, omit it; do not send `false`.

## Current-table SQL

When the chat has a current table, query these virtual tables. Pass the explicit `tableId` when you know which table you are debugging; if omitted, the workbench uses the current active table — always check the returned `tableId`/`tableName` receipt before treating an empty result as proof that data is empty. Do not add `table_id` filters inside `table_rows`/`table_cells`; those virtual tables are already scoped by the `tableId` argument and do not expose a `table_id` column.

| Table | Use for | Key columns |
| --- | --- | --- |
| `table_rows` | Visible current-table row values and aggregates | `row_id`, `insert_order`, `status`, plus visible column SQL names |
| `table_cells` | Per-cell status, provider errors, skipped cells, JSON/text values | `row_id`, `column_id`, `column_name`, `sql_name`, `status`, `error_code`, `error_message`, `skip_reason`, `retry_count`, `retry_after`, `value_text`, `value_json` |

Each visible column in `table_rows` has a main SQL column (sanitized from the column id/name) plus helper `<name>_text` / `<name>_json` columns (may be suffixed to avoid collisions). **Visible column SQL names must be exact** — do not guess friendly aliases like `domain`; use the real `sql_name` (`company_domain`). If unclear, inspect first:

```sql
SELECT column_id, column_name, sql_name
FROM table_cells
GROUP BY column_id, column_name, sql_name
ORDER BY column_name
```

Use `table_rows` for counts/filters/samples; use `table_cells` when runtime truth matters:

```sql
SELECT row_id, column_name, status, error_code, error_message, skip_reason
FROM table_cells
WHERE status IN ('failed', 'skipped')
ORDER BY row_id, column_name
LIMIT 25
```

In `table_cells.status` and `workspace_cells.status`, `NULL` means no runtime cell status exists — do not treat it as failed or complete. Status SQL is a point-in-time observation: use the returned `observedAt` as the time of truth. `pending`, `claimed`, `polling_wait`, or `retry_scheduled` means "observed running/unfinished then", not "wedged now" — do not diagnose stuck/stale execution from a status query alone; prove that with a fresh execution-health read showing no active run or wake path.

## Workspace SQL

Metadata: `workspace_tables` (id/name, description, tags, row count, status, dates, recipe id, session id), `workspace_columns` (id/name/type/source/tool/formula/prompt/condition/semantic type/hidden/lookup config), `workspace_links` (app table relationships). Old-table rows/cells: `workspace_rows`, `workspace_cells`.

Rules for `workspace_rows` / `workspace_cells`: every alias must have a literal single-quoted `table_id` filter; use `AND` scope, not broad `OR`; no dynamic table ids from another table or expression; keep the scope small (the snapshot rejects too many tables/rows/cells).

```sql
-- Good
SELECT c.row_id, c.value_text FROM workspace_cells c
WHERE c.table_id = 'table_123' AND c.column_id = 'company_domain'
-- Bad: no table_id scope
SELECT row_id FROM workspace_cells WHERE column_id = 'company_domain'
```

## JSON and paths

First read visible rows with `copilot_read_rows({ showRowData: true })` when a small sample is enough. Use `query` for one exact visible-value structure question: path sample, null coverage, array length, first-item keys, or which candidate path contains the requested value. Query the visible `value_json` (or `<name>_json`), then use a formula, an AI-normalization column, or `copilot_expand_array` on that visible value. Do not let query become a second workflow or a replacement for durable columns.

## Repair failed SQL

Read the tool error and repair the same query shape.

| Error shape | Fix |
| --- | --- |
| `table_not_allowed` | Use only `table_rows`, `table_cells`, `workspace_tables`, `workspace_columns`, `workspace_links`, `workspace_rows`, or `workspace_cells`. |
| `workspace_table_id_scope_required` | Add literal `table_id = '...'` filters to every `workspace_rows`/`workspace_cells` alias. |
| `Referenced column "table_id" not found` | Remove `table_id` filters from `table_rows`/`table_cells`; pass the table id as the tool's `tableId` argument. |
| `Referenced column "..." not found` on `table_rows` | Use the exact names in `Candidate bindings`, or inspect `table_cells.sql_name`; do not retry with another guessed alias. |
| `schema_qualified_table_not_allowed` | Remove `main.`, `public.`, or other schema prefixes. |
| `multi_statement_sql_not_allowed` | Keep one SELECT only. |
| empty result | Check the returned table receipt, table id, column id, status/null meaning, and whether you queried metadata vs row/cell values. |

Do not respond to a SQL failure by guessing a broad new workflow. Tighten the query, read one small sample if needed, then continue.
