---
name: workflow-execution
description: Use the Arnie CLI before recipe/table writes, multi-step table work, widening, validation reads, and async/polling columns — the durable execution lifecycle, path lock, and post-write check.
---

# Workflow Execution

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Use tables to do the work. Loop: **choose one path, prove the smallest real risk, commit, observe once, classify, repair or continue.**

This skill owns every durable table/workflow write, same-path repair, changed-root sample, widening, and validation read. For manual recipe structure (shell, inputs, pagination, arrays, formula shape) see the **recipe-creation** skill; for reusable row work see the function-first rule in **arnie-workflows**; for spend gates see **cost-approval**. Do not repeat those owners' rules here.

## Hard rules

- **One row = one item.** The finished user-facing row matches the requested output shape; seed/control rows are only a route to it.
- **Build the machine before collecting results.** Pattern: `Control -> Filters -> Loop -> Items -> Work`. If it can filter, paginate, loop, retry, or poll, make that state visible as columns before scale. API calls and scrapes are columns that read visible workflow state; returned records become child rows through `copilot_expand_array`, not imported provider output.
- **Row provenance.** Before `copilot_create_recipe`, `add_rows`, or `copilot_expand_array`, probe/search/scrape/`external_call` output is evidence only, not target rows. Production rows come from approved static rows, existing rows, stable control rows that runtime columns consume, or committed column output expanded with `copilot_expand_array`.
- **Cardinality gate.** Before `add_column`/`update_column`, decide once-total setup vs row work. A once-total side effect (create a campaign/list/audience/sheet, register a resource, send one summary) is NEVER a column — run it once with `copilot_workflow_workbench(action:"external_call")` using literal inputs (no table rows), store the returned id in a source input column via `update_settings`/`add_rows`, and reference that column later. Bulk is not a loophole; never batch row data through one `external_call` even when the provider accepts arrays. If an action produces one output per row, it is row work: use a matching function first, else one owner column with a `condition` and idempotency.
- **Discovery and skills are context, not progress.** Never guess action names. Probes and durable writes are not discovery.
- **Recipe writes are real commits, not probes.** Plan the workflow in the conversation and get the user's explicit go-ahead before the first `copilot_create_recipe`.
- **After writes, do one bounded check** with `copilot_read_rows` or `copilot_workflow_workbench(action:"query")`. Reading rows is the truth — "the call returned" is not validation.
- **Validation is not monitoring.** After one real sample proves an upstream path, widen it, add dependents, and queue their cells with `rerun_columns` immediately; `add_column` never auto-runs. Never wait for all rows to finish.
- **A status query is not a live progress feed.** Treat `table_cells.status` counts as observed at that moment. Running/blocking statuses are `pending`, `claimed`, `pending_repair`, `polling_wait`, `retry_scheduled`; terminal are `complete`, `skipped`, `error_permanent`. `NULL` means no runtime cell status — not complete, not pending. Only diagnose stuck execution after a fresh read proves no active run, no future wake, and runnable cells left unexecuted.
- **Fix bad columns at the bad column.** Preserve existing rate/concurrency. Only actual table cells showing provider rate-limit failures allow an ingredient repair; then add or lower pacing only for the failing ingredient before retrying or widening.
- **If one valid path remains, use it; do not ask.**

## Discovery and tool fit

Discovery is a contract lookup, not a ritual. Order: (1) visible context, tool history, current recipe/ingredient contracts; (2) `copilot_search_workspace` for the current action, functions first when reusable row work may exist; (3) only when the workspace cannot prove the contract, ask the user or add a search-capable ingredient column. Saved ingredients and connected inventory are memory, not the provider's full surface — a missing or thin ingredient means "not saved yet", never "not possible": discover the API and save/update the ingredient.

Mandatory same-provider recon: if the user names a provider/API, or the capability is something that provider should plausibly support, do not declare it missing/unsupported until you have done exact capability search, returned-id/schema inspection, adjacent action-family search, and one safe same-provider probe. Impossibility claims need receipts.

Provider config ids (workspace/account/list/campaign ids) are data to resolve, not default questions. Check workspace assets and saved capabilities, then the canonical same-provider list/get-current action; prove one cheap read with `external_call` and use the returned id. Never ask the user to copy an opaque id the provider can resolve.

## Path lock

Provider comparison chooses the route; successful same-path proof or a committed user-facing column then locks it for that current value. A candidate proof never locks the route while provider comparison is unresolved. Do not replace a locked provider/tool/source because another looks cheaper, cleaner, easier, or faster. Switch only when the locked route has an actual failure point and cannot continue, the user approves, or the new branch is an explicit conditional fallback. Proof must match the path you will wire: a saved action/ingredient needs `copilot_workflow_workbench(action:"external_call")` proof for that exact path.

## Required value first

**Required value first beats coverage and fallback.** Until one correct-row path produces the first required user-facing value, keep driving that path — do not add fallback providers, alternate query columns, extra coverage rows, or a second strategy first. Helper values (URLs, IDs, handles, domains, queries) are bridge inputs only. Never substitute AI/formula inference for a fact that needs a real source; return Unknown/null and continue.

## Branch admission

A branch is a competing/alternate route for the same unresolved value; a normal next step that consumes the current result is not a branch. Before creating a branch, prove one admission reason: the locked route failed and cannot continue, the route reached the required value and validation shows a real gap, the branch runs only when the prior final value is missing, or the user explicitly asked for multiple strategies. **No admission proof means continue the current route.**

## Conditional fallbacks

Every fallback provider, query path, or strategy must have a visible `condition` that checks the prior final/scalar value is empty, failed, or unusable. **No condition means do not add the fallback column.**

## Post-write check

One bounded live check, not permission to wait until cells finish. After `copilot_create_recipe`, `copilot_workflow_workbench`, `add_rows`, `rerun_columns`, widening, or `copilot_expand_array`, run at most one targeted `copilot_read_rows` or `query` for the rows/columns the next decision needs. Classify:

- `good`: values present and usable (a real domain/email/listing, not metadata or a job-handle ID), expected arrays have items, no API errors hidden inside cell payloads.
- `repair`: values missing, unusable, failed, skipped, empty, or in the wrong place. Fix the owning bad column, then check again.
- `running`: continue the proven dependent chain or independent setup; do not watch it finish.
- `missing`: a needed column is not in the recipe — add or repair it, then check again.

Sleep is allowed only for non-table async status/result endpoints, tool-requested retry waits, or user-requested monitoring — never for table post-write or widening settlement.

## Async and polling columns

Model async providers as visible columns (job id, status, retry token), not a sleep loop. When a column uses `columnDefinition.polling`, the system re-calls the polling ingredient until `statusField` matches `completeStatus`.

**Polling means:** call a safe read/status endpoint, with the same existing job/run/task ID, until that existing job reaches a terminal state. It is NOT "try the API again until data appears."

Cost-safety gate (non-negotiable — a bad setup can call the same API hundreds of times or create duplicate paid tasks):
- If the endpoint/action starts work (names like `start`, `run`, `create`, `trigger`, `scrape`, `crawl`, `search`, `submit`, `batch`) — do NOT add polling to that column.
- If the response already contains the real requested data — do NOT treat it as polling just because it has a `status` field. A `status` field alone is not proof of polling.
- Allowed polling shape: input is a single run/task/job ID (`runId`, `taskId`, `jobId`); response is mostly status metadata (`id`, `status`, `startedAt`, `progress`). Danger shape: input is start/search inputs (`query`, `url`, `maxItems`); response contains real records.

After setting up a polling chain, if the tool result says to wait, stop — do not run more cells in the same turn. Otherwise follow the normal sample -> verify -> widen flow.

## User-owned blockers

Missing credentials, approvals, paid access, quota, billing, spend limits, destinations, and workspace limits are user-owned blockers. If the strongest or locked path only needs one unblock, ask for it in the conversation instead of silently weakening row shape, source, coverage, or output. Ask outcome-shaping questions once near the beginning (2-4, never more than 5); there is no selection card on this surface — you are already talking to the user. After durable work starts, do not stop with "what next?" or provider menus; continue on the clarified goal and reasonable defaults.
