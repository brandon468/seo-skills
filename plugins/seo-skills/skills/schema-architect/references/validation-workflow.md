# Validation Workflow

This is the exact procedure for Phase 7 of the skill. **Both validators
must pass before a block is delivered.** No shortcuts.

---

## Validator 1 — validator.schema.org

### Endpoint

```
POST https://validator.schema.org/validate
Content-Type: application/x-www-form-urlencoded
Body: code=<URL-encoded JSON-LD payload>
```

The endpoint returns JSON. Look for these top-level keys in the response:

- `tripleGroups[]` — parsed entities (used to confirm it parsed)
- `errors[]` — array of error objects with `severity`, `message`, `path`
- `warnings[]` — array of warning objects (non-blocking)

### How to call it

**Preferred path: Firecrawl HTTP**

If `firecrawl` is available, use a `scrape` call with `method: "POST"`:

```
firecrawl scrape \
  --url "https://validator.schema.org/validate" \
  --method POST \
  --headers '{"Content-Type":"application/x-www-form-urlencoded"}' \
  --body 'code=<URL-encoded JSON-LD>'
```

Parse the JSON response.

**Fallback path: Claude-in-Chrome**

If Firecrawl is unavailable:

1. `mcp__Claude_in_Chrome__navigate` → `https://validator.schema.org/`
2. `mcp__Claude_in_Chrome__form_input` → fill the textarea (selector
   `textarea[name="code"]` or the visible code editor) with the JSON-LD
3. Click the "Run Test" / "Validate" button
4. Wait 3-5 seconds for results to render
5. `mcp__Claude_in_Chrome__get_page_text` and parse for the
   "Errors" and "Warnings" panels

### Pass criteria

- `errors[]` is empty (or `length === 0`).
- All entities in `tripleGroups` correspond to types you submitted (sanity check).

### Common errors and fixes

| Error message | Fix |
|---------------|-----|
| `The property X is not recognized by the schema (e.g., schema.org)...` | Misspelled property name. Cross-check `references/schema-templates.md`. |
| `Required property X is missing` | Ask user, OR drop the parent type if it's optional. |
| `URL value is not valid` | URL is relative or malformed. Make absolute. |
| `Date value is not in ISO 8601 format` | Convert. Include timezone. |
| `The value of X is not of the expected type Y` | Type mismatch (string where Number expected, etc.). |

---

## Validator 2 — Google Rich Results Test

**No public API.** You must drive the UI.

### URL

```
https://search.google.com/test/rich-results
```

### Preferred path: Firecrawl with `actions`

Firecrawl's `scrape` endpoint supports an `actions` array. Use the
following sequence:

```json
{
  "url": "https://search.google.com/test/rich-results",
  "formats": ["html", "screenshot"],
  "actions": [
    { "type": "wait", "milliseconds": 2000 },
    { "type": "click", "selector": "[role='tab']:nth-child(2)" },
    { "type": "wait", "milliseconds": 500 },
    { "type": "write", "selector": "textarea, [contenteditable='true']", "text": "<JSON-LD wrapped in <script> tag>" },
    { "type": "click", "selector": "button[type='submit'], [aria-label*='Test']" },
    { "type": "wait", "milliseconds": 15000 },
    { "type": "screenshot" }
  ]
}
```

The "Code" tab is what we need (Firecrawl can paste HTML+JSON-LD there).
After submission, wait 10-15 seconds — the test takes that long.

Then parse the returned `html` for:
- `[data-test-id="results-found"]` or text "Page is eligible"
- Items detected (e.g., "1 valid item detected")
- Any error rows (text "Errors" followed by detail rows)
- Any warning rows

### Fallback path: Claude-in-Chrome

```
1. mcp__Claude_in_Chrome__navigate → https://search.google.com/test/rich-results
2. Click "Code" tab (so we can paste, not URL-test)
3. mcp__Claude_in_Chrome__form_input on the code textarea — paste the
   full <script type="application/ld+json"> ... </script> block
4. Click "Test code" button
5. Wait 10-15 seconds
6. mcp__Claude_in_Chrome__read_page — extract:
   - "Page is eligible for rich results" / "Page cannot be processed"
   - List of detected items + their type
   - Errors (red icons) and warnings (yellow icons) per item
```

### Pass criteria

- "Page is eligible for rich results" message present, OR at least one
  item detected with zero errors.
- Errors panel shows zero errors for the type(s) you submitted.

### Common errors and fixes

| Error message | Fix |
|---------------|-----|
| `Either "offers", "review", or "aggregateRating" should be specified` | Product needs at least one. Add Offer (most common). |
| `Missing field "image"` | Image is required for most types. Provide an absolute URL. |
| `Missing field "publisher"` (Article) | Add `publisher` referencing your `Organization` `@id`. |
| `Missing field "logo"` (Organization → Article publisher) | Logo image must be present and ≥112px tall. |
| `The property X is not a known property of Y` | Cross-check spelling and capitalization. |
| `Cannot find any markup` | Ensure JSON-LD is wrapped in `<script type="application/ld+json">`. |
| `Invalid JSON` | Run JSON.parse locally first. Check trailing commas. |

---

## Repair Loop

```
attempts = 0
while attempts < 3:
    schema_org_result = validate_with_schema_org(jsonld)
    google_result = validate_with_google(jsonld)

    if schema_org_result.errors.length == 0 and google_result.errors.length == 0:
        mark_validated(block)
        break

    fixes = []
    for err in schema_org_result.errors + google_result.errors:
        fix = lookup_fix(err)
        if fix.requires_user_data:
            ask_user(err.field)
        else:
            fixes.append(fix)

    apply_fixes(jsonld, fixes)
    attempts += 1

if attempts == 3 and not_validated:
    surface_errors_to_user(jsonld, remaining_errors)
    do_not_mark_validated()
```

**Never mark a block validated without zero errors from BOTH validators.**
Warnings are surfaced in the report but do not block delivery.

---

## Validation Report Format

Write `./schema/validation-report.md`:

```markdown
# Schema Validation Report

Generated: <timestamp>
Total blocks: N | Validated: N | Failed: 0 | Warnings: M

## Per-block results

| URL | Schemas | schema.org | Rich Results | Warnings |
|-----|---------|-----------|--------------|----------|
| /products/foo | Product, Offer, BreadcrumbList | ✓ | ✓ | 1 |
| /blog/bar | Article, BreadcrumbList | ✓ | ✓ | 0 |

## Warnings (non-blocking)

### /products/foo
- `aggregateRating` is recommended for products with reviews. Consider
  collecting reviews via [tool] before adding this field.

## Failed blocks (if any — do NOT install these)

(None — all blocks passed validation.)
```
