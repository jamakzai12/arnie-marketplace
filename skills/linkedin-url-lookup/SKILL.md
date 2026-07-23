---
name: linkedin-url-lookup
description: Use to find, resolve, or verify standard LinkedIn profile (/in/) URLs from names with company/title/location anchors over remote MCP — with name-only recovery and strict identity validation against false matches.
---

# LinkedIn URL Lookup

Resolve the right person's standard LinkedIn profile URL. Do not return the first plausible search result. Discover people-search and profile-resolver capabilities with `copilot_search_workspace`; prove and price them through **cost-approval**. Own the motion here: decide which identity anchors are strong enough, preserve candidate and validation evidence in the table, order the lookup and fallback stages, and decide `High | Medium | Low | No match`.

## Classify the input first

| Input | Treatment |
|---|---|
| Full name + verified company domain | Strongest normal lookup input. Keep domain in every search and validation step that accepts it. |
| Full name + company + current title | Strong, but resolve/verify the company domain when ambiguity remains. |
| Work email + employer | Use the email/domain as identity evidence; do not reveal extra contact data unless requested. |
| Name + location/event/role context | Useful for event and RSVP lists, but still validate the returned profile name. |
| Name only | Weak. Add role, event, geography, or company context before paid scale; leave `No match` when ambiguity remains. |
| Existing `linkedin.com/in/...` candidate | Normalize and validate it. Do not re-search unless it fails. |
| Sales Navigator `/sales/lead/` URL | Not a standard profile URL. Resolve identity/domain through a compatible route first. |

If contacts must first be found at target companies, use **find-qualified-titles** or **prospecting**. This skill starts once a known person row or candidate identity exists.

## Keep the evidence visible

One source-person row per person. Preserve inputs when available: `source_first_name`, `source_last_name`, `source_full_name`, `source_company`, `source_domain`, `source_title`, `source_location`, `source_context`. Add visible lookup/decision fields instead of hiding reasoning in one opaque cell: lookup query and exact provider/tool id; raw candidate result or candidate array; candidate URL, normalized `/in/` URL, and rank; scraped profile name, current company, current title, location; `name_match`, `company_domain_match`, `role_or_location_match`; matched anchors, rejection reason, confidence, and final LinkedIn URL. Downstream work reads the final accepted URL, not a raw provider response.

## Arnie-native workflow

1. Read existing people rows and strongest identity fields.
2. Discover the current stage's capability with a focused `copilot_search_workspace` capability query; exact-read the serious candidate and use its `underlyingProvider`.
3. Prove the smallest real risk on one person — `copilot_workflow_workbench(action:"external_call")` for a one-off contract proof, or add the production provider column and run one sample row when row formulas are the risk.
4. Lock that route after same-path proof. Do not prebuild every provider in the ladder.
5. Extract candidate URLs into visible scalar or array fields; if several candidates must be audited independently, expose the candidate array and `copilot_expand_array` into a linked candidate table.
6. Validate the candidate with a profile resolver/scraper on the same sample row.
7. Accept only a validated standard `/in/` URL. Otherwise keep driving the locked route or admit one conditional fallback.
8. Read the real sample row once; fix the first bad column before widening. Apply **cost-approval** before paid scale — count only admitted rows.

Stage discovery: a people-search capability that returns standard LinkedIn URLs directly is the cheapest primary route when the row's anchors fit it; a general search that discovers candidate `/in/` URLs plus a profile-scraper to validate is a broad fallback (search discovery is not identity proof); a semantic search is a weak name-only fallback behind a missing-value condition; a paid structured people/enrichment route is a last resort when a verified domain and title context exist and cheaper validated routes missed. Stop at the first validated match.

## Conditional fallbacks and final value

Do not wire the full ladder at once. Add the next provider only after the current route cannot produce a valid URL, then gate it on the prior accepted scalar being empty. Shape: `primary raw -> primary candidate URL -> profile validation -> primary accepted URL -> conditional fallback raw -> fallback validation -> fallback accepted URL -> final URL`. Every fallback column needs a visible `condition` (the logical equivalent of "prior accepted URL is empty AND this row has the fallback's required anchors"). Build the final value with a formula over already-validated scalars; never coalesce unvalidated raw candidates.

## Mandatory name and identity validation

Validate every looked-up URL — accepting search results without this gate produces a large share of wrong-person matches. Normalize before comparison: remove accents, punctuation, emoji, honorifics, extra whitespace; compare case-insensitively; keep quoted nicknames and common variants (Mike/Michael, Bob/Robert, Bill/William, Liz/Elizabeth, Dan/Daniel, Sara/Sarah).

Gates:
- **Last name:** require exact or safe substring agreement for real hyphenated/compound names. Reject single-character abbreviation matches.
- **First name:** accept exact match, a meaningful 3+ character prefix, a known nickname, or a quoted nickname in the profile.
- **Company/domain:** current company or domain agreement is the strongest supporting anchor. Old employment does not prove current identity.
- **Title/location/context:** supporting evidence, never a replacement for a real name match.
- **Current role:** inspect current active experience; do not trust a provider's top-level title if an old job, board seat, or advisor role may have outranked the real current job.

Set confidence honestly: `High` = first and last name match plus company/domain and another supporting anchor; `Medium` = strong name match plus one credible supporting anchor; `Low` = plausible but insufficient; `No match` = wrong name, no usable `/in/` URL, or unresolved ambiguity. Return null for `Low` and `No match` unless the user explicitly asked for raw candidates.

## Special scenarios

- **Event and RSVP lists:** when only names and event context exist, add likely role-family terms to the query instead of pretending the name is unique. Keep event, role-keyword, and geography scores visible; if several people remain plausible, ask for another anchor before paid work or return `No match`. A high keyword score never bypasses the name gate.
- **Nickname search:** expand common variants in the query, but validate the resulting profile against the same source row.
- **Existing URL verification:** normalize the supplied URL to `https://www.linkedin.com/in/<slug>`, strip query params and trailing slashes, then validate the profile. Do not replace it with a different provider result unless it fails and the user asked for recovery.

## Exit checks

Before widening or declaring completion: inspect real rows for correct `/in/` normalization; verify every accepted URL passed name validation; verify company/domain evidence refers to the current employer when required; inspect several `No match` rows so nulls are honest, not hidden provider failures; confirm every fallback has a condition and every downstream column reads the final URL; prove current price and admitted-row count before paid scale. Report the funnel: source rows → candidates → profile-resolved → validated → no match.
