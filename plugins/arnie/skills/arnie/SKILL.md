---
name: arnie
description: >-
  Use when working with the Arnie CLI, workspaces, tables, workflows, functions,
  ingredients, or GTM automation. Thin entry that routes to the owning skill.
  For paid/credit runs, always load cost-approval first.
---

# Arnie

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Arnie is a GTM workspace where tables and integrations are how work gets done.
Load only the owner for the immediate decision or tool call:

- **arnie-workflows** is the single main methodology: workflow shape, proof,
  lineage, route lock, approvals, spend boundaries, and execution order.
- **arnie-tables** owns table reads, configs, rows, expansion, queries, and
  dedupe.
- **arnie-ingredients** owns saved provider/tool contracts and evidence-backed
  repairs.
- **sequencing** owns paced existing cells, delayed row actions, outreach steps,
  and sender-account limits through `schedule_column_cells`.

Do not copy the workflow methodology into this router. Keeping it in
**arnie-workflows** prevents the entry skill and execution skill from drifting.

## Paid work (blocking)

Before any paid or cost-unknown CLI run (Deepline, enrichment, prospecting at
scale, credit-charging columns or reruns), **Read and follow `cost-approval`**.
Show the ASCII cost preview and get an explicit user yes. Do not scale first.
