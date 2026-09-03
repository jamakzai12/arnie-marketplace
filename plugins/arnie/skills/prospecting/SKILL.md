---
name: prospecting
description: Use the Arnie CLI for GTM prospecting — net-new companies/people, qualification, scoring, verification, and enrichment order. Routes to the focused motion skills and owns the general motion.
---

# Prospecting

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Owns the general prospecting motion: company-first vs the rare person-first route, company/person row shape, qualification and scoring order, coverage and completion rules, and when contact enrichment and outreach may begin. It does not own table mechanics (**workflow-execution**), paid-run approval (**cost-approval**), or outreach copy (**personalization-loop**). Discover providers with `copilot_search_workspace`; compare cost before locking (see **cost-approval**).

## Route the motion before work

Pick the focused owner for the specific motion, then work the general order below:

| User goal | Owner skill |
|---|---|
| Account map, org chart, buying committee, or people around one anchor | **account-orgchart** |
| Company TAM or a large ICP account universe | **build-tam** |
| Real role holders from company title rosters | **find-qualified-titles** |
| Resolve or validate LinkedIn profile URLs | **linkedin-url-lookup** |
| Compare Closed Won vs Closed Lost accounts for niche signals | **niche-signal-discovery** |
| Fund, accelerator, association, membership, or portfolio sourcing | **portfolio-prospecting** |
| Local businesses, storefronts, or service-area companies | **small-business-prospecting** |
| Qualification-informed outreach copy or personalization | **personalization-loop** |

## Required prospecting order

Goal: move from an ICP to the best N qualified, complete prospects. One row = one company or person. The table is the loop.

1. Define the requested output, filters, target count, and completion rule.
2. Build or accept the company set.
3. Qualify and keep company rows.
4. Find and qualify people only at kept companies.
5. Reveal and verify only the requested contact details for kept people.
6. Over-provision at the top, then filter to the best N complete rows.
7. Prove provider-backed row work on a tiny real sample.
8. Price, cap, and get approval before paid widening.
9. Widen the proven path, report the funnel, then hand off outreach.

## Hard motion rules

### Company first

For B2B company-first prospecting this order is mandatory: build the company set → score and qualify company rows → keep the qualified companies → find and qualify people only at those companies → reveal or verify email/phone only for the kept people. Do not use a broad people search to discover the company universe. If the user supplied named companies or domains, the company set already exists — keep the company/domain anchor and continue from qualification.

Known companies plus nuanced roles are not person-first. For "AI leadership at Mount Sinai" or "RevOps buyers at these accounts", use **find-qualified-titles**: real title roster → qualify exact roster titles → find exact-title holders. Use person-first only when person fit genuinely stands alone from company fit (creator, community, personal-brand targeting); state the reason before opening the exception.

### Circle and Star

Circle = the broad queryable pool. Star = the specific criteria that need evidence and qualification. Make the Circle large enough to contain the Stars, but no larger. Search the Circle, then add visible qualification columns for the Stars the provider cannot filter reliably.

### Qualify before contact enrichment

Filter to qualified rows before email, phone, decision-maker, or contact-detail enrichment. Contact enrichment starts only after the qualified-lead filter exists — do not spend it on unqualified candidates.

### Over-provision, then filter — never chase missing rows

When the user asks for N complete prospects, source about 1.4×N candidates unless real evidence supports another ratio. Filter to the best N complete rows at the end; drop incomplete rows instead of retrying or hand-patching them. Do not rerun a whole enrichment set to fill a few gaps. Report the funnel: raw → qualified → contacts found → verified. Coverage is usually a property of the company, not effort.

### Web evidence is not list truth

Web/search results require verification — they are candidates, not final leads. Do not deliver web-search lead lists without enrichment/scrape plus AI verification unless the user explicitly asks for raw candidates.

### Contact matching needs company/domain anchors

A name alone never confirms identity — match on company/domain plus at least one more fact (role, location, linked site). Never deliver a guessed email pattern as the final answer; verify with an email-finder capability (name + domain in, verified email out). For match/scoring judgments use `High | Medium | Low | No match`.

### Credit and approval gate

Paid or cost-unknown row work follows this blocking order: prove the exact path on 1–3 real rows → read the real output, behavior, and billing receipt (or honest unknown cost) → count only rows the final condition admits → estimate the full scope and cap → use **cost-approval** for one concise confirmation → run only that approved scope with `showApprovalCard:false`. For TAM sizing, prefer a free count endpoint, or `limit:1` only when the live contract proves the response includes the total. Stop after a pilot with low usable coverage, wrong matches, missing required fields, or high cost per usable row — repair the route before buying the same failure at scale.

## Handoffs

Prospecting builds and qualifies the list. It never treats list completion, copy approval, or paid-run approval as send approval. Durable tables/columns/reruns → **workflow-execution**. Paid widening → **cost-approval**. Outreach copy after qualified evidence → **personalization-loop**.
