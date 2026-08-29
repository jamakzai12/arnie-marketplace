---
name: sequencing
description: "Use to spread existing ingredient cells over time from the ingredient's durable capacity. Not for cron, webhooks, polling, provider setup, or a campaign engine."
---

# Cell Scheduling

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Use the table as the sequence. Schedule existing executable ingredient cells;
do not create provider, account, campaign, or step tables.

## Before Scheduling

For an external or customer-facing action, get the user's clear confirmation
before scheduling the cells.

Read the exact executable column and its saved ingredient. Use an existing time
capacity when present. An explicit user-chosen sequence pace is authorized
automation input; it is not proof of a provider rate limit. If the user did not
give or approve a clear pace, ask instead of inventing one. Do not use Redis as
weekly capacity truth.

The exact saved-ingredient read must show its exact ingredient ID, complete `rateLimitConfig`
including `requestsPerWeek`, and `usedInRecipes` so shared
impact is visible. If shared usage is unavailable, retry the exact read and do
not change pacing until it succeeds. It must also show `isGlobal: false`.
Never change a global ingredient for user-chosen pacing: its cross-user impact
is not visible here. Stop and explain that the sequence needs a user-owned
ingredient; do not clone or replace the global ingredient automatically.
Then follow this order:

1. If the saved time-based pace already matches, keep it.
2. If it is missing or different, explain which recipes use the shared
   ingredient. Reuse the same ingredient; do not make a pacing-only duplicate.
3. Preserve every unrelated `rateLimitConfig` value. Never loosen a stricter
   limit that came from a real provider failure.
4. Call `copilot_update_ingredient` with that exact `ingredientId` and the full
   preserved `rateLimitConfig` plus the approved time-based pace first; this
   field is a full replacement. Do not target this edit by name.
5. Exact-read the ingredient again. Verify the saved pace before calling
   `copilot_workflow_workbench` with `schedule_column_cells`.
6. Validate a small sample and confirm the cells have different times.

Provider-safety tuning remains evidence-gated: only actual table-cell provider
rate-limit failures allow a guessed repair, and that repair may only add or
lower the failing ingredient's limits. The user-pacing exception applies only
to the explicit pace needed for the requested sequence.

## Schedule Cells

Use the existing `copilot_workflow_workbench` action:

```json
{
  "action": "schedule_column_cells",
  "recipeId": "...",
  "columnId": "send_message",
  "rowIds": ["..."],
  "startAt": "2026-08-03T09:00:00.000Z"
}
```

Omit `rowIds` and `rowIndexes` only when the user clearly means every row.

The tool spreads the cells in table order. For `50/week`, 100 cells take about
two weeks. It does not put them all at the same time.

## Timing Boundary

For a new sequence, use `startAt` plus the approved ingredient pace in
`schedule_column_cells`. Do not create or change a per-column `schedule` object,
do not make a formula read the clock, and do not add a schedule/time column to
drive execution. The cell-scheduling action owns the durable not-before write
and applies ingredient capacity. Keep the normal condition visible so a reply,
failed/missing prior action, or cancellation can stop a later action cell.

## Runtime Meaning

Postgres stores the cell times and the shared ingredient weekly cursor. BullMQ
only wakes the table. When a cell becomes due, the runtime checks again:

1. is its normal condition still true?
2. is its optional row date or upstream-completion delay due?
3. does the ingredient have durable capacity now?

If capacity is busy, the cell gets a later time. It does not count as a failed
provider attempt.

## Validate

On a small sample, check that:

- selected cells have different scheduled times
- future cells do not run early
- due cells recheck their conditions
- skipped cells do not spend capacity
- the existing ingredient runs without a replacement scheduler

Do not create one BullMQ job per cell and do not babysit the table by polling.
