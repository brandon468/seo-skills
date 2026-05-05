---
name: schema-architect
description: Use when generating, auditing, or validating JSON-LD schema markup for a website. Triggers on "generate schema", "schema markup for my site", "structured data", "rich results", "schema for my products / blog / locations", "fix invalid schema", "add JSON-LD". Detects site type (ecommerce, B2B SaaS, local, blog, etc.), ingests site architecture from a sitemap, Firecrawl crawl, Screaming Frog export, or manual URL list, scopes to a section if requested, recommends correct schema types, gathers missing entity data without fabricating, generates JSON-LD, and validates every block against validator.schema.org AND Google's Rich Results Test before delivery.
---

# /schema-architect — Validated Schema Markup Generator

Most schema generators dump a generic template and leave the user to fix it.
This skill does the opposite: it analyzes the site, recommends the right
schema for each page archetype, only fills fields it actually has data for,
and refuses to deliver any block that hasn't passed both
`validator.schema.org` and Google's Rich Results Test.

**Core principle: never fabricate schema data, never deliver unvalidated
JSON-LD.** Missing fields get asked, not invented. Invalid output gets
fixed and re-validated, not shipped.

---

## When to Use

- "Generate schema for my site" / "Add JSON-LD to my pages"
- "I want rich results for my products / articles / locations"
- "Audit my structured data" / "My schema isn't validating"
- "What schema should an ecommerce / SaaS / local site use?"

## When NOT to Use

- The user only wants a single template snippet for one known type
  (use `/searchfit-seo:schema-markup` or `/searchfit-seo:generate-schema`).
- The user has not yet decided whether they need schema at all
  (point them at `/searchfit-seo:seo-audit` first).

---

## Required Output Directory

All generated files go to `./schema/` at the project root.

```
./schema/
  {archetype}-{slug}.jsonld     # one block per URL/archetype
  INSTALL.md                    # platform install instructions
  validation-report.md          # validator results + warnings
```

---

## Workflow (8 Phases)

You MUST run these in order. Do not skip ahead. Do not deliver until
Phase 7 reports `Validated ✓` on every block.

### Phase 1 — Detect Site Type

1. Ask the user: "What's the root URL of the site?"
2. `WebFetch` the homepage. Look for signals:
   - Cart / checkout / `/products/` paths → **Ecommerce**
   - Pricing tiers, "Sign up", "Book demo", `/app.` subdomain → **B2B SaaS**
   - NAP block, hours, "Call now", service-area copy → **Local Business**
   - Article feed, author bylines, `/blog/` or `/posts/` → **Blog / Publisher**
   - Product/service pages without cart, lead form → **Informational / Lead-gen**
   - `mailto:` author, single-person bio → **Personal Brand**
   - User-listed inventory, multi-vendor → **Marketplace**
   - 501(c)(3), donate button → **Nonprofit**
3. Use `AskUserQuestion` to confirm the detected type. Show your reasoning
   ("I see a cart on the homepage and `/products/` URLs, so this looks like
   ecommerce — confirm?").

### Phase 2 — Ingest Site Architecture

Use `AskUserQuestion` (multiSelect) to let the user choose one or more
input methods:

| Option | What you do |
|--------|-------------|
| **Sitemap URL** | `WebFetch` `{root}/sitemap.xml` (and any nested sitemaps). Parse `<loc>` entries. Group by path pattern. |
| **Firecrawl** | Confirm permission. Check `FIRECRAWL_API_KEY` is set. Invoke the `firecrawl` skill with `crawl` mode (limit 100 pages by default; ask before going higher). Extract per-URL `metadata.title`, `metadata.description`, key entities. |
| **Screaming Frog export** | Ask for the file path. Use the `xlsx` skill for `.xlsx` or `pandas` via Bash for `.csv`. Read columns: Address, Status Code, Title 1, Meta Description, H1, Indexability. Filter to indexable 200s. |
| **Manual URL list** | Accept paste. One URL per line. |

Normalize all four into one internal table:
`{ url, page_archetype, title, h1, description, key_entities }`.

Classify `page_archetype` by URL pattern + content heuristics:
`home`, `about`, `contact`, `pdp`, `collection`, `blog_post`,
`blog_index`, `service_page`, `location_page`, `pricing`, `faq`,
`legal`, `author`, `tag`, `category`.

### Phase 3 — Scope

Ask the user:

> "Schema for the entire site, or a specific section?"

Options: Entire site, Blog only, PDPs only, Collection/Category only,
Service pages only, Location pages only, FAQ only, About/Org only,
Single URL.

Filter the internal table to the chosen scope before continuing.

### Phase 4 — Recommend Schema Types

Load `references/schema-types-by-business.md`. For each archetype in scope,
look up the recommended schema stack for the detected site type.

Present the recommendation as a table:

| URL pattern | Archetype | Recommended schema | Why |
|-------------|-----------|--------------------|----|
| `/products/*` | PDP | `Product` + `Offer` + `AggregateRating` + `BreadcrumbList` | Drives price/review rich results in Google Shopping |

Confirm with the user. Allow edits.

### Phase 5 — Fill Missing Entity Data

Inspect fetched pages for the fields each chosen schema type requires
(see `references/schema-templates.md` for required vs. recommended fields).

For each REQUIRED field that's missing across the dataset, ask the user
via `AskUserQuestion`. **Batch related questions per archetype** — do not
ask one at a time:

- **Organization-wide once:** legal name, logo URL (absolute), founders,
  founding date, NAP, `sameAs` social profile URLs, `contactPoint`.
- **LocalBusiness:** specific subtype (Plumber, Restaurant, Dentist…),
  hours, areas served, accepted payment, price range.
- **Product (per item or batched if uniform):** GTIN, SKU, MPN, brand,
  price, currency, availability, condition, shipping cost.
- **Article (per post):** author entity, publish date, modified date,
  featured image absolute URL.
- **FAQ:** Q/A pairs (extract from page content first; ask only for gaps).

**Iron rule: never invent data.** If a field is not provided and not on
the page, omit it. Do not guess a phone number, address, GTIN, price,
or date.

### Phase 6 — Generate JSON-LD

Render templates from `references/schema-templates.md`. Rules:

1. One `<script type="application/ld+json">` block per URL/archetype.
2. Cross-link entities with `@id`. Define `Organization` once with
   `@id: "{root}#organization"` and reference it from every `Article`'s
   `publisher`, every `Product`'s `brand` (when applicable), etc.
3. URLs absolute. Dates ISO 8601. Currencies ISO 4217. Country codes ISO 3166.
4. Drop empty / null fields. Do not emit `""` placeholders.
5. Match `@type` to the actual page (don't slap `Article` on a service
   page just because it has paragraphs).

### Phase 7 — Validate (MANDATORY — do not skip)

Follow `references/validation-workflow.md` exactly. For every generated
block:

**Validator 1 — `validator.schema.org`**
- POST the JSON-LD to `https://validator.schema.org/validate` (form-encoded
  field `code`) using the `firecrawl` skill's HTTP capability or
  `mcp__Claude_in_Chrome__navigate` + form fill.
- Parse response for `errors[]` and `warnings[]`.

**Validator 2 — Google Rich Results Test**
- No public API exists. Use one of:
  - `firecrawl` with `actions` (navigate → fill `#code-input` → click
    Test button → wait for results → extract DOM), OR
  - `mcp__Claude_in_Chrome__navigate` to
    `https://search.google.com/test/rich-results` and drive the form.
- Read the "Detected items" panel and any errors/warnings.

**Repair loop:**
- If either validator returns errors → identify the offending field →
  fix the JSON-LD → re-run BOTH validators → repeat (max 3 iterations).
- After 3 failed iterations, surface the remaining errors to the user
  with a clear explanation and a question. Do NOT mark the block as
  validated.

A block is `Validated ✓` only when BOTH validators report zero errors.
Warnings are surfaced but don't block delivery.

### Phase 8 — Deliver

1. Write each validated block to
   `./schema/{archetype}-{slug}.jsonld`.
2. Write `./schema/INSTALL.md` with platform-specific install snippets:
   - **Next.js (App Router):** `<Script type="application/ld+json"
     dangerouslySetInnerHTML={{ __html: ... }} />` in the route's `page.tsx`
     or in `generateMetadata`'s `other` field.
   - **WordPress:** Yoast / RankMath JSON-LD field, or `wp_head` action
     in `functions.php`.
   - **Shopify:** edit `theme.liquid` and inject inside `<head>`, or use a
     metafield + `{% schema %}` block.
   - **Webflow:** Page Settings → Custom Code → Inside `<head>`.
   - **Raw HTML:** before `</head>`.
3. Write `./schema/validation-report.md`: table of `URL → schemas →
   schema.org status → Rich Results status → warnings`.
4. Print a one-screen summary to terminal: counts by type, validation
   pass rate, files written, next steps.

---

## Quick Reference

| Step | Tool | Output |
|------|------|--------|
| Detect site type | `WebFetch` + `AskUserQuestion` | Confirmed business model |
| Ingest architecture | `WebFetch` / `firecrawl` / `xlsx` / paste | URL table |
| Scope | `AskUserQuestion` | Filtered URL table |
| Recommend schema | Lookup in `schema-types-by-business.md` | Approved schema plan |
| Fill data | `AskUserQuestion` (batched) | Populated entity data |
| Generate | Templates + entity data | Raw JSON-LD blocks |
| Validate | Firecrawl / Claude-in-Chrome | Pass/fail per block |
| Deliver | `Write` to `./schema/` | Files + install guide + report |

---

## Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| Filling required fields with placeholder text or guesses | Ask the user. Drop the field if they don't have it. |
| Using relative URLs (`/logo.png`) | Convert to absolute (`https://site.com/logo.png`) before validation. |
| `@type: Article` on every page that has text | Match type to actual page purpose. |
| Skipping Rich Results Test because it has no API | Use Firecrawl `actions` or Claude-in-Chrome — both are in the user's stack. |
| Marking a block validated when only schema.org passed | Both validators must pass. No exceptions. |
| Inventing reviews / ratings to get rich result eligibility | Hard NO. Google penalizes fake review markup. |
| Outputting one giant `@graph` array when individual blocks would be clearer | Use `@graph` only when entities cross-reference; otherwise one `<script>` per type. |

---

## Red Flags — STOP and Re-validate

- "I'll just fill in a placeholder phone number" → No. Ask the user.
- "schema.org passed, that's good enough" → No. Run Rich Results too.
- "Validation is timing out, I'll skip it" → No. Retry, or fall back to
  the alternate validation path. Never skip.
- "The user is in a rush" → Validation is non-negotiable. The whole
  point of this skill is validated output.

---

## Dependencies

- `firecrawl` — for crawling and validator submission. **Required.**
  Skill prompts the user for `FIRECRAWL_API_KEY` if not set.
- `mcp__Claude_in_Chrome__*` — fallback validator path. Optional but
  recommended.
- `xlsx` — for Screaming Frog `.xlsx` exports. Optional.
- `WebFetch` — sitemap and homepage detection. Built in.
- `AskUserQuestion` — structured prompts throughout. Built in.

## References

- `references/schema-types-by-business.md` — site-type × archetype → schema lookup
- `references/schema-templates.md` — JSON-LD templates with required/recommended fields
- `references/validation-workflow.md` — exact validation procedure
