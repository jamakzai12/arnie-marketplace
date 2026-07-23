---
name: outcome-tracking
description: Use over remote MCP to declare the Outcome-matching contract — build one final row-level outcome table with a match tag, or tag a campaign's identity so results match back to campaign rows.
---

# Outcome Tracking

Outcome matching works off two facet tags you declare on column config. The tags ride the normal column contract (`add_column` / `update_settings`) — you author the table with ordinary workflow work; the Outcome backend only validates and stores the exact table/column binding, it does not choose among candidates or verify how you built the table.

Two separate jobs:

- **SOURCE BUILD** — the source is already chosen. Use any safe provider workflow to create one final row-level table, then tag exactly one final identity column with `tracking: { role: "match", kind: "<metric>", identityKind: "<identity namespace>" }`. If it should refresh, use the normal table scheduler.
- **CAMPAIGN TAG** — tag the campaign's stable identity input column with `tracking: { role: "identity", kind: "<identity namespace>" }`.

The final source table and the exact tagged column are the whole contract.

## Canonicalize the identity

Canonicalize the final identity values to the same shape used by campaign tags. The matcher only trims, lowercases, and compares values inside the exact same `identityKind` — it does not repair phones, strip email aliases, collapse domains, deduplicate rows, or infer entities. So you must canonicalize phone, domain, provider ids, and other identity values in the tables yourself. Matching is one database join on `identityKind` plus the trimmed, lowercase scalar value; **name alone is never enough.**

## Hard rules

- Never rediscover the seeded source.
- Never match on a person's or company's name alone.
- Never create a second campaign registry or an Outcome-specific schedule — refresh uses the normal table scheduler.
- Never put secrets or full private message bodies in the final table.
- A missing final table or missing exact tag is **broken**, never a trustworthy zero.
- Every final tracker row counts: the backend does not deduplicate, rank, promote, merge, or exclude matches across campaign tables, and there is no separate event time, conversation reference, or cross-identity entity graph.
- Workflow construction details and provider-specific fields are not part of the contract — keep them out of any claim about what the backend guarantees.

Every ready tracker checks every campaign table that carries identity tags, but compares only within the same identity namespace. Missing or moved bindings project "broken" from current table truth — there is no special Outcome teardown or table-move owner.
