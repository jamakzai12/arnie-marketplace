---
name: account-orgchart
description: Use to map an account or person-centred org neighbourhood over remote MCP — org charts, stakeholder maps, buying committees, decision makers, reporting lines, and multi-threading.
---

# Account Org Chart

Build an account map as Arnie tables: real people, deal-role hypotheses, relationship evidence, and next actions, all visible and inspectable. Discover people/profile/contact providers with `copilot_search_workspace`; prove and price them through **cost-approval**. Arnie owns the tables, linked rows, columns, formulas, AI judgments, conditions, pagination, validation, and path lock.

## Capture the deal context

Accept one reliable anchor: verified company domain or stable company id; company LinkedIn URL + verified identity; verified person LinkedIn URL + current company; or full person name + verified company domain. **A person name alone is not enough** — ask for company context when it cannot be discovered safely.

Capture context that changes who matters (ask at most one or two critical questions if missing): product/service being sold, deal stage, existing CRM contacts/relationships, target function, company size/segment, known champion/blocker/buyer. Right-size the committee: SMB 2–3, mid-market 4–6, enterprise 6–12.

## Pick one mode before provider discovery

- **Company-wide mode** — "map the GTM org", "find the decision makers", a bare company/domain, or a buying-committee request. Source the relevant functions and levels; map a full roster only when the user explicitly wants one.
- **Person-centric mode** — "two up/two down around Jane", "Jane's manager and reports", one anchor profile. Build a tight neighborhood; do not run a company-wide employee search.

Confusing the modes is a major failure: company-wide wants coverage; person-centric wants a small, ranked candidate set.

## The hard truth about reporting lines

Normal people databases, LinkedIn-derived data, and profile scrapers do not expose a reliable `reports_to` field. Better search improves who Arnie finds; it does not verify the edges between them. Real names, titles, profiles, and verified contact fields may be high confidence, but reporting edges are inferred from seniority gap, function/team, location, and tenure overlap — at a large enterprise one candidate edge may be only 10–20% likely. Never render an inferred edge as confirmed; keep multiple tied candidates instead of inventing one manager or report; show evidence and confidence for every edge. Use `Confirmed | Inferred | Unknown` for relationship state — `Confirmed` requires a real source (CRM notes, the user, a company org page, explicit public evidence); title proximity alone is always `Inferred`.

## Arnie table model

**Account parent row** — company name/domain, stable company id, target function and deal context, segment, known contact/champion/blocker, selected mode, selected provider/tool id and source timestamp, pagination/cursor state, requested committee size, and gap flags like `missing_economic_buyer`.

**Linked people rows** — one row = one candidate person. Keep stable person id + verified LinkedIn URL; name, current title/company, function/team, location; current-role evidence, start date, tenure, source timestamp; source provider and source account row; CRM relationship/owner/stage when available; seniority rank and display level; committee role and role evidence; disposition, access level, influence level; champion-potential and priority scores; relationship target/state/confidence/evidence; key concern, how to win, risk if ignored, next action; email/phone only after qualification and explicit channel need.

Provider calls return person arrays inside an account cell — keep the committed array column, then `copilot_expand_array` into the linked people table. Set the child dedupe key immediately using stable person id or verified LinkedIn URL, never name alone.

## Spend-safe source order

Read connected CRM first — merge known contacts into linked people rows, preserve their contact fields, mark the CRM relationship, and do not repurchase that data. Then select ONE current people source through capability discovery and prove it on one account/tiny page. Do not fire every source as a copied "waterfall": the first proven route becomes the production path; add a second source only after visible evidence shows a required function/tier is missing or the locked route fails and cannot continue, with the gap/failure condition on the fallback column and its people deduped into the same child table. Apply **cost-approval** for paid scale. If a provider returns names without profile URLs, use the **linkedin-url-lookup** motion — name-to-profile and profile-to-person hydration are different operations.

## Seniority and tenure

Classify titles in order; first clear match wins. Use AI for messy-title semantic classification and a formula for deterministic scoring on the clean result.

| Rank | Level | Typical patterns |
|---:|---|---|
| 0 | CEO | CEO, chief executive |
| 1 | C-level | CTO, CFO, COO, CMO, CRO, chief + function |
| 2–4 | EVP / SVP / VP | EVP, SVP, VP, area vice president |
| 5–6 | Senior Director / Director | senior director, head of, director |
| 7–8 | Senior Manager / Manager | senior manager, manager |
| 9–11 | Principal / Lead / Senior | principal, staff, lead, senior |
| 12 | IC | everything else |

Apply tenure only when a start date exists: Manager/Director under 6 months → treat influence one level lower; at least 2 years → one level higher; VP/C-level under 3 months → `newly_hired_exec`; IC/Senior at least 5 years → `long_tenured_ic`. Per team, `velocity_ratio = people with tenure < 90 days / people in the team`: ≥0.30 flags high staleness (re-verify before a large send). For display, collapse to `Exec | Director | Manager | IC`.

## Map the buying committee

Titles are starting evidence, not proof. Keep `reporting_relationship` separate from `committee_role`.

| Role | Likely starting signals |
|---|---|
| Economic buyer | CFO, CRO, GM/BU owner, P&L owner, owning-function VP |
| Champion | operator closest to the pain (often RevOps/Ops/Enablement) — behavior matters more than title |
| Coach | friendly former user, partner, CS lead, well-connected IC |
| Technical evaluator | engineering/IT leader, architect, CISO, privacy/data owner |
| Procurement/legal | procurement, vendor management, legal, compliance |
| Blocker/final authority | CISO, compliance, general counsel, vendor management |
| End user / influencer | relevant IC, Staff/Principal/Lead/Architect |
| Executive sponsor | relevant C-suite/SVP |

Use job posts, recent hires/promotions, technographic ownership, public content, CRM notes, and warm-path evidence. Do not label someone a champion because of title or score alone. Create separate AI columns for `committee_role`, `committee_role_reason`, `disposition`, `access_level`, `influence_level`, `key_concern`, `how_to_win`, `risk_if_ignored`, `next_action` — each must cite visible row evidence and return `Unknown` when evidence is weak.

## Multi-threading and scores

Do not recommend "contact everyone". No known contacts → access line (coach/warm path → champion → buyer). Active single-threaded deal → dual track (champion + one evaluator/end user). Stuck deal → power line (sponsor/buyer + blocker/procurement). Small account → thin committee (2–3). Enterprise → layered committee (6–12 sequenced over time). For a normal mid-market account, choose 3–5 immediate people; put others into `monitor` or `ask_champion`.

Use scores as transparent prioritization, not truth, and keep each component visible. Priority score example: Exec `+40`, Director `+25`, Manager `+10`; verified email `+25`, phone `+10`; warm connection `+30`, verified CRM relationship `+50`; new exec under 90 days `+20`; cap/normalize to the table's scale. Champion-potential 0–60 with bands `45–60 High / 30–44 Medium / 15–29 Low / 0–14 Unknown` — a Coach is not automatically a Champion; set champion disposition only from behavioral evidence. Relationship-candidate: exactly one level above `+10`, two levels `+5`, same team `+8`, related team `+3`; below 5 is too weak; highest score never becomes confirmed — show ties and evidence.

## Contact reveal and validation

The org map is useful before email/phone. Reveal channels only after the person is qualified: LinkedIn/identity first, work email only for kept people when requested, phone only for priority people when requested. Validate current company and deliverability before any send-ready state; keep `valid`, `catch_all`, `invalid`, and `unknown` separate. Every paid reveal needs a tiny proof, actual cost receipt, admitted-row count, cap, and **cost-approval** before widening.

Keep the table as the artifact — add views/filters (priority-sorted committee, warm/CRM paths, recent hires, team view with growing-team flags, immediate-vs-monitor, missing roles) rather than a polished external chart that hides why an edge was inferred.

Before widening, verify: the anchor currently belongs to the company; real rows represent the requested function/geography; the route/pagination can support the scope; every inferred edge has evidence and a confidence label; committee roles cite deal evidence and do not overclaim champions; dedupe uses stable identity, not name alone; paid contact fields ran only for qualified rows. Stop when the map is useful — a complete-looking but invented hierarchy is worse than an honest shortlist with unknown edges.
