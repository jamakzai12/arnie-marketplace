---
name: ai-columns
description: Use for copilot_workflow_workbench ai_generated columns over remote MCP — extract, normalize, classify, score, summarize, or write from row evidence. AI reasons over evidence already in the table; it does not fetch missing facts.
---

# AI Columns

Use this before `copilot_workflow_workbench` adds or updates an `ai_generated` column. AI columns are for per-row extraction, normalization, reasoning, and writing from evidence already present in the table — the default tool for messy text or semantic work. They do NOT fetch missing facts. Before scaling a chargeable AI column, apply the **cost-approval** gate (AI cells charge per cell).

## Best uses and the formula handoff

Use AI columns for inference, cleanup, scoring, validation of noisy text, summaries, categories, semantic extraction, normalization, match confidence, and copy from evidence already in the row. Good examples: company-name cleanup from a scraped page title, business-description summary from scraped page text, ICP fit from verified signals, title cleanup, candidate match confidence, lead scoring.

Use formulas only when the value is deterministic (known path/delimiter/URL pattern/fixed prefix/exact gate/validated array position). **Absolute handoff rule:** when the source is messy variable text — snippets, search results, bios, descriptions, comments, headlines, rendered markdown/body — use an AI column as the extractor/normalizer first; formulas run after AI produces a clean scalar or bounded JSON. AI is also right for normalizing scraped rendered markdown/HTML/text when the same recipe must support variable site layouts (scrape first, then AI into a bounded schema or row array with source URL and Unknown/null handling — this creates candidate fields, it does not verify external facts).

## Model choice and repair

AI columns run on OpenAI models ONLY — never set another provider (non-OpenAI column writes are rejected). Every new `ai_generated` column must set `model` explicitly from the allowlist:

- **`gpt-4o-mini`** (default): cheap mini tier for normal table-scale work — cleanup, simple labels, short summaries, simple scoring, classification, low-risk extraction. Start here.
- **`gpt-5.4`** (stronger): pick only when the task genuinely needs it — precise generation, personalization, candidate match confidence, long/messy evidence, strict JSON-array normalization, outreach copy where accuracy matters.

Validate `gpt-4o-mini` on a representative sample; switch up only when the sample shows missed evidence, weak reasoning, bad tone, unstable JSON, or low-confidence checks; switch back down once the prompt/evidence are stable and the task is simple. If AI is the right method but output is weak, repair the AI column first (tighten the prompt, add missing evidence columns, lower temperature, or use a stronger model) — do not switch to formula just because the cheap model struggled. Do not set `maxOutputTokens`; use the default budget.

## Boundaries — never invent a fact

Do not use an AI column to create or guess a fact that is not present in the row evidence. AI can normalize or extract a fact visibly present in source-backed evidence; it cannot browse or invent the missing one. Blocked: finding a missing email/phone/URL/profile/employer/salary/headcount/revenue with no source-backed evidence; turning a guessed value into a verified fact; browsing/researching new facts; parsing raw JSON/object/array payloads to find fields; leaving a raw-JSON result as another raw-JSON blob; classifying rows before the needed evidence columns exist.

If the workflow needs a verified fact the row does not contain, continue the real-data chain first (discovery, enrichment, scraping, extraction, deterministic cleanup). If it has JSON/object/array evidence, shape the field first with an enrichment/formula column, then use AI only on the resulting plain scalar. This boundary is about machine JSON/object/array payloads — it does not ban AI from normalizing rendered markdown, HTML, snippets, page titles, or page text after scraping (for repeated records, a short JSON array is allowed only as an intermediate that will be `copilot_expand_array`-ed into rows).

**Identifiers are not identity facts.** A handle, username, profile ID, email, URL, or slug identifies a record but is not automatically a name, employer, or title. Do not jump from one platform identifier to another platform's search query without bridge evidence (real name, company, domain, title, known reused username).

## One output per column; prefer score/tier

If the model wants multiple distinct outputs, split them into multiple columns — do not ask one column for category + score + reason + next step. Good pattern: `lead_fit_score`, then `lead_fit_reason`, then `lead_fit_tier`. Use short JSON only when a downstream step truly needs structured output and the values belong together (or as an intermediate before `copilot_expand_array`).

When the judgment is fuzzy, prefer score/tier over boolean (`1-10`, `High | Medium | Low`, `Strong | Weak | None`) — fuzzy yes/no is brittle and hard to review. Boolean is okay only for strict, narrow binary outputs.

**Rating examples are required.** For ratings, lead scoring, fit tiers, qualification, match confidence, and subjective labels, the prompt MUST include ≥1 clear good/high example and ≥1 clear bad/low example tied to the exact row evidence the score should use. This is part of the prompt contract, not optional polish; add a medium/edge example when the boundary is fuzzy.

**Qualification column for action lists:** when a list will drive action and has 15+ rows, add exactly one qualification/tier column at the END (after all factual enrichment lands), scaled `High | Medium | Low` or `1-10`, its prompt citing the evidence columns it scores from so each row is traceable to real data. If the user asked for a binary filter ("CTOs only"), that's an input filter on the source, NOT a qualification column. Skip qualification for single-row lookups, data-cleanup recipes, small lists (≤15 rows), or when the user asked to keep the list flat.

## Candidate match confidence

Use AI as a limited final checker when the workflow already produced a candidate match and accuracy matters. Good: judge whether `candidate_linkedin_url` matches the row's source identity and the candidate's extracted profile evidence. Bad: ask AI to FIND the missing URL/employer/email. Pattern: real source/search/enrichment creates the candidate → if the candidate URL can be safely read/enriched, fetch it → extract all useful readable evidence → add one AI confidence column → gate final/outreach-ready work on high confidence. A search-result snippet alone is usually not enough for `High`; `High` requires a strong company/domain/employer anchor plus the requested known-person name or role/title/function.

```text
Task: judge whether the candidate profile/contact/company result matches the requested target.
Return ONLY: High | Medium | Low | No match.
Evidence: source anchors {{person_name}}, {{company_name}}, {{company_domain}}, {{title}};
candidate anchors {{candidate_url}}, {{candidate_profile_name}}, {{candidate_headline}}, {{candidate_company}}, {{candidate_domain}}, {{candidate_about}}, {{candidate_snippet}}.
Rules: use only evidence above; do not search; do not invent facts. High requires company/domain/employer
agreement plus requested name or role/title/function. Name-only agreement is Low. Missing candidate evidence lowers confidence.
```

## Prompt contract and patterns

Good prompts reference exact plain/scalar columns, say what to do when evidence is missing, ask for one clean value, state the output format, label inferred outputs, include good/bad examples for subjective ratings, and ban made-up facts. Bad prompts ask AI to find web facts, guess factual values, parse raw JSON/object/array payloads, score from labels alone, produce several unrelated outputs, hide uncertainty, or leave arrays as final displayed values when each item should be a row.

```text
Task: classify / score / normalize / validate / write.
Return ONLY: <one value or one short sentence>.
Evidence: {{column_id}}, {{column_id_2}}
Rules: use only evidence above; do not invent facts; safe output when evidence is weak.
Examples: High/good means <concrete evidence pattern>; Low/bad means <concrete weak or missing evidence pattern>.
```

## Before adding, and validation

Do not add an AI extraction + scoring + outreach stack in one pass. Add or update one AI or support column, run the changed owner column with `copilot_workflow_workbench(action:"rerun_columns")` on a small sample (set `autoExpandDownstream:true` when the result names dependents in `sideEffects.rerun`), then `copilot_read_rows` before adding the next dependent. A scrape or `external_call` probe proves the route only; the committed column still needs its own sampled row values. After adding/updating, read rows and check: the column completed for representative rows; the value is readable and matches the requested output type; missing evidence produced a safe blank/unknown/0/labeled explanation; no generated value is treated as a fetched fact; multi-output requests were split into separate columns.
