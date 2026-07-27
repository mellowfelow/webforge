# WebForge — Performance reference (v9) — Core Web Vitals + Lighthouse gate

Read when building or auditing any site. Performance targets are **platform-neutral** — they apply to **both** Vercel and Cloudflare. Core Web Vitals are measured by Google in a real browser regardless of host; the target has nothing to do with where the site is deployed.

> **v9 delta.** v8 kept performance targets in `references/cloudflare.md` only, which (a) left Vercel builds ungoverned and (b) used the **retired FID metric** (FID was replaced by INP as a Core Web Vital in March 2024). v9 relocates these here as platform-neutral standards, replaces FID with INP, and adds a measured Lighthouse gate to MODE 4 AUDIT F. The structural half is enforced statically by the crosscheck; the measured half needs a live URL.
>
> **v9.1 delta.** Fonts guidance strengthened from "self-host or preconnect" to **self-host by default**, with an explicit callout that a Google Fonts CSS `@import` is a serial-chain render-blocker distinct from (and worse than) a `<link rel="stylesheet">`. Added explicit LCP-preload guidance for CSS `background-image` heroes, which the browser's preload scanner cannot see until CSS parses. Both came from a live PageSpeed audit where an `@import`-based hero site measured FCP 3.0s / LCP 3.5s (Performance 83) primarily on this chain.

---

## The two halves of performance

- **Structural (Tier 1 — static, pre-ship crosscheck):** things provable from the build output — image format/sizing, minification, deferred scripts, `font-display`, no render-blocking resources, code-splitting, page-weight budget. These gate the build.
- **Measured (Tier 2 — live, AUDIT F):** the actual Lighthouse run and field/lab Core Web Vitals. These need the deployed URL and a headless browser, so they run post-deploy — never from a ZIP.

---

## Targets (both targets, every site)

| Metric | Target | Tier |
|---|---|---|
| Lighthouse — Performance | **≥ 95** | 2 (live) |
| Lighthouse — SEO | **≥ 95** | 2 (live) |
| Lighthouse — Accessibility | **≥ 95** | 2 (live) |
| Lighthouse — Best Practices | **≥ 95** | 2 (live) |
| LCP (Largest Contentful Paint) | **< 2.5s** | 2 (live) |
| CLS (Cumulative Layout Shift) | **< 0.1** | 2 (live) |
| INP (Interaction to Next Paint) | **< 200ms** | 2 (live) |
| Total homepage weight | **< 1.5MB** | 1 (static) |
| Individual image weight | **< 150KB** (hero may be larger, still optimise) | 1 (static) |

> INP replaced FID as a Core Web Vital in March 2024. Do not reference FID anywhere.

---

## Structural standards (Tier 1 — enforced by crosscheck)

Every build must satisfy these, provable from `out/` (static) or the Vercel build:

- **Images:** AVIF + WebP (see image handling below); correct `width`/`height` (or `next/image` intrinsic sizing) so **CLS contribution from images is 0**; `loading="lazy"` below the fold, `eager` for the LCP image; responsive sizing.
- **CSS minified**, JS terser-minified + `defer`; keep `.source` copies (never hand-strip `//` comment lines — a one-line join can comment out the file).
- **No render-blocking resources:** no synchronous third-party scripts in `<head>`; chat widgets load deferred (after first interaction or 3s idle); analytics deferred.
- **Fonts: self-host by default.** `font-display: swap`, subset to latin/latin-ext where the site is English-only. **Never a CSS `@import` for a font stylesheet** — `@import url('https://fonts.googleapis.com/...')` is strictly render-blocking in serial: the browser must download and parse the main CSS file before it even discovers the `@import`, then fetch and parse *that* CSS before it discovers the actual font file — three sequential round-trips before a single character can paint. On Vercel, `next/font` self-hosts automatically and is the default. On a manual/static build, fetch the Google Fonts CSS2 API response once at build time, download the referenced woff2 files to `public/fonts/` (check for repeated URLs across weight/stretch declarations first — a variable font family usually needs only 2-4 unique files for latin + latin-ext, not one per weight), and write local `@font-face` rules. This also removes `fonts.googleapis.com`/`fonts.gstatic.com` from CSP entirely. A `<link rel="stylesheet">` in `<head>` (parallel-discoverable) beats `@import` if self-hosting genuinely isn't possible, but treat that as the fallback, not the default.
- **Preload the LCP element.** `<link rel="preload" as="image" fetchpriority="high">` for the hero/above-the-fold image on its page — this matters most when the hero is a CSS `background-image` rather than an `<img>` tag, because the browser's preload scanner parses raw HTML before it parses CSS, so a background-image hero is invisible to it until the stylesheet loads; an explicit preload closes that gap. Also preload the 1-2 font files actually used above the fold (hero heading + body text) with `<link rel="preload" href="/fonts/x.woff2" as="font" type="font/woff2" crossorigin>` — the rest of the font set can load normally.
- **Code-splitting:** Next.js route-level splitting (automatic); no giant client bundles — server components by default, `'use client'` only where required (cart, forms, steppers, search, gallery, nav toggle).
- **Caching headers:** immutable long-cache for `/images/*`, `*.webp`, `*.avif`, `*.css`, `*.js`; HTML short-cache/revalidate. (Generated into `vercel.json` or `_headers` per target.)
- **Page-weight budget:** homepage < 1.5MB transferred; flag the hero image first if over (most common culprit).

## Measured gate (Tier 2 — AUDIT F, needs the deployed URL)

Run Lighthouse (CI or PageSpeed Insights) against the deployed URL, mobile profile, and assert all four category scores ≥ 95 and the three Core Web Vitals within target. If a score misses, triage in this order:
1. **LCP:** hero image weight/format, render-blocking CSS/fonts, server response time.
2. **CLS:** missing image dimensions, injected banners/ads, late-loading fonts (mitigated by `font-display: swap` + preconnect).
3. **INP:** heavy client JS on interaction, unsplit bundles, long tasks from third-party widgets.
4. **Best Practices/SEO:** console errors, missing meta, non-HTTPS subresources, invalid structured data.

Tooling: Lighthouse CI or the PageSpeed Insights API. This cannot run from a ZIP — it is a **post-deploy** step. Report scores; auto-fix is limited to structural regressions (Tier 1) — CWV regressions are reported with the triage above, not silently "fixed."

---

## Image handling — target-conditional (v9 decision)

Vercel is the default workflow; Cloudflare is retained for existing/legacy static sites. Image optimisation now differs by target, because `next/image`'s runtime optimiser cannot run on a static export.

| | **Vercel** (default) | **Cloudflare** (static export, legacy) |
|---|---|---|
| Mechanism | `next/image` `<Image>` — runtime optimisation | pre-optimised files from `scripts/images.mjs` |
| `next.config` images | `{ formats: ['image/avif','image/webp'] }` | `{ unoptimized: true }` |
| Formats served | AVIF → WebP fallback, responsive `srcset`, auto lazy | AVIF + WebP emitted by the pipeline, served via `<picture>` |
| CLS | width/height (or `fill` + sized container) | width/height on `<img>` |

**Both targets ship AVIF.** On Vercel it is automatic via `next/image`. On Cloudflare the pipeline (`scripts/images.mjs`) emits `<slug>.avif` **and** `<slug>.webp`, and the markup uses `<picture>` with an AVIF `<source>` + WebP `<img>` fallback. This keeps existing Cloudflare sites AVIF-capable for audits without a runtime optimiser.

Implementation note: use one wrapper component so pages don't branch. It renders `next/image` when `SITE.target === 'vercel'` and a `<picture>` (AVIF+WebP) when `SITE.target === 'static'`. Pages and cards call the wrapper, never a raw `<img>`.

> The v8 rationale for `unoptimized: true` on both targets was byte-parity between targets. With Vercel as the primary workflow that parity is no longer a goal, so Vercel switches to real optimisation. Cloudflare keeps the pipeline for the existing sites.

## Reference pipeline — `scripts/images.mjs` (uniform, clean, budget-safe)

The pipeline that produces uniform cards from inconsistent client photos:
**trim whitespace → scale to fill ~90% (enlarging small sources) → sharpen only
when upscaling → adaptive quality under the 150 KB budget → AVIF + WebP.**
This is what stops small or heavily white-padded sources from floating tiny in
the frame. Copy verbatim for static/Cloudflare (Vercel uses `next/image`, but the
same trim/fill/quality logic is still worth running to normalise sources).

```js
import sharp from 'sharp'
import { readdirSync, existsSync, mkdirSync, statSync, writeFileSync } from 'node:fs'
import { resolve, dirname, basename, extname } from 'node:path'
import { fileURLToPath } from 'node:url'

const root = resolve(dirname(fileURLToPath(import.meta.url)), '..')
const srcDir = resolve(root, 'assets/product-photos')
const outDir = resolve(root, 'public/images')
const photos = existsSync(srcDir)
  ? readdirSync(srcDir).filter((f) => /\.(jpe?g|png|webp|avif|tiff?)$/i.test(f)) : []
if (!photos.length) { console.log('images: none — using placeholder.'); process.exit(0) }
mkdirSync(outDir, { recursive: true })

const W = 1600, H = 1200, FILL = 0.90, CAP = 145 * 1024
const sleep = (ms) => new Promise((r) => setTimeout(r, ms))
// Windows AV can briefly lock a freshly written file — retry the write.
const writeRetry = async (out, buf) => {
  for (let i = 0; ; i++) { try { writeFileSync(out, buf); return }
    catch (e) { if (i >= 8) throw e; await sleep(150) } }
}

for (const file of photos) {
  const slug = basename(file, extname(file)).toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/(^-|-$)/g, '')
  const src = resolve(srcDir, file)
  const meta = await sharp(src).rotate().metadata()
  let buf = await sharp(src).rotate().toBuffer()

  // 1) trim uniform border, guarded against collapsing the image
  let tw = meta.width, th = meta.height
  try {
    const t = await sharp(buf).trim({ threshold: 12 }).toBuffer({ resolveWithObject: true })
    if (t.info.width >= meta.width * 0.12 && t.info.height >= meta.height * 0.12) {
      buf = t.data; tw = t.info.width; th = t.info.height
    }
  } catch { /* no uniform border */ }

  // 2) scale to fill ~90% (enlarge small sources); 3) sharpen only when upscaling
  const scale = Math.min((W * FILL) / tw, (H * FILL) / th)
  let inner = sharp(buf).resize(Math.round(W * FILL), Math.round(H * FILL), { fit: 'inside', kernel: 'lanczos3' })
  if (scale > 1.1) inner = inner.sharpen({ sigma: 1 })
  const innerBuf = await inner.toBuffer()

  const canvas = sharp({ create: { width: W, height: H, channels: 4, background: { r: 255, g: 255, b: 255, alpha: 1 } } })
    .composite([{ input: innerBuf, gravity: 'centre' }]).flatten({ background: { r: 255, g: 255, b: 255 } })

  // 4) adaptive quality → buffer → single write
  const encode = async (fmt, out, qStart, qFloor) => {
    let outBuf, q
    for (q = qStart; ; q -= 6) { outBuf = await canvas.clone()[fmt]({ quality: q }).toBuffer()
      if (outBuf.length <= CAP || q <= qFloor) break }
    await writeRetry(out, outBuf); return q
  }
  await encode('webp', resolve(outDir, `${slug}.webp`), 88, 40)
  await encode('avif', resolve(outDir, `${slug}.avif`), 62, 40)
  if (Math.min(tw, th) < 500) console.log(`  low-res source (reshoot): ${file}  (${tw}x${th}, up ${scale.toFixed(1)}x)`)
}
```

Crosscheck still asserts: every non-hero raster ≤150 KB, AVIF sibling for every
WebP, SmartImage (never raw `<img>`). The adaptive-quality loop is what keeps
detailed/upscaled images under the 150 KB gate without hand-tuning per image.
