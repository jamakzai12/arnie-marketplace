---
name: arnie-ingredients
description: Use when Claude Code works with Arnie saved ingredients, provider configs, API credentials, reusable API/tool contracts, or ingredient-backed workflow columns through remote MCP — saving, updating, rate-limit tuning, and the credential flow.
---

# Arnie Ingredients

An ingredient is a reusable **API/tool contract**. Keep its role lean: **provider tuning plus optional saved config only.** It is not a place for campaign-specific row data, and it is not where endpoint knowledge lives — that belongs in memory notes and the provider's own docs. Save an ingredient by reusable operation, not by the current campaign, table, industry, or niche.

## Discover before you save

`copilot_search_workspace` first, to find existing ingredients, functions, and connected apps. A durable write is not discovery: do not use `copilot_save_ingredient` or `copilot_update_ingredient` to explore. Prove the exact endpoint or action first, then commit.

- `copilot_save_ingredient` — only for a genuinely reusable provider operation that does not already exist.
- `copilot_update_ingredient` — when the same operation exists but its contract changed, or live evidence proved the saved config wrong. Fix stale ingredients in the same turn you discover the drift.
- To replace rather than duplicate, pass the existing id so the save overwrites it.

Prefer a function over a raw ingredient. If a `copilot_search_workspace` capability result lists `relatedFunctions`, exact-read those function ids before rebuilding from the raw ingredient — the function already wires the ingredient into row work. Save ingredients as `ingredientType:"enrichment"` and call them from `copilot_workflow_workbench` columns; source ingredients are disabled, so an API that would seed rows must be recreated as enrichment and consumed by the connected machine.

## Contract discipline

- `inputs[]` must include every documented API request parameter, required and optional; mark optional ones `required:false`. Do not send placeholder values for optional params in recipe formulas.
- Ingredient formulas must use `request({ method, url, path, query, body, headers })`. Plain objects like `({ request_body: ... })` are not allowed. For Composio, only `request({ query: { ...actionArgs } })`. `url` is relative; `path` fills `{id}` / `:id` placeholders.
- Never put credential, auth, or secret keys in formulas — credentials are injected outside formula code.

## Provider tuning: research rate limits before saving

Before saving, know the provider's real allowance. A saved ingredient with the wrong rate-limit/concurrency config produces `RATE_LIMITED` cells that the retry schedule already owns — do not hammer manual reruns or batch workarounds. Fix the cause: set the ingredient's rate limit and concurrency to match the provider's documented allowance with `copilot_update_ingredient`, then let scheduled retries run or rerun once after the config fix. (A null rate-limit config defaults to a conservative rate — set it deliberately.)

## Credentials

- Store credentials with the credential flow: `copilot_request_credential` for missing/expired/wrong credentials, `copilot_set_secure_fields` when a secret field must be stored. Connect a new OAuth app with `copilot_get_connect_url`.
- **Never ask the user to paste API keys, tokens, cookies, passwords, or auth headers in chat.** Let Arnie collect and store them safely.
- On an auth failure, do not guess: check key value, header name, header prefix, trailing whitespace, token scope, and account/project. Study the provider through `copilot_search_workspace`, then current provider docs, before claiming the API cannot do it. A saved operation failing proves only that operation is off — not that the provider lacks the capability.

## Proof order

Discover the saved contract → prove the exact endpoint/action with `copilot_workflow_workbench(action:"external_call")` → commit the column. When manual proof works and no reusable function covered it, save the missing ingredient/function before moving on, so the next chat, run, or table can reuse it.
