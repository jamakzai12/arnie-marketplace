---
name: build-tam
description: Use the Arnie CLI to build or complete a durable TAM, large ICP company list, or account universe — market counts, paginated coverage, qualify companies before contacts.
---

# Build TAM

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Build a durable company universe in Arnie as account rows with a proved coverage funnel — not a headline market-size guess. The table is the runtime. Discover providers with `copilot_search_workspace`; prove and price them through the **cost-approval** gate. Do not paste probe results into production rows.

## Core rules

1. **Companies first, then people.** Build and qualify the company universe before finding contacts. The exception is a user-supplied named company list — that universe already exists.
2. **Known source before reconstruction.** If the requested universe lives in a named directory, registry, portfolio, filing, or public list, extract that source instead of rebuilding it through search providers (use **portfolio-prospecting** for fund/accelerator/association/portfolio universes).
3. **Count first.** Use a dedicated count capability when one exists. Otherwise prove the final filter shape with `limit:1` / `per_page:1` and inspect the returned total before pulling pages.
4. **One fixed production route.** After a tiny same-path proof succeeds, lock one exact ingredient into one source column or recipe step. Do not choose a different provider per row.
5. **Search responses are source data.** Extract firmographics the source already returned (headcount, funding, HQ, category, growth). Do not pay to enrich the same fields again.
6. **Scale through Arnie.** Probe output is evidence only. Production entity rows come from a committed source column, then `copilot_expand_array`, pagination/control rows, formulas, conditions, and linked tables.

## Define the universe

Make the source filters visible before provider choice: geography; industry/category/problem space; employee or revenue range; business model; funding stage/investor/maturity; technologies/hiring signals; exclusions; target count. Split several distinct ICPs into explicit segment/control rows or a visible `segment` column — do not hide several markets inside one opaque search prompt.

Use **Circle and Star**: Circle = the broad pool the source can query (category, geography, size, known directory). Star = the facts/judgments that qualify a company (tool usage, hiring, buying signal, recent change, subjective ICP fit). Make the Circle large enough to contain the Stars, but no larger — search the Circle, then add visible qualification columns for the Stars the provider cannot filter reliably.

## Arnie table shape

Use two stages when a source returns many entities.

**Source/control table** — one row = one admitted segment and one page/cursor state. Keep `segment`, source route and fixed tool id, exact filters/query, `page`/`cursor`/`offset`, page size, source timestamp, raw source result, returned total when available, and next page/cursor. The fixed source column consumes these control values; extract its returned company array into a dedicated array column and `copilot_expand_array` into company rows. Never copy companies from a proof call into `add_rows` or seed CSV.

**Company table** — one row = one company. Keep stable provider company id, normalized name, verified domain, company LinkedIn URL when returned, source provider/tool id, source segment/query/filters/page/timestamp, source evidence, returned firmographics used for qualification, qualification result and reason, and a stable dedupe key (prefer provider id, then normalized domain). Add people only after a qualification condition exists, in a linked child table with the parent company id/domain preserved.

## Provider discovery and production contract

Start with exact Arnie capability discovery via `copilot_search_workspace`. If a connected/default integration fits the required filters, entity shape, coverage, pagination, and price, prefer it. If no exact fit exists, search for a direct ingredient for company rows from the needed filters and exact-read at most two serious results; inspect the input/output schema, credential mode, availability, pagination, advertised unit price, and gotchas. Validate enum-like values with the provider's autocomplete when required — never guess fields or reuse a stale example as the contract.

Run a count or tiny same-path proof, then lock one route. The production source column uses that exact ingredient; only its filters/page/cursor/segment vary per row. Treat `billing.cost_usd` as the actual spend receipt and show it beside the advertised unit price. Missing or invalid pricing means unknown, not free. Apply the **cost-approval** gate before the first paid proof and before scale — a count result is not permission to pull the full market.

Escalate to another provider only when the current route lacks a required filter or has a proved coverage failure; when structured databases return zero for pre-revenue startups, niche verticals, or non-US companies, admit a concept-search or known-source-extraction fallback as a separate segment. Do not fire all providers in parallel.

## Pagination and coverage

First page proves shape, not coverage.

1. Prove the page/cursor fields and returned `next`/total shape.
2. Commit the fixed source column.
3. Materialize page/cursor/offset state as explicit control rows.
4. Expand returned arrays into company rows through one shared downstream path.
5. Stop when the target is reached, the provider reports no next page, or the approved cap is reached.

Do not make separate first-page and tail-page pipelines — they produce the same entity and must converge on the same extraction, identity, dedupe, and qualification columns. Pull more source rows than the final target when downstream attrition is expected (about `1.4 × requested usable rows` is a useful start), but validate the actual funnel before widening further.

## Qualification and people

Search filters define candidates; visible columns prove fit. Put deterministic returned fields into formula/extract columns; use AI normalization for messy descriptions, snippets, job text, or subjective ICP criteria; preserve evidence and return `Unknown`/null when the source cannot prove a criterion. Add a visible condition for qualified companies before any contact-detail column. Hiring is usually a qualification layer, not the only discovery route — build plausible company rows first, then add hiring evidence against known company domains.

Find people only after company qualification. Search with company domains whenever possible, using 1–2 broad function keywords plus seniority (`Growth` + `VP`/`Director`), not strict title lists — for companies under 500 employees narrow titles often return zero, and under 50 employees classic databases may have almost no coverage. For ambiguous company names add domain, batch, or location context; if more than 30% of a sample is unaffiliated, stop that candidate route. Expand person arrays into a linked child table preserving the parent company id/domain, then run contact-detail waterfalls only for qualified people behind visible missing-value conditions.

## Convergence and honest reporting

Filter, do not restart — remove poor matches and supplement a proved gap instead of discarding a working source. A second provider may own a clearly separate admitted segment; it must not silently compete for the same rows. Stop at good enough: if roughly 80% of the target survives qualification and the remaining gap is expensive or low-confidence, ship the proved result and report the gap. Easier extraction, cheaper price, faster response, or cleaner payload is not enough to switch after lock.

After each stage, use `copilot_read_rows` or `copilot_workflow_workbench(action:"query")` to inspect visible values — a successful column write does not prove good data. Report separate counts: provider-reported total (when reliable), rows pulled, rows with valid company identity, deduplicated companies, qualified companies, companies with candidate people, verified contacts. Never present a provider total as a verified TAM — name the source, filters, date, coverage limit, and missing segment.

## Anti-patterns

Broad people search before building the company universe; reconstructing a known directory with repeated queries; pasting proof-call entities into production rows; firing several providers before routing and proof; guessing filter names/enum values/response paths/price; using generic web search as the primary structured list source; re-enriching firmographics the search already returned; contact enrichment before company qualification; hiding pages/cursors inside raw JSON instead of visible control rows; switching the locked provider only because another looks nicer; claiming the headline total without row-level coverage evidence.
