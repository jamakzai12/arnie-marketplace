---
name: small-business-prospecting
description: Use for local or SMB prospecting over remote MCP — dentists, plumbers, med spas, agencies, storefronts, service-area businesses; build structured Maps or local-business lists with location and contact fields.
---

# Small-Business Prospecting

Build a structured local-business table for local and service-area businesses. Search pages are query evidence, not the final dataset. Discover local-business and scrape capabilities with `copilot_search_workspace`; prove and price them through **cost-approval**. Own the motion here: define the search controls and business row shape, choose the traversal, normalize/deduplicate/qualify businesses before contact recovery, and preserve website and social identity evidence.

## Make the request visible

Capture the few inputs that change the route or row shape: business category/service; city/region/postcode/country/radius or map bounds; locale/language when relevant; open/active status when requested; rating or review-count threshold when requested; required output fields; target count. Ask only when a missing choice changes the first durable table commit; default ordinary fields and keep moving.

## Control rows, then business rows

**Search-control table** — one row per independent category-location search. Keep `query_id`; category/service; city/region/postcode/country or lat/long/radius/map bounds; provider query text; page index/offset/cursor/area tile when the contract needs it; target count and source locale. Use deterministic page or area values as control inputs. When pagination depends on a previous response, keep the continuation value in a visible next-page column on the same control route — do not copy probe cursors into ad-hoc seed rows.

**Business table** — the locked production source column returns a business array; expose the array, then `copilot_expand_array` into one business row per returned place. Keep provider place id and source query id; business name, primary and extra categories; full address, city/region/postcode/country, coordinates when available; phone, official website, rating, review count, open/closed status; source provider/tool id, source URL, fetch time; qualification state and rejection reason. Set a dedupe key immediately — prefer stable place id; when absent, the strongest safe combination of normalized phone, domain, and full address. Do not deduplicate by business name alone.

## Production source over search snippets

Prefer a structured local-business capability (category + location, map-bounded area, or point + radius) when the final table needs phone, address, rating, website, place identity, and optional contact extraction. Exact-read the live contract before deciding which geography fields, result paths, contact flags, and pagination controls exist. Use a fast Maps-style search only for discovery and query tuning — to see how the category and location are interpreted — not as the production source for a bulk list, and never copy its probe businesses into final rows. Prove one control row and one small page, then lock the exact endpoint and payload after the sample returns correct business rows. Do not prewire multiple source providers; add a fallback only after the locked route fails or validation proves a real gap, with a visible `condition` on the fallback.

## Qualification before contact recovery

Qualify the business itself first — requested category, geography, operating status, minimum rating/reviews, and any website requirement. Keep states separate: qualified and open; closed/inactive; duplicate; wrong category/geography; missing website; provider failure. Run contact recovery only where the qualification state admits it — do not spend contact work on rejected businesses.

## Public contact-email recovery

Do not start a name + domain work-email waterfall unless the row contains a named person. A local business's public contact email is a website/social evidence problem first.

**1. Official website first.** When a normal official homepage exists: keep the canonical website/domain visible; use a scrape-capable ingredient to inspect the homepage and likely contact/about/booking/location page; extract candidate public emails with source URL and page evidence; validate the site belongs to the same business before accepting the email. Do not accept an email from a directory, unrelated franchise location, or old domain merely because the name is similar.

**2. Social profiles only when admitted.** Treat social profiles as optional candidate sources, not mandatory steps. Admit the social route when the row's only website is a social profile, the official site is missing/thin, the row already has a verified social URL/handle, or a small ground-truth sample shows public profile fields are the best source. Discover a Facebook/Instagram profile-contact capability via `copilot_search_workspace`, exact-read it, add one fixed column, and run a tiny qualified sample; add a posts capability only when the profile result has no accepted email and recent post text is a justified source, gated on the prior accepted-email scalar being empty.

**3. Keep social evidence separate.** Add audit fields (`facebook_url`, `instagram_url`, `social_email`, `social_email_source`, `social_source_url`, `social_identity_evidence`, `social_contact_confidence`, `social_fetched_at`). Do not overwrite the canonical email directly — build a final email formula only from accepted scalars after each route's validation. Accept social contact data only when at least two identity anchors match (business name, address/locality, phone, official website/menu/booking link, Maps place identity). Reject a social email when the account belongs to a different branch, similarly named business, former brand, fan page, or unrelated creator.

Example gated flow: `qualified business -> official website email -> social profile email (condition: official email empty + verified social URL) -> social posts email (condition: profile email empty) -> final accepted email`.

## Pagination, coverage, and validation

Prove coverage mechanics before widening: for page/offset keep page state visible and verify no repeated page; for cursor preserve the returned continuation on the route that produced it; for nearby/area search make radius/map bounds/area tiles explicit and expect overlap (deduplicate after expansion). Do not claim complete local coverage from one query or one map viewport; keep the source query on each business row so missing areas and duplicates can be explained. If a broad city query truncates, partition by postcode, neighborhood, category, or explicit map area only after the first route is proven — do not build a giant grid before the provider's page/area behavior is known.

Before widening: confirm returned businesses are inside the requested geography; confirm category precision on several real rows; inspect duplicates across nearby/overlapping areas; prove pagination/map traversal with distinct next results; inspect rows with and without websites; verify open/closed status is not hidden; check current provider cost and returned billing receipt; verify expanded rows preserve place id and source query. Treat as real failures: search snippets presented as a complete list; repeated first page or ignored map bounds; nearby search leaking outside the radius; branches collapsed because dedupe used name alone; social profiles accepted on name match only; social emails overwriting a stronger official email; missing website confused with provider failure; empty provider output hidden inside a `complete` cell. Report the funnel: control queries → raw places → unique businesses → qualified/open → official-site contacts → validated social contacts → no public contact.
