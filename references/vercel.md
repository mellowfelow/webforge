# WebForge — Vercel target reference (v8)

Read when the deploy target is Vercel (the default). Cloudflare → `references/cloudflare.md`.

## The workflow

```
edit src/config/site.js  →  npm run build  →  npm run crosscheck  →  commit  →  push  →  Vercel deploys
```
No ZIP. No upload. The repo IS the deliverable.

## First-time setup (~2 min, once per site)

1. **Empty GitHub repo.** No README, no .gitignore — truly empty, so the first push isn't a merge conflict.
2. **Local:** `git init` → `.gitignore` → commit → `git remote add origin …` → `git branch -M main` → `git push -u origin main`.
3. **Vercel → Add New → Project → import the repo.**
   - **Framework Preset: Next.js** ← get this wrong and the site 404s
   - Build command / output directory: **leave blank** — `vercel.json` and `next.config.mjs` handle it
4. Every push to `main` now deploys automatically.

## vercel.json — generated, never hand-written

`scripts/gen-agent-files.mjs` emits it from `SITE.domain`. Contains:
- `"trailingSlash": true` — **must match `next.config.mjs`**, or Vercel 301-bounces every URL and splits canonicals
- www → apex 301 (`has.host = www.[DOMAIN]`). **http → https is automatic on Vercel** — don't add it. ⚠️ This assumes apex is canonical — **verify that against what Vercel's dashboard actually set as primary before shipping it** (see the redirect-loop warning below). If www is primary instead, flip this rule (or drop it and let Vercel's own domain-level redirect handle it) and set `SITE.domain` to the `www.` form.
- Security headers + CSP
- The RFC 8288 `Link` header on `/*` advertising the agent files
- `Content-Type` + `Access-Control-Allow-Origin: *` for `llms.txt`, `auth.md`, every `.well-known/*`
- `Cache-Control`: `/_next/static/*` immutable, `/images/*` a week, HTML must-revalidate

**Vercel does NOT read `_headers` or `_redirects`.** Those are Cloudflare-only. A site shipped to Vercel with only `_headers` silently loses its entire CSP, cache policy and agent `Link` header, and nothing warns you. Crosscheck item 14 catches it.

## Environment variables

| Var | When | Where |
|---|---|---|
| `RESEND_API_KEY` | `provider: 'resend'` | Vercel → Settings → Environment Variables. **Never in the repo** (crosscheck B5). |
| `TARGET` | building for Cloudflare | local/CI only — never set on Vercel |

Anything reaching the browser needs `NEXT_PUBLIC_`. SMTP credentials, `ADMIN_PASSCODE`, and `RESEND_API_KEY` are all server-only secrets — never `NEXT_PUBLIC_`, never in `src/config/site.js`.

## Connecting a domain later

The normal path — sites are usually built before the domain exists.

1. Vercel → Settings → Domains → add it → set the DNS records it shows.
2. Add the apex **and** `www` (the www→apex 301 needs www to resolve).
3. In `src/config/site.js`: `domain: 'DOMAIN.com'` → `domain: 'realdomain.com'`.
4. `npm run build && npm run crosscheck` → commit → push.
5. Every generated file — `robots.txt`, `llms.txt`, all `.well-known/*`, sitemap, canonicals, OG, `vercel.json` — is now correct.

**That is the entire procedure.** No find-and-replace. If you're editing a domain in more than one file, the build is broken — fix the generator.

> ⚠️ **Redirect-loop trap, confirmed live.** Vercel's own Domains dashboard applies its own apex↔www redirect at the platform level, independent of `vercel.json`, and which direction it picks depends on how the domain was added (often www ends up primary, not apex — the opposite of this skill's default assumption). If `vercel.json` ALSO carries a `redirects` rule for the same pair but pointing the other way, the two fight: apex → www (dashboard) → apex (your rule) → www (dashboard) → … an infinite 308 loop. Page routes can look fine for a while because the CDN edge-caches a few successful responses (`/`, `/shop/`) from before the conflict, while every asset and every uncached path (images, JS, CSS, `robots.txt`, `.well-known/*`) loops and fails — this reads as "random" breakage if you're only spot-checking the homepage. **Before shipping the `redirects` rule, check the actual live behavior**: `curl -sI -X GET https://[apex]/ | grep -i location` and the same for `https://www.[domain]/` — whichever one 200s directly (not a 308) is what Vercel actually treats as primary. Match `SITE.domain` and the `vercel.json` redirect direction to that, don't assume apex. If Vercel's dashboard is already redirecting one way, the simplest fix is often to delete the `redirects` array entirely and let the dashboard's own redirect be the only one.

Then: GSC verify → submit sitemap → Bing → test all three forms → `isitagentready.com/[domain]`.

## API routes (live agent layer)

`src/app/api/**/route.js`. Present on Vercel, **dropped by `output: 'export'`**. See `references/agent-live.md`.

Runtime: default (Node). Only use `export const runtime = 'edge'` if a route needs global low latency — the catalog routes don't, and edge complicates Resend.

## Troubleshooting — every 404 cause, in order of likelihood

**Deployment succeeds, site 404s:**
1. **`package.json` is not at the repo root.** Look at the repo on GitHub: if you see one wrapper folder containing everything, that's it. Vercel looks at the root. Fix: move contents up one level.
2. **Framework Preset isn't Next.js.** Settings → General. A wrong preset overrides `vercel.json`.
3. **A ZIP was committed instead of files.** GitHub does not unpack ZIPs — it stores them as a binary blob. The build finds no `package.json`. Extract first, commit the *contents*.
4. **`trailingSlash` mismatch** between `next.config.mjs` and `vercel.json` → infinite 301 or 404 on directory routes.

**Some pages 404:** `generateStaticParams` missing or incomplete for that route.
**`.well-known/*` 404s:** files must be in `public/.well-known/`. If uploaded via a browser, they were **silently skipped** — browsers skip dotfolders. Use git.
**Forms fail:** `provider: 'smtp'` needs `SMTP_HOST/USER/PASS` set in Vercel — check `sendMail()` isn't silently returning `{sent:false}`. `provider: 'resend'` needs `RESEND_API_KEY` set in Vercel AND a verified domain.

## Verify the deploy — never assume

A green build is not a working site. After every first deploy and every domain change:

```bash
curl -s -o /dev/null -w "%{http_code}" https://[domain]/
for f in llms.txt auth.md robots.txt sitemap.xml \
         .well-known/ucp .well-known/api-catalog \
         .well-known/agent-skills/index.json .well-known/mcp/server-card.json; do
  echo "$(curl -s -o /dev/null -w '%{http_code}' https://[domain]/$f)  $f"
done
curl -sI https://[domain]/ | grep -i "^link:"                    # agent Link header present?
curl -sI https://www.[domain]/ | grep -i "^location:"            # www → apex 301?
curl -sI https://[domain]/.well-known/api-catalog | grep -i content-type   # linkset+json?
```
Vercel serves 403 to some datacenter IPs (bot protection) — a 403 from a server is not necessarily a site fault. Verify from a browser too.

**v9 — full audit tier (post-deploy).** Beyond the sampled checks above, run the complete live tier (MODE 4 AUDIT F):
```bash
# Full-sitemap 200 crawl — every URL, not a sample
curl -s https://[domain]/sitemap.xml | grep -oE '<loc>[^<]+' | sed 's/<loc>//' \
  | while read u; do echo "$(curl -s -o /dev/null -w '%{http_code}' "$u")  $u"; done
# (child sitemaps: repeat for /sitemap/0.xml .. /sitemap/3.xml)

# Lighthouse >=95 x4 + Core Web Vitals (mobile)
npx lighthouse https://[domain]/ --preset=perf --form-factor=mobile --quiet --chrome-flags="--headless"
# Accessibility (axe-core)
npx @axe-core/cli https://[domain]/
```
Assert: all crawl URLs 200; Lighthouse Performance/SEO/Accessibility/Best-Practices >=95; LCP<2.5s, CLS<0.1, INP<200ms; axe zero critical/serious. See `references/performance.md` + `references/accessibility.md`.

**Rich Results Test — not a CLI step, don't skip it because it isn't in the command block above.** Google's Rich Results Test has no scriptable API for this workflow; run it live via the in-app browser at `search.google.com/test/rich-results` against one representative URL per schema type shipped (product page, FAQ page, homepage, a blog post, etc.). See `references/seo.md` § JSON-LD validity and SKILL.md AUDIT F.

**Vercel target only:** also assert `/api/mcp` answers `tools/list`, and that its tools match `server-card.json` exactly (crosscheck B8).
