---
name: arnie-ingredients
description: Use when Claude Code works with Arnie saved ingredients, provider configs, API credentials, reusable API/tool contracts, or ingredient-backed workflow columns through remote MCP — saving, updating, rate-limit tuning, and the credential flow.
---

# Arnie Ingredients

An ingredient is a reusable **API/tool contract**. Keep its role lean: **provider tuning plus optional saved config only.** It is not a place for campaign-specific row data, and it is not where endpoint knowledge lives — that belongs in memory notes and the provider's own docs. Save an ingredient by reusable operation, not by the current campaign, table, industry, or niche.

## Discover before you save

`copilot_search_workspace` first, to find existing ingredients, functions, and connected apps. A durable write is not discovery: do not use `copilot_save_ingredient` or `copilot_update_ingredient` to explore. Prove the exact endpoint or action first, then commit.

- `copilot_save_ingredient` — only for a genuinely reusable provider operation that does not already exist.
- `copilot_update_ingredient` — when the same operation exists but its contract changed, or live evidence proved the saved config wrong. It can affect every recipe in `usedInRecipes`, so fix stale ingredients in the same turn you discover the drift.
- To replace rather than duplicate, pass the existing id so the save overwrites it.

**One operation = one ingredient.** For custom HTTP, the same base URL + endpoint path + HTTP method is usually the same ingredient; for Composio, the same app + same action is the same ingredient. Different props, literal values, or test data do NOT create another ingredient — never save a duplicate just to try another configuration. Create a variant only for a proved different required-input contract or a provider mode/schema branch. Ask for explicit confirmation before creating a custom non-Composio ingredient.

Prefer a function over a raw ingredient. If a `copilot_search_workspace` capability result lists `relatedFunctions`, exact-read those function ids before rebuilding from the raw ingredient — the function already wires the ingredient into row work. Save ingredients as `ingredientType:"enrichment"` and call them from `copilot_workflow_workbench` columns; source ingredients are disabled, so an API that would seed rows must be recreated as enrichment and consumed by the connected machine.

## Contract discipline

- `inputs[]` must include every documented API request parameter, required and optional; mark optional ones `required:false`. Do not send placeholder values for optional params in recipe formulas.
- Ingredient formulas must use `request({ method, url, path, query, body, headers })`. Plain objects like `({ request_body: ... })` are not allowed. For Composio, only `request({ query: { ...actionArgs } })`. `url` is relative; `path` fills `{id}` / `:id` placeholders.
- Never put credential, auth, or secret keys in formulas — credentials are injected outside formula code.

Prove the **minimum runnable contract** before saving: auth, endpoint/action, method, every required request field, only the optional fields used now, the useful response shape, and any pagination/async behavior. Do not guess.

## Input fields

Use only `inputs[]`. Each included input carries:

- `name` — stable runtime name.
- `apiKey` — only when the provider's wire key differs from `name`.
- `location` — `query`, `body`, `path`, or `header`.
- `type` — the JSON value type; `required` — whether the provider requires it.
- `description` — the value's purpose, not transport details.
- `configOrData` — `config` for fixed recipe setup, `data` for per-row values.
- `sampleValue` — a safe, real live-test value, **never a runtime default**.

Do not use empty strings, nulls, fake IDs, or placeholders for an unused optional input — leave it out. Recipe columns pass current values with a `request(...)` formula using the exact input names.

`configOrData:"config"` is a recipe handoff, not automatic runtime storage: before using the ingredient in a recipe, create or reuse a real source input column, persist the resolved value with `copilot_workflow_workbench(action:"update_settings", settingsUpdates:{ inputDefaults })`, and reference that column in `request(...)`. A `data` input maps from the current row or an upstream column. Never use `sampleValue` or a hidden formula literal as the recipe value. For POST and Composio operations, set `responseConfig.externalActionCapability` to `read` or `write`; add timeouts, field mappings, proxy settings, and rate limits only when the proven operation needs them.

## Description and searchable metadata

- Write a short outcome summary: what the operation does and when it is useful. Add a short `Response:`/`Returns:` section only when the stable useful outputs are known. Do not copy provider documentation or a raw response schema, and do not repeat request/location/sample transport details the runtime already derives from `inputs[]`.
- Do not set `name` or `displayName` — the runtime derives both from the saved operation.
- Tags help future discovery: `capabilityTags` (what it does), `inputTags` (concepts it accepts), `outputTags` (concepts it returns), `useCaseTags` (why a future workflow searches for it). Use specific `snake_case` tags; avoid broad labels like `api`, `data`, `search`, `tool`, or `integration`, and do not duplicate provider/action names (those canonical fields are already searched). Save `freshnessClass` directly as `current`, `historical`, or `unknown`. Omit metadata you have not proved.

## Auto-test safety

The ordinary save/update auto-test calls the real provider with `sampleValue` and may retry. Use one low-cost read sample. For a write or uncertain action, get approval for the exact side effect and target first, and resolve ambiguous target IDs with a read/list call before the write — never discover a write payload through repeated saves, and remember a local rollback cannot undo a provider write. `success:true` does not prove usefulness: inspect auth, the requested entity, useful fields, errors, and whether a write actually occurred, then rediscover the saved operation to confirm its identity and searchable metadata.

## Provider tuning: research rate limits before saving

Before saving, know the provider's real allowance. A saved ingredient with the wrong rate-limit/concurrency config produces `RATE_LIMITED` cells that the retry schedule already owns — do not hammer manual reruns or batch workarounds. Fix the cause: set the ingredient's rate limit and concurrency to match the provider's documented allowance with `copilot_update_ingredient`, then let scheduled retries run or rerun once after the config fix. (A null rate-limit config defaults to a conservative rate — set it deliberately.)

## Credentials

- Store credentials with `copilot_set_secure_fields` (pass the secret values as tool arguments — this is the credential path here). OAuth-connected apps are set up in the Arnie web app (Settings → Integrations), not from this client.
- **Never ask the user to paste API keys, tokens, cookies, passwords, or auth headers in chat.** Let Arnie collect and store them safely.
- On an auth failure, do not guess: check key value, header name, header prefix, trailing whitespace, token scope, and account/project. Study the provider through `copilot_search_workspace`, then current provider docs, before claiming the API cannot do it. A saved operation failing proves only that operation is off — not that the provider lacks the capability.

## Proof order

Discover the saved contract → prove the exact endpoint/action with `copilot_workflow_workbench(action:"external_call")` → commit the column. When manual proof works and no reusable function covered it, save the missing ingredient/function before moving on, so the next chat, run, or table can reuse it.
