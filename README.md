# SEO Skills for Claude Code

Production-grade SEO skills for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview). Open source. MIT licensed. Designed for founders, agencies, and in-house SEO teams who want senior-level structured data work without copying templates from a blog post.

---

## What's Inside

### `/schema-architect`

A guided workflow that generates JSON-LD schema markup for your site — and won't ship it until both `validator.schema.org` AND Google's Rich Results Test report zero errors.

**The skill:**

1. Detects your site type (ecommerce, B2B SaaS, local business, blog, marketplace, nonprofit, personal brand, lead-gen)
2. Ingests your site architecture from your choice of:
   - Sitemap URL
   - Firecrawl API crawl (with permission)
   - Screaming Frog CSV/XLSX export
   - Manual URL list
3. Lets you scope to entire site or a specific section (PDPs, blog, service pages, locations, FAQ, single URL)
4. Recommends the right schema stack per page archetype with reasoning
5. Asks for missing entity data (NAP, social profiles, logo, GTIN, etc.) — never fabricates
6. Generates JSON-LD with proper `@id` cross-linking
7. **Validates every block via Firecrawl/browser against schema.org + Google Rich Results — repairs errors automatically (up to 3 retries)**
8. Delivers files to `./schema/` with platform-specific install instructions (Next.js, WordPress, Shopify, Webflow, raw HTML)

---

## Install

```
/plugin marketplace add brandon468/seo-skills
/plugin install seo-skills@seo-skills
```

After install, run:

```
/schema-architect
```

---

## Requirements

- **Claude Code** (latest)
- **Firecrawl** — required for crawling and validator submission. Install via `/plugin install firecrawl@anthropic-skills` (or your preferred source). Get an API key at https://firecrawl.dev — set as `FIRECRAWL_API_KEY` env var.
- **Claude in Chrome MCP** (optional) — fallback validator path when Firecrawl can't drive a JS-heavy form.
- **xlsx skill** (optional) — for Screaming Frog `.xlsx` exports.

---

## Why Another Schema Tool?

Most schema generators (including the one shipped with `searchfit-seo`) hand you a template with empty fields and call it a day. That's fine if you already know:

- Which schema types apply to your business model
- Which fields are actually required for rich result eligibility
- How `@id` cross-linking works between Organization, WebSite, Article publisher, Product brand, etc.
- Which LocalBusiness subtype fits your business
- How to validate JSON-LD against both schema.org AND Google's Rich Results Test

If you don't, you'll ship invalid markup. Google won't surface rich results, and worse — fabricated review/rating markup can earn manual penalties.

`schema-architect` walks the workflow with you and refuses to deliver unvalidated output. That's the entire pitch.

---

## Roadmap

- `/internal-linking-architect` — analyze a site's link graph, identify orphan pages, recommend internal links per page archetype
- `/technical-seo-audit` — Core Web Vitals, render-blocking, indexation, crawl budget
- `/serp-monitor` — track ranking volatility for a keyword cluster across a date range
- `/content-refresh` — detect ranking decay and recommend updates per article

If you want to contribute one of these, open an issue.

---

## Repo Structure

```
seo-skills/
├── README.md
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── seo-skills/
        ├── plugin.json
        └── skills/
            └── schema-architect/
                ├── SKILL.md
                └── references/
                    ├── schema-types-by-business.md
                    ├── schema-templates.md
                    └── validation-workflow.md
```

---

## Contributing

PRs welcome. Two rules:

1. **No fabricated data.** If a skill needs a field, it asks the user or omits it. Period.
2. **Validation before delivery.** Any skill that generates markup, sitemaps, or other crawlable artifacts must validate before writing to disk.

---

## License

MIT — do whatever you want with it. Attribution appreciated.

Built by [Brandon Jarman](https://github.com/brandon468) / [Digital Jarmarketing](https://digitaljarmarketing.com).
