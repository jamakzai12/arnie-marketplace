---
name: formulas-conditions
description: Use the Arnie CLI for copilot_workflow_workbench formula or condition columns — {{column_id}} syntax, the safe evaluator rules, conditional gates, and waterfall final-value formulas.
---

# Formulas & Conditions

## CLI access

Use the `arnie` CLI for every `copilot_*` tool below. The shorthand
`copilot_x({...})` means `arnie call copilot_x '<same JSON>'`. Never configure
or call MCP directly. If `arnie` is missing, run
`npm install --global github:jamakzai12/arnie-marketplace`, then `arnie login`
and `arnie tools`.

Formula syntax and condition gates. Formulas and conditions consume clean scalar outputs; use an AI-generated column for semantic classification or extraction from messy text, then let formulas read that clean output.

**Formula mindset:** formulas are only for deterministic transforms, gates, cleanup, assembly, and deterministic extraction. ABSOLUTE rule: use a formula extractor only when the stable path or pattern is known before commit. If the source is messy variable text — bios, snippets, search results, descriptions, comments, headlines, page titles, rendered markdown — do not use a formula as the extractor. Use an AI column first to normalize/extract a clean scalar or bounded JSON with explicit Unknown/null handling; then formulas may consume that output. After 3 failed formula previews on a messy extraction, switch to AI normalization. Feed AI plain text/markdown, not raw HTML.

## Formula columns

Formulas produce one value. Use a simple JavaScript expression by default; for local variables or early returns, use a synchronous arrow IIFE. **Column references MUST use `{{column_id}}` syntax.**

```javascript
{{first_name}} + ' ' + {{last_name}}                    // Combine
{{employees}} >= 50 ? 'Enterprise' : 'SMB'              // Conditional
{{email_primary}} || {{email_backup}} || null           // Fallback
{{job_title}}?.toLowerCase().includes('ceo') || false   // Optional chaining (dedicated title field only)
{{data}}.items?.[0]?.name                               // Property access OUTSIDE {{}}
(() => {                                                // Multiline IIFE
  const title = String({{job_title}} || '').toLowerCase();
  if (!title) return null;
  return title.includes('ceo') ? 'Executive' : 'Other';
})()
```

**Rules:**

- The formula must produce one final value; simple formulas are one expression. Multiline logic must be a sync IIFE.
- No `async`, `await`, `Promise`, `import`, `eval`, `Function`, network calls, timers, or side effects.
- `{{}}` contains ONLY the column ID — property access goes OUTSIDE: `{{col}}.prop`, never `{{col.prop}}`. Never use bare column names.
- Use `===`/`!==`/`>`/`<`; a single `=` is assignment and is not allowed for row fields.
- Row input data is read-only — copy before sorting/reshaping. If a JSON column is stored as a string, parse it: `const p = JSON.parse({{json_col}} || '{}')`.
- `__APP_URL__` is a string token, not a column: `'__APP_URL__/app?tab=tables&tableId=' + {{table_id}}`. Never hard-code localhost or a prod host; use `tableId` exactly.
- Enrichment input formulas must use `request(...)` (see the recipe-creation skill); never put credential/auth/secret keys in formulas.

**Enrichment object access:** `{{enrichment_col}}` is the materialized cell value, already a JS object — read stable paths with property access outside the brackets (`{{reddit_posts}}?.data?.children?.length || 0`). Do NOT `JSON.parse({{enrichment_col}})`. Do not create a formula that only returns the whole enrichment object and then feed it into an AI prompt or tool input.

**Do not use formulas** on free-text fields (headline/bio/about/description/page title) to derive company, domain, title, URL, or other real-world facts. **Identity fields require identity evidence** — never build real-name/identity fields from handles, usernames, IDs, emails, URLs, or slugs. Semantic checks (`includes()`, regex, keyword lists, `.split()`) as the main classifier belong in an AI column first; formulas/conditions read the clean output.

**Search-query bridge rule:** formulas may assemble a search query from clean bridge fields, but must not invent or extract those fields from messy text. If the target surface differs from the source surface, first create bridge evidence (company, domain, website, real name, title). For contact/profile queries, company/domain is the primary anchor; name alone is not enough.

## Formula pre-commit preview

When the response shape is complex or unclear, test the formula before adding the real column with `copilot_workflow_workbench(action:"external_call")`:

```json
{
  "action": "external_call",
  "columnDefinition": { "id": "clean_value", "source": "formula", "formula": "{{raw_value}}?.items?.[0]?.name || null" }
}
```

With `{{column}}` refs it evaluates against a few existing rows. A standalone `external_call` (literal inputs, no `{{column}}` refs) runs ONCE and needs no table rows — one-off side effects go through that standalone form, never per sample row. Preview does NOT add the column, stage cells, queue work, or write row data. If the preview is right, call the same `add_column` again without `external_call`.

## Waterfall final-value formula

Chooses between already-returned values.

```javascript
{{email_primary}} || {{email_fallback}} || null
```

- Extract provider outputs into visible scalar columns first; never use `source:"input"` for derived provider output.
- Gate every secondary/fallback enrichment with a `condition` that checks the prior scalar is missing before it runs. **No condition means do not add the fallback column.**
- Put the final formula before cleanup, qualification, scoring, external calls, or hiding raw provider columns; downstream columns reference the final value column.
- Do not use the formula to create a missing fact — it only chooses between values real sources already returned. For numeric/boolean values use explicit null checks, not `||`, so valid `0`/`false` are not dropped.

Example fallback gate:

```json
{ "id": "fallback_linkedin_profile_url", "source": "enrichment", "tool": "secondary_profile_resolver", "condition": "!{{primary_linkedin_profile_url}} && !!{{best_linkedin_query}}" }
```

## Condition gates

Gate column execution with `condition` when upstream values are required. Conditions work for formula, AI, enrichment, and fallback columns, so the dependency chain stays explicit.

**Sentinel text is a missing value.** "Unknown", "N/A", "No match", "None", "Not found" pass a non-empty check because they are text — gate against missing, sentinel, junk, and off-target inputs so paid dependent columns never run on them. To compare a column to a boolean literal (`{{col}} === true`), the column's declared type must be `boolean`; on a text/AI column the stored value is text and the comparison silently skips every cell.

```javascript
condition: "!!{{company_domain}}"                                 // field exists
condition: "!!{{linkedin_url_candidate}} && (!!{{company_domain}} || !!{{company_name}})" // candidate + anchor
condition: "Number({{employees}}) > 100"                          // threshold
condition: "!!{{email}} && !!{{name}} && !!{{company}}"           // push to CRM only if complete
condition: "!{{primary_result}}"                                  // fallback only if primary failed
```

**Every enrichment column costs API credits; conditions and formulas cost zero.** Always add conditions when the user names a filter ("only if", "when", "who have"), when an enrichment depends on possibly-missing data, when a formula/AI/lookup/match depends on a non-empty upstream, when a provider-backed enrichment pushes to a destination, or when enrichments chain (gate expensive downstream calls). Placement: extract/normalize evidence -> compute checks with formulas -> gate enrichments -> gate external calls. Cascade skip is automatic — if a condition skips one column, dependents skip too.

After adding formula or condition columns, verify with `copilot_read_rows` or `copilot_workflow_workbench(action:"query")` that outputs are correct. `success: true` means the column was added, not that it produces correct values.
