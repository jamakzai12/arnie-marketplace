---
name: personalization-loop
description: Use the Arnie CLI to tune, lock, and scale truthful per-row outbound copy — enrich first, tune on a tiny sample, turn edits into rules, lock the prompt, then scale behind cost approval.
---

# Personalization Loop

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Use after company/person discovery and research. Personalized copy is not a one-shot prompt sprayed across every row: enrich first, tune on a tiny sample, turn edits into rules, lock the prompt after it converges, then scale. The runtime is Arnie — evidence columns, AI columns, formulas, review rows, locked prompts, cost approval, and paced sending.

This skill owns copy-readiness checks on already-qualified evidence, copy-specific fit bands, fit rationale and missing-evidence notes, subject lines / first lines / body / sequence design, strict structured outputs, copy QA, prompt-tuning and lock rules, and safe scaling after approval. It does not own general company/contact qualification or lead scoring (**prospecting**), company/contact discovery, external research, sending-account access, or send approval.

## Required inputs

Use visible Arnie columns or exact user/workspace context for: person (name, current title/company, role evidence); company (name, domain, description/category, size/stage when relevant); qualification (ICP rules, disqualifiers, required signals); product/offer (value proposition, proof points, limits, approved claims); research (source-backed trigger, pain language, recent event, relevant evidence); copy rules (tone, length, banned phrases, CTA, channel, sequence length). Missing evidence must become `Unknown`, `No match`, `REVIEW`, or a static truthful fallback — not an invented fact.

## Context pack

Before generating, make these explicit in the table/recipe or current prompt. **ICP:** best-fit companies, best-fit personas, required qualifiers, disqualifiers, anti-fit signals. **Product and proof:** problem solved, relevant value propositions, approved proof points, claims that must not be made, offer and CTA. **Copy rules:** channel, voice/tone, max words, banned phrases, allowed personalization fields, allowed CTA types, required output shape. Keep facts separate from assumptions — broad ICPs produce generic copy.

## Evidence before writing

The writing column reads source-backed scalar evidence, never only the raw lead row. Good evidence includes verified role/company, company product/use case, hiring/technology signal, relevant recent post/launch, case-study fit, the person's own words, or a proven company pain/initiative. Richer evidence in produces better copy — if copy stays generic, improve the evidence or angle before changing adjectives. Do not use AI to browse invisibly inside the writing prompt: external research belongs in separate source-backed columns; the writing column only uses those results.

## Output contracts

Use a structured AI column when later filters/workflows depend on qualification:

```json
{ "score": 8, "score_label": "Strong fit: 8/10", "fit_band": "STRONG_FIT",
  "rationale": "Short evidence-based summary.",
  "answers": [{ "question": "string", "answer": "Yes", "confidence": "High", "rationale": "string" }],
  "positives": ["string"], "risks": ["string"], "next_checks": ["string"] }
```

Use `Yes | No | Unknown` for evidence questions and `High | Medium | Low | No match` for confidence. Unknown evidence must not be scored as confirmed. For sequences, default to four steps only when the user did not choose another length, returning `emails[{ step, subject, core_value_prop, email }]` plus a `sequence_rationale`. Keep one generated column when only one output is needed; when multiple fields must be filtered/reused, return structured output and extract only the user-facing scalars.

## Formula versus AI

Use a formula for deterministic output: fixed template with clean scalar fields, approved static fallback, CTA assembly, whitespace/casing cleanup, length/status checks, coalescing approved variants. Use an AI column for qualification judgment, evidence selection, message angle, subject-line variation, copy generation/critique, or structured reasoning over several evidence columns. Avoid mail merge disguised as personalization — if every message has the same structure and only swaps first name/company, it is a template; each personalized message must use a relevant source-backed difference.

## Recommended workflow

1. Confirm discovery and qualification are complete enough.
2. Define ICP, product proof, copy rules, channel, CTA.
3. If the eligible set is not already proven, return to **prospecting** for general qualification/scoring; use a copy-specific readiness band here only when the decision is about message eligibility.
4. Filter to qualified rows before generating copy.
5. Build separate research/evidence columns for any missing message inputs.
6. Generate one sample row.
7. Run the convergence loop.
8. Lock the prompt.
9. Apply **cost-approval** to the eligible row count.
10. Scale the locked AI column.
11. Run copy QA and spot-checks.
12. Send only through the separate approved/paced action path.

## Convergence loop

- **Round 0 — one sample.** Generate one real row; show the exact output with its evidence. Do not batch yet.
- **Rounds 1..N — batches of 10.** Generate ten rows, collect row-level edits. Every repeated edit becomes a prompt rule (drop compliment openers; never say "I noticed"; shorten the proof sentence; one direct CTA; prefer the company trigger over personal trivia). Do not hand-patch rows without improving the rule that created them.
- **Lock** after two consecutive zero-edit rounds (one is not enough). The locked prompt becomes the durable recipe/AI-column instruction — do not change it during the full run unless validation finds a real repeated defect.
- **Scale** in batches after the prompt is locked and spend is approved. Spot-check at least five random rows per variant before any send.

## Hard copy rules

- Never fabricate. Use `Unknown` or `REVIEW` when evidence is missing.
- Keep generated variable fields to three or fewer per variant unless the user requires more. One CTA per message. Default email length 60–90 words unless the user/channel requires another.
- Prefer direct evidence-supported claims over vague hedging. Do not use "leverage", "synergy", "I noticed", "I came across", "hope this finds you well", or "circling back" — extend the banned list from user edits.
- Do not output empty placeholders, "cannot be generated", or "as an AI" — mark the row `skipped` and use approved static fallback copy.
- Favor company triggers and self-authored signals over scraped personal trivia. Every personalized claim must trace to a visible source-backed field.

## Copy QA and edits-into-rules

Before sending, compute visible checks: word count, CTA count, banned-phrase hit, unsupported-claim review, evidence field used, empty/placeholder leakage, duplicate-copy similarity at scale, and a final `READY | REVIEW | SKIPPED`. Do not send rows in `REVIEW` or `SKIPPED`. A user edit is a rule, not a one-off correction — when a row is rewritten, identify what the edit means and update the prompt so later rows inherit it. If edits never converge, the evidence may be too thin, the ICP too broad, the angle wrong, the template over-variabled, or the product proof unclear — fix the owning issue instead of repeatedly regenerating.

## Spend and sending boundary

Regenerating a locked AI column costs again — lock once, then scale once. Before widening, use **cost-approval** with a proven sample, eligible row count, generation cost estimate/cap, send/action cost when relevant, and the exact approved scope. **Copy-generation approval is not send approval.** `READY | REVIEW | SKIPPED` is copy-quality state, never send authorization. The final sending route uses the owning action path, verified account, and the user's explicit conversational send authorization — access to a sender does not authorize a send. Load **sequencing** and call `copilot_workflow_workbench(action:"schedule_column_cells")` on the existing executable send cells; never create a recurring column schedule or approval/review/status column for sending. Before handoff or send, read real rows and confirm only qualified rows received copy, every claim traces to evidence, copy follows the locked prompt, user edits became durable rules, no banned/placeholder leakage remains, one CTA and the requested length are respected, the full run stayed inside the approved cap, and send authorization stays separate and explicit.
