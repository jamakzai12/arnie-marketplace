---
name: find-qualified-titles
description: Use the Arnie CLI for exact titles and qualified role holders at known companies or company domains — read the real title roster, select matching titles verbatim, then find every current holder.
---

# Find Qualified Titles

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Start from known companies. Read each company's real title roster, select the titles that match the user's persona, then find every current holder of those exact strings. Discover the roster and holder capabilities with `copilot_search_workspace`; prove and price them through **cost-approval**. This owns roster qualification, exact-title semantics, row shape, pagination, and staged contact reveal.

## When this motion fits

Use it for "what titles exist at these companies?", "find qualified titles at these accounts", or a known company list plus a plain-English ICP where guessed title keywords would miss non-standard titles like "Revenue Architect" or "GTM Systems". Do not use it to build the company list — the universe must already exist as user-supplied rows, approved static rows, or committed output from an earlier company-source workflow. Use ordinary broad people search when the user does not care about exact company-specific titles and wants volume across a large market.

## Why exact matching is correct here

Exact title matching is normally brittle because guessed strings miss variants — that problem disappears when the candidate titles come from the company's own roster:

1. The roster capability returns verbatim titles that actually exist at the company.
2. An AI column selects from those strings without rewriting them.
3. The holder lookup receives those exact strings (one exact-title list is an OR across the selected strings).

The important invariant is **provenance**: every selected title must be an unchanged member of the source roster. Never let the model invent a nicer title or normalize the string before the exact holder lookup.

## Inputs

Required per company: verified `company_domain` (or a stable id the roster contract accepts) and a plain-English persona/ICP with include and exclude rules. Useful optional: company name and LinkedIn URL, target function, min/max seniority, locations, explicit exclusions (recruiters, support, interns, plain sales reps), desired channels. A company name alone is weak — resolve and verify a domain before paid person work when the contract is domain-based.

## Arnie table model

**Company parent table** — one row = one known company. Keep name, verified domain, stable provider id; persona/ICP text and include/exclude rules; fixed roster tool id and fixed holder tool id; raw roster result and source timestamp; deterministic `title_roster` array; `matched_titles` array; `title_match_reason`; holder page/cursor, page size, completion state, failure state; returned people array per ready page.

**Linked people table** — `copilot_expand_array` provider-returned holder arrays into one child row per person. Keep source company row/domain; stable person id and verified LinkedIn URL; full name and exact current title; current-company evidence; department/seniority/location when returned; source tool/page/timestamp; matched roster title; role-fit decision and reason; optional email/phone plus validation only after qualification. Set the child dedupe key immediately using stable person id or LinkedIn URL, never name alone.

## Workflow

**1. Prove the company identity.** Validate the domain/id belongs to the intended company. Keep `wrong_company`, `missing_domain`, and `provider_failure` separate.

**2. Fetch the full title roster.** Commit one fixed roster column with a non-empty company-anchor condition. Inspect the real returned object before writing an extraction formula — direct-execution envelopes and persisted Arnie cells are different surfaces, so use a targeted read/query to locate the actual array, then one deterministic formula returning it. Keep the full roster (do not pre-trim); preserve exact spelling, punctuation, seniority markers, and language; return `[]` only when the provider clearly returned no titles; use a separate error/status column for failure — never turn a provider error into an empty roster.

**3. Select matching titles with AI.** One structured AI column consuming the full roster and the persona (semantic matching — no keyword formulas as the main classifier):

```text
Task: select only titles from the exact company roster that match the persona.
Return ONLY a JSON array of exact roster strings, max 100.
Persona: {{persona}}   Include: {{include_rules}}   Exclude: {{exclude_rules}}
Exact roster: {{title_roster}}
Rules: never invent, normalize, shorten, translate, or rewrite a title; every output must
byte-for-byte match one roster item; return [] when none match.
```

Keep `matched_titles` and `title_match_reason` as separate outputs. After the sample completes, validate deterministically that every selected title is a member of `title_roster` — any invented/changed string fails the row and blocks the holder lookup.

**4. Materialize a real array.** The holder provider must receive a real array, not text that looks like JSON. If the AI column returns a structured array, pass it through unchanged; if storage materializes JSON text, add one deterministic parse+validate formula. Gate the holder lookup on a non-empty, valid array — sentinel text like `Unknown` or `[]`-as-a-string does not admit paid work.

**5. Find all exact-title holders.** Commit one fixed holder column using the exact company anchor and `matched_titles`, preserving the provider's expected array/object shape. Path-lock: once the exact-title route works, keep it — do not swap to fuzzy title filters because they look easier, cheaper, or return more people. `No matching title` is a valid business result, not an exact-route failure. Any broad fallback must be a visible, admitted branch with its own status/condition, marked `broad_fallback`, never described as exact-roster coverage.

**6. Paginate without truncation.** A default first page is not "all holders". Keep `page_number`/cursor, page size, returned count, and completion state visible; choose a page size that matches the expected holder volume; expand every ready page into the same linked table with the same mappings and dedupe key. When the provider offers no total, stop on the documented end signal (empty page, `has_more=false`, missing next cursor) — not a guessed page count.

**7. Validate and qualify.** For representative rows verify current company/domain matches the source account, the returned current title exactly matches one selected roster string, role evidence is not stale, all pages land in the same child table, and duplicates are removed by stable identity. Use an AI confidence column only after real source evidence exists.

## Tiered contact reveal

Identity/LinkedIn first; buy only the channels the user needs. Work email only on kept role holders when requested (prefer verified LinkedIn URL or first+last name + verified domain as anchors; validate current company and email domain; keep `valid`/`catch_all`/`invalid`/`unknown` separate). Phone only for priority contacts when explicitly requested, after a verified person identity. Apply **cost-approval** separately for the holder pull, email reveal, and phone reveal — count only rows the visible condition admits.

## Conditions and failure states

```text
roster lookup: verified company anchor exists
AI title match: title_roster is a non-empty array
holder lookup: matched_titles is a non-empty valid subset of title_roster
email reveal: role qualified AND verified person identity exists AND email requested
phone reveal: priority person AND verified identity exists AND phone requested
```

Keep these states different: `No roster title matches persona`, `Roster provider returned no titles`, `Roster provider failed`, `AI changed or invented a title`, `Exact holder lookup returned no people`, `Exact holder provider failed`, `Pagination incomplete`, `Wrong/stale current company`, `Contact reveal not requested`, `Contact enrichment failed`. An empty valid result must not trigger a fuzzy route automatically. Report the funnel honestly: companies → roster success → companies with matching titles → exact holders → qualified current holders → optional verified email/phone.
