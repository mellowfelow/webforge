# SEO Reference — Meta, Schema, Sitemap, Robots, GSC, AI Visibility Content

> **v8 delta — read first.** The formulas, schema shapes and checklists below are still authoritative. **The mechanics changed:**
> - **Meta is never hand-written.** Use Next.js `generateMetadata` — see `references/nextjs.md`. The HTML examples below show the OUTPUT to aim for, not code to type.
> - **Domain is never hard-coded.** Everything derives from `SITE.domain` in `src/config/site.js`. No `[domain]` find-and-replace — see SKILL.md Rule 9.
> - **`sitemap.xml` and `robots.txt` are generated** by `app/sitemap.js` / `app/robots.js` from `SITE`. Never write them by hand. Exclude every `noindex` page from the sitemap.
> - **`SearchAction` is Google-dead** — the sitelinks search box was deprecated on 21 Nov 2024. Emit it ONLY when `/search/` really exists, and only for agents (SKILL.md Rule 10).
> - **JSON-LD is built as a JS object and `JSON.stringify`d** — never templated as a string. That is how invalid schema ships.
> - **`AggregateRating` / `Review` for real, attributable reviews — including reviews migrated from the client's existing site or business profiles (preserve them, with their real counts/ratings + source).** Never invent reviews that do not exist.
> - Meta description band: **110–158 chars**. Titles ≤60. The crosscheck enforces both.

> **v11.1 delta.** JSON-LD validity checking was static-only (JSON parses, required fields present) — this confirms the schema is *valid*, not that Google treats it as *rich-result-eligible*, which is a stricter, narrower bar. Static checks can't tell the difference; a live Rich Results Test run can. See § JSON-LD validity below and SKILL.md AUDIT F.

---

---

## Google Search Console verification tag

Add to `<head>` of **every page** during build. Use GSC code from intake Section A.

```html
<!-- Google Search Console verification -->
<meta name="google-site-verification" content="[GSC_VERIFICATION_CODE]">
```

If no GSC code provided at intake:
```html
<!-- GSC verification — PENDING: replace before deploying -->
<meta name="google-site-verification" content="GSC_CODE_PENDING">
```

Log status in PROJECT.md → Hosting → GSC verification code.

---

## IndexNow key

Generate a random 32-character alphanumeric string at build time. Same key used across all pages.

Add to every page `<head>`:
```html
<meta name="IndexNow-key" content="[KEY]">
```

Create file `[KEY].txt` at site root containing only the key string.
Log key in PROJECT.md → Hosting → IndexNow key.

---

## Meta title and description formulas

### Homepage
- Title: `[Site Name] — [Primary Keyword] | [Key Differentiator]`
- Description: `[Site Name] — [primary keyword]. [Key product types]. [Differentiator]. [Shipping offer].`

### Shop page
- Title: `Shop [Primary Keyword] | [Site Name]`
- Description: `Shop [product range]. [Category count] products. [Differentiator]. [Shipping offer].`

### Category page
- Title: `[Category Name] | [Site Name]`
- Description: `Buy [category keyword]. [Sub-types]. [Differentiator]. [Shipping offer].`

### Product page
- Title: `[Product Name] — [Site Name]`
- Description: `[Product name]. [Key spec/variant]. [Price point]. [Differentiator]. [CTA].`

### Blog post
- Title: `[Post Title] | [Site Name] Blog` (≤60 chars)
- Description: Matches first 155 chars of intro paragraph.

### FAQ page
- Title: `FAQ — [Site Name]`
- Description: `Answers to common questions about [primary keyword], ordering, shipping, and more.`

### Contact page
- Title: `Contact [Site Name] | Get in Touch`
- Description: `Contact [Site Name] for order enquiries, product questions, and support. We respond within 24 hours.`

### Order page
- Title: `Place an Order | [Site Name]`
- Description: `Order [primary keyword] from [Site Name]. Fill out our simple order form and we'll confirm within hours.`

### Wholesale page
- Title: `Wholesale [Primary Keyword] | [Site Name]`
- Description: `Apply for a [Site Name] wholesale account. Competitive pricing, flexible minimums, fast fulfilment.`

### Legal pages
- Title: `[Policy Name] | [Site Name]`
- Description: Short factual summary of the policy.

---

## og:image — derivation and sizing

`og:image` is required by the "Complete social meta" crosscheck item but was never specified beyond "present" — this closes that gap.

- **Size: 1200×630** (the standard OG preview-card size every major platform expects — Facebook, X, LinkedIn, WhatsApp, Slack).
- **Derive it from whatever image already anchors that page, never source a fresh image for it.** Homepage/category pages: crop from the page's hero at 1200×630 (`object-fit: cover`, same subject-visible rule as the hero itself). Product pages (no hero — see `pages.md` § Product): crop from the primary gallery image instead. Blog posts: crop from the post's thumbnail image.
- One per key page (home, shop, each blog post) — not every single page needs a bespoke one; a page with no natural hero/thumbnail (e.g. a legal page) falls back to the homepage's og:image rather than going without one.
- If the image-sourcing skill produced the site's imagery, its sourced hero/thumbnail is the input to this crop — this skill does not source a separate image, only crops/resizes what already exists.
- Feeds `opengraph.images` in `generateMetadata` (`references/nextjs.md`) exactly like any other OG field — this is a sizing/derivation rule for that same array, not a new mechanism.

---

## AI visibility content requirements

Apply these during Mode 3 (build) and Mode 4 (audit). Full AI SEO reference is in `references/ai-visibility.md`.

### Homepage content requirements
1. **Brand entity statement** — first paragraph of hero section, add CSS class `brand-statement`:
   ```html
   <p class="brand-statement">[Brand entity statement from PROJECT.md]</p>
   ```
2. **Concrete factual claims** — shipping speed, product count, founding year, geographic coverage
3. **FAQ section** — minimum 4 questions including "What is [Brand]?" in direct-answer format
4. **Trust signals** — review reference, quality claim, shipping promise

### About page content requirements (minimum 300 words)
Structure:
1. Opening paragraph with `class="about-intro"` — brand entity statement expanded
2. Founding story — when, why, who started it
3. Mission — what the brand stands for
4. Product range — what they offer and who it's for
5. Geographic coverage — where they ship
6. Contact / support promise

### FAQ content format (AEO standard)
Every FAQ answer must follow this format:
- H3 for question (or use FAQPage structure)
- Direct answer in FIRST SENTENCE — **homepage FAQ: 40–55 words; dedicated FAQ page: up to 65 words — never exceed 65 in any answer** (matches keyword-engine's `content-planning.md` Answer Rules when keyword-engine supplied the content)
- Optional: expand with detail after first sentence
- Never bury the answer — AI assistants extract the first sentence

Required FAQ questions (minimum — add niche-specific ones):
- "What is [Site Name]?"
- "Where does [Site Name] ship?"
- "How do I place an order with [Site Name]?"
- "What payment methods does [Site Name] accept?"
- "How long does shipping take?"
- "What is [Site Name]'s return policy?"
- "Is [Site Name] legit?" or "Is [Site Name] trustworthy?"
- "[Niche-specific question relevant to the brand's product]"

---

## Schema markup — required per page type

### Organization JSON-LD — enhanced for AI visibility (homepage + about page)
`knowsAbout[]` sources from **both** the homepage AND shop-page rows of `docs/keyword-map.md` (each row's Primary + Secondary Keywords) — never homepage alone. Falls back to the site's own primary + top secondary keywords plus product categories when no keyword-engine docs exist.

**`@id` + `sameAs` disambiguation (low-cost hedge, same spirit as Speakable below — not a proven ranking lever, just cheap to get right since the fields are already being written).** `@id` gives this Organization node a stable internal identifier that other schema blocks on the site can carry alongside their own inline fields — e.g. an Article's `publisher` keeps its required `name`/`logo` inline (Google's Article rich-result check wants those present on the page, not a bare cross-page pointer) and additionally carries the same `"@id": "https://[domain]/#organization"`, so systems that stitch entities together by `@id` can tell it's the same Organization as the homepage's. Add a Wikidata and/or Wikipedia `sameAs` entry **only when the brand genuinely has one** — never create or claim one that doesn't exist; most small/new brands won't have either, and that's fine, the social `sameAs` entries below still apply.
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "@id": "https://[domain]/#organization",
  "name": "[Site Name]",
  "url": "https://[domain]/",
  "logo": "https://[domain]/images/logo.webp",
  "description": "[Brand entity statement — same text as brand-statement element]",
  "foundingDate": "[year or omit if unknown]",
  "areaServed": "[country or Worldwide]",
  "knowsAbout": [
    "[homepage primary keyword]",
    "[homepage secondary keyword 1]",
    "[shop page primary keyword]",
    "[shop page secondary keyword 1]",
    "[product category 1]"
  ],
  "sameAs": [
    "https://www.instagram.com/[handle]",
    "https://www.facebook.com/[handle]"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "[phone]",
    "contactType": "customer service",
    "availableLanguage": "English"
  }
}
```
When the brand has a real Wikidata and/or Wikipedia entry, append its URL(s) to the `sameAs` array above alongside the social links — never add one that doesn't exist.

### Speakable JSON-LD (homepage + about page)
Speakable is a **Google-only, still-beta** mechanism — limited to US/English users and news-article content. It does nothing on Alexa, Siri, or Bixby, and won't apply to most homepage/about-page use the way it's scoped here. Keep it as a zero-cost hedge, not a relied-upon lever.
```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "speakable": {
    "@type": "SpeakableSpecification",
    "cssSelector": [".brand-statement", ".hero-description", ".about-intro"]
  },
  "url": "https://[domain]/"
}
```

### WebSite JSON-LD with SearchAction (homepage only)
```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "[Site Name]",
  "url": "https://[domain]/",
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://[domain]/search/?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

### BreadcrumbList JSON-LD (all inner pages)
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {"@type": "ListItem", "position": 1, "name": "Home", "item": "https://[domain]/"},
    {"@type": "ListItem", "position": 2, "name": "[Category]", "item": "https://[domain]/shop/[category]/"},
    {"@type": "ListItem", "position": 3, "name": "[Page]", "item": "https://[domain]/[path]/"}
  ]
}
```
Adjust depth to match actual hierarchy.

### Product JSON-LD (product pages)
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "[Product Name]",
  "description": "[Product description]",
  "image": "https://[domain]/images/products/[slug].webp",
  "brand": {"@type": "Brand", "name": "[Site Name]"},
  "sku": "[product-slug]",
  "offers": {
    "@type": "Offer",
    "price": "[price]",
    "priceCurrency": "[currency code]",
    "availability": "https://schema.org/InStock",
    "url": "https://[domain]/product/[slug]/"
  }
}
```
Only include aggregateRating if real reviews exist — client-provided or migrated from their existing site/profiles (preserve those). Never invent reviews that do not exist.

### FAQPage JSON-LD (FAQ page + homepage FAQ section)
Google deprecated the FAQ rich result in 2026 — this markup no longer produces a visible snippet in classic Google Search. Keep it anyway: it structures Q&A content for AI/LLM extraction (AEO) — AI assistants and answer engines (ChatGPT, Perplexity, AI Overviews) can still parse this structured Q&A data when forming an answer.
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is [Site Name]?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "[Brand entity statement — direct answer in first sentence]"
      }
    },
    {
      "@type": "Question",
      "name": "[Question]",
      "acceptedAnswer": {"@type": "Answer", "text": "[Answer]"}
    }
  ]
}
```

### Article JSON-LD — enhanced (blog posts)

**`author` default: Organization (shown below) — unless `docs/PROJECT.md`'s "Named content author" field (intake Section N) names a real person who actually writes or reviews the content.** When it does, swap to the `Person` variant underneath. **Never invent a named author** — a `Person` author is only ever as real as what the client confirmed at intake; absent that confirmation, Organization is correct, not a fallback to apologize for.
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "[Post Title]",
  "description": "[Meta description]",
  "image": "https://[domain]/images/blog/[slug].webp",
  "author": {
    "@type": "Organization",
    "name": "[Site Name]",
    "url": "https://[domain]/"
  },
  "publisher": {
    "@id": "https://[domain]/#organization",
    "@type": "Organization",
    "name": "[Site Name]",
    "logo": {"@type": "ImageObject", "url": "https://[domain]/images/logo.webp"}
  },
  "datePublished": "[ISO date]",
  "dateModified": "[ISO date]",
  "about": {"@type": "Thing", "name": "[primary topic]"},
  "mentions": [{"@type": "Thing", "name": "[related topic]"}]
}
```
**`author` — Person variant** (only when a real named author was confirmed at intake). Everything else in the block above is unchanged — only `author` swaps:
```json
  "author": {
    "@type": "Person",
    "name": "[Author Name]",
    "jobTitle": "[Role/credential, e.g. Founder]",
    "sameAs": ["https://www.linkedin.com/in/[handle]"]
  },
```
Drop `sameAs` entirely if no LinkedIn URL was given — never link to a guessed profile. The visible byline on the post (`pages.md` § Blog: "date + author") must show this same name — the rendered page and the schema must agree, the same rule `Breadcrumbs` already follows for `BreadcrumbList`.

### LocalBusiness JSON-LD (contact page — if physical location)
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "[Site Name]",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "[City]",
    "addressRegion": "[State/County]",
    "addressCountry": "[US/GB/etc]"
  },
  "telephone": "[phone]",
  "email": "[email]",
  "url": "https://[domain]/"
}
```

---

## Sitemaps — split index + image sitemap (v9)

`app/sitemap.js` emits a **sitemap index** plus child sitemaps via Next's `generateSitemaps()` — never a single flat file, never hand-written. Children: pages, products, categories, blog. Product entries carry `<image:image>` (Next supports an `images: []` field per entry). See `references/nextjs.md` for the generator.

Output shape (what to aim for — do not hand-write):
- `/sitemap.xml` — index referencing the children
- `/sitemap/0.xml` — static pages (home, shop, about, contact, faq, legal…)
- `/sitemap/1.xml` — products (each with `<image:image>`)
- `/sitemap/2.xml` — categories
- `/sitemap/3.xml` — blog posts

Rules: canonical URLs only; `<lastmod>` on every URL; **exclude** /cart/, /account/, /tracking/, /search, all /thank-you-*/, not-found, and any noindex page; no duplicate URL across children; no redirected or 404 URL. The crosscheck (item 41) validates the index, each child, and the image entries.

---

## robots.txt template

```
User-agent: *
Allow: /

Disallow: /cart/
Disallow: /account/
Disallow: /tracking/
Disallow: /search?
Disallow: /thank-you-contact/
Disallow: /thank-you-order/
Disallow: /thank-you-wholesale/

Sitemap: https://[domain]/sitemap.xml
```

UK/EU sites add: `Disallow: /cdn-cgi/`

---

## Image SEO requirements

```html
<img
  src="/images/[name].webp"
  alt="[Descriptive alt text including keyword where natural]"
  width="[W]"
  height="[H]"
  loading="lazy"
>
```

Hero images and logo: `loading="eager"`

**Format (v9):** every image ships as **AVIF + WebP**. On Vercel, `next/image` negotiates format/responsive sizing automatically. On Cloudflare (static export), the pipeline emits `<slug>.avif` + `<slug>.webp` and markup uses `<picture>` (AVIF `<source>` + WebP `<img>` fallback). Pages call the SmartImage wrapper, never a raw `<img>`. See `references/performance.md` → image handling.

Alt text rules:
- Describe what the image shows
- Include primary or secondary keyword if it fits naturally
- Product images: "[Product Name] — [variant] — [Site Name]"
- Category images: "[Category Name] from [Site Name]"
- Hero images: "[Site Name] [product type] — [hero headline keywords]"

---

## Internal linking standards

Every page links to minimum:
- Homepage (via logo)
- Shop (via nav)
- 2-3 relevant category or product pages (contextual in body copy)
- FAQ or Contact (via CTA or footer)

Blog posts must include:
- 2-3 internal links to relevant product or category pages
- 1 link to another related blog post (when one exists)

Product pages must link to:
- Parent category page (via breadcrumb)
- 3-4 related products (via related products section)

No orphan pages — every page linked from at least one other.

---

## Google Search Console setup checklist

After deploying:
1. Add property → Domain type → verify via DNS TXT record (Cloudflare DNS → add TXT record)
   OR use HTML tag method → GSC verification meta tag already in every page from build
2. Submit sitemap.xml → GSC → Sitemaps → Submit
3. Request indexing for homepage and shop page
4. Check Coverage report after 48h
5. Verify Product rich results live via SKILL.md AUDIT F's Rich Results Test pass, not just assumed from static JSON validity. **FAQ rich results were deprecated by Google in 2026** — FAQPage schema no longer produces a visible snippet in classic Search; keep the markup for AI/LLM extraction value (AEO), not for a Rich Results Test pass on FAQ specifically.
6. Set preferred domain (non-www)

## Bing Webmaster Tools setup
1. Import from GSC (fastest method)
2. Submit sitemap.xml
3. Verify IndexNow key — already in meta tags and [key].txt file from build
4. Once verified, check the AI Performance report (Bing Webmaster Tools) alongside GSC — tracks citation counts in Copilot/Bing AI answers.

**DuckDuckGo:** no separate webmaster program exists — DuckDuckGo sources most organic links from Bing, so the Bing setup above already covers it.

## IndexNow
Key generated during build. Already placed in:
- `<meta name="IndexNow-key" content="[key]">` on every page
- `[key].txt` file at root
Log key in PROJECT.md → Hosting → IndexNow key.

---

## v4 additions — validity traps, freshness, canonicalization, webmaster lag

### JSON-LD validity (MANDATORY — validate every block with a JSON parser)
- BreadcrumbList: ListItem objects MUST be joined with `},{` not `}{`. The `}{` bug makes the whole block invalid and silently kills ALL structured data on the page.
- NO bare `{"@type":"Product","name":...}` on non-product pages. Google flags "Product snippets: 1 invalid item detected" (GSC + Bing). For homepage price ranges use AggregateOffer (offerCount/lowPrice/highPrice/priceCurrency) — never a nested itemOffered Product. Audit homepage for ANY "@type":"Product".
- Real product PAGES keep full Product+Offer (name, image, description, offers w/ price+priceCurrency+availability).

**JSON-parse validity is necessary but not sufficient.** A block can parse as perfectly valid JSON and still fail Google's rich-result eligibility (e.g. missing a field Google requires to actually render the rich snippet, even though schema.org doesn't strictly require it). The static crosscheck only catches parse errors and the known traps above — it cannot confirm eligibility. That requires an actual live run through Google's Rich Results Test post-deploy — see SKILL.md AUDIT F.

### Meta description bands
- Google truncates >160. Bing flags outside ~120-158 (stricter). Target ~150. Keep og:description + twitter:description in sync with the meta description.

### RankMath score-lifters (from 75 → 96 on a real build)
- One H1 per page (hero slider: H1 slide 1 only, others styled divs).
- Plaintext Emails (Security): entity-encode ALL emails (mailto href + visible text), and remove email from JSON-LD too — RankMath scans whole source. Keep phone in schema.
- Minify CSS + JS. Never hand-strip // comment lines from JS (one-line join turns a surviving // into a comment that kills the file). Use `npx terser script.source.js -o script.js --compress --mangle`; keep .source copies. CSS hand-minify safe — verify brace balance.
- Cache-Control headers in _headers for /images/*, *.webp, *.css, *.js (max-age=31536000).
- Complete OG (og:type, og:site_name) + Twitter card tags.
- WWW Canonicalization: _redirects forces www→apex AND http→https with 301!, placed FIRST.
- Content Freshness: <lastmod> on EVERY sitemap URL + og:updated_time on every page + HTML revalidation Cache-Control.
- "Mobile Speed: Test execution failed" = RankMath's own PageSpeed backend timing out, NOT a site defect. Re-run; don't chase in code.

### Webmaster-tool lag (GSC + Bing)
URL-inspection "Index" tabs show the LAST CRAWL, not the live page. After deploy: purge Cloudflare cache, use the LIVE test (GSC "Test Live URL", Bing "Live URL"), then Request Indexing. Don't re-edit correct code chasing a stale cached report. The live test reflects the current file; the index tab catches up on the engine's schedule.
