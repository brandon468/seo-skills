# JSON-LD Schema Templates

Templates for every type the skill supports. Each one lists:
- **Required** fields (Google + schema.org both require → if missing, ASK USER)
- **Recommended** fields (boost rich result eligibility)
- **Optional** fields (drop if not provided)

Wrap every block in:

```html
<script type="application/ld+json">
{ ... }
</script>
```

When emitting multiple types for one URL, prefer one `<script>` per type
unless they cross-reference, in which case use a single `@graph`.

---

## Organization (site-wide singleton)

**Required:** `name`, `url`
**Recommended:** `logo`, `sameAs`, `contactPoint`, `description`
**Optional:** `foundingDate`, `founder`, `numberOfEmployees`, `address`

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "@id": "https://example.com/#organization",
  "name": "",
  "url": "https://example.com",
  "logo": {
    "@type": "ImageObject",
    "url": "https://example.com/logo.png",
    "width": 600,
    "height": 60
  },
  "description": "",
  "sameAs": [
    "https://twitter.com/...",
    "https://linkedin.com/company/...",
    "https://instagram.com/..."
  ],
  "contactPoint": [{
    "@type": "ContactPoint",
    "telephone": "+1-555-555-5555",
    "contactType": "customer support",
    "email": "support@example.com",
    "areaServed": "US",
    "availableLanguage": ["en"]
  }]
}
```

## WebSite (site-wide singleton, with optional SearchAction)

**Required:** `name`, `url`
**Recommended:** `publisher`, `potentialAction` (sitelinks search)

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "@id": "https://example.com/#website",
  "name": "",
  "url": "https://example.com",
  "publisher": { "@id": "https://example.com/#organization" },
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://example.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

## LocalBusiness (or specific subtype)

**Required:** `@type` (use specific subtype), `name`, `address`, `telephone`
**Recommended:** `geo`, `openingHoursSpecification`, `priceRange`,
`image`, `url`, `sameAs`, `areaServed`

```json
{
  "@context": "https://schema.org",
  "@type": "Plumber",
  "@id": "https://example.com/#localbusiness",
  "name": "",
  "image": "https://example.com/storefront.jpg",
  "url": "https://example.com",
  "telephone": "+1-555-555-5555",
  "priceRange": "$$",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "",
    "addressLocality": "",
    "addressRegion": "",
    "postalCode": "",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 0,
    "longitude": 0
  },
  "openingHoursSpecification": [{
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": ["Monday","Tuesday","Wednesday","Thursday","Friday"],
    "opens": "09:00",
    "closes": "17:00"
  }],
  "areaServed": [{ "@type": "City", "name": "" }],
  "sameAs": []
}
```

LocalBusiness subtypes (use the most specific one): `AnimalShelter`,
`AutoBodyShop`, `AutoDealer`, `AutoRepair`, `Bakery`, `BarOrPub`,
`BeautySalon`, `BikeStore`, `BookStore`, `Brewery`, `CafeOrCoffeeShop`,
`ChildCare`, `ClothingStore`, `Dentist`, `DryCleaningOrLaundry`,
`Electrician`, `EmergencyService`, `EmploymentAgency`, `EntertainmentBusiness`,
`EventVenue`, `ExerciseGym`, `FastFoodRestaurant`, `FinancialService`,
`Florist`, `FurnitureStore`, `GeneralContractor`, `GroceryStore`,
`HairSalon`, `HardwareStore`, `HealthAndBeautyBusiness`, `HomeAndConstructionBusiness`,
`HousePainter`, `HVACBusiness`, `InsuranceAgency`, `JewelryStore`,
`LegalService`, `Library`, `Locksmith`, `MedicalBusiness`, `MovingCompany`,
`Notary`, `Optician`, `Pharmacy`, `Physician`, `Plumber`, `PostOffice`,
`ProfessionalService`, `RadioStation`, `RealEstateAgent`, `Restaurant`,
`RoofingContractor`, `School`, `SelfStorage`, `ShoeStore`, `SkiResort`,
`SportsClub`, `Store`, `TattooParlor`, `TaxiService`, `TelevisionStation`,
`TennisComplex`, `TireShop`, `TouristAttraction`, `TouristInformationCenter`,
`TravelAgency`, `VeterinaryCare`, `Winery`.

## Product

**Required:** `name`, `image`, `description`, `offers` (with price + currency + availability)
**Recommended:** `brand`, `sku`, `gtin`, `mpn`, `aggregateRating` (real),
`review` (real)

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "",
  "image": ["https://example.com/p1.jpg"],
  "description": "",
  "sku": "",
  "gtin13": "",
  "brand": { "@type": "Brand", "name": "" },
  "offers": {
    "@type": "Offer",
    "url": "https://example.com/products/foo",
    "priceCurrency": "USD",
    "price": "0.00",
    "priceValidUntil": "2027-12-31",
    "availability": "https://schema.org/InStock",
    "itemCondition": "https://schema.org/NewCondition",
    "shippingDetails": {
      "@type": "OfferShippingDetails",
      "shippingRate": {
        "@type": "MonetaryAmount",
        "value": "0",
        "currency": "USD"
      },
      "shippingDestination": {
        "@type": "DefinedRegion",
        "addressCountry": "US"
      }
    },
    "hasMerchantReturnPolicy": {
      "@type": "MerchantReturnPolicy",
      "returnPolicyCategory": "https://schema.org/MerchantReturnFiniteReturnWindow",
      "merchantReturnDays": 30
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "0",
    "reviewCount": "0",
    "bestRating": "5"
  }
}
```

## SoftwareApplication

**Required:** `name`, `applicationCategory`, `operatingSystem`
**Recommended:** `offers`, `aggregateRating`, `description`, `screenshot`

```json
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "description": "",
  "url": "https://example.com",
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  }
}
```

## Article / BlogPosting / NewsArticle

**Required:** `headline` (≤110 chars), `image`, `datePublished`, `author`, `publisher` (with logo)
**Recommended:** `dateModified`, `mainEntityOfPage`, `description`, `articleSection`, `wordCount`

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "",
  "description": "",
  "image": ["https://example.com/post-hero.jpg"],
  "datePublished": "2026-01-15T08:00:00-08:00",
  "dateModified": "2026-01-20T08:00:00-08:00",
  "author": {
    "@type": "Person",
    "name": "",
    "url": "https://example.com/authors/jane"
  },
  "publisher": { "@id": "https://example.com/#organization" },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://example.com/blog/post-slug"
  }
}
```

For news sites use `@type: "NewsArticle"` instead of `Article`.

## TechArticle (for developer docs)

Same fields as Article, but `@type: "TechArticle"` and add:
- `proficiencyLevel`: `Beginner` | `Expert`
- `dependencies`: free text (e.g., "Node.js 20+")

## FAQPage

**Required:** `mainEntity` array of `Question`s, each with one `acceptedAnswer`

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": ""
      }
    }
  ]
}
```

Rules: Q&A must exist on the page (not just in markup). No promotional
language. No advertising.

## HowTo

**Required:** `name`, `step` (array of `HowToStep`)
**Recommended:** `image`, `totalTime` (ISO 8601 duration), `tool`, `supply`

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "",
  "description": "",
  "image": "https://example.com/howto-hero.jpg",
  "totalTime": "PT30M",
  "step": [
    {
      "@type": "HowToStep",
      "name": "",
      "text": "",
      "image": "https://example.com/step1.jpg",
      "url": "https://example.com/howto#step1"
    }
  ]
}
```

## Recipe

**Required:** `name`, `image`, `recipeIngredient`, `recipeInstructions`
**Recommended:** `prepTime`, `cookTime`, `totalTime`, `recipeYield`,
`nutrition`, `aggregateRating`, `author`, `datePublished`

```json
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "",
  "image": [],
  "author": { "@type": "Person", "name": "" },
  "datePublished": "",
  "description": "",
  "prepTime": "PT15M",
  "cookTime": "PT30M",
  "totalTime": "PT45M",
  "recipeYield": "4 servings",
  "recipeCategory": "",
  "recipeCuisine": "",
  "recipeIngredient": [],
  "recipeInstructions": [
    { "@type": "HowToStep", "text": "" }
  ]
}
```

## Event

**Required:** `name`, `startDate`, `location`
**Recommended:** `endDate`, `image`, `description`, `eventStatus`,
`eventAttendanceMode`, `organizer`, `offers`, `performer`

```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "",
  "startDate": "",
  "endDate": "",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "",
      "addressLocality": "",
      "addressRegion": "",
      "postalCode": "",
      "addressCountry": "US"
    }
  },
  "image": [],
  "description": "",
  "organizer": { "@id": "https://example.com/#organization" },
  "offers": {
    "@type": "Offer",
    "url": "",
    "price": "0",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock",
    "validFrom": ""
  }
}
```

## VideoObject

**Required:** `name`, `description`, `thumbnailUrl`, `uploadDate`
**Recommended:** `duration` (ISO 8601), `contentUrl`, `embedUrl`,
`interactionStatistic`

```json
{
  "@context": "https://schema.org",
  "@type": "VideoObject",
  "name": "",
  "description": "",
  "thumbnailUrl": [""],
  "uploadDate": "",
  "duration": "PT1M30S",
  "contentUrl": "",
  "embedUrl": ""
}
```

## BreadcrumbList

**Required:** `itemListElement` array, each with `position`, `name`, `item`

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://example.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Category",
      "item": "https://example.com/category"
    }
  ]
}
```

## Person

**Required:** `name`
**Recommended:** `url`, `image`, `sameAs`, `jobTitle`, `worksFor`,
`description`, `alumniOf`

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "@id": "https://example.com/about/#person",
  "name": "",
  "url": "https://example.com/about",
  "image": "",
  "jobTitle": "",
  "worksFor": { "@id": "https://example.com/#organization" },
  "sameAs": [
    "https://twitter.com/...",
    "https://linkedin.com/in/..."
  ],
  "description": ""
}
```

## Service

**Required:** `name`, `provider`
**Recommended:** `serviceType`, `areaServed`, `offers`, `description`

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "",
  "serviceType": "",
  "provider": { "@id": "https://example.com/#organization" },
  "areaServed": { "@type": "City", "name": "" },
  "description": "",
  "offers": {
    "@type": "Offer",
    "price": "",
    "priceCurrency": "USD"
  }
}
```

## Review (standalone, not nested)

**Required:** `itemReviewed`, `reviewRating`, `author`
**Recommended:** `reviewBody`, `datePublished`, `publisher`

```json
{
  "@context": "https://schema.org",
  "@type": "Review",
  "itemReviewed": { "@type": "Product", "name": "" },
  "reviewRating": {
    "@type": "Rating",
    "ratingValue": "5",
    "bestRating": "5"
  },
  "author": { "@type": "Person", "name": "" },
  "reviewBody": "",
  "datePublished": ""
}
```

## JobPosting

**Required:** `title`, `description`, `datePosted`, `hiringOrganization`,
`jobLocation`
**Recommended:** `validThrough`, `employmentType`, `baseSalary`

```json
{
  "@context": "https://schema.org",
  "@type": "JobPosting",
  "title": "",
  "description": "",
  "datePosted": "",
  "validThrough": "",
  "employmentType": "FULL_TIME",
  "hiringOrganization": { "@id": "https://example.com/#organization" },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "addressLocality": "",
      "addressRegion": "",
      "postalCode": "",
      "addressCountry": "US"
    }
  },
  "baseSalary": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": {
      "@type": "QuantitativeValue",
      "value": "0",
      "unitText": "YEAR"
    }
  }
}
```

## Course

**Required:** `name`, `description`, `provider`
**Recommended:** `hasCourseInstance`, `offers`

```json
{
  "@context": "https://schema.org",
  "@type": "Course",
  "name": "",
  "description": "",
  "provider": { "@id": "https://example.com/#organization" }
}
```

## CollectionPage / ItemList

For category and tag pages:

```json
{
  "@context": "https://schema.org",
  "@type": "CollectionPage",
  "name": "",
  "url": "",
  "mainEntity": {
    "@type": "ItemList",
    "itemListElement": [
      { "@type": "ListItem", "position": 1, "url": "" },
      { "@type": "ListItem", "position": 2, "url": "" }
    ]
  }
}
```

## ProfilePage (author / seller pages)

```json
{
  "@context": "https://schema.org",
  "@type": "ProfilePage",
  "mainEntity": {
    "@type": "Person",
    "name": "",
    "url": ""
  }
}
```

## PodcastSeries / PodcastEpisode

```json
{
  "@context": "https://schema.org",
  "@type": "PodcastSeries",
  "name": "",
  "url": "",
  "description": "",
  "webFeed": ""
}
```

```json
{
  "@context": "https://schema.org",
  "@type": "PodcastEpisode",
  "name": "",
  "url": "",
  "datePublished": "",
  "associatedMedia": {
    "@type": "MediaObject",
    "contentUrl": ""
  },
  "partOfSeries": {
    "@type": "PodcastSeries",
    "name": ""
  }
}
```

---

## Cross-linking with @graph

When emitting multiple types for one URL, prefer `@graph` so all entities
share `@context`:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Organization", "@id": "https://example.com/#organization", "name": "..." },
    { "@type": "WebSite", "@id": "https://example.com/#website", "publisher": { "@id": "https://example.com/#organization" } },
    { "@type": "WebPage", "@id": "https://example.com/page#webpage", "isPartOf": { "@id": "https://example.com/#website" } },
    { "@type": "Article", "mainEntityOfPage": { "@id": "https://example.com/page#webpage" }, "publisher": { "@id": "https://example.com/#organization" } }
  ]
}
```

---

## Validation Pre-flight Checklist (before submitting to validators)

- [ ] All URLs absolute (no leading `/`).
- [ ] All dates ISO 8601 with timezone where applicable.
- [ ] No `""` empty-string values — drop the field instead.
- [ ] No `null` values — drop the field instead.
- [ ] `@id` values are URLs, unique per entity, stable across pages.
- [ ] `@type` matches actual page content.
- [ ] LocalBusiness uses the most specific subtype.
- [ ] Article has publisher with logo (≥112px tall).
- [ ] Product has at least one offer with price + currency + availability.
- [ ] FAQ Q&A appears on the visible page (not markup-only).
- [ ] AggregateRating reflects real reviews (count > 0).
- [ ] Currency codes are ISO 4217 (USD, EUR, GBP).
- [ ] Country codes are ISO 3166 (US, GB, DE).
- [ ] Durations are ISO 8601 (PT1H30M, P1D).
