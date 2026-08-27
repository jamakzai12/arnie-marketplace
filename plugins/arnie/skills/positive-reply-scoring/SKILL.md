---
name: positive-reply-scoring
description: Use the Arnie CLI to classify cold-outreach replies and compare campaigns by positive-reply rate, not raw reply volume.
---

# Positive Reply Scoring

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Reply *count* lies — auto-replies, "unsubscribe", and "not interested" all count as replies. The metric that matters is the **positive-reply rate**: genuine interest divided by real human replies. Classify each reply, then roll up.

Needs a table of replies, one row per reply, with the reply text plus a lead and (ideally) campaign/variant id. If the replies aren't in a table yet, pull them in first via the sending-tool action/recipe or an import — this skill scores what's in the table. Use the **workflow-execution** rules before changing the table, **ai-columns** for the classifier, **cost-approval** before widening the paid AI column, and **workbench-query** after labels exist. If targeting quality is the problem, use **list-quality-scorecard**.

## One classification column

Add a single `ai_generated` **category** column over the reply text. Labels:

- `positive` — genuine interest (wants a call/info, asks a buying question, refers you on)
- `objection` — engaged but pushing back (price, timing, "send more info") — a soft yes worth a reply
- `neutral` — acknowledges, no signal either way
- `negative` — clear no / "not interested"
- `unsubscribe` — opt-out / "remove me" / "stop"
- `auto_reply` — out-of-office, autoresponder, ticket bot
- `bounce` — mailer-daemon / delivery failure

Rules (see **ai-columns** for the exact column mechanics):
- One output per column — classification only. A one-line reason or suggested next action are **separate** columns, not crammed in.
- Put ≥1 positive and ≥1 negative **example reply** in the prompt so the boundary between `positive`, `objection`, and `neutral` is anchored — that boundary is where classifiers drift.
- Default the cheap OpenAI mini model (`gpt-4o-mini`). Reply classification is short-text labeling, not reasoning — a stronger model here is wasted spend; escalate to `gpt-5.4` only if the cheap model misreads the sample.
- Score the sample with `copilot_read_rows` first; correct the prompt until the labels match human judgment on ~10–20 replies, then run the rest.

## The north-star rollup

`copilot_workflow_workbench(action:"query")` over the classified column:

```
positive_reply_rate = positive / (total - auto_reply - bounce)
```

Exclude `auto_reply` and `bounce` from the denominator — they aren't human replies. Report, per campaign and per variant when the ids exist: positive-reply rate (the headline); positive + objection counts (the real pipeline); unsubscribe rate (a deliverability/targeting warning if it climbs).

## Reading it

- Compare **variants/lists by positive-reply rate**, never by raw reply count — one variant can get more replies and fewer real opportunities. For a disciplined single-variable comparison, change one thing at a time (list **or** copy, not both).
- A spike in `negative` + `unsubscribe` usually means a targeting (list) problem, not a copy problem — use **list-quality-scorecard**.
- `positive` and `objection` rows are the handoff list: surface them as the follow-up segment, freshest first.
