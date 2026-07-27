# WebForge — Next.js reference (v8)

Read when building or updating any page. **React + Next.js App Router. Never hand-roll a prerenderer.**

## The contract

`src/config/site.js` is the single source of truth. Every route, meta tag, JSON-LD block, sitemap entry and nav link **derives** from it. Adding a product = one object. Never hand-write a page that a config entry could generate.

```js
export const SITE = {
  brand: 'Brand Name',
  tagline: '…',
  domain: 'DOMAIN.com',              // ← THE ONLY PLACE A DOMAIN APPEARS
  get url() { return `https://${this.domain}` },
  target: 'vercel',                  // 'vercel' | 'static'  → drives generation + Rule 10
  email: 'info@DOMAIN.com', ordersEmail: '…', wholesaleEmail: '…',
  currency: 'AUD', locale: 'en-AU',
  minOrder: 250, freeShipOver: 500, flatShip: 20,
  founded: '2022', foundedPlace: 'Sydney, Australia',
  gscCode: '…', indexNowKey: '…',
  payments: ['…'],
}
export const FORMS = { provider: 'web3forms', web3formsKey: '…', resendFrom: '', turnstileSiteKey: '' }
export const CHAT   = { channels: [{ type: 'whatsapp', value: '61400000000' }] }
export const CATEGORIES = [{ slug, name, kw, desc }]
export const PRODUCTS   = [{ slug, name, price, cat, badge, short, desc, images: [] }]
export const POSTS      = [{ slug, title, date, kw, excerpt, body: [['h2','…'],['p','…']] }]
export const FAQS       = [{ q, a }]
```

`SITE.url` as a getter means changing `domain` propagates everywhere with zero find-and-replace.

## Static params

Every dynamic route must enumerate itself, or static export fails:

```jsx
// src/app/product/[slug]/page.jsx
import { PRODUCTS } from '@/config/site'
export function generateStaticParams() {
  return PRODUCTS.map(p => ({ slug: p.slug }))
}
```
Same for `shop/[cat]` (from `CATEGORIES`) and `blog/[slug]` (from `POSTS`).

## Metadata — use the API, never hand-write tags

```jsx
export async function generateMetadata({ params }) {
  const p = PRODUCTS.find(x => x.slug === params.slug)
  if (!p) return {}
  const url = `${SITE.url}/product/${p.slug}/`
  return {
    title: `${p.name} | ${SITE.brand}`.slice(0, 60),
    description: p.short.slice(0, 158),
    alternates: { canonical: url },
    openGraph: { title: p.name, description: p.short, url, images: [`${SITE.url}/images/${p.images[0]}`], type: 'website' },
    twitter: { card: 'summary_large_image' },
  }
}
```
- **Titles ≤60 chars. Descriptions 110–158.** The crosscheck enforces both.
- Thank-you pages: `robots: { index: false, follow: true }`.
- `metadataBase: new URL(SITE.url)` in the root layout so relative OG images resolve.

## JSON-LD

Server components only — a `<script type="application/ld+json">` with `dangerouslySetInnerHTML`:

```jsx
<script type="application/ld+json"
  dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }} />
```
Build the object in JS and `JSON.stringify` it — never template a JSON string by hand. That is how invalid schema ships. The crosscheck parses every block.

Rules: exactly one `Product` per product page, none elsewhere. `BreadcrumbList` on every non-home page. `FAQPage` only where FAQs actually render. `Organization`/`Store` once, on home. **`SearchAction` only if `/search/` exists** (Rule 10).

## sitemap.js / robots.js

Next natives — they read `SITE.domain` and work in both targets. **v9: split via `generateSitemaps()` into an index + children, with `<image:image>` on product entries.**

```js
// src/app/sitemap.js
import { SITE, PRODUCTS, CATEGORIES, POSTS } from '@/config/site'

export async function generateSitemaps() {
  // one child per section -> /sitemap/0.xml (pages), 1 (products), 2 (categories), 3 (blog)
  return [{ id: 0 }, { id: 1 }, { id: 2 }, { id: 3 }]
}

export default function sitemap({ id }) {
  const now = new Date()
  if (id === 1) {
    return PRODUCTS.map(p => ({
      url: `${SITE.url}/product/${p.slug}/`,
      lastModified: now,
      images: (p.images || []).map(img => `${SITE.url}/images/${img}`), // <image:image>
    }))
  }
  if (id === 2) return CATEGORIES.map(c => ({ url: `${SITE.url}/shop/${c.slug}/`, lastModified: now }))
  if (id === 3) return POSTS.map(p => ({ url: `${SITE.url}/blog/${p.slug}/`, lastModified: now }))
  // id === 0 -> static pages (exclude every noindex page: thank-you-*, not-found, cart, account, tracking, search)
  const pages = ['', '/shop/', '/blog/', '/about/', '/contact/', '/faq/', '/shipping/', '/refund/', '/privacy/', '/terms/']
  return pages.map(r => ({ url: `${SITE.url}${r}`, lastModified: now }))
}
```
Next emits `/sitemap.xml` as the index automatically when `generateSitemaps` is used, and `/sitemap/[id].xml` for each child. **Exclude every `noindex` page.** On static export, confirm the child files exist in `out/` (crosscheck item 36). Robots references the index at `/sitemap.xml`.

## Static export vs Vercel

| | `TARGET=static` (Cloudflare) | default (Vercel) |
|---|---|---|
| `output` | `'export'` → `out/` | server build |
| `src/app/api/**` | **dropped from the build** | live routes |
| `resend` forms | **impossible** — crosscheck B3 | supported |
| MCP / JSON API | not present; `.well-known` must not claim them (B2) | present |

```js
// next.config.mjs
const isStatic = process.env.TARGET === 'static'
export default {
  output: isStatic ? 'export' : undefined,
  trailingSlash: true,
  images: isStatic
    ? { unoptimized: true }                       // Cloudflare: pre-optimised AVIF+WebP pipeline
    : { formats: ['image/avif', 'image/webp'] },  // Vercel: runtime optimisation
}
```
**`trailingSlash: true` on both.** Every route becomes a directory with `index.html`; every internal link ends in `/`. Mixing styles splits canonicals and 301-bounces on Vercel.

**Images are target-conditional (v9).** Vercel uses `next/image` runtime optimisation (`formats: ['image/avif','image/webp']`, responsive `srcset`, auto lazy). Cloudflare static export can't run the optimiser, so it keeps `unoptimized: true` and serves pre-optimised AVIF+WebP from `scripts/images.mjs` via `<picture>`. Pages never use a raw `<img>` — they call the **SmartImage** wrapper, which renders `next/image` when `SITE.target === 'vercel'` and a `<picture>` (AVIF+WebP) when `SITE.target === 'static'`. See `references/performance.md`.

## Client vs server components

Default to **server**. `'use client'` only for: cart state, quantity steppers, forms, nav toggles, search filter, image gallery. A product page is a server component that renders a small client island for the cart button — never the whole page.

Cart lives in `localStorage` under `<brand>-cart`, with a custom `cart-change` event so the nav badge updates. **Never** use `localStorage` in a server component or during SSR — read it in `useEffect`.

## Component conventions

- `Nav` — server shell + client island for the mobile toggle and cart badge
- `ProductCard` — client (quantity stepper + add to cart)
- `WebForm` — client; provider logic per `references/forms.md`
- `ChatHub` — client; renders `CHAT.channels`
- `Breadcrumbs` — server; mirrors the `BreadcrumbList` schema exactly (visible and structured must agree)
- `SmartImage` — server; renders `next/image` when `SITE.target==='vercel'`, else a `<picture>` (AVIF+WebP). All product/category/hero images go through it — never a raw `<img>`.

## Traps

1. **`generateStaticParams` missing** → export build fails or silently drops routes.
2. **`localStorage` at module scope** → build crashes (no `window` on the server).
3. **A `.jsx` page importing a `'use client'` component is fine.** The reverse (client importing a server component) is not.
4. **Hand-editing generated files** (`llms.txt`, `.well-known/*`, `vercel.json`, `_headers`) → wiped on the next build. Crosscheck item 13 catches the drift. Edit the config.
5. **Absolute URLs in JSON-LD** — always `SITE.url + path`, never a relative path.
