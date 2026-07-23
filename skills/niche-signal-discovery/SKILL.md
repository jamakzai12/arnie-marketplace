---
name: niche-signal-discovery
description: Use for Won-vs-Lost ICP analysis over remote MCP — discover differential website, jobs, tech-stack, maturity, and firmographic signals; compute Laplace-smoothed lift; turn reliable signals into an account score and actionable prospects.
---

# Niche Signal Discovery

Discover which observable account signals separate buyers from non-buyers. Compare Closed Won and Closed Lost companies using source-backed website, job, technology, maturity, and firmographic evidence; calculate Laplace-smoothed lift; turn reliable signals into an account score and actionable prospects. Discover research/scrape/enrichment capabilities with `copilot_search_workspace`; run aggregate analysis with `copilot_workflow_workbench(action:"query")`; price paid evidence collection through **cost-approval**; use **prospecting** for the final sourcing/qualification.

## Input gate

Required: a target company/product to model; Closed Won company rows with a verified domain; Closed Lost company rows with a verified domain; a reliable `won | lost` outcome label; enough context to describe what the target sells, the buyer, and the problem. Recommended sample: at least 20 Won and 10 Lost; continue smaller only with a visible dataset caveat. If fewer than 15 Won accounts exist, user-approved lookalikes may supplement Won but are reported as inferred fit, not real buyers. Before prospect delivery, ask whether the user has a customer/CRM/opportunity/past-outbound list for deduplication. Do not invent missing outcome labels or silently treat all customers as Won — if the input cannot support a real comparison, explain the gap and stop before paid evidence collection.

## Arnie table model

1. **Account cohort table** — one row = one input company. Keep stable account id; name; supplied domain and normalized apex domain; outcome `won | lost`; outcome source/date; cohort source (real outcome or approved lookalike); domain-validation status/evidence; cross-group-conflict status; source lineage. Remove exact duplicates inside each cohort. If the same apex domain appears in both Won and Lost, exclude every conflicting row from differential analysis and keep the conflict reason visible.
2. **Signal catalog table** — one row = one candidate signal. Keep signal id and category; the exact keyword/tool/role/firmographic rule; positive-fit / anti-fit / migration / neutral intent; expected source (website, job, technology, firmographic, news, analyst); buyer/seller ambiguity risk; vertical rationale; active/inactive review state. Generate it from the target and buyer ecosystem — never copy a stale catalog from another vertical.
3. **Evidence table** — one row = one account-signal-source observation. Keep account id, apex domain, outcome; signal id; present/absent state; source type and URL; page title or job title; exact supporting quote or structured value; source timestamp; buyer/seller/migration/anti-fit/ambiguous interpretation; provider/tool id and raw evidence pointer. Keep the account as the parent and `copilot_expand_array` pages/jobs/findings into linked evidence rows rather than one opaque cell.
4. **Differential result table** — one row = one analyzed signal. Keep Won/Lost sample sizes; Won count and rate; Lost count and rate; Laplace-smoothed lift; source-breakdown counts; evidence count; reliability tier; buyer/seller interpretation; proposed score points and reason.
5. **Prospect table** — one row = one actionable company. Keep score, matched signals, evidence, dedupe category, exclusion state, and later contact coverage separate from the analysis cohort.

## Pipeline (in order)

**Phase 0 — understand the target.** Research it first: product category, target buyer personas, key differentiators, problem and desired outcome, example customers, likely competitors/substitutes. The entire signal catalog depends on this context.

**Phase 0.5 — map the buyer ecosystem.** 3–5 competitors/adjacent vendors; 10–15 tools common in the buyer's workflow grouped by function; 10–15 real buyer/user/implementer job-title variations. Keep competitor products in a migration segment; "sells the same product category" is structural anti-fit. Competitor usage and competitor identity are different facts.

**Phase 1 — prepare and validate cohorts.** Normalize and verify domains before enrichment (a domain must belong to the intended company, not a blog platform, target subdomain, or case-study host). Use apex-domain identity as the primary key, handling multi-label suffixes (`co.uk`, `com.au`). Keep valid / duplicate / cross-group-conflict / domain-mismatch / outcome-missing / source-failure states separate. Do not treat missing evidence as signal absence until the account passed the evidence quality gate.

**Phase 1.5 — design vertical-specific signals** from the target research: product/business-model indicators, buyer pain language, niche technology/integration indicators, maturity/security/compliance/analyst indicators, buyer/user/implementation roles, job/growth indicators, structural anti-fit, migration opportunity. Generic terms ("platform", "automation", "integration", common infra like AWS/GitHub/Slack) are context, not differentiators, unless the cohort data proves otherwise.

**Phase 2 — collect multi-source evidence.** Never inspect only the homepage — for each cohort company collect the relevant pages (product, features, integrations, customers, security, pricing, careers, about, docs, API, compliance). Use search only to discover/verify relevant URLs for already-known accounts; use a scrape-capable ingredient for proved public-page extraction; use a structured job/technology capability when its exact contract fits. For every external step: check exact capabilities via `copilot_search_workspace`, prove one fixed tool on a tiny sample, lock that tool into one column, and widen only after quality and cost are understood. Keep raw evidence and extracted scalars separate — deterministic paths/formulas for stable fields, AI only to normalize/interpret messy source-backed content (AI must not invent the fact or count cohort presence). Evidence priority: job listings (active investment/pain) > analyst validation > compliance/security infra > pain language in careers/ops content > niche tech stack > website marketing language (may describe a seller, not a buyer).

**Phase 3 — evidence quality gate.** Confirm cohort counts still match the admitted input; both groups have usable evidence; domain validation passed; website evidence coverage is normally at least 80% or the shortfall is explained; content covers several relevant pages, not one shallow page; job fields were parsed from the actual returned shape; provider failures remain distinct from valid no-data results; high-lift matches were spot-checked for substring and seller/buyer errors. If one cohort has weak coverage, repair or narrow the comparison before calculating lift.

**Phase 3.5 — review the catalog against real data.** Present in <10% of enriched accounts: possibly too narrow. Present in >90%: probably generic. Product-category terms common in Won website pages: check whether those companies are competitors. Expected buyer roles absent from job evidence: revisit the persona/title catalog. Substring match catching unrelated words: replace with a safer rule. Changes must be visible — do not silently rewrite a signal after seeing the result and present it as pre-planned.

**Phase 4 — calculate differential lift** with deterministic aggregation over the Evidence and Account tables (`copilot_workflow_workbench(action:"query")`). One account contributes at most one presence count per signal per cohort.

```text
won_rate_smoothed  = (won_count  + 0.5) / (won_total  + 1)
lost_rate_smoothed = (lost_count + 0.5) / (lost_total + 1)
lift = won_rate_smoothed / lost_rate_smoothed
```

Laplace smoothing avoids infinite ratios when Lost has zero matches. Always show raw counts and ordinary percentages beside lift — a large smoothed ratio does not rescue a tiny sample. Reliability: `n=1` flag as single-company evidence, never Tier 1; `n=2` exploratory; `n>=3` eligible for scoring only with source-backed interpretation; positive candidate: lift at least 1.5 and Won count at least 3; strong positive: lift above 2.0 and Won count at least 3; anti-fit candidate: lift below 0.5 with meaningful Lost evidence. Keep source breakdowns — a signal found mainly in jobs means something different from the same words on product pages.

**Phase 5 — build the report:** dataset sizes/lineage/coverage/caveats; a quick decision dashboard; positive, anti-fit, and migration signals; raw Won/Lost counts, rates, and lift; source breakdown and interpretation; 3–5 live citations for each top signal when available; a reconciled 0–100 account score; clear actions and the prospect output. Every top signal needs cited evidence — fewer than three independent account citations: demote it and label the gap.

**Phase 6 — interpret before scoring.** Every scoring input must be observable before sales engagement. Exclude AE activity artifacts (note counts, recorded champion/decision-maker counts, opportunity-contact-role counts, MEDDPICC fields) — they measure work already done on the deal, not whether the account was a good buyer before outreach. Loss reasons can diagnose a weak top-of-funnel ICP but do not become account-scoring inputs. Build positive-fit, buying-intent, infrastructure-readiness, anti-fit, and migration segments separately; point values must be explained by cohort evidence.

**Phase 7 — ship actionable prospects.** Top 10 actionable prospects are a required output (ten is a ceiling, not a reason to pad — deliver a smaller set and explain why if fewer clear the bar). Use the discovered signals to source/qualify a net-new company universe through **prospecting**. Deduplicate against current customers, active opportunities, earlier losses, past outbound, and prior results; keep categories visible: Net-new, Account-only, Re-engage, Active-open opportunity, Current customer. Exclude active opportunities and customers from net-new outreach; categorize other known accounts for review rather than silently discarding them. Company cards are required; contacts and emails are optional (cost) — always offer contact discovery and run it only with approval, qualifying companies first, then buyer-persona contacts, then validated work emails. Never publish an email whose apex domain does not match the current employer.

## Scoring model shape

A transparent 0–100 model only after the differential analysis is complete: Core Fit 0–40, Buying Intent 0–30, Infrastructure Readiness 0–30. Suggested bands: 60–100 Tier 1 (immediate evidence-led review), 35–59 Tier 2 (trigger/timed outreach), below 35 nurture or skip. These are starting priors, not universal truth — reconcile every point value between the detailed model, summary, and prospect rows.

## Stop conditions and success contract

Stop and fix the owning layer when: target/buyer context is missing; Won/Lost labels are unreliable; domains do not map to the named companies; cross-group conflicts remain; one cohort lacks usable evidence; paid evidence collection lacks approval; the selected provider contract or pricing is unknown; high-lift results come from n=1, substring errors, or seller pages; a score uses post-engagement CRM activity; prospect dedupe cannot be audited.

A completed run produces: validated Won and Lost cohorts; a vertical-specific signal catalog; source-backed account evidence; deterministic Won/Lost counts and Laplace-smoothed lift; interpreted positive/anti-fit/migration signals; an evidence-backed scoring model with caveats; a report with live citations; up to ten deduped actionable company prospects; optional approved contacts and validated work emails. Do not send outreach in this skill — hand copy to **personalization-loop**.
