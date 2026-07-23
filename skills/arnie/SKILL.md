---
name: arnie
description: Use when Claude Code is working with Arnie remote MCP tools, Arnie workspaces, workflows, tables, saved functions, ingredients, or GTM automation through the Arnie Claude pack. Entry point that routes to the deeper arnie-tables, arnie-workflows, and arnie-ingredients skills.
---

# Arnie

Arnie is a GTM workspace where tables and integrations are how work gets done. Your standing brief lives in `~/Arnie/CLAUDE.md`; this skill is the map to the deeper per-area skills.

**Tables are workflows**, not spreadsheets. Rows are the target items (one row = one requested thing). Columns are the installed implementation — each is data, proof, judgment, or a draft action, and each runs once per row while the executor loops. Functions are reusable row-work; manual columns are fallback only.

## First moves

- Start with `copilot_search_workspace` before guessing any tool, table, function, ingredient, or provider capability. Internal discovery comes before web search.
- Values Arnie returns are native JSON: arrays stay arrays, booleans stay booleans, objects stay objects, numbers stay numbers. Never stringify them, and never send placeholder strings like `none`, `n/a`, `unknown`, or `x123`.
- Ask before real send, delete, bulk-spend, or customer-facing actions. Never ask the user to paste secrets — use `copilot_request_credential`.

## The core loop

`Choose -> Prove -> Lock -> Commit -> Observe Once -> Repair or Continue`. Work one saved, testable step at a time; prove the smallest real risk before you widen; observe real row values once (tool success is not proof), then repair bad output at its root or continue.

## Operating rules

Marketplace installs get no `~/Arnie/CLAUDE.md`, so the invariant brain lives here. These outrank convenience — breaking one wastes the user's provider credits, pollutes tables, or breaks trust.

- **Tool-fit proof happens before durable writes.** `copilot_create_recipe`, `copilot_workflow_workbench` (add_column / call_function / external_call / rerun_columns), `copilot_save_ingredient`, and `copilot_update_ingredient` are commits, not discovery. Find and prove the best-fit tool first; if you only know a goal or category, discover the tool before writing.
- **Internal discovery before web search.** Discovery is a contract lookup, not a ritual: use `copilot_search_workspace` before guessing and before any web search. Research the open web only after internal discovery cannot prove the provider, endpoint, auth, parameter, or response contract you need.
- **Plan before you build.** `copilot_create_recipe` runs immediately over MCP — there is no approval card and nothing pauses for sign-off. Before the first `copilot_create_recipe`, lay out the whole workflow (tables, columns, providers, spend) in the conversation and get the user's explicit go-ahead; only then create the recipe.
- **Function-first row work is mandatory.** Before hand-building a column, run one `copilot_search_workspace({ mode:"capabilities", provider:"function", query:"get X from Y" })`. An empty result is the justification to build by hand.
- **Sample before widening.** After every `add_column`, `update_column`, `call_function`, or expansion, rerun 2–3 rows of exactly what changed, read the real values, and fix what is wrong. After an `add_rows` append, do NOT rerun — new rows auto-cascade every committed column across the connected lineage (linked/child tables included), so READ 2–3 of the new-row cells instead; a rerun after `add_rows` double-charges. Never widen, queue the full table, or `expand_array` an unproven column. Once the sample proves it, execute all remaining rows in that same step.
- **Prove risky routes first.** A successful `copilot_workflow_workbench(action:"external_call")` proves the route; a committed column does not. Prove the risky saved path before manual commit or widening.
- **Required value first beats coverage and fallback.** Until one correct-row path produces the first required user-facing value, keep driving that path — do not add fallback providers, alternate query columns, or extra coverage rows before that value exists. Never substitute AI/formula inference for a fact that needs a real source.
- **Branch admission before competing routes.** A branch is a competing/alternate route for the same unresolved value; a normal next step that consumes the current result is not a branch. Prove one admission reason first. No admission proof means continue the current route.
- **Path lock.** Same-path proof or a committed user-facing column locks the route for that value. Do not switch provider/source/tool because another looks easier or cheaper — switching a large recipe can cost the user real money. Switch only after the locked path actually fails and cannot continue, or the user approves.
- **Condition gates for non-empty checks.** Use a `condition` so a dependent column skips rows whose anchor is missing. Every fallback provider or strategy needs a visible `condition` that checks the prior final value is empty, failed, or unusable. No condition means do not add the fallback column.
- **Sentinel text is a missing value.** "Unknown", "No match", "N/A", "None", "Not found" pass a non-empty check because they are text — treat them exactly like empty. A provider call spent on a sentinel input is a wasted credit and a false result.
- **Confidence is labeled, not invented.** For match/scoring judgments use `High | Medium | Low | No match`. Label estimates as estimates, never as verified facts.
- **Contact matching needs company/domain anchors.** A name alone never confirms identity — match on company/domain plus at least one more fact (role, location, linked site). Never deliver a guessed email pattern as the final answer.
- **Install-then-Reconcile.** After `call_function`, read every installed column in ONE `copilot_workflow_workbench(action:"get_column_config", columnIds:[...])` call and fix each prompt / condition / formula / destination-id mismatch before any rerun. A carried literal id can write rows to the source's campaign or list.
- **Row Provenance Rule.** `add_rows` carries only control/seed state — user-chosen values, static config, approved fixtures. It never carries result data; real data enters through runtime columns and the `copilot_expand_array` cascade.
- **Same intent, same workflow lineage.** When the current table can serve the intent, continue it. Data leaves a table only through `copilot_expand_array` (the child IS the linked table). Do not copy current-table rows into a disconnected new recipe.
- **Do not use AI as a JSON parser.** Values Arnie returns are native JSON — shape a structured payload with formula columns, row reads, or `copilot_expand_array` first; AI reasons over plain scalars, not raw payloads.
- **Ask before real send, delete, bulk-spend, or customer-facing actions.** Rows cost real provider credits: never rerun with `runScope:"all"` or whole-column scope by default — rerun the exact rows or failed-only cells the step needs (`rerun_columns` with `rowIds`/`rowIndexes`, or `runScope:"cells:failed"`). A whole-column rerun on a large table is a spend decision — take it only when the column config changed for every row, and confirm the spend tradeoff with the user first when it would repeat expensive provider calls at scale.
- **Never ask the user to paste API keys, tokens, cookies, passwords, or auth headers in chat.** Use `copilot_request_credential`. Ask directly for user-owned blockers (credentials, approvals, quota, destinations); do not route around them with a weaker path.

## Where to go next

- **Reading, querying, or growing tables** (samples, SQL, `copilot_expand_array`, Install-then-Reconcile, insert_order): load **arnie-tables**.
- **Building or running workflows** (`copilot_create_recipe`, `copilot_workflow_workbench`, GTM motions, prospecting, enrichment, confidence checks, approval flow): load **arnie-workflows**.
- **Saving or updating API/tool contracts** (`copilot_save_ingredient`, provider tuning, credentials, rate limits): load **arnie-ingredients**.

Skills are documentation, not a checklist. Load only the one the immediate tool call needs, then act.
