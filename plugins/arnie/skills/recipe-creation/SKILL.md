---
name: recipe-creation
description: Use the Arnie CLI for copilot_create_recipe or complex copilot_workflow_workbench authoring — row shape, seed rows, formula inputs, pagination, arrays, and modify rules.
---

# Recipes — Creation & Modification

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

The short authoring guide for manual recipe structure: shell, input columns/defaults, pagination, arrays, expansion, formula/request shape, insert-time dedupe, or structural repair. Execution lifecycle, path lock, sampling, and validation live in the **workflow-execution** skill; spend gates in **cost-approval**; reusable-function-first row work in **arnie-workflows**.

## Plan first, then shell

Lay out the whole workflow in the conversation (tables, columns, providers, and the spend it implies) and get the user's explicit go-ahead. Only then call `copilot_create_recipe`.

- Preferred clean path: `copilot_create_recipe` as a table shell with name/intent and no initial rows or columns. Then `copilot_workflow_workbench(action:"update_settings")` for input columns and dedupe, then `add_rows` for seed rows, then `add_column`/`call_function` for the work.
- Before creating the shell for any integration/provider/search/list source, decide pagination (`none`, vertical independent pages/offsets, or horizontal cursor/next-url columns) — it changes seed input columns, `inputDefaults`, and child shape.
- Pass `assetIntent` (description + tags) when the goal is clear; later use `copilot_update_workspace({ id, description, tags })` for description/tag changes.

## Cardinality gate

Before `add_column`/`update_column`, decide once-total setup vs row work. Once-total setup is NEVER a column: run it once with `copilot_workflow_workbench(action:"external_call")` using literal inputs (it needs no table rows), then carry the returned id/handle forward with a source input column and persistent `inputDefaults`. Bulk is not a loophole; never batch row data through one `external_call`, even when the provider accepts arrays. If an action consumes table rows, uploads/updates/sends/enriches per row, or creates one output per row, it is row work.

## Row Provenance Rule

Before `copilot_create_recipe`, `add_rows`, or `copilot_expand_array`, name where each row comes from. Allowed production row origins:

- user/developer-approved static rows or existing workflow rows
- stable seed/control rows (query, URL, page, deterministic offset, date window, filter, account/list/source id, category) whose committed columns fetch the final data at runtime
- committed column output expanded with `copilot_expand_array`

Tool/probe/search/scrape/`external_call` output is evidence only — use it for schemas, paths, formulas, source URLs/ids, control rows, and validation; do not copy returned target entities into `add_rows`, seed rows, or final rows. If a probe returns the target entities, commit the runtime source/control column, expose its array, then `copilot_expand_array`. Bridge URLs/IDs are runtime routes, not final rows. A `deduplicationKey` does not make bulk `add_rows` of real data acceptable.

## Shape first

What is one row? The finished row should match the user's requested output shape; seed rows are only a way to reach it. For GTM work, do not leave evidence rows (search results, snippets, scraped blocks, raw URLs) as the final table — promote them into canonical account, contact, signal, or campaign rows once identity is clear. When a seed/control row points to a collection, prove the collection surface (item unit, traversal/multiplier, item identity, available fields, detail path) before creating final item rows.

Classify each requested value before wiring: real-world facts (names, emails, phones, domains, URLs, titles, IDs, revenue, headcount) need a source/API/scrape or user value — never AI/formula invention. AI/formulas may derive labeled estimates (fit score, likely category) from existing evidence, returning Unknown/null on missing evidence.

## Seed rows vs enrichment

Valid starting rows are user-approved CSV/static seeds, existing rows, or rows created through committed `copilot_expand_array` lineage. Provider results, scrape results, and copied rows from another table are not valid `add_rows` seeds.

- Add future seed/input columns with `update_settings.addInputColumns`; set `deduplicationKey` there before repeated inserts. If the table already has rows or visible duplicates, include `dedupeExisting:"actively_dedupe"` in that same settings update instead of creating a new table.
- Use `inputDefaults` for static config/user values in real source input columns. For a saved-ingredient input marked `configOrData:"config"`, create the input column, persist the resolved value as its default, and reference the column in `request(...)`; `sampleValue` is test data, not the recipe default. Defaults persist on the table, backfill existing rows, apply to future `add_rows`, and pass through `copilot_expand_array` children. Explicit `add_rows` values override defaults.
- After `add_rows`, do NOT rerun existing columns — new rows automatically enter the table cascade across the connected lineage, and a rerun double-charges. If you add/update columns after that, rerun only those changed columns.
- Do not use formulas or hidden static values for user/config values (ids, keywords, deterministic pages/offsets, URLs, list/campaign ids, date windows, fixed filters). Put those in source input columns referenced as `{{input_column}}`.

## Pagination

Decide the growth axis before creating the shell:

- **Vertical growth:** independent page/offset/date/URL rows added with `add_rows` (seed values only) — use only when the next page value is independent/deterministic.
- **Horizontal growth:** page columns when page N+1 needs page N's returned cursor/token/next URL. Returned cursors/tokens/next URLs go in extracted continuation columns and next-page formulas — never copied into `add_rows` seeds. The parent is one row per independent seed with columns like `page_1_items`, `page_1_next_cursor`, `page_2_items`; the child is one row per returned item. If page columns return the same item type, expand every page's array into the same child table.

Do not invent or decode future cursors/tokens. First page proves shape, not coverage.

## Arrays and expansion

When a column holds an array of items that should become records, make a visible array column, then `copilot_expand_array` into a child table.

- Precondition: `sourceColumnId` must already hold a real array — read the column and verify each `extractPaths.from` against one real item. Point at the array itself, not a wrapper like `{data:[...]}`; if the list is buried, pass the exact `sourceArrayPath`.
- Omit `targetTableId` to create a new child; pass it to merge into an existing child. Reuse the same child for the same item type; `from:""` means blank/null for that target column, not whole-item extraction.
- After expansion, set the child `deduplicationKey` with `update_settings` using a plain input column already inserted with the child row (raw profile URL, handle, item ID, email, domain, slug). Formula/extracted/AI/enrichment columns and name-alone are not valid dedupe keys.
- After shape proof, expand every ready source array across all relevant parent rows/pages into that child before downstream child work; sample expansion is proof, not completion.

## Formula and object shape

The column formula is a JavaScript expression returning a `request(...)` intent for a saved ingredient. It must use `request({ method, url, path, query, body, headers })`; plain objects like `({ request_body: ... })` are not allowed. For Composio, only `request({ query: { ...actionArgs } })`. `url` is relative; `path` fills `{id}`/`:id`. Never put credential/auth/secret keys in formulas. Use `{{column_id}}` refs (dependencies auto-extract); put nested access outside the placeholder: `{{company_profile}}?.organization?.primary_domain`. Details in the **formulas-conditions** skill.

**Object-first:** do not create one visible column per nested path just because an enrichment returned a rich object. Keep the object whole when one downstream formula/condition/enrichment can read the paths directly. Extract visible scalar columns only for final/user-facing fields, values reused by multiple steps, filter/gate/dedupe/audit values, or arrays to expand. Enrichment/AI columns should normally have at least one upstream dependency — a column with only fixed/static inputs is disconnected and usually fails the cardinality gate.

## Modify existing recipes

Before modifying any column, ALWAYS call `copilot_workflow_workbench(action:"get_column_config")` for it (pass `columnIds` to read several at once), plus a row read. Read it like a code file: `dependencies` = imports, `dependents` = consumers. An edit that changes the column's output shape requires re-running its dependents.

- Same tool, different params -> `update_column`.
- Wrong identity (wrong source type, provider/tool/ingredient, upstream dependency, row entity, or output purpose; or AI/formula used where deterministic shaping or real enrichment was required) -> `remove_column` then `add_column`, and update/rebuild downstream consumers.
- `baseUrl`/`endpointPath`/`httpMethod` change -> new ingredient, not a mutated one.
- Do not call a provider/tool swap "wrong identity" just because another path seems better — the existing path must have lock-break evidence (failed call, blocked access, empty/unusable validation, impossible output, user-approved swap, or a conditional fallback design).

**Install-then-Reconcile (after `call_function`):** installed columns are the source table's snapshot — prompts, conditions, formula/request literals, declared types, and lookup scopes carry over verbatim; only `{{column}}` refs were rewritten. Read every installed column in ONE `get_column_config` call and fix each mismatch against the current intent before any rerun. A carried literal destination id can write your rows to the source's campaign or list.

## Cost facet on paid columns

Any column that calls a provider (enrichment, BYOK ingredient, AI call) declares its `cost` facet when added/modified. Read the provider RESPONSE and pick the first rung that applies: (1) the provider's own TOTAL usage field -> that usage `unit` + a `unitsFormula` reading it; (2) component usage fields only -> `unitsFormula` sums them (never sum a total WITH its components); (3) no units field -> omit `cost`, runtime records one counted, unpriced call. Dollar pricing comes only from explicit human/admin configuration — never state a cost or dollar figure you did not read from a tool result, and never write `unitsEstimate`. Before scaling a paid column across many rows, apply the **cost-approval** gate.
