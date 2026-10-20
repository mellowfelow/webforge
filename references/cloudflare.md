# Cloudflare Pages — Deploy Configuration (v9)

> **v9 delta — read first.**
> - **Cloudflare is now one of two targets, not the default.** Vercel is default (`references/vercel.md`). Cloudflare remains **first-class** — for clients already on Cloudflare, ZIP-upload workflows, anyone who doesn't want Git, and **any site with backend = Yes**.
> - **Choosing Cloudflare does NOT imply a backend.** Cloudflare + backend = No is a plain static site. Only the reverse is forced: backend = Yes → Cloudflare.
> - **The site is a Next.js static export**, not hand-written HTML. `npm run build:static` (`TARGET=static` → `output:'export'`) → `out/` → zip the CONTENTS of `out/`.
> - **`_headers` and `_redirects` are GENERATED** from `SITE.domain` by `scripts/gen-agent-files.mjs`. Never hand-edit them — crosscheck item 13 fails on drift. The templates below show the OUTPUT to aim for.
> - **`src/app/api/**` does not exist in an export build.** No MCP server, no `resend` forms. `.well-known/*` must not claim these (crosscheck B2/B3). **Forms and the Reply Portal now run on Cloudflare too, via Pages Functions** (`functions/api/**`) — a separate directory Cloudflare deploys as edge functions alongside the static export, not the Next.js API routes that `output:'export'` drops. See "Forms & Reply Portal (Pages Functions)" below and `references/forms.md` / `references/reply-portal.md`.
> - **Agent ceiling ~79** on the free plan. Accepted. Functions exist for forms/portal correctness, not to chase agent-readiness points.
>
> **ZIP handover — say all three, every time:**
> 1. Extract it. GitHub/hosts do not unpack ZIPs.
> 2. Contents go at the ROOT — not inside a wrapper folder.
> 3. Browsers silently skip dotfolders on drag-upload, so `.well-known/` will be MISSING. Cloudflare's ZIP upload handles it; a browser drag-upload to GitHub does not.

---

## _headers file (place at project root)

This file sets security headers and cache policies for all Cloudflare Pages deployments.

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  X-XSS-Protection: 1; mode=block

/images/*
  Cache-Control: public, max-age=31536000, immutable

/styles.css
  Cache-Control: public, max-age=86400, must-revalidate

/script.js
  Cache-Control: public, max-age=86400, must-revalidate

/*.html
  Cache-Control: public, max-age=3600, must-revalidate

/
  Cache-Control: public, max-age=3600, must-revalidate
```

For UK/EU sites, add Content-Security-Policy after testing (don't add blindly — test first):
```
/*
  Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com; connect-src 'self' https://formspree.io https://api.emailjs.com; frame-ancestors 'none'
```
Adjust connect-src based on actual third-party services used.

---

## _redirects file (place at project root)

```
# www to non-www (primary pattern)
https://www.[domain]/* https://[domain]/:splat 301

# Common legacy URL patterns — add as needed
/home /  301
/index.html / 301
/shop/index.html /shop/ 301

# 404 fallback
/* /404.html 404
```

For sites migrating from another domain:
```
https://[old-domain]/* https://[new-domain]/:splat 301
```

---

## Cloudflare Pages project setup

### Build settings (static sites — no build step)
```
Build command:        (leave blank)
Build output directory: /
Root directory:       (leave blank)
```

### Environment variables (set in Cloudflare dashboard if needed)
```
NODE_VERSION: 18
```
(Only needed if adding a build step later)

### Custom domain setup
1. Cloudflare Pages dashboard → your project → Custom domains
2. Add domain → enter domain name
3. If domain is already on Cloudflare DNS: auto-configured
4. If not: add CNAME record pointing to [project].pages.dev

### DNS records (if domain is on Cloudflare)
```
Type: CNAME
Name: @ (root)
Target: [project-name].pages.dev
Proxy: ON (orange cloud)

Type: CNAME
Name: www
Target: [project-name].pages.dev
Proxy: ON (orange cloud)
```

### SSL/TLS settings
- SSL mode: Full (strict)
- Always Use HTTPS: ON
- HSTS: Enable (after confirming site works on HTTPS)
- Minimum TLS version: TLS 1.2

### Speed settings (Cloudflare dashboard)
- Auto Minify: HTML ✓ · CSS ✓ · JavaScript ✓
- Brotli compression: ON
- Rocket Loader: OFF (can break inline scripts — test before enabling)
- Early Hints: ON

### Caching rules
- Browser Cache TTL: Respect existing headers
- Cache Level: Standard

---

## Deploy checklist (run before going live)

### Files present
- [ ] index.html
- [ ] styles.css
- [ ] script.js
- [ ] robots.txt
- [ ] sitemap.xml
- [ ] _headers
- [ ] _redirects
- [ ] All pages/ subdirectory pages
- [ ] All images in /images/ as BOTH .avif and .webp (served via <picture>)
- [ ] PROJECT.md
- [ ] keyword-map.md
- [ ] blog-plan.md (plus faq-bank.md / product-gaps.md if keyword-engine produced them)

### Cloudflare config
- [ ] Project created in Cloudflare Pages
- [ ] Custom domain connected
- [ ] SSL set to Full (strict)
- [ ] Always Use HTTPS = ON
- [ ] Auto Minify enabled
- [ ] www → non-www redirect working

### Post-deploy tests
- [ ] Homepage loads on custom domain
- [ ] All nav links work (no 404s)
- [ ] Shop page loads with products
- [ ] Cart add/remove works
- [ ] Checkout form submits correctly
- [ ] WhatsApp button opens correct chat (if enabled)
- [ ] Mobile: no horizontal scroll at 375px
- [ ] Mobile: hamburger menu works
- [ ] Page speed: run PageSpeed Insights on homepage
- [ ] GSC: submit sitemap
- [ ] BWMT: verify via GSC import
- [ ] Rich Results Test — full pass, all schema types (see SKILL.md AUDIT F / `references/seo.md` § JSON-LD validity), not just a Product+FAQ spot-check

---

## File versioning (cache busting)

When updating CSS or JS files, append a version query string in the HTML:
```html
<link rel="stylesheet" href="/styles.css?v=1.1">
<script src="/script.js?v=1.1"></script>
```
Increment version number with each deploy that changes these files.

---

## Deployment trigger (re-deploy process)

Since these are static sites with no build step:
1. Update files locally
2. Push to GitHub repo connected to Cloudflare Pages → auto-deploys
   OR
3. Drag-and-drop upload in Cloudflare Pages dashboard (for quick single-file updates)
4. Purge Cloudflare cache after deploy: Dashboard → Caching → Purge Everything

---

## Performance targets

**Performance targets are platform-neutral and now live in `references/performance.md`** (they apply to Vercel too — Core Web Vitals are measured in-browser regardless of host). Summary: Lighthouse ≥95 on Performance/SEO/Accessibility/Best-Practices; LCP<2.5s, CLS<0.1, **INP<200ms** (INP replaced FID in March 2024 — do not use FID); homepage <1.5MB; images <150KB. Measured in MODE 4 AUDIT F against the deployed URL.

If score is below target, check:
- Hero image size (most common culprit — compress to < 200KB)
- Render-blocking scripts (move to bottom of body or add defer)
- Unused CSS (audit and remove)
- Google Fonts loading (use font-display: swap)

---

## v4 — Canonicalization + caching + freshness (required)

`_redirects` — put canonicalization FIRST:
```
https://www.DOMAIN.com/* https://DOMAIN.com/:splat 301!
http://www.DOMAIN.com/* https://DOMAIN.com/:splat 301!
http://DOMAIN.com/* https://DOMAIN.com/:splat 301!
# then any legacy redirects…
```

`_headers` — CSP `connect-src` needs only `'self'` for forms now (same-origin Pages Functions, no third-party form API) plus any chat widget origin; add caching + HTML revalidation:
```
/*
  Content-Security-Policy: …connect-src 'self' https://*.jivosite.com https://*.jivochat.com wss://*.jivosite.com wss://*.jivochat.com; …
  Cache-Control: public, max-age=3600, must-revalidate
/images/*
  Cache-Control: public, max-age=31536000, immutable
/*.webp
  Cache-Control: public, max-age=31536000, immutable
/*.css
  Cache-Control: public, max-age=31536000
/*.js
  Cache-Control: public, max-age=31536000
```

Content freshness: every sitemap URL gets `<lastmod>YYYY-MM-DD</lastmod>`; every page gets `<meta property="og:updated_time" content="YYYY-MM-DDT12:00:00+00:00">`.

After deploy: PURGE Cloudflare cache (dashboard → Caching → Purge Everything) so webmaster Live Tests fetch the new version.

---

## Forms & Reply Portal (Pages Functions) — every Cloudflare site now, not opt-in

Unlike the CMS backend below, this is not gated by Section O — **every Cloudflare site needs working contact/order/wholesale forms**, and Section P (Reply Portal) is offered on Cloudflare exactly as it is on Vercel. Both run on the same mechanism: a `functions/` directory at the project root, deployed alongside the static `out/` export as edge functions. No build step, no framework — Cloudflare auto-routes `functions/api/contact.js` to `/api/contact`, `functions/api/admin/orders.js` to `/api/admin/orders`, and so on, matching the same paths the front-end's `fetch()` calls already use on Vercel.

**Setup sequence (forms-only — no Reply Portal):**
1. **Pages project** — Workers & Pages → Create → Pages → Upload assets → drag the WHOLE site folder (must include `functions/`, not just `out/`'s contents — the static export and the Functions directory sit side by side at the project root) → Deploy.
2. **Secrets** — Project → Settings → Variables and Secrets → add `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` (tick Encrypt on each), optionally `SMTP_FROM`, `CONTACT_EMAIL`, `ORDER_EMAIL`, `WHOLESALE_EMAIL`.
3. **Redeploy** — Deployments → latest → ⋯ → Retry deployment. **Secrets only apply to deployments made after they're added** — same rule as D1/R2 bindings below.
4. **Compatibility flag** — Project → Settings → Functions → add `nodejs_compat` under Compatibility Flags, for **both** Production and Preview. This is dashboard-only, same as everything else here: a `wrangler.toml` dropped into the uploaded ZIP is **not** read by a direct/drag-and-drop upload — that file only takes effect via the `wrangler` CLI or a Git-connected Pages project, neither of which this workflow uses. Without this flag set in the dashboard, `cloudflare:sockets` isn't available and the mailer silently can't connect.

**Additional steps when Section P = Yes (Reply Portal):**
5. **KV namespace** — Storage & Databases → KV → Create → name `[site-slug]-kv`.
6. **Binding** — Project → Settings → Bindings → Add → KV namespace → variable name **`REPLY_KV`** (fixed name, matches the code exactly — see `references/reply-portal.md` §2). Also add the `ADMIN_PASSCODE` secret alongside the SMTP ones in step 2.
7. **Redeploy again** — same mandatory-redeploy rule.

This can share the same Pages project as a D1/R2 backend (Section O) — a site can have both a CMS (D1/R2) and a Reply Portal (KV) simultaneously; they use different bindings, and **different UI paths on purpose** — the Reply Portal owns `/admin`, the CMS owns `/cms` (`references/backend.md`). Both are Next.js/static-exported into the same output root, so distinct paths aren't a style choice here, they're what stops one from overwriting the other's files at build time.

Deploy config specific to this: `_headers` → `/admin/*` gets `Cache-Control: no-store`; `robots.txt` → `Disallow: /admin/`. CSP `connect-src 'self'` covers the same-origin Functions calls — no third-party form API origin needed now that Web3Forms is retired.

---

## Backend-enabled sites (only when backend = Yes — see references/backend.md)

Backend sites add D1 + R2 + Pages Functions to the SAME Pages project — still 100% Cloudflare, still dashboard-only (no command line). The full click-by-click guide is generated per build as `CLOUDFLARE-SETUP.md`; this is the canonical sequence it follows:

1. **D1 database** — Storage & Databases → D1 → Create → name `[site-slug]-db`.
2. **Tables** — open the DB → Console → paste all of `schema.sql` → Execute.
3. **R2 bucket** — R2 Object Storage → Create bucket → name `[site-slug]-media`.
4. **Pages project** — Workers & Pages → Create → Pages → Upload assets → drag the WHOLE site folder (includes `functions/` + `cms/`) → Deploy.
5. **Bindings** — project → Settings → Bindings → Add: D1 with variable name `DB`; R2 with variable name `MEDIA` (names must match the code exactly).
6. **Secrets** — Settings → Variables and Secrets → add `SESSION_SECRET` and `ADMIN_SETUP_TOKEN`, both Encrypted.
7. **Redeploy** — Deployments → latest → ⋯ → Retry deployment (bindings/secrets only apply to deployments made after they're added).
8. **Admin login** — visit `[site-slug].pages.dev/cms/setup?token=YOUR_ADMIN_SETUP_TOKEN` once → set email + password → it self-disables.
9. **Custom domain** — Settings → Custom domains → add `[domain]`.

Backend-specific deploy config:
- `_headers`: add `/cms/*` → `Cache-Control: no-store` (never cache the dashboard); keep functions responses uncached. CSP `connect-src 'self'` for the same-origin API.
- `robots.txt`: `Disallow: /cms/` and `Disallow: /api/`. (**Not** `/admin/` — that path belongs to the Reply Portal now, see "Forms & Reply Portal" above. The two are deliberately distinct so a site with both never has one overwrite the other's output.)
- Future updates: re-upload the site folder (Deployments → Create new deployment). D1, R2, bindings, secrets, and all client content persist across redeploys.
- Content (posts/products) is added via `/cms → Import` of a `*-import.json` file, NOT by editing HTML.

Note: bindings take effect only on the deployment made AFTER they're added — the redeploy (step 7) is mandatory, not optional.
