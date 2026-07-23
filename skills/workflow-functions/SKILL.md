---
name: workflow-functions
description: Use before reusable row work over remote MCP — saved-function lookup, call binding, installed-column reconciliation, conflicts, roots, and saving. Function-first beats a hand-built column.
---

# Workflow Functions

Reusable row work is function-first. The mandatory shape:

`search -> exact-read -> provider proof -> call -> inspect every installed config -> update mismatches -> sample roots -> widen`

This skill owns the reusable function lifecycle. It does not choose the business motion (that's the motion skill) or prove external provider contracts (see **workflow-execution**'s discovery rules). The durable table run is owned by **workflow-execution**; manual recipe shape/expansion by **recipe-creation**.

## Lookup and call

1. Search `copilot_search_workspace({ mode:"capabilities", provider:"function", query:"get <output> from <available input>" })`.
2. Exact-read the best function id like code.
3. Check the target table, source columns, sample input values, params, credentials, connection, and recent failures.
4. Check whether any returned list needs `copilot_expand_array` (plan the child-row and dedupe shape first — see **recipe-creation**).
5. Complete provider proof before `call_function` when the function wraps a risky external operation; finding a function does not satisfy it.
6. Bind params by meaning to target input column ids.
7. Call `copilot_workflow_workbench(action:"call_function", functionId, params)`.
8. Treat the result as **installation only** — it does not run cells, add rows, copy defaults, or save expansion config.

**Decompose before searching.** Functions are indexed by one generic transformation, not a whole goal. Run one search per transformation and strip every table, industry, campaign, and niche qualifier — search the way you would NAME it (see Naming). "companies with recent funding hiring for a role from my list" is three searches (recent funding, hiring signal, list join), not one.

If exact-read shows a list/array output that must become rows, plan the child-row and dedupe shape before the call. The function still installs columns only; `copilot_expand_array` remains a separate durable action.

## Reconcile before any run

Read `installedColumnIds`, `rootColumnIds`, `resolvedParams`, `autoMappedParamKeys`, warnings, and credential requests from the call result. Then read every installed column together with `copilot_workflow_workbench(action:"get_column_config", columnIds: installedColumnIds)` and reconcile:

| Field | Required check |
|---|---|
| Prompt | Rewrite source-table, industry, company, threshold, and output wording for the current intent |
| Formula/request | Replace hardcoded source ids, URLs, filters, dates, and destinations with current input references |
| Condition | Confirm it gates the correct target value against missing, sentinel, junk, and off-target inputs |
| Declared type | Keep the output parseable or update the type with the prompt |
| Params | Verify every manual and automatic binding by meaning |
| Workspace lookup | Replace source workflow table scopes when they do not belong to the target |
| Cost usage | Use the provider's current total usage field when present; never substitute item count, quota, or an estimate |
| Rate limits | Change only after docs/headers or observed table attempt-rate evidence isolates pacing (see **arnie-ingredients**) |
| Dependencies | Confirm every input exists and carries the current value |
| Dependents | Note which columns need downstream refresh after an update |

Use `copilot_workflow_workbench(action:"update_column")` for every mismatch **before** any rerun or sample. A carried literal destination id can write your rows to the source's campaign or list — sampling an unreconciled function column is as serious as widening an unproven one. Then rerun only `rootColumnIds` on 2-3 rows, refresh dependents only when required, and inspect real values before widening.

## Update, remove, or keep

Use `update_column` when the column identity is still right and only its prompt, formula, condition, type, params, polling, or cost metadata is wrong. Remove and rebuild when the installed column has the wrong provider, tool family, row entity, upstream identity, or output purpose. **Never call the function again to repair installed columns.**

## Inputs, defaults, and params

A function never copies source `inputDefaults` and never creates missing target input columns. When a missing-param error names a source default: create or use a target input column, resolve the current value from workspace state / a provider lookup / the user, and bind that target column to the param. Never reuse the source table's sample or default value.

The params object uses the saved function's param keys; values are target table input column ids. **Bind by meaning, not matching id** — source param `hotel_name` may bind to target `company_name` when the meaning matches. Treat `autoMappedParamKeys` as untrusted until checked by meaning.

## Reinstall and conflicts

- Same function + same params skips identical installed columns.
- Same function + different params installs a second parallel set.
- Re-calling over installed columns already edited creates an install conflict — fix bad installs with update or remove, never by re-calling with new params.
- If expansion was not saved, rerun roots first, then `copilot_expand_array` when child rows are needed.
- If credentials block installation, follow returned credential requests and retry the same call after connection.

## Risk proof and manual fallback

Run a small same-path `copilot_workflow_workbench(action:"external_call")` before installation when exact-read shows recent failures, provider risk, expensive side effects, uncertain params, or stale proof. Clean exact functions can install first, then prove through reconciled root samples. If params require niche source-only columns or a misleading old shape, do not force the call — build and prove the smaller correct route, then update the stale function.

Manual `add_column` is allowed only when no function fits, required params do not fit, exact-read proves the function stale/broken, or same-path proof fails. If a saved ingredient lists `relatedFunctions`, exact-read those function ids before rebuilding from the ingredient. After a manual route proves on real rows, save or update the missing reusable function before moving to the next stage.

## Naming

Name by reusable transformation, not the current table, campaign, industry, or niche.

- Good: `Company name/domain -> decision-maker contact`, `Contact name + domain -> verified email`, `LinkedIn profile -> company domain`.
- Avoid narrow names unless the logic truly requires that niche.

## When to save, roots, and save layers

Save after real row proof when the same route would help another table. Do NOT save: one-off cleanup; table-specific filters/lists/campaigns/views; hidden source-table-only dependencies; `expand_array` sync config. Search by the generic transformation before saving; overwrite a stale function by passing its exact saved id.

A **root** is the reusable output to install later. Saving a root includes its upstream dependency cascade, not downstream dependents.

1. Inspect candidate root configs and dependencies.
2. Save the smallest useful proven component first.
3. Save a broader end-to-end root only after the full cascade proves.
4. Use `copilot_workflow_workbench(action:"save_function", functionDefinition:{ id?, name, description, tags, rootColumnIds })`.

Example save order: `company/domain -> executive contact`; `contact + domain -> verified email`; later, `company/domain -> contact + verified email + LinkedIn`.

**Session save sweep.** Before ending, ask once: anything ELSE proven this session — a route, a normalization, a working param binding — that another table would reuse? Save the reusable ones; skip genuine one-offs deliberately rather than by forgetting.
