# Schema Types by Business Model

This is the lookup table the skill uses in Phase 4 to recommend schema
types. Find the user's confirmed business model, then for each page
archetype in scope, recommend the listed schema stack.

Multiple types per page are normal. Cross-link them with `@id`.

---

## Ecommerce

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `Organization` + `WebSite` (with `SearchAction`) + `BreadcrumbList` | Brand entity + sitelinks searchbox eligibility |
| collection / category | `CollectionPage` + `ItemList` + `BreadcrumbList` | Lets Google understand category pages, not just PDPs |
| pdp | `Product` + `Offer` + `AggregateRating` + `Review` (only if real) + `BreadcrumbList` | Price, availability, reviews → Shopping rich results |
| blog_post | `Article` (or `BlogPosting`) + `Person` (author) + `BreadcrumbList` | Author credibility, publish date |
| about | `AboutPage` + `Organization` (full detail) | Reinforces org entity |
| contact | `ContactPage` + `Organization` with `contactPoint` array | Phone/email rich results |
| faq | `FAQPage` | Direct rich result eligibility |
| cart / checkout | none | Don't waste markup on non-indexable pages |

## B2B SaaS

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `Organization` + `SoftwareApplication` + `WebSite` | Establishes app entity for AI assistants and Google |
| pricing | `Product` + `Offer` (one per plan) + `SoftwareApplication` | Surfaces pricing in SERPs and AI answers |
| features / use-case | `WebPage` + `BreadcrumbList` | Light schema; main weight goes on org/app |
| integrations | `WebPage` + `ItemList` of integrated `SoftwareApplication`s | Helps "X integrations" queries |
| docs | `TechArticle` + `BreadcrumbList` | TechArticle is preferred for developer docs |
| changelog / blog | `Article` + `Person` author + `Organization` publisher | Standard publishing schema |
| case-study | `Article` + `Organization` (the customer, as `about`) | Surfaces case study queries |
| testimonials | `Review` + `Organization` reviewed | Only if reviews are verifiable, not paraphrased |
| about | `AboutPage` + `Organization` (founders, foundingDate, employees) | Strengthens entity |
| pricing-comparison | `Product` array + `Offer` per plan | Comparison tables |

## Local Business

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home (single location) | `LocalBusiness` (or specific subtype) + `WebSite` | Subtype is critical — `Plumber`, `Restaurant`, `Dentist`, `LegalService`, `AutoRepair`, `HairSalon`, etc. |
| home (multi-location) | `Organization` + `LocalBusiness` per location (or link to location pages) | Clear org → branch hierarchy |
| location_page | `LocalBusiness` (subtype) with full NAP, hours, geo, areaServed | Map pack signals |
| service_page | `Service` + `LocalBusiness` (provider) | Service offering schema |
| reviews | `LocalBusiness` with `aggregateRating` (only from real reviews) | Star ratings — never fabricate |
| about | `AboutPage` + `LocalBusiness` (full detail) | Entity reinforcement |
| contact | `ContactPage` + `LocalBusiness` | NAP must match Google Business Profile exactly |
| blog_post | `Article` + `Person` author + `LocalBusiness` publisher | Establishes E-E-A-T |
| faq | `FAQPage` | Common service questions |

**LocalBusiness subtype reference:** when in doubt, use the most specific
subtype that exists. Full list:
https://schema.org/LocalBusiness#subtypes

## Blog / Publisher

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home / blog_index | `WebSite` + `Blog` + `Organization` | Establishes publication entity |
| blog_post | `Article` (or `NewsArticle` for news) + `Person` author + `Organization` publisher + `BreadcrumbList` | Standard publishing stack |
| author | `ProfilePage` + `Person` | Author credibility, E-E-A-T |
| tag / category | `CollectionPage` + `ItemList` of articles | Topic clustering |
| about | `AboutPage` + `Organization` (Publisher with logo) | Required for `Article` publisher field |

For news sites add `NewsArticle` instead of `Article` and consider
`LiveBlogPosting` for live coverage.

## Informational / Lead-gen

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `Organization` + `WebSite` | Light footprint |
| service_page | `Service` + `Organization` provider | Surfaces service queries |
| pricing | `Service` + `Offer` | If services have public pricing |
| case-study | `Article` + `Organization` about | Same as SaaS |
| blog_post | `Article` + `Person` + `Organization` | Standard |
| faq | `FAQPage` | Lead-gen pages benefit from FAQ |
| contact | `ContactPage` + `Organization` | Standard |

## Personal Brand

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `Person` + `WebSite` | Person entity is the brand |
| about | `AboutPage` + `Person` (full bio, sameAs to socials) | Knowledge panel signals |
| blog_post | `Article` + `Person` author (self-link) | Self-publishing pattern |
| portfolio item | `CreativeWork` (or `Article`/`VideoObject`/`ImageObject` per medium) | Per-piece markup |
| services | `Service` + `Person` provider | If solo practitioner |
| speaking | `Event` (past + upcoming) + `Person` performer | Speaker discoverability |
| podcast | `PodcastSeries` + `PodcastEpisode` per episode | Podcast rich results |
| course | `Course` + `Person` instructor | Course rich results |

## Marketplace

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `Organization` + `WebSite` (with `SearchAction`) | Sitelinks searchbox |
| listing | `Product` + `Offer` + `Seller` (`Organization` or `Person`) + `AggregateRating` | Marketplaces need seller entity |
| seller profile | `ProfilePage` + `Organization` or `Person` | Seller trust signals |
| category | `CollectionPage` + `ItemList` | Browse pages |
| reviews | `Review` per individual review | Granular review markup |

## Nonprofit

| Archetype | Recommended schema | Why |
|-----------|--------------------|-----|
| home | `NGO` (subtype of `Organization`) + `WebSite` | NGO type is recognized |
| program / cause | `Service` (or `CreativeWork` for content programs) | Describes the work |
| events | `Event` | Fundraisers, galas |
| donation | `DonateAction` (advanced) or `WebPage` | Donation flow |
| board / staff | `ProfilePage` + `Person` | Leadership credibility |
| annual report | `Report` + `Organization` author | Publication markup |
| about | `AboutPage` + `NGO` (foundingDate, founder) | Mission entity |

---

## Required Site-wide Entities

Regardless of business model, every site should define ONE of these
once with a stable `@id` and reference it from every page:

- `Organization` (or `LocalBusiness`/`NGO`/`Person` for solo) at
  `{root}#organization` (or `#person`).
- `WebSite` at `{root}#website`, with `publisher: { @id: ... }` pointing
  to the org.

This becomes the publisher reference for every Article and the brand
reference for every Product.

---

## Eligibility Cheat Sheet — What Earns Rich Results

| Schema | Rich result | Hard requirements |
|--------|-------------|-------------------|
| `Product` + `Offer` + `AggregateRating` | Price + stars in SERP | Real reviews; avg, count, max |
| `FAQPage` | Expandable FAQ | Genuine Q&A; not promotional |
| `HowTo` | Numbered how-to (limited rollout) | Real steps with images |
| `Article` (with publisher logo) | Top stories, news carousel | Logo ≥112px tall, AMP optional |
| `Recipe` | Recipe card | Ingredients, instructions, totalTime |
| `Event` | Event listing | startDate, location, name |
| `LocalBusiness` (full) | Knowledge panel data | NAP must match Google Business Profile |
| `VideoObject` | Video thumbnail | thumbnailUrl, uploadDate, duration ISO 8601 |
| `JobPosting` | Jobs in SERP | datePosted, hiringOrganization, location |
| `Course` | Course listing | provider, name, description |
| `Review` (standalone) | Critic review | Real verified reviewer; not anonymous |
| `BreadcrumbList` | Breadcrumb path in SERP | Position-ordered ListItem array |
