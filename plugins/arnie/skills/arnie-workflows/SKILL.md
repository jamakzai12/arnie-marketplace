---
name: arnie-workflows
description: Use for Arnie workflow decisions and execution, including table shape, proof, lineage, spend boundaries, approval boundaries, and routing paced actions to scheduled cells.
---

# Arnie Workflows

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

## Shared Arnie Workflow Methodology

This is the decision layer for every Arnie surface. Tool names may change by
surface, but these decisions do not.

### Workflow model

- **Tables are reusable workflows.** Rows hold the changing items or control
  scope. Columns hold shared work that repeats for rows. Functions are reusable
  row-work; manual columns are fallback only.
- Build the connected machine before collecting production results:
  `Control -> Filters -> Loop -> Items -> Work`.
- Lists become child rows through a visible array plus `expand_array`. Set a
  stable plain-input deduplication key before repeated expansion.
- Work one saved path at a time:
  `Choose -> Prove -> Lock -> Commit -> Observe Once -> Repair or Continue`.

### Decisions that must stay the same

- **Effort matches risk.** Act directly when the request and next action are
  clear, cheap, bounded, and reversible. Investigate first when the route,
  contract, spend, side effect, or scope is materially unclear. Ask only for a
  remaining user-owned choice or boundary.
- **Tool-fit proof happens before durable writes.** Discovery is a contract
  lookup, not a ritual. Exact-read the best fitting reusable capability before
  creating a table, adding manual work, calling a provider, or saving a tool.
- **Sample before widening.** Prove 2-3 real rows of exactly what changed and
  read the values. Repair the producing step before widening. New rows already
  auto-cascade committed columns, so never rerun them after `add_rows`.
- **Prove risky routes first.** Compare serious provider routes before lock.
  Required value first beats coverage and fallback. Prefer the lowest total
  expected cost among routes that meet the requirement; use a visible
  `condition` for every fallback.
- **Function-first row work is mandatory.** Install-then-Reconcile: inspect all
  installed configs and fix prompts, conditions, formulas, mappings, literal
  ids, and cost settings before any sample run.
- **Cardinality decides the shape.** Once-total setup is not a column. Work that
  consumes or produces one result per row is row work. Bulk payload support is
  not a reason to hide row work in one external call.
- **Row Provenance Rule.** `add_rows` carries only approved seeds, control state,
  and static config. Provider, scrape, probe, or existing-table result data
  enters through committed runtime columns and connected `expand_array`
  lineage, never copied result rows.
- **Work-set continuity preserves lineage, not current-row or schema reuse.**
  New scope with the same control shape is another control row. Repeated output
  entities become connected child rows. Do not open a disconnected replacement
  route without branch-admission proof.
- **Path lock.** Successful same-path proof or a committed user-facing column
  then locks it for that current value. No admission proof means continue the
  current route. Do not switch because another path looks easier, cleaner,
  faster, or cheaper after lock.
- **Conditions are part of column config.** Sentinel text is a missing value.
  Values such as Unknown, No match, N/A, None, and Not found must not admit a
  dependent paid or state-changing cell.
- **Facts need sources.** Do not use AI as a JSON parser or invent external
  facts. Contact matching needs company/domain anchors; a name alone never
  confirms identity. Confidence labels are `High | Medium | Low | No match`.
- **Never ask the user to paste secrets.** Use the surface's safe credential or
  connection flow.

### Approval and action boundary

- Financial approval is only for real credits or money. For CLI agents: **Read cost-approval**, show the ASCII cost preview, and get an explicit user yes before paid scale. Free account actions
  are outside the financial cost gate.
- A clear request authorizes the described table automation. Send authorization
  is confirmed in the conversation; it is not a table column.
- Never create an approval, review, send-approval, or status column unless the
  user explicitly asks for a per-row human approval button. Approval inputs are
  opt-in and must never be invented as a normal safety step.
- Direct state-changing work outside the requested table automation still needs
  a clear instruction. For row-scale account actions, prove one safe cell, keep
  an objective eligibility condition and durable completion result, and never
  auto-retry an unknown or failed write.

### Time and pacing boundary

- **Paced actions use scheduled cells, not scheduled columns.** Load the
  `sequencing` owner and use `schedule_column_cells` on the existing executable
  ingredient column.
- Never create a schedule column, approval column, sender/account/campaign/step
  table, or recurring column schedule to imitate per-cell pacing.
- The user chooses the pace. Reuse the exact user-owned ingredient, preserve
  unrelated and stricter proven limits, verify the saved pace, then schedule
  its existing cells in table order.
- Recurring schedules may rerun exact read, refresh, list, cleanup, or tracking
  columns on a clock. They do not own outreach steps, delayed actions,
  sender-account limits, or other paced row actions.

## Marketplace CLI map

- Discover with `copilot_search_workspace`. Exact-read the best result before a
  durable call.
- Start a table shell with `copilot_create_recipe`, then use
  `copilot_workflow_workbench` for settings, seed rows, functions, columns,
  samples, repairs, widening, and validation.
- Use `copilot_expand_array` for connected child rows.
- Use `copilot_save_ingredient` and `copilot_update_ingredient` only after live
  evidence proves a reusable contract or saved contract needs changing.
- For per-cell pacing, load **sequencing** before
  `copilot_workflow_workbench(action:"schedule_column_cells")`.

There is no selection-card tool on this surface. Ask the user directly in the
conversation when a real user-owned answer is still needed. A clear request is
already authorization for its described table automation; do not ask again and
do not turn that authorization into an approval column.

## Marketplace execution notes

- Before `copilot_create_recipe`, decide the table shape, pagination shape, and
  likely paid path. Describe material choices when review is useful; do not add
  a ritual plan approval to a clear, bounded request.
- A first page or successful probe proves only the method. It is not full
  coverage and its output is not seed data.
- Vertical pagination uses independent static page/offset/date/URL control rows.
  Horizontal pagination keeps returned cursor or next-URL state in columns.
- A once-total side effect uses
  `copilot_workflow_workbench(action:"external_call")`. Per-row actions use a
  fitting function or one executable owner column with a condition and
  idempotency.
- After widening or expansion, do not sleep for completion. Run one bounded
  observation, continue independent ready work, or say that background work is
  still running.
