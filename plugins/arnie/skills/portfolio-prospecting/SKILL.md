---
name: portfolio-prospecting
description: Use the Arnie CLI to source companies from a named fund, VC, investor, accelerator, association, membership, or public portfolio — the official source defines membership; qualify companies before contacts.
---

# Portfolio Prospecting

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Find companies from the exact named portfolio or directory, then qualify companies, find people, verify contacts, and prepare outreach in Arnie. **The official public source defines membership** — it is not reconstructed through providers. Discover scrape/extract/enrich capabilities with `copilot_search_workspace`; prove and price them through **cost-approval**. Do not paste probe output into production rows.

## Core insight: portfolio data is public

Major funds, accelerators, associations, and membership groups normally publish their company universe — start with that official source (a fund's `/portfolio` or `/our-companies` page, an accelerator's companies directory). If the user names a batch, sector, geography, or status, preserve it in the source URL or visible control columns.

## Non-negotiable rules

1. **Exact source first.** Use the official portfolio/accelerator/association source. Search only to recover that source URL when unknown.
2. **Do not reconstruct membership through providers.** Generic investor filters are not the membership authority. People-first search followed by investor verification wastes most of the work.
3. **Companies first, then people.** Extract, identify, and qualify company rows before people search.
4. **Known-source extraction is one production route.** Once a tiny same-path extraction works, lock it. Do not switch from the official source to a generic search provider because its payload is easier.
5. **Probe results are evidence only.** Production companies come from a committed source/extraction column and `copilot_expand_array`, never copied into seed rows.
6. **Preserve lineage.** Every company and person row retains the source, filter/batch, source timestamp, and parent identity.

Provider enrichment may correct company identity or firmographics; it does not rewrite source membership.

## Arnie table architecture

**Source/control rows** — one row = one official collection surface and one traversal state. Keep portfolio/directory name; official source URL; batch/category/geography/status filter; page/cursor/offset/tab/collection URL; source timestamp; extraction route and fixed tool id when provider-backed; raw source result; source-visible total when available; next page/cursor state. The committed extraction column returns a company array — extract it into one dedicated column and `copilot_expand_array` into company rows. If the source has collections and pages, use collection rows first, then page/cursor rows, then one shared company-expansion path.

**Company rows** — one row = one listed organization. Keep portfolio/directory name; source page and filter/batch; listed name; listed website/domain and description; source evidence proving membership; source timestamp; normalized name; verified domain; provider company id and LinkedIn URL when returned; qualification result/reason/evidence; membership state and enrichment state as separate fields; stable dedupe key. Use distinct states: `listed by source`, `not listed by source`, `source extraction failed`, `identity/domain unresolved`, `enrichment failed`.

**People rows** — a linked child table preserving parent company id/name/domain, source membership evidence, candidate role, affiliation evidence, and the provider that won each contact field.

## Prove the official source

Use the exact URL when given; if only a fund/group name is given, use a precise search to recover the official page, then stop searching and work from the source. For a collection site or JS-rendered directory, use a scrape-capable ingredient (discovered via `copilot_search_workspace`) as reconnaissance to learn collection/batch URLs, repeated company blocks, filters/tabs, next-page/cursor/load-more behavior, detail links, and available domain/description fields. Reconnaissance is a map, not production data — convert the map into source/control rows and the same extraction step that will run in the recipe.

If no exact built-in or installed ingredient fits the source, stop and explain the missing capability — name the provider/API and what it takes to connect or save, and proceed on approval instead of quietly settling for a generic company database that replaces the source.

## Traversal, qualification, and people

First page proves method, not coverage. Materialize batches/categories/sectors/statuses/regions/tabs as control rows when they split the source; prove the page/cursor/load-more pattern on one representative collection; use one shared extraction and company-expansion path for every page; stop on no-next-page, source-visible completion, target completion, or approved cost cap. Do not manually scrape page 2, 3, and a tail page into separate pipelines. When the source displays a total, compare it with extracted raw and deduplicated counts; if not, report the specific collections/pages traversed rather than inventing completeness.

Use source-returned domains directly, then validate them; resolve missing domains only after company rows exist. Add visible company qualification columns before people work (geography, employee range, ICP fit, active/inactive status, hiring for the requested function, growth/funding evidence, exclusions) — formula/extract columns for stable returned fields, AI normalization/qualification for messy descriptions, preserving evidence and allowing `Unknown`/null. Hiring is a qualification layer on the extracted set, not a discovery route.

Search people only at qualified companies, by verified domain when possible, keeping company membership and qualification evidence attached to every candidate. Use broad function keywords plus seniority, not strict titles — under 500 employees strict titles often return zero; under 50 remove the narrow title filter and select the best current-role match. Disambiguate common company names with domain, batch, product, or location; a profile name/title match is not enough — validate current company/domain plus another anchor, and if more than 30% of a sample is unaffiliated, stop and change the route. Expand candidate arrays into the linked people table.

## Contact waterfall and outbound handoff

Contact finding happens after company qualification and person affiliation/current-role validation. Use one provider per waterfall column, gate each later provider with a visible condition that the required value is still missing, record which provider won the final field, and verify email/sendability before outreach. Do not retry or hand-patch companies with naturally missing coverage forever. For very small companies, a broad domain-pattern email finder often fills poorly — discover and prove a small-company-fit capability rather than defaulting to one that returns zero. Every provider-backed waterfall column uses one fixed ingredient; never choose the provider per row. Apply **cost-approval** before scale.

Personalization comes after verified company/person/contact rows — keep evidence columns (why the company belongs to the source, company ICP/hiring signal, person's current role and affiliation, relevant detail, verified contact) and generate copy with a bounded Arnie AI column; pilot on 2–3 rows, read outputs, edit the prompt, lock it, then scale through the same column (see **personalization-loop**). Outbound actions still require their normal approval, idempotency, and channel owner — this skill builds qualified evidence and copy; it does not silently send.

## Path lock, validation, and pitfalls

Lock the source route after one real same-path extraction succeeds; lock each later provider-backed field after its own tiny proof. Switch only after an actual failure that cannot continue (source inaccessible or missing the requested set, tool unavailable/credential-blocked, no usable target shape, proved pagination cannot cover the source, sampled coverage materially wrong, observed cost violates the cap) — not because another provider is easier, faster, cleaner, or looks cheaper after the route works.

After every commit, use `copilot_read_rows` or `copilot_workflow_workbench(action:"query")` — a successful write does not prove correct values. Report the funnel: source-visible total (when available), raw extracted rows, valid identities/domains, deduplicated listed companies, qualified companies, companies with affiliated people, verified contacts, personalization-ready rows. Name every uncovered collection, hidden-source limitation, and provider gap; do not claim complete portfolio coverage from one page.

Common pitfalls: discovering portfolio companies through generic provider search (misses real members); replacing source membership with an investor filter (false positives); pasting proof results into rows (production cannot rerun/paginate/repair); starting with people (most results unaffiliated); strict titles at small startups (zero candidates); treating first page as complete (hidden batches/pages skipped); blending source, identity, and enrichment errors into one status; switching providers after a working proof (duplicate spend, divergent data).
