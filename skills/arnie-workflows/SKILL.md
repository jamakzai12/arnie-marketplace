---
name: arnie-workflows
description: Use when Claude Code creates, edits, approves, runs, or scales Arnie workflows and recipes through remote MCP — GTM motions like prospecting, enrichment, and identity triangulation, plus confidence checks, pagination, branch admission, and the approval flow.
---

# Arnie Workflows

A workflow is a connected table machine. Build the machine before collecting results: design the visible table state that can produce, filter, continue, retry, and rerun the data — then let the rows run.

## Build order

1. **Discover.** `copilot_search_workspace` for saved functions, ingredients, connected apps, and current tables. For row-work, search functions first: `copilot_search_workspace({ mode:"capabilities", provider:"function", query:"get <outcome> from <input>" })`, e.g. `get email from domain`. An empty result is the justification to build by hand.
2. **Plan, then shell.** `copilot_create_recipe` runs immediately over MCP — there is no approval card and nothing pauses for sign-off. Before you call it, lay out the whole workflow in the conversation (tables, columns, providers, and the spend it implies) and get the user's explicit approval of that plan. Only then `copilot_create_recipe` with the table name/intent and no initial rows/columns.
3. **Configure.** `copilot_workflow_workbench` actions: `update_settings` (`addInputColumns`, `deduplicationKey`, persistent `inputDefaults`), then `add_rows` for seed rows, then `add_column` / `call_function` for the work.
4. **Prove, then widen.** Preview a risky path with `copilot_workflow_workbench(action:"external_call")`, sample 2–3 rows with `rerun_columns`, read the values, then execute the rest.

**Function-first row work is mandatory** — a matching function beats a hand-built column. Manual `add_column` is fallback only after no function fits, params do not fit, or exact-read proves the saved function stale.

## Control → Filters → Loop → Items → Work

- **Control:** the highest stable source object (account, base, search, category, source URL).
- **Filters:** visible inputs that change the result (query, view, date window, page size, geography, persona).
- **Loop:** visible continuation/wait state (page, offset, cursor, next URL, job id, status, retry token).
- **Items:** returned records — these become child rows through `copilot_expand_array`, not imported provider output.
- **Work:** enrichment, cleanup, qualification, scoring, and draft-action columns on the item rows.

## One row = one requested item

The final user-facing row must match the requested output shape; seed/control rows are allowed only as a route to it. If the best start is a bridge shape, say it plainly: source/seed rows → scrape/search/enrich → expand/extract → requested rows. A first page or one successful probe proves the method only — it is not full coverage. Never loop user items inside one probe or one AI prompt; a column handles one row and the executor loops. Do not invent missing external facts (email, phone, title, company, revenue) with AI or a formula — back them with a real source, or return Unknown/null and continue the real-data chain.

## Pagination

Decide the plan before creating the shell, because it changes seed columns and child shape:
- **Vertical growth:** independent page/offset/date/URL rows added with `add_rows` (seed values only).
- **Horizontal growth:** page columns when page N+1 needs page N's returned cursor/token/next URL. Returned cursors go in extracted continuation columns and next-page formulas — never copied into `add_rows` seeds.

## GTM motions

- **Prospecting (net-new companies/people):** promote evidence rows to real entities only when identity is clear. Account rows = company name, domain, description, fit reason, proof. Contact rows = person, role, company/domain anchor, contact fields, confidence.
- **Enrichment:** one owner column per fact, each backed by a real source/API — never invent salary, email, phone, title, or company with AI or a formula. Gate every dependent column with a `condition` that skips missing-or-sentinel anchors.
- **Identity triangulation / contact matching needs company/domain anchors.** A name alone never confirms identity — match on company/domain plus at least one more fact (role, location, linked site, photo). Never deliver a guessed email pattern as the final answer; verify with an email-finder provider (name + domain in, verified email out).
- **Waterfalls:** an ordered real-data chain. Provider B runs only where A is empty/null/skipped (coverage fill), or adds a different source-backed angle (strategy stack). Do not research fallback providers before the primary path hits an actual failure. End every column-level waterfall in one final scalar formula column (`{{primary}} || {{fallback}} || null`) that downstream steps read.

## Confidence and sentinels

- Label match/scoring judgments as `High | Medium | Low | No match`. Estimates and signals are labeled as estimates, never presented as verified facts.
- **Sentinel text is a missing value.** "Unknown", "No match", "N/A", "None", "Not found" pass a non-empty check because they are text — treat them exactly like empty. A provider call spent on a sentinel input is a wasted credit and a false result.

## Staying on one route

- **Same intent, same workflow lineage.** When the current table serves the intent, continue it. Per-entity rollups (unique companies from contacts, unique channels from videos) are an array/identity column plus `copilot_expand_array` with a child `deduplicationKey` — never a new `create_recipe` fed by rows copied from table data.
- **Branch admission before competing routes.** A new or alternate table/route is not a normal next step. Before opening one, prove one admission reason. **No admission proof means continue the current route.**
- **Path lock.** Once a path is proven or a user-facing column is committed, do not switch provider/source/tool because another looks easier or cheaper. Switch only after the locked path actually fails and cannot continue, or the user approves.

## One-time setup is not a column

A once-total side effect — create a campaign/list/audience/webhook/sheet, register a resource, send one summary — is never an `add_column`. Run it once with `copilot_workflow_workbench(action:"external_call")`, store the returned id in a source input column via `update_settings`/`add_rows`, and reference that input column in later formulas. If an action produces one output per row, it is row work: use a function or one owner column with a `condition` and idempotency. Bulk is not a loophole.

## Clarify early, then execute

Before durable writes, inspect visible context, current table/work-set state, and workspace candidates. If outcome-shaping details are missing (end goal, data type, filters, geography, volume, recency, personas, whether the output needs contactable emails), ask the user directly in this conversation once near the beginning — one message with usually 2–4 questions, never more than 5. There is no selection-card tool on this surface; you are already talking to the user. After durable table work has started, do not stop with "what next?" or provider menus; continue on the clarified goal and reasonable defaults. Ask again only for a user-owned value, credential, approval, destination, spend/coverage tradeoff, or irreducible ambiguity. Do not ask for self-resolvable config (workspace/list/campaign ids) — resolve it from memory, workspace assets, or a provider lookup, and confirm a recalled id still exists with a cheap read before committing rows to it.

## Async work runs in the background

After widening or expansion, never sleep just to wait for results. If the sample already proved the step, widened rows keep running while you continue independent ready work — or stop and say background execution is running. For polling/async providers, model start → status → result as visible columns (job id, status, retry token), not a sleep loop.

After an `add_rows` append, never `rerun_columns` the new rows — `add_rows` already cascades every committed column across the connected lineage; a rerun double-charges. READ 2–3 new-row cells to confirm, then continue.

## Tables scale later, so keep them connected

Every table can be automated after its core path is proven. Schedules re-run exact columns on existing or control rows; inbound webhooks insert event rows. List outputs become new rows only through `copilot_expand_array` lineage. Keep result tracking, saved views, cleanup, schedules, and webhooks with their focused owners.
