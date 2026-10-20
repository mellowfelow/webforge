# WebForge — Pages reference (v8)

**Rewritten for React + Next.js App Router.** v5/v6 described hand-written static HTML — that is gone. Pages are React components; meta comes from the metadata API; content comes from `src/config/site.js`.

Read alongside `references/nextjs.md` (patterns) and `references/seo.md` (meta formulas + schema).

---

## The rule that replaces most of this file

**Never hand-write a page a config entry could generate.** Products, categories and posts are data. `page.jsx` + `generateStaticParams` renders all of them. If you find yourself writing a second near-identical page, stop — it belongs in `src/config/site.js`.

Only these are truly bespoke: home, about, contact, wholesale, faq, cart, order, search, the four legal pages, three thank-you pages, `not-found`.

## Base requirements (every page)

Handled ONCE in `src/app/layout.jsx` — never repeated per page:
- `<html lang>` from `SITE.locale`, viewport, `metadataBase: new URL(SITE.url)`
- GSC + IndexNow verification tags
- Announcement bar · sticky nav · footer · chat hub · `webmcp.js` (deferred)
- Fonts (preconnect + display swap), global CSS

Per page, via `generateMetadata` (see `nextjs.md`): title ≤60 · description 110–158 · canonical · OG · Twitter. **Never hand-write meta tags.**

Per page, in the component: exactly one `<h1>` · breadcrumbs (except home) · JSON-LD built as an object and stringified.

## Writing quality (every page that has drafted copy)

**Vary sentence length and structure — flat, uniform-length sentences read as generic regardless of how good the content is.** Mix short and long sentences within a section; don't default to the same subject-verb-object shape three sentences in a row. This is a prose-quality rule, not an SEO mechanic — it sits alongside (not instead of) the keyword/structure rules elsewhere in this file.

**Shift tone to match what each section is doing**, not one flat register for the whole page: a spec/trust-list reads plainly and precisely, a CTA reads more persuasively, an authority section reads more narratively. All of it stays inside the one-line brand voice collected at intake (Section A) — "warm and casual" still varies sentence rhythm and still writes a spec list plainly; it doesn't mean every sentence sounds like a chat message.

This is a writing-quality check, not a visual one — it's the content-side sibling to `design-quality.md`'s rule 8 (dated AI-tells), which only scans the rendered page's visual design (palettes, nav style, typefaces). Flat, uniform prose is the same "reads as generic/templated" failure rule 8 catches visually, just in the copy instead of the layout.

---

## Home (`app/page.jsx`)

Primary keyword in H1. 2–3 secondaries across H2s and body. 800–1200 words of indexable copy.

1. **Hero** — H1 (primary keyword), sub-headline (brand entity statement, condensed), two CTAs: Shop → `/shop/`, secondary → chat deep-link or `/wholesale/`. **Composition (design-quality.md):** deliberate image treatment — full-bleed background + contrast scrim, or a sized `object-fit: cover` image grid with tuned `object-position` so subjects show; min-height ≥70vh desktop / ≥60vh mobile (or a sized container); text legible over imagery.
2. **Trust bar** — 4 items: shipping speed · quality/credential · support channel · payment/discount
3. **Category grid** — every category, linked. **Uniform (design-quality.md):** one shared tile with a single fixed aspect (1:1 or 4:3), image `object-fit: cover`, equal heights, consistent gap, identical label treatment on every tile — never mixed sizes or ragged heights.
4. **Featured products** — 6–8 from `PRODUCTS`, `<ProductCard>`
5. **Authority section** — 600–800 words: brand entity statement, founding, differentiators, real milestones, coverage, order rules. Internal links to categories + blog. **This is the AI-visibility payload** — see `ai-visibility.md`.
6. **Stat strip** — 4 real numbers only (founded, product count, dispatch time, coverage). **Never invent a statistic.**
7. **FAQ** — 4 items from `FAQS`, `<details>`, FAQPage schema
8. **Blog grid** — 3 latest from `POSTS`

**Schema:** `Store`/`Organization` · `WebSite` (+ `SearchAction` **only if `/search/` exists**) · `FAQPage` for the 4 shown.

## Shop (`app/shop/page.jsx`) + Category (`app/shop/[cat]/page.jsx`)

One component, two routes. `generateStaticParams` from `CATEGORIES`.

- H1: shop = "Buy [primary keyword] — Full Range"; category = category name
- Intro 100–150 words with the category keyword
- Category chips (current one active) — links, not JS filters, so each category is a real crawlable URL
- Product grid → `<ProductCard>`
- Closing internal links: buyer's guide, legal/FAQ, wholesale

**Never** put products behind a JS-only filter. Every category is a URL with its own meta, schema and copy.

**Schema:** `BreadcrumbList` (+ `OfferCatalog` on `/shop/`). **No bare `Product` on listing pages.**

## Brand pages (`app/brands/[slug]/page.jsx`) + Brand Hub (`app/brands/page.jsx`) — if Section G enabled

**Individual brand pages nest under the hub route** (`/brands/[slug]/`, e.g. `/brands/sur-ron/`) — matching keyword-engine's URL convention exactly (`page-mapping.md` § Brand Hub, `ai-visibility.md`'s worked example). Never a separate top-level `/brand/` route. One component, `generateStaticParams` from the brands present in `PRODUCTS`. Each: H1 = brand name, short brand intro (50–150 words), product grid filtered to that brand, breadcrumb Home > Brands > [Brand]. **Schema:** `Brand` + `BreadcrumbList`.

**Brand Hub** (`/brands/`) exists only when the site stocks 3+ brands — a single aggregator page linking to every individual brand page, grid layout matching the category-grid uniformity rule, anchored Informational/Navigational with lighter treatment like the shop page (never built "conditional-pending" below 3 brands — just skipped). Under 3 brands, skip the hub; individual brand pages still exist at `/brands/[slug]/` if Section G said yes.

## Best Sellers (`app/best-sellers/page.jsx`) + New Arrivals (`app/new-arrivals/page.jsx`) — if Section G enabled

Each independent, built only if its own Section G answer was yes. H1 = "Best Sellers" / "New Arrivals". Intro 50–100 words. Product grid pulling from `PRODUCTS` (best-seller flag / most recently added, respectively) — same `<ProductCard>` grid as Shop. **Never let this silently duplicate the Shop page's content** — this is exactly the four-way cannibalisation check keyword-engine's audit mode runs (Shop / Category / Best Sellers / New Arrivals all pulling overlapping products needs each to target a distinct keyword angle, not just a relabeled product list). **Schema:** `BreadcrumbList` (+ `ItemList` if the full set is enumerable).

## Sale / Deals (`app/sale/page.jsx`) — if Section G enabled

H1 = "Sale" / "Deals". Intro 50–100 words. Product grid filtered to `PRODUCTS` currently flagged on-sale (requires a sale-price field). **Three-way cannibalisation check** (keyword-engine's audit mode, mirroring the Best Sellers/New Arrivals four-way check above) — against the homepage/shop and the relevant category page(s), since all three can end up listing the same discounted products under overlapping keyword angles. **Active-Conditional per keyword-engine's Guarantee Flag model** — if the sale ends and the page goes dormant/empty, either redirect it or keep it live with an "no current sale — check back" state; never leave it 404ing or indexable-but-empty. **Schema:** `BreadcrumbList` (+ `OfferCatalog` for the discounted set).

## Product (`app/product/[slug]/page.jsx`)

`generateStaticParams` from `PRODUCTS`.

1. Breadcrumb: Home > Shop > [Category] > [Product]
2. **Gallery** — white 4:3 frame; thumbnails only when `images.length > 1`; primary `loading="eager"` (LCP), rest lazy. No lightbox library.
3. H1 product name · price · full description
4. `<QtyStepper>` + Add to cart + View cart
5. Trust list — dispatch, free-ship threshold, discount, min order (from `SITE`)
6. Compliance note if the vertical needs one (see `niche-variants.md`)
7. Related products — same category, 3
8. Closing links to guide + wholesale

**Schema:** exactly one `Product` (+ `Offer`, full `images` array, `AggregateRating` **only if reviews are real** — client-provided or migrated; never invented) · `BreadcrumbList`.

## Blog (`app/blog/page.jsx`, `app/blog/[slug]/page.jsx`)

Index: H1, lead, post cards (date, title, excerpt, read link).
Post: H1 (cluster keyword), date + author (`SITE.author.name` if set, else `SITE.brand` — same value the Article schema's `author` field uses, see `seo.md` § Article JSON-LD), `body` array rendered as H2/P (plus a `<table>`/`<ul>`/`<ol>` wherever the content is list-shaped — see below), closing CTA to a category.

800–1500 words. Internal links to products/categories in-body — not just at the end. Topics come from `docs/blog-plan.md`'s build queue — take the next `Pending` row in priority order, then flip it to `Built` once published. (`docs/keyword-map.md` carries page-level keyword targets only; it does not track the blog queue.)

**Each H2 section answers its own heading in the first sentence, then expands.** Same front-loading principle the FAQ answers already use (`ai-visibility.md` AEO section) — applied to blog body sections too: a reader or an AI assistant pulling a passage out of context gets the actual answer immediately, not after a paragraph of lead-in. Write the heading as the real question it's answering where natural ("How long does a leather leash last?" beats "Durability"), and open that section with the direct answer.

**Comparison, spec, and step-by-step content renders as a real `<table>` or `<ul>/<ol>`, never a prose paragraph describing the same thing.** If a section is inherently list-shaped — "X vs Y," a spec sheet, a numbered process — write it as the structural element, not as sentences that just happen to mention several items in a row. This is what both search and AI-answer extraction pull most reliably; a paragraph that goes "first you do X, then Y, then Z" should be a numbered list instead.

**Schema:** `Article` (`datePublished`, `dateModified`, publisher, `mainEntityOfPage`) · `BreadcrumbList`.

## Search (`app/search/page.jsx`) — v8

Client component. Filters `PRODUCTS` + `POSTS` in memory (already in the bundle — no backend). Matches name, short, category. Reads `?q=`. Empty state links to `/shop/`.

Exists so `SearchAction` is honest and agents have a real endpoint (Google's sitelinks searchbox died Nov 2024 — this is not an SEO play). Nav entry when `PRODUCTS.length > 12`.

## About (`app/about/page.jsx`)

H1 · lead (brand entity statement) · story 300–500 words · milestones timeline (**real only**) · differentiators grid · who we supply · coverage · contact.

**Schema:** `AboutPage` with `mainEntity` = the Organization object.

## Contact · Wholesale · Order

All use `<WebForm>` (see `forms.md`). Never hand-roll a form.

- **Contact** — name, email, message → `/thank-you-contact/`
- **Wholesale** (if enabled) — tier table, how it works, company/name/email/phone/requirements → `/thank-you-wholesale/`
- **Order** — chat deep-link with the cart pre-composed FIRST (converts better on mobile), divider, then the form: name, email, phone, address, payment select, order textarea prefilled from cart → `/thank-you-order/`

Always show an entity-encoded email as a fallback near the form.

## Cart (`app/cart/page.jsx`)

Client. `localStorage` read in `useEffect` — never at module scope.

Rows: image · name · unit price · `<QtyStepper>` · line total · **remove button**. Plus: undo-remove, empty-cart with confirm, discount toggle, totals (subtotal → discount → shipping → total), free-ship hint, **min-order block** disabling checkout below `SITE.minOrder`.

Empty state links to `/shop/`.

## FAQ (`app/faq/page.jsx`)

8–12 questions from `FAQS`, `<details>` accordion, `FAQPage` schema covering **all** rendered questions. Answers are self-contained and quotable — see `ai-visibility.md` (AEO).

## Page-level mini-FAQ (product · category · brand · blog · best sellers/new arrivals · sale · compare · wholesale)

**Not the homepage/FAQ-page content above — a separate, smaller FAQ block on every other mapped page that has question-intent keywords in its cluster** (keyword-engine's Phase 7 Page-Level FAQ, `content-planning.md` § Page-Level FAQ). 2–4 questions, `<details>` accordion matching the dedicated FAQ page's style, placed after the page's main content (e.g. after Related Products on a product page, after the category grid on shop/category). Its own `FAQPage` schema instance — never merged into the homepage/FAQ-page schema block, and never a duplicate of a question already answered elsewhere (link to it instead). Skip only when a page's cluster genuinely has nothing to draw on — never invent filler questions. Sourced from `keyword-map.md`'s own Per-Page "Page-Level FAQ" field when keyword-engine ran (not `faq-bank.md` — that file covers homepage/FAQ-page/blog content only); drafted from the page's own obvious buyer questions when it didn't (e.g. a product page almost always has a sizing/compatibility question even with no keyword-engine data).

## Journey-Stage internal linking (cross-cutting — applies to blog + every Purchase-stage page)

**keyword-engine assigns every mapped page a Journey Stage** (`keyword-map.md`'s per-page field: Awareness/Consideration/Purchase/Loyalty) **and mandates bidirectional linking between stages** (`page-mapping.md` § Journey-Stage Internal Linking) — this is the build-side rule that actually wires those links into the rendered pages, so the field doesn't just sit in the doc unused.

- **Purchase-stage pages** (Category/Subcategory, Product, Shop, Sale, Wholesale, Best Sellers, New Arrivals) are the link *targets*.
- **Awareness/Consideration-stage blog posts** are the link *source*: when building a post from `blog-plan.md`, read the Journey Stage + "funnels toward" page(s) `keyword-map.md` assigns it, and add an in-body internal link to that specific Purchase-stage page (category/product), not just a generic "shop now" CTA — this is on top of the existing "2-3 internal links to products/categories in-body" rule in the Blog section above, same links, now driven by that field when it exists.
- **The Purchase-stage page links back**: the category/product page's "closing internal links" (already called out per page type above — "buyer's guide, legal/FAQ, wholesale" on Shop/Category, "guide + wholesale" on Product) gains the specific blog post(s) that funnel toward it, not just a generic guide link, when `keyword-map.md` names one.
- **Loyalty-stage content** (care/maintenance/accessory posts, FAQ) links forward the same way but is additive — it doesn't replace the Awareness/Consideration linking above.
- **No keyword-engine docs for this project:** skip this section — there's no Journey Stage field to read. Fall back to the existing generic internal-linking rules per page type (unchanged).
- **A missing link beats a broken one:** if `keyword-map.md` names a target page that doesn't exist on this build (e.g. a conditional page type Section G said no to), skip that link rather than point at a 404 — don't invent the page just to satisfy it.

## Legal — shipping · refund · privacy · terms

Real content, not boilerplate. Restate the actual numbers from `SITE` (thresholds, windows, methods). Terms carries the vertical's permitted-use and compliance language (`niche-variants.md` + the site's compliance block). Reference the correct consumer law for `SITE` country.

## Thank-you × 3

`robots: { index: false, follow: true }`, excluded from sitemap, disallowed in robots.txt. Confirmation + what happens next + CTA back to shop + chat link.

## `not-found.jsx`

Real content, `noindex`, links to `/shop/` and `/`.

---

## Components

| Component | Type | Notes |
|---|---|---|
| `Nav` | server + client island | mega dropdown, hamburger ≤700px, cart badge via `cart-change` event |
| `ProductCard` | client | white 4:3 frame, badge, `<QtyStepper>`, add-to-cart, qty resets after add |
| `QtyStepper` | client | − / typed number / + · min 1 max 999 · `aria-label` per button |
| `WebForm` | client | provider logic per `forms.md` |
| `ChatHub` | client | renders `CHAT.channels`; single link channel → direct button |
| `Breadcrumbs` | server | **must mirror `BreadcrumbList` exactly** — visible and structured must agree |
| `Footer` | server | brand + shop + company + legal, compliance line, entity-encoded email |

## Mobile (every page)

Design and verify at **~380px first**. Zero horizontal scroll is a release blocker.

```css
html,body{overflow-x:hidden;max-width:100vw}
*{box-sizing:border-box;min-width:0}
img,video,iframe{max-width:100%;height:auto}
@media(max-width:600px){
  [style*="grid-template-columns"]{grid-template-columns:1fr!important}
  [style*="display:grid"]{display:block!important}
}
```
Columns 4→2→1 · hamburger ≤700px · tap targets ≥44px · sticky add-to-cart on mobile PDP · footer single column · `prefers-reduced-motion` respected.

**Read `references/mobile-native.md` for the platform-level layer** — tap highlight, the `dvh`/`svh`
height bug, input zoom, safe-area insets, overscroll, and the full "never ship" checklist. The rules
above are the floor; that file is what separates "survives on a phone" from "feels installed."

## Accessibility

`aria-label` on every icon-only button (cart, hamburger, close, qty ±, remove) · qty/remove buttons `type="button"` · `aria-hidden` on decorative icons · visible focus rings · labels tied to inputs · `aria-live` on cart totals.

---

## Removed in v8 — do not reintroduce

| v5/v6 | Why gone |
|---|---|
| Hand-written `<head>` per page | metadata API |
| Formspree / EmailJS | superseded — see `forms.md` |
| Hero **slider** | 4 slides = 4 LCP candidates and an H1 trap. **One hero, one H1.** |
| Quick-view modals | a real product page is better for SEO, agents and users |
| Newsletter bar | only if the client actually has a list |
| Sidebar price-slider / JS filters | category URLs are crawlable; sliders are not |
| Square 1:1 product frames | landscape products cap at ~66% fill — v8 uses white 4:3 |
| `keyword-cluster.txt` in the site root | leaked the strategy publicly → now `docs/keyword-map.md`, never deployed |
