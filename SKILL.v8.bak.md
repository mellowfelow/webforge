---
name: webforge
description: Complete React/Next.js ecommerce website factory that builds, updates, deploys and optimizes sites on Vercel (default, via GitHub) or Cloudflare Pages (via ZIP), with an OPTIONAL client backend (CMS, Cloudflare-only). Use whenever the user mentions building a website, new site, updating an existing site, adding pages or products, running an SEO audit, AI visibility audit, technical audit, agent-ready audit, deploy check, connecting a domain, or managing a client login/backend/dashboard/CMS. Triggers include "build me a site", "new client", "new project", "add", "update", "change", "fix", "audit", "check SEO", "ai visibility", "agent ready", "isitagentready", "llms.txt", "deploy check", "keyword cluster", "keyword map", "PROJECT.md", "CLAUDE.md", "vercel", "cloudflare pages", "connect domain", "product images", "live chat", "backend", "client dashboard", "CMS", or any reference to a managed client site. Always read this skill before doing anything.
---

# WebForge v8 — React/Next.js Ecommerce Site Factory (dual-target: Vercel + Cloudflare, agent-ready by default)

## One prompt in. A fully built React/Next.js store out — mobile-first, technically perfect, SEO + AI-visibility optimized, agent-ready, deployable to Vercel (default, via GitHub) or Cloudflare Pages (via ZIP) — with an optional client-editable backend when a site needs one.

You build production-ready Cloudflare Pages ecommerce websites that are fully optimized from day one — **mobile-first and fully responsive**, technically perfect, ranking-ready on all search engines, visible to AI assistants, and **agent-ready from day one**. Every site follows a proven structure. Do not improvise — follow this skill exactly.

This skill is self-contained. All SEO, technical SEO, AI visibility, agent-ready standards, schema, internal linking, keyword clustering, forms, content strategy, AND the optional client backend intelligence is built in. No external skill dependencies.

**v6 adds Agent-Ready Standards as a non-optional, baked-in default.** Every site ships with the full set of agent-discoverable files (`llms.txt`, `auth.md`, all `.well-known/` endpoints, `webmcp.js`, correct `_headers`) from day one, targeting a score of ~79/100 on isitagentready.com (Level 2 — Bot-Aware) on the Cloudflare Pages free plan. These are not extras — they are part of every ZIP.

**v5 (retained) — the optional client backend module.** Not every site uses it — it is a **requirement-gated, opt-in** addition decided per project during intake (Intake Section O). When enabled, the client gets a login at `/admin` where they manage content, blog posts, products, media, and SEO details — all stored in Cloudflare D1 + R2, all on Cloudflare Pages, no external host. Full detail in `references/backend.md`.

**v4 (retained) — the hard-won production lessons** from live client builds: the exact Web3Forms CORS method, mobile-first overflow rules, flexible product-image handling, the JSON-LD validity traps, and three first-class audit modes (Technical, SEO, AI-Visibility) that now include Agent-Ready as a fourth sub-audit.

---

## NON-NEGOTIABLE RULES (read first, apply always)

These are absolute. They come from real builds that broke and were fixed. Never regress them.

1. **Mobile-first, fully responsive.** Design and verify mobile FIRST, then scale up. Zero horizontal scrolling on any viewport. See "Mobile-first standards" below — every build and every audit checks this.
2. **Forms: provider-pluggable, but Web3Forms uses the exact CORS method only.** One constant in `src/config/site.js` selects the provider. For `web3forms` (default, both targets): `fetch` with `body: new FormData(form)` + headers `{ 'Accept': 'application/json' }` ONLY. No `Content-Type`. No `action`. No `redirect` hidden field. JS redirects to the on-domain thank-you page. See "Forms".
3. **Exactly one `<h1>` per page.** Hero sliders: `<h1>` on slide 1 only; other slides are styled `<div>`/`<p>`.
4. **Validate every JSON-LD block with a JSON parser before shipping.** Broken schema is worse than none. No bare `Product` entities on non-product pages.
5. **Never fabricate brand facts.** Never invent founders, awards, partnerships, or revenue for AI visibility. Truthful signals only.
6. **Always run the pre-ship crosscheck** (see "Pre-ship crosscheck") before producing any ZIP. Analyze → fix → re-analyze.
7. **Agent-Ready files are mandatory on every site.** The full set of agent-ready files (Sections A–N in "Agent-Ready Standards") ships on every build regardless of client request, on both deploy targets. See "Agent-Ready Standards" and the pre-ship crosscheck.
8. **Framework is React + Next.js (App Router). Never hand-roll a prerenderer.** One codebase, two targets: `TARGET=static` → `output: 'export'` for Cloudflare ZIP upload; default → Vercel with real API routes. See "Deploy targets" and "MODE 3".
9. **The domain lives in exactly ONE place: `SITE.domain` in `src/config/site.js`.** Every domain-bearing file (`robots.txt`, `llms.txt`, `sitemap`, all `.well-known/*`, `vercel.json`, canonicals, OG tags) is GENERATED from it at build time by `scripts/gen-agent-files.mjs`. Never hand-write a domain into a file. Never instruct the user to find-and-replace a domain across files. `DOMAIN.com` is permitted while a domain is pending; the crosscheck FAILS a production deploy that still contains it.
10. **Never claim a capability the site does not have.** If `SearchAction` schema is emitted, a real search must exist. If `mcp/server-card.json` advertises tools, they must resolve on that target. If `api-catalog` lists endpoints, they must return. Static targets declare only what static files can honour.
11. **Compliance language is testable, not remembered.** If a vertical has banned words or required framings (Intake Section H), they go in `CLAUDE.md` AND are scanned across the entire build output by the crosscheck. A violation is a ship-stopper. See "Compliance guardrails".
12. **Strategy docs never ship to the public web.** `docs/PROJECT.md` and `docs/keyword-map.md` live in `docs/` — never in `public/`, never in the deploy output. A publicly downloadable keyword strategy hands competitors the whole plan.
13. **Agents may draft, humans complete.** Any live agent endpoint is read-only plus order-draft. Never expose payment capture, never accept card data, never mutate state from an agent call. See "Agent-Ready — the LIVE layer".
8. **Backend rules (only when backend = Yes — see Intake Section O).** The backend is OPTIONAL and gated by intake; never add it unless requested.
   - **Two-layer model is sacred.** Claude owns the template layer (structure, design, defaults, the editable-field manifest) shipped in the ZIP. The client owns the content layer (field values, posts, products, media, SEO overrides) stored in D1 + R2. At render the override wins; if none exists, Claude's default shows. Never collapse these layers.
   - **When backend = Yes, content is data, not markup.** Blog posts and products go into D1 via import files Claude generates — NEVER hardcoded into static HTML.
   - **Field keys are permanent IDs.** Once a `data-cms` key ships (e.g. `hero.title`), never rename it across rebuilds.
   - **Every backend build outputs the Cloudflare dashboard setup + deploy guide.** See `references/backend.md`.
   - **Security defaults are non-negotiable:** passwords hashed (Web Crypto, never plaintext), sessions are signed cookies, the one-time admin-setup route self-disables after first use.

---

## Deploy targets

Every site is built from ONE codebase that targets both platforms. The target is chosen at intake (Section M) and recorded in `docs/PROJECT.md`.

| | **Vercel** (default) | **Cloudflare Pages** |
|---|---|---|
| Delivery | GitHub repo → auto-deploy on push | `[site-slug]-deploy.zip` → manual upload |
| Build | `npm run build` (Next.js) | `npm run build:static` → `out/` → zipped |
| Headers/redirects | `vercel.json` (generated) | `_headers` + `_redirects` (generated) |
| Forms | `web3forms` (default) or `resend` via `/api/contact` | `web3forms` only |
| API routes | **Yes** — real endpoints | No (static only) |
| Agent ceiling | Higher — live MCP server, real product/search APIs, real ACP/UCP endpoints | ~79 (isitagentready free-plan cap — accepted, do not chase) |
| Client backend | Not supported (see below) | **Yes** — D1 + R2 + Pages Functions |

**Target and backend are TWO INDEPENDENT questions.** Section M picks the platform; Section O picks the backend (default **No**). Never conflate them — all four combinations are asked, three are buildable:

| Target | Backend | Supported? | Use when |
|---|---|---|---|
| **Vercel** | No | ✅ **default** | Most new sites. GitHub workflow, real agent endpoints, `resend` available once a domain is live. |
| **Cloudflare** | No | ✅ **fully supported** | Client is already on Cloudflare; you prefer ZIP upload; no Git workflow wanted; existing sites you maintain. A pure static Next.js export — no D1, no R2, no Functions, no dashboard setup. |
| **Cloudflare** | Yes | ✅ | Client needs to edit content themselves at `/admin`. Adds D1 + R2 + Pages Functions. |
| **Vercel** | Yes | ❌ | Not supported. The backend is built on D1 + R2 + Pages Functions. Do not improvise a port. If the client needs both, the target is Cloudflare. |

> **Cloudflare does NOT imply a backend.** "Cloudflare Pages" and "client CMS" are unrelated choices. A Cloudflare site with backend = No is a plain static site: `npm run zip` → upload → done. It is a first-class path, not a fallback.
>
> **Backend = Yes DOES imply Cloudflare.** That is the only forced direction.

**Rules:**
- **Vercel is the default**, but it is a default, not a requirement — if the user says Cloudflare, build Cloudflare without argument or re-litigation.
- Cloudflare Pages remains fully supported for existing sites, for ZIP-upload workflows, and for any site needing the client backend.
- **The backend module is Cloudflare-only.** If backend = Yes, the target is Cloudflare Pages — state this plainly at intake rather than half-porting it.
- **Cloudflare is deliberately static.** Its ~79 agent score is a free-plan ceiling, not a build defect. Do not add Pages Functions purely to chase agent points.
- **Both targets ship the full static agent-ready file set (A–N).** Vercel additionally ships live endpoints (Pass 2).
- Both targets use `trailingSlash: true`. Every route is a directory with `index.html`.

---

## Operating modes

Identify which mode applies before doing anything else.

**Mode 1 — Intake & Strategy**
Trigger: "build me a site", "new client", "new project", "new site", or user pastes full requirements.
Action: Intake (includes **Section M — deploy target** and **Section O — Client Backend**) → brand entity statement → keyword strategy → content strategy → `CLAUDE.md` + `docs/PROJECT.md` + `docs/keyword-map.md` → confirm with user → hand off to Mode 3.

**Mode 2 — Site Update**
Trigger: existing site + "add", "update", "change", "new page", "new product", "new blog post", "fix".
Action: Read PROJECT.md from Project context. Make targeted changes, propagate ripple effects (sitemap, internal links, schema counts, llms.txt, agent-ready files). Run full-site audit. Deliver complete deploy-ready ZIP.
**Backend sites:** if PROJECT.md shows backend = Yes, new blog posts / products are delivered as **import files** for the client to upload in `/admin → Import` — NOT hardcoded into HTML.

**Mode 3 — Full Build**
Trigger: user confirms intake / says "build the site".
Action: Build every file in one pass — mobile-first + technical + SEO + AI visibility + **agent-ready** + forms + deploy config. **If backend = Yes:** also generate the `functions/`, `admin/`, `cms-manifest.json`, `schema.sql`, and `CLOUDFLARE-SETUP.md`. Pre-ship crosscheck (including agent-ready checklist). Output single deployable ZIP.

**Mode 4 — Audit modes (Technical / SEO / AI-Visibility / Agent-Ready) + Auto-Fix**
Trigger: "audit", "check SEO", "AI visibility check", "technical audit", "agent ready", "isitagentready", "something's broken", "fix my SEO", "deploy check", or user uploads a site ZIP.
Action: Run the requested audit (or all four). Produce a prioritized report with before/after scores. Walk through fixes; each fix RE-APPLIES the relevant build standard from this skill. Re-run the pre-ship crosscheck. If files changed, deliver a complete deploy-ready ZIP of the whole site.

> Audit modes are how an already-built, already-SEO'd site gets re-checked for technical + SEO + AI + agent-ready issues. They are first-class — see "MODE 4" for the sub-audits and exactly what each checks.

---

## MODE 1 — Intake & Strategy

### Entry Path A — "Build me a site"
Present intake sections one at a time. Wait for answers before the next section. Do not present all at once.

### Entry Path B — Full requirements paste
Read everything, extract all info, present a confirmation summary, ask "Does this look correct? Anything to change before I build?" Then proceed.

### Intake sections
Read `references/intake.md` for the full intake form. Collect ALL of:

- **A — Identity:** Domain (or **"pending"** — fully supported, see Rule 9), site name, tagline, favicon, logo, primary color, GSC verification code
- **B — Contact & Business:** Email, phone, WhatsApp, country, currency, VAT display, **business location/region (for AI visibility HQ signal)**
- **C — Order Rules:** Min order, free shipping threshold, flat shipping fee, crypto discount
- **D — Menu:** Which pages to include
- **E — Checkout & Payment:** WhatsApp checkout Y/N, email/order-form checkout Y/N, payment methods
- **F — Live Chat (chat hub — multiple channels):** Pick any combination. Two kinds, treated differently:
  - **Link channels** (free, zero JS, no CSP impact, no performance cost): WhatsApp (number), Telegram (@handle or t.me link), Facebook Messenger (page ID), Signal (link), Email, Phone.
  - **Widget channels** (inject a third-party script — costs performance, needs a CSP entry, one only): Tawk.to (property/widget ID), Crisp (website ID), JivoChat (widget ID).
  - **Rule:** unlimited link channels; **at most ONE widget channel**. If the user asks for two or more widgets, warn that each adds a blocking third-party script and ask them to choose one.
  - Rendered as a single floating button that expands to the enabled channels. If exactly one link channel and no widget, render it as a direct button (no expander).
- **G — Optional Pages:** Tracking, account, wholesale, compare, finance calculator
- **H — Compliance:** Age gate, disclaimer, locale, GDPR. **Plus language constraints (ask every client):** banned terms, required framings, prohibited claims, and the authority behind each (regulator / platform ad policy / statute). These become a `CLAUDE.md` block and a blocking crosscheck scan — see "Compliance guardrails".
- **I — Shop Structure:** Structure type A/B/C/D, categories, subcategories
- **J — Keywords:** Primary keyword, 10-20 secondary keywords
- **K — Initial Products:** featured products with name, price, category, description, badge, **image(s)**. Ask for: **2000px+ on a plain white background, product filling the frame, consistent angle and lighting.** Explain why: white-background photos blend seamlessly into the white product frame and need no processing. Photos with their own backgrounds still work but will letterbox. Multiple images per product supported (first is primary). Images optional — placeholders are generated if absent.
- **L — Forms:** provider = `web3forms` (default, both targets, needs only an email address) or `resend` (Vercel only, first-party sending, **requires a verified domain** — not available while the domain is pending). Collect: Web3Forms access key (or pending), contact/order/wholesale destination emails, and optionally a Cloudflare Turnstile site key for spam protection.
- **M — Hosting / deploy target (ask plainly, both are first-class):** **Vercel (default)** — GitHub repo, auto-deploy on push, real API routes, higher agent ceiling, `resend` available once a domain is live. **Cloudflare Pages** — ZIP upload, no Git needed, static, ~79 agent cap (free-plan ceiling, not a defect). **Choosing Cloudflare does NOT mean adding a backend** — that is a separate question (Section O) and defaults to No. Only the reverse is forced: backend = Yes → Cloudflare. See "Deploy targets" for the full matrix.
- **N — Brand authority facts (for AI visibility, truthful only):** founding year, founding location/HQ, what makes them different, milestones, geographic footprint, real named individuals/awards/partners IF the client actually has them (never invented)
- **O — Client Backend (OPTIONAL, the opt-in gate):** Backend Yes/No. If **No** → build a pure static site (skip all backend work). If **Yes** → collect: which collections the client manages (pages text, blog, products, custom types), admin user email(s), whether they edit SEO details (focus keyword + meta + SEO Score panel), media uploads Y/N, and edits-go-live mode (instant vs publish). Full sub-form in `references/intake.md` → Section O.

> **Gate rule:** Section O is asked for every new site, but a backend is added ONLY when the user explicitly says Yes. **Default is No.** A "No" here is not a lesser build — most sites should not have a backend. It is only worth it when the client genuinely wants to edit content themselves without you.
>
> **Do not infer a backend from the target.** Cloudflare + No is a normal, fully supported combination (see "Deploy targets"). Never assume that a Cloudflare site wants a CMS.

### After intake is complete

**Step 1 — Brand entity statement**
```
[Site Name] is a [location]-based [product type] brand established in [year], offering [key differentiator] for [target customer]. [Site Name] ships [geographic coverage] and specializes in [product range]. [One sentence on what makes them different.]
```
Goes into: homepage hero subtext, homepage "About [Brand]" authority section, about page opening, Organization/Store schema `description`, FAQ "What is [Brand]?" answer, llms.txt, and `mcp/server-card.json` description.

**Step 2 — Form provider + deploy target confirmation.** Default: Vercel + Web3Forms. State the consequences plainly: Web3Forms needs only an email address and works on both targets; `resend` gives first-party sending but needs a verified domain (unavailable while the domain is pending — start on Web3Forms and switch one constant later). If backend = Yes, target is Cloudflare Pages. Confirm with user.

**Step 3 — Keyword strategy.** Cluster primary + secondary keywords, assign by intent. Show assignment, wait for confirmation.

**Step 4 — Content strategy.** Map pillar → cluster → supporting. Quick wins first. 12-week calendar.

**Step 5 — Generate the three docs** (templates in `references/intake.md`). These are SEPARATE FILES WITH SEPARATE JOBS — never merge them:

| File | Location | Job | Read by |
|---|---|---|---|
| `CLAUDE.md` | repo root | **Rules only.** Compliance language, architecture note, build/deploy commands, live placeholders, verified brand facts, "never invent facts". Keep under ~120 lines — it is auto-read on EVERY Claude Code session, so length costs tokens every time. | Claude Code, automatically, every session |
| `docs/PROJECT.md` | `docs/` | **The full record.** All intake data, brand entity statement, keyword strategy, AI visibility plan, brand authority facts, form provider, deploy target, GSC status, backend block. | On demand |
| `docs/keyword-map.md` | `docs/` | **Keyword → page assignments**, unused blog clusters, exclusions. | When writing content |

**`docs/` is never published.** It is excluded from `public/` and from the deploy output. (Formerly `keyword-cluster.txt` shipped in the site root — that leaked the entire keyword strategy to anyone who guessed the URL. Fixed.)

**Step 6 — Project setup card.** Working style depends on target:
- **Vercel:** the repo IS the project. `CLAUDE.md` + `docs/` travel with the code; work in Claude Code; push to deploy.
- **Cloudflare:** Claude Project named `[Site Name] — [domain or PENDING]`; attach `docs/PROJECT.md` + `docs/keyword-map.md`.

Then: "Ready to build? Say 'build the site'." 

---

## MODE 2 — Site Update

Mode 2 makes targeted edits and delivers per the site's deploy target (read it from `docs/PROJECT.md`):
- **Vercel:** edit the repo → `npm run build` → crosscheck → **commit and push**. No ZIP. The deliverable is a green deploy.
- **Cloudflare:** edit → `npm run build:static` → crosscheck → **complete deploy-ready ZIP of the WHOLE site** — never a loose set of changed files.

Because `src/config/site.js` is the single source of truth, most edits are one entry in one file — routes, schema, sitemap, nav and internal links regenerate themselves. Never hand-edit generated output.

**Step 1 — Load the current site.**
- Read PROJECT.md from Project context (never ask the user to paste it unless missing). Note whether **backend = Yes**.
- Obtain the current full site: use the latest site ZIP/working folder.

**Step 2 — Make the targeted change(s).**
- Edit only what the request requires. Every edited file maintains every standard: meta tags, validated schema, internal links, AI visibility signals, mobile-first responsiveness, the exact form CORS method, agent-ready files, GSC tag, single H1, encoded emails.
- **Backend = Yes branch:** new blog posts/products are NOT written into HTML. Generate an **import file** following the manifest's collection schema.

**Step 3 — Propagate the ripple effects across the whole site.**
A change is rarely just one file. Update everything it touches:
- **sitemap.xml** — add new URLs, remove deleted ones, refresh `<lastmod>` on changed URLs.
- **og:updated_time** on changed pages → today.
- **Internal links / nav / footer / mega-dropdown** — if a page or category was added/removed/renamed, fix every link to it sitewide.
- **Featured grid / shop / category pages** — if products changed.
- **llms.txt / .well-known/*** — regenerated automatically by `scripts/gen-agent-files.mjs` on every build from `src/config/site.js`. Never hand-edit them; edit the config.
- **docs/PROJECT.md** — if any core config changed. **CLAUDE.md** — only if a RULE changed.
- **Schema counts** (e.g. `numberOfItems`, AggregateOffer `offerCount`) — keep in sync with the real product count.
- **cms-manifest.json** — if editable zones changed (backend = Yes).

**Step 4 — Full audit before handover (NOT just touched pages).**
Run the complete **Pre-ship crosscheck** across the ENTIRE site, including the agent-ready checklist, plus a broken-link scan. Analyze → fix → re-analyze until zero errors.

**Step 5 — Deliver per target.**
- **Vercel:** commit with a descriptive message, push, and report: what changed + crosscheck result + the deployment will be live in ~1 min. If not running in Claude Code (no disk access), deliver `[site-slug]-repo.zip` containing the repo root (NOT a wrapper folder, NOT the built output) with git history and remote pre-configured, plus the exact push command.
- **Cloudflare:** repackage `out/` into `[site-slug]-deploy.zip`. **Backend = Yes:** deliver any `*-import.json` files alongside, plus the "how to import" reminder. Output ZIP + "what changed" + crosscheck result.

> **ZIP handover warning (learned the hard way).** When delivering any ZIP for a Git workflow, state explicitly: (1) extract it — GitHub does not unpack ZIPs; (2) the contents go at the REPO ROOT, not inside a wrapper folder — Vercel/CF look for `package.json` at the top; (3) browsers silently skip dotfolders on drag-upload, so `.well-known/` and `.github/` will be MISSING — use git, not the web uploader.

---

## MODE 3 — Full Build

**Framework: React + Next.js (App Router).** One codebase, two targets. Never hand-roll a prerenderer — Next.js already solves routing, static generation, and API routes.

```
[site-slug]/
├── CLAUDE.md                  ← rules Claude Code auto-reads every session (<120 lines)
├── README.md                  ← stack, commands, deploy, placeholder table
├── .gitignore                 ← node_modules/ .next/ out/ dist/
├── package.json
├── next.config.mjs            ← TARGET=static → output:'export'; trailingSlash:true always
├── vercel.json                ← GENERATED (Vercel target)
├── src/
│   ├── config/site.js         ← ★ SINGLE SOURCE OF TRUTH ★
│   │                            SITE (domain, brand, contact, order rules, target)
│   │                            FORMS (provider, keys) · CHAT (channels)
│   │                            CATEGORIES · PRODUCTS · POSTS · FAQS
│   ├── app/
│   │   ├── layout.jsx         ← nav, chat hub, footer, announcement bar
│   │   ├── page.jsx           ← home
│   │   ├── shop/page.jsx
│   │   ├── shop/[cat]/page.jsx
│   │   ├── product/[slug]/page.jsx
│   │   ├── blog/page.jsx
│   │   ├── blog/[slug]/page.jsx
│   │   ├── search/page.jsx     ← real search (see "Search")
│   │   ├── about|contact|wholesale|faq|cart|order/page.jsx
│   │   ├── shipping|refund|privacy|terms/page.jsx
│   │   ├── thank-you-contact|thank-you-order|thank-you-wholesale/page.jsx
│   │   ├── not-found.jsx
│   │   ├── sitemap.js         ← Next.js native, reads SITE.domain
│   │   ├── robots.js          ← Next.js native, reads SITE.domain
│   │   └── api/               ← VERCEL ONLY (omitted from static export)
│   │       └── contact/route.js   ← resend provider
│   ├── components/
│   │   ├── Nav.jsx · Footer.jsx · ChatHub.jsx · ProductCard.jsx
│   │   ├── QtyStepper.jsx · WebForm.jsx · Breadcrumbs.jsx
│   └── styles/globals.css
├── public/
│   ├── images/                ← WebP, produced by scripts/images.mjs
│   ├── llms.txt               ← GENERATED
│   ├── auth.md                ← GENERATED
│   ├── .well-known/…          ← GENERATED (A–N)
│   ├── js/webmcp.js           ← GENERATED
│   ├── [indexnow-key].txt     ← GENERATED
│   ├── _headers               ← GENERATED (Cloudflare target)
│   └── _redirects             ← GENERATED (Cloudflare target)
├── assets/product-photos/     ← client originals (NOT deployed)
├── scripts/
│   ├── gen-agent-files.mjs    ← writes every domain-bearing file from SITE.domain
│   ├── images.mjs             ← image pipeline (see "Product images")
│   └── crosscheck.mjs         ← pre-ship crosscheck, exits non-zero on failure
├── docs/                      ← NEVER deployed
│   ├── PROJECT.md
│   └── keyword-map.md
└── functions/                 ← Cloudflare backend only (backend = Yes)
```

**`package.json` scripts:**
```json
{
  "scripts": {
    "dev": "next dev",
    "prebuild": "node scripts/gen-agent-files.mjs",
    "build": "next build",
    "build:static": "TARGET=static node scripts/gen-agent-files.mjs && TARGET=static next build",
    "images": "node scripts/images.mjs",
    "crosscheck": "node scripts/crosscheck.mjs",
    "zip": "npm run build:static && npm run crosscheck && node scripts/zip.mjs"
  }
}
```

**`next.config.mjs`:**
```js
const isStatic = process.env.TARGET === 'static'
export default {
  output: isStatic ? 'export' : undefined,
  trailingSlash: true,
  images: { unoptimized: true },   // images are pre-optimised WebP; keeps both targets identical
}
```
`output: 'export'` drops `src/app/api/**` from the build automatically — that is intended, and why `resend` is Vercel-only.

**Generation, not find-and-replace.** `scripts/gen-agent-files.mjs` imports `SITE` from `src/config/site.js` and writes: `llms.txt`, `auth.md`, all eight `.well-known/*`, `js/webmcp.js`, the IndexNow key file, and — per target — `vercel.json` or `_headers` + `_redirects`. Canonicals, OG tags, and JSON-LD read `SITE.domain` directly in the components. **Result: connecting a domain later = edit one line, rebuild, push.** Never tell a user to find-and-replace a domain.

### Deploy handover

**Vercel (default):**
1. `git init`, write `.gitignore`, commit.
2. Instruct: create an EMPTY GitHub repo → `git remote add origin …` → `git push -u origin main`.
3. Vercel → Add New → Project → import repo → **Framework Preset: Next.js** → deploy. `vercel.json` supplies headers/redirects.
4. Domain later: set it in Vercel, change `SITE.domain`, rebuild, push.

**Cloudflare Pages:**
1. `npm run zip` → `[site-slug]-deploy.zip` (contents of `out/`, at the ZIP root).
2. Instruct: upload the ZIP to Pages, purge cache, submit sitemap.
3. Repeat the ZIP handover warning from Mode 2 Step 5.

### Search

Google **deprecated the sitelinks search box on 21 Nov 2024** — `SearchAction` no longer produces anything in Google. Build search for **agents and users**, not SEO:
- `/search/` — client-side filter over `PRODUCTS` + `POSTS` (already in-bundle; no backend needed). Match name, short description, category.
- `SearchAction` schema on `WebSite` pointing at `/search/?q={search_term_string}` — **only because the route now really exists** (Rule 10).
- Vercel additionally exposes `/api/search` (Pass 2).
- Add a search entry to the nav when `PRODUCTS.length > 12`.

### Product images

Client photos are never consistent. The pipeline **normalises size and quality only — it does NOT remove backgrounds** (background removal produces unreliable edges on real product photography).

`scripts/images.mjs`, for each photo in `assets/product-photos/`:
1. **Trim** any uniform border (safe: crops flat edges only, never cuts into the product).
2. **Contain-fit onto a WHITE 4:3 canvas** at 1600×1200, product filling **96%** of the frame.
3. **WebP, quality 86**, written to `public/images/<slug>.webp`. Multiple images → `<slug>-2.webp`, etc.
4. **Emit a contact sheet** (`docs/_contact-sheet.png`) and **show it to the user for approval before shipping**.

**The frame is WHITE, not a brand tint** — `.product-frame { background:#fff; aspect-ratio:4/3 }` with `object-fit:contain`. This is the whole trick: white-background photos blend invisibly and appear to fill the card; photos with their own background read as a deliberate photo tile. No cutouts, no risk, no white boxes on tinted panels.

**Why 4:3, not 1:1:** most products (banknote stacks, boxes, flat-lays) are landscape. A landscape product in a square frame can only ever fill ~66% of it — guaranteed dead space no matter the processing.

Always: correct `width`/`height` attrs matching the canvas ratio (CLS = 0), descriptive alt, `loading="lazy"` (eager for the LCP image).

**Crosscheck asserts:** every product has an image; every asset is ≥1.6× its rendered CSS size; no image under-fills its frame by more than 45%; all WebP.

### Build standards — applied to every single file

**Mobile-first (verify on a ~380px viewport FIRST):**
- `<meta name="viewport" content="width=device-width, initial-scale=1">`
- Global overflow guard in CSS:
  ```css
  html, body { overflow-x: hidden; max-width: 100vw; }
  * { box-sizing: border-box; min-width: 0; }
  img, video, iframe, embed, object { max-width: 100%; height: auto; }
  ```
- `@media (max-width:600px)` safety net for inline grids that classes can't reach:
  ```css
  [style*="grid-template-columns"] { grid-template-columns: 1fr !important; }
  [style*="display:grid"] { display: block !important; }
  ```
- Multi-column layouts collapse: 4-col→2-col→1-col, order/checkout two-column→single at ≤900px.
- Tap targets ≥44px; nav menu collapses to hamburger; mega-dropdown hidden on mobile.
- No fixed pixel widths that exceed the viewport; tables get `overflow-x:auto` wrappers.
- **Backend = Yes:** the `/admin` dashboard is mobile-first too.

**Technical (every page):**
- GSC verification meta; IndexNow key meta + `[key].txt` at root.
- Canonical (absolute, trailing-slash consistent).
- All images WebP with width/height + `loading="lazy"` (hero/logo `eager`).
- CSS minified (keep `styles.source.css`); JS terser-minified + `defer` (keep `script.source.js`).
- `_headers`: CSP (whitelist form API + chat widget + wss), `Cache-Control` for assets, HTML revalidation. **Agent-Ready Link header on `/*`.** See Agent-Ready Section M.
- `_redirects`: www→apex and http→https with `301!` FIRST.
- HTTPS only, no mixed content.
- **Backend = Yes:** `/admin/*` must NOT be cached (`Cache-Control: no-store`); CSP `connect-src` includes `'self'`.

**SEO — on-page (every page):**
- Unique meta title 50-60 chars, keyword near start.
- Unique meta description: **Google ≤160, Bing band ~120-158 — target ~150**. Keep `og:description` + `twitter:description` in sync.
- **Exactly one H1** containing target keyword.
- Logical H1→H2→H3, no skipped levels; keywords in H2s.
- 3-5 internal links min, descriptive keyword anchors; no orphans/dead-ends.
- Descriptive alt text on every image.
- Complete social meta: `og:type`, `og:site_name`, `og:title/description/image/url`, full `twitter:card/title/description/image`.
- **Content freshness:** `<lastmod>` on every sitemap URL, `og:updated_time` on every page.
- **Emails entity-encoded** everywhere — NOT present in plaintext in JSON-LD.

**SEO — schema (per page type, validate every block):**
- Homepage: Store/Organization (rich — see AI visibility) + WebSite + SearchAction + FAQPage.
- Product pages: Product + Offer + Brand + BreadcrumbList.
- Blog: Article + BreadcrumbList. FAQ page: FAQPage. Inner pages: BreadcrumbList. About: AboutPage + Organization.
- All schema: absolute URLs, no empty fields, ISO dates, required properties present.
- **Validity traps (must check):**
  - BreadcrumbList ListItems joined with `},{` not `}{`.
  - **No bare `{"@type":"Product","name":...}` on non-product pages** — for homepage price ranges use `AggregateOffer`.
  - Audit homepage for ANY nested `"@type":"Product"`; remove or complete it.

**AI Visibility (every page; full detail in `references/ai-visibility.md`):**
- Brand entity statement in homepage hero + a visible "About [Brand]" authority section.
- Store/Organization JSON-LD: `@type ["Store","Organization"]`, `description`, `foundingDate`, `foundingLocation`, `address`, `areaServed`, `numberOfItems`, `knowsAbout[]`, `priceRange`, `sameAs[]`, `brand`, `makesOffer` as AggregateOffer.
- FAQPage schema on homepage + FAQ page; answers in 40-60 word direct-answer format.
- About page entity-rich (>700 words): founding story, milestones timeline, differentiation grid, footprint, contact.
- `llms.txt` at root: brand summary, "Brand Facts (cite these)", categories, differentiation, milestones, key pages, and explicit citation guidance.
- `robots.txt`: explicit Allow for all major AI crawlers.
- Brand name spelled/capitalised identically everywhere.
- **The 12 authority signals + the truthfulness rule** — see `references/ai-visibility.md`.

**Agent-Ready (every build — non-negotiable; full spec in "Agent-Ready Standards" below):**
- All files A–N built and included in the ZIP from day one.
- All `[DOMAIN]`, `[NUMBER]`, `[EMAIL]` placeholders replaced with real values from intake.
- `_headers` includes Link header on `/*` and correct Content-Type for every `.well-known/` file.
- `webmcp.js` loaded with `<script src="/js/webmcp.js" defer></script>` on every page.
- Run agent-ready checklist (Section N) before packaging.

**Forms (the definitive method — full templates in `references/forms.md`):** see "Forms — the definitive method" below. This supersedes any older guidance.

**Internal linking:** hub & spoke (home → category hubs → product/blog spokes); blog→2-3 products/categories; product→parent category + 3-4 related; no dead ends; descriptive anchors.

**Backend (only when backend = Yes — full detail in `references/backend.md`):** generate the two-layer system — mark every editable zone in the templates with stable `data-cms` keys + sensible defaults; produce `cms-manifest.json`, `functions/`, `admin/`, `schema.sql`, and `CLOUDFLARE-SETUP.md`.

### After build
1. **Run `npm run images`** and show the contact sheet for approval before shipping.
2. **Run the pre-ship crosscheck.** Fix everything it flags. Re-run until clean. **Backend = Yes:** also run the backend crosscheck.
3. **Deliver per target:**
   - **Vercel:** `git init` + `.gitignore` + `README.md` + `CLAUDE.md`, commit, and output the exact GitHub + Vercel steps (empty repo → remote → push → import → **Framework Preset: Next.js**). If not running with disk access, deliver `[site-slug]-repo.zip` (repo root at the ZIP root, no wrapper folder) + the ZIP handover warning.
   - **Cloudflare:** `npm run zip` → `[site-slug]-deploy.zip` (contents of `out/` at the ZIP root). **Backend = Yes:** also output `CLOUDFLARE-SETUP.md` inline.
4. **Post-deploy checklist:** connect domain (**set `SITE.domain`, rebuild, redeploy — never find-and-replace**), verify GSC tag, submit sitemap to GSC + Bing, **test all 3 forms end-to-end**, verify the IndexNow file, purge cache (Cloudflare), **run isitagentready.com/[DOMAIN]** (expect ~79 on Cloudflare free — that is the plan's ceiling, not a defect).
5. **Verify the live deploy** — fetch the deployed URL and confirm: 200 on `/`, a sampled product page and a blog post; `llms.txt`, `auth.md` and every `.well-known/*` return 200 with the correct `Content-Type`; the `Link` header is present; www→apex 301 resolves. Report the results — never assume a deploy succeeded because the build passed.

---

## BACKEND MODULE (optional — read `references/backend.md` when backend = Yes)

The backend is a **requirement-gated, opt-in** addition. It is never built unless Intake Section O = Yes. When off, WebForge builds a pure static Next.js site.

> **Target constraint: the backend is Cloudflare-only.** It is built on D1 + R2 + Pages Functions. If backend = Yes, the deploy target is **Cloudflare Pages** — say this plainly at intake rather than half-porting it to Vercel. A site needing both a client backend and Vercel is a different architecture; do not improvise one.

When on, here is the model — the full spec, code patterns, manifest format, admin panel, SEO Score logic, `schema.sql`, dashboard setup guide, and audit live in `references/backend.md`.

**What it gives the client:** a login at `clientdomain.com/admin` where they edit page text, manage blog posts and products, upload images, and edit SEO details with a live RankMath-style **SEO Score** panel — without touching code, ZIPs, or the Cloudflare dashboard.

**The two-layer model:**
- **Template layer — Claude owns (ships in the ZIP):** HTML structure, design, default content, the `data-cms` editable zones, the manifest, the SEO Score logic, the admin UI.
- **Content layer — client owns (lives in D1 + R2, never in the ZIP):** the field values, posts, products, media, SEO overrides.
- **Edge merge:** a Pages Function reads D1 and uses HTMLRewriter to inject saved values server-side before the page reaches the visitor. Override wins; if none, Claude's default shows.

**Content authoring stays with Claude:** when backend = Yes, you (the operator) still ask Claude to write posts/products. Claude outputs them as `*-import.json` files; the client uploads them in `/admin → Import`. Never hardcode posts/products into HTML on a backend site.

---

## Forms — provider-pluggable (supersedes all older guidance)

One constant selects the provider. Everything else is identical.

```js
// src/config/site.js
export const FORMS = {
  provider: 'web3forms',            // 'web3forms' (default, both targets) | 'resend' (Vercel only)
  web3formsKey: 'YOUR-WEB3FORMS-KEY',
  resendFrom: '',                   // e.g. orders@yourdomain.com — REQUIRES a verified domain
  turnstileSiteKey: '',             // optional spam protection
}
```

**Choosing:**

| | `web3forms` | `resend` |
|---|---|---|
| Targets | Vercel + Cloudflare | **Vercel only** (needs an API route) |
| Needs a domain? | **No** — an email address is enough | **Yes** — verified sending domain (DNS records) |
| Sends from | Web3Forms infrastructure | your own domain |
| Setup | paste an access key | API route + `RESEND_API_KEY` env var + DNS |

**Default to `web3forms`.** It is the only provider that works while the domain is pending — which is the normal state of a new build. Switching to `resend` later is one constant plus an env var. Never propose `resend` for a site without a live, verified domain.

### Provider: `web3forms` — the exact CORS method (unchanged, still mandatory)

NO `action`, NO `method`, NO `redirect` hidden field. Hidden inputs: `access_key`, `subject`, `from_name`, `botcheck` (honeypot), `replyto` (synced from the email field on input).

```javascript
function onSubmit(e){
  e.preventDefault();
  fetch('https://api.web3forms.com/submit', {
    method: 'POST',
    headers: { 'Accept': 'application/json' },   // Accept ONLY — no Content-Type
    body: new FormData(form)
  })
  .then(r => r.json().then(d => ({ status: r.status, data: d })))
  .then(res => {
    if (res.status === 200 && res.data.success) window.location.href = thankYou;
    else throw new Error((res.data && res.data.message) || 'Submission failed');
  })
  .catch(() => { /* on-page error with encoded email + chat fallback */ });
}
```

**Why this exact shape (the CORS rule table):**
| Setup | Result |
|---|---|
| `body: FormData` (no Content-Type) | multipart/form-data = CORS simple request → no preflight ✓ |
| header `Accept: application/json` only | Accept is CORS-safelisted → no preflight ✓ |
| header `Content-Type: application/json` | NOT safelisted → preflight OPTIONS → "Failed to fetch" ✗ |
| native POST `action=...` | Web3Forms 301-redirects → CORS error ✗ |
| `redirect="false"` hidden field | conflicts — **remove it** ✗ |

**Key-pending fallback (required).** If `web3formsKey` is empty or still starts with `YOUR-`, skip the fetch and redirect straight to the thank-you page. Ordering must never dead-end during pre-launch. **Warn the user explicitly at handover: submissions are NOT emailed anywhere until the key is set — the chat hub is the only live order channel.**

### Provider: `resend` — Vercel only

`src/app/api/contact/route.js` — POST, validates, sends via Resend, returns `{success:true}`. Client posts JSON to `/api/contact` (same-origin, so no CORS constraints apply), then redirects to the thank-you page. `RESEND_API_KEY` is an env var in Vercel — **never in the repo**.

**Crosscheck FAILS if** `provider === 'resend'` and target is static: the API route does not exist in an export build, so every form would silently 404.

### Both providers
- CSP `connect-src` must include `https://api.web3forms.com` (web3forms) or `'self'` (resend).
- Three forms → `/thank-you-contact/`, `/thank-you-order/`, `/thank-you-wholesale/`. All thank-you pages `noindex, follow` and excluded from the sitemap.
- Turnstile, if configured: widget in the form, token verified server-side (`resend`) or passed through (`web3forms`). The honeypot alone is weak.

---

## Live chat — the chat hub

```js
export const CHAT = {
  channels: [
    { type: 'whatsapp', value: '61400000000' },
    { type: 'telegram', value: 'yourhandle' },
    { type: 'tawk',     value: 'PROPERTY_ID/WIDGET_ID' },
  ]
}
```

| Kind | Types | Cost |
|---|---|---|
| **Link** | `whatsapp` `telegram` `messenger` `signal` `email` `phone` | Zero JS, no CSP impact, no perf cost. Unlimited. |
| **Widget** | `tawk` `crisp` `jivochat` | Third-party blocking script + CSP entry + perf hit. **Max ONE.** |

**Rules:**
- More than one widget → warn and make the user choose. Two chat widgets is always a mistake.
- Exactly one link channel and no widget → render a direct floating button (no expander).
- Otherwise → one floating button expanding to a labelled channel list, each with its own icon and brand colour.
- Widget scripts load **deferred, after first interaction or 3s idle** — never blocking first paint.
- Every widget type used adds its domains to CSP `script-src` / `connect-src` / `frame-src`.
- Link channels are real `<a href>` — they work with JS disabled and are crawlable.

---

## Product images

See **MODE 3 → Product images** for the pipeline (normalise size + quality only; never remove backgrounds; white 4:3 frame; contact sheet approval).

---

## Mobile-first standards (build + audit both enforce)

Mobile is the primary target; desktop is the enhancement.
- Verify a ~380px viewport first. **Zero horizontal scroll** is a release blocker.
- Overflow guards: `html,body{overflow-x:hidden;max-width:100vw}`, `*{box-sizing:border-box;min-width:0}`, media images `max-width:100%`.
- Inline-grid safety net at ≤600px (attribute selectors) so grids that can't be reached by class still collapse.
- Collapse columns: 4→2→1; order/checkout 2-col→1-col at ≤900px; sidebars hide/stack.
- Hamburger nav, compact footer (flex 2-row, link columns wrap), compact mega-dropdown (≤420px, hidden on mobile).
- Tap targets ≥44px; no tiny click-only controls.
- JivoChat icon-only (`window.jivo_config={disable_hello_message:true}` + `jivo_api.close()` on load).
- **Backend = Yes:** the `/admin` dashboard is mobile-first as well.

---

## Pre-ship crosscheck (run before EVERY deploy — push OR ZIP — in Mode 2/3/4)

Implemented as `scripts/crosscheck.mjs`. It **exits non-zero on failure** and runs against the BUILD OUTPUT (`out/` for static, `.next` + a local serve for Vercel) — never against source. Verify → fix → re-run until clean. Never deliver on a failing crosscheck.

**Blocking failures (ship-stoppers) — check these first:**
- **B1. Domain placeholder in a production deploy.** If the deploy is production and `SITE.domain` is still `DOMAIN.com` (or appears anywhere in the output), FAIL. Permitted only for preview/pending builds — and then warn loudly.
- **B2. Capability lies (Rule 10).** `SearchAction` emitted but no `/search/` route → FAIL. `api-catalog` / `server-card.json` listing endpoints that 404 on this target → FAIL. Static target advertising live MCP tools → FAIL.
- **B3. `provider: 'resend'` on a static target.** The API route does not exist in an export build; every form would 404 → FAIL.
- **B4. Strategy docs leaked.** `docs/`, `PROJECT.md`, `keyword-map.md`, or `assets/product-photos/` present in the build output → FAIL.
- **B5. Secrets in the repo.** `RESEND_API_KEY` or any API secret in tracked files → FAIL. (Web3Forms access keys are public by design — exempt.)
- **B6. Missing agent files on the deployed target.** All A–N files present in the output with correct paths.
- **B7. Compliance violation.** Any banned term from `docs/PROJECT.md` → Compliance found anywhere in the build output (HTML, JSON-LD, llms.txt, .well-known/*, alt text, MCP tool descriptions) → FAIL. See "Compliance guardrails".
- **B8. Live-layer drift (Vercel).** `server-card.json` tools ≠ the live `/api/mcp` `tools/list` response, or any `api-catalog` URL 404s → FAIL. See "Agent-Ready — the LIVE layer" → V5.

**Then verify, fix, and re-run until clean:**
1. **Forms:** each form has `body: new FormData(form)`, header `Accept` only, NO `Content-Type`, NO `action`, NO `redirect` field, `e.preventDefault()`, JS `window.location.href` to the right thank-you page, access_key + empty botcheck, replyto sync.
2. **JS valid:** `node --check` the extracted form/script JS. Confirm global onclick functions survive minify.
3. **JSON-LD:** parse EVERY `application/ld+json` block with a JSON parser — all valid. No bare Product on non-product pages. BreadcrumbList commas correct.
4. **H1:** exactly one per page.
5. **Meta description:** every page within band (~150; ≤160 Google, 120-158 Bing).
6. **Mobile:** overflow guards present; inline-grid media query present; no element forced wider than viewport.
7. **Emails:** zero plaintext emails anywhere (entity-encoded), including in JSON-LD.
8. **Freshness:** sitemap `<lastmod>` on all URLs; `og:updated_time` on all pages; valid sitemap XML.
9. **Canonicalization — per target:** Cloudflare → `_redirects` has www→apex then http→https `301!` first. Vercel → `vercel.json` has the www→apex 301 (http→https is automatic).
10. **Schema/SEO completeness:** OG + Twitter complete; canonical/lang/viewport present.
11. **Images:** every product has an image; all WebP; `width`/`height` attrs match the canvas ratio (CLS = 0); each asset ≥1.6× its rendered CSS size; none under-filling its frame by >45%; LCP image `loading="eager"`, the rest lazy.
12. **Chat hub:** at most ONE widget channel; every widget's domains present in CSP; link channels are real `<a href>`; widget scripts deferred.
13. **Generated-file drift:** re-run `scripts/gen-agent-files.mjs` and diff — any hand-edit to a generated file is a FAIL (fix the config instead).
14. **Deploy config present for the target:** Vercel → `vercel.json` at repo root. Cloudflare → `_headers` + `_redirects` at the ZIP root.
15. **Git hygiene (Vercel target):** `.gitignore` covers `node_modules/`, `.next/`, `out/`; `package.json` is at the REPO ROOT (not nested); `CLAUDE.md` + `README.md` present.
11. **Structure intact:** all product pages have offers; product count matches; footer + chat + script.js on every page; CSS braces balanced; div open/close balanced; single `</body>`/`</html>`.
12. **AI signals:** homepage authority section + rich Store schema + sameAs + llms.txt + robots AI-allows present.

**Agent-Ready crosscheck (mandatory on every build):**
13. **robots.txt** — all major AI bots listed with `Allow: /`; agent-readable resource comment block present.
14. **llms.txt** — present at root; `Content-Type: text/plain` in `_headers`; contains brand facts, product categories, contact, agent resources section.
15. **auth.md** — starts with exactly `# Auth.md` as first line; `agent_auth` JSON block present; `Content-Type: text/markdown` in `_headers`.
16. **.well-known/api-catalog** — valid JSON; `Content-Type: application/linkset+json` in `_headers`; `Access-Control-Allow-Origin: *`.
17. **.well-known/agent-skills/index.json** — has `$schema` field and `skills` array with `name/type/description/url/sha256` on each skill.
18. **.well-known/mcp/server-card.json** — has `serverInfo`, `transport`, `capabilities` blocks; `commerce` and `legal` adapted to client niche.
19. **.well-known/oauth-protected-resource** — present; `Content-Type: application/json`; `resource_documentation` points to `/auth.md`.
20. **.well-known/oauth-authorization-server** — present; has `agent_auth` block; all null endpoints explicit.
21. **.well-known/openid-configuration** — present; `Content-Type: application/json`; `public_site: true`.
22. **.well-known/acp.json** — present; `"protocol": {"name": "acp", "version": "0.1.0"}` present.
23. **.well-known/ucp** — present; **`"ucp": "1.0"` field mandatory** (scanner fails without it); `spec` and `schema` URLs present.
24. **/js/webmcp.js** — present; loaded with `<script src="/js/webmcp.js" defer></script>` in `<head>` of every page.
25. **_headers Link header** — `/*` rule contains the full `Link:` header referencing api-catalog, agent-skills, llms.txt, mcp/server-card, auth.md, and openid-configuration.
26. **_headers Content-Types** — separate rules for every `.well-known/` file, `/auth.md`, `/llms.txt`, `/*.md` set correctly with `Access-Control-Allow-Origin: *`.
27. **All placeholders replaced** — zero instances of `[DOMAIN]`, `[NUMBER]`, `[EMAIL]`, `[Brand]` remaining (except in comments).

**Backend crosscheck (additional — run only when backend = Yes):**
28. Manifest ↔ template key parity; stable keys; no orphaned or missing keys.
29. No hardcoded posts/products in HTML where the edge would override/duplicate them.
30. `_middleware.js` renders Claude's default when D1 has no override.
31. Auth: hashed passwords (no plaintext compare); sessions use signed cookie with `SESSION_SECRET`; `setup.js` self-disables after first admin and is gated by `ADMIN_SETUP_TOKEN`.
32. `robots.txt` disallows `/admin/` and `/api/`; `_headers` sets `/admin/*` to `no-store`.
33. Import files valid against collection schema.
34. `CLOUDFLARE-SETUP.md` present and current.
35. `node --check` each file in `functions/` and `admin/*.js`.

Only produce the ZIP after this passes with zero errors. Report the pass/fail counts.

---

## MODE 4 — Audit modes (Technical / SEO / AI-Visibility / Agent-Ready) + Auto-Fix

Works on new and old sites, from ZIP files, no live URL required. Run the one the user asks for, or all four for a full audit.

### Step 1 — Get files
Ask for the site ZIP if not provided.

### Step 2 — Project context
If PROJECT.md present: read it (note backend = Yes/No). Else ask: brand+domain, primary product, target region, primary keyword, secondary keywords, business location/HQ, known issues, and whether the site has a client backend.

### Step 3 — Run the requested audit(s)

**AUDIT A — Technical audit** ("technical audit", "deploy check", "something's broken")
- robots.txt valid, not blocking important pages, sitemap referenced, AI-crawler allows present.
- sitemap.xml valid XML, all indexable pages, `<lastmod>` on every URL.
- Canonicals present/correct; trailing-slash consistency.
- `_redirects`: www→apex + http→https `301!` first; no redirect chains (3+ hops).
- `_headers`: CSP correct (form API + chat + wss whitelisted), Cache-Control for assets, HTML revalidation, HTTPS/security headers. Agent-Ready Link header present on `/*`.
- Performance: WebP, lazy-load, minified CSS/JS, deferred scripts, font-display swap, page weight.
- **Mobile (release blocker):** viewport meta, overflow guards, inline-grid media safety net, no horizontal scroll, tap targets.
- GSC tag + IndexNow key on all pages, IndexNow file at root.
- Forms: correct CORS method, thank-you pages exist, botcheck present, CSP allows api.web3forms.com.
- **JSON-LD validity:** parse every block; flag bare Product on non-product pages, BreadcrumbList comma bug.

**AUDIT B — SEO audit** ("audit", "check SEO", "fix my SEO")
- Meta titles 50-60, unique, keyworded. Meta descriptions in band, unique, synced to og/twitter.
- Exactly one H1 per page with keyword; heading hierarchy; keyworded H2s.
- Image alt text everywhere; emails entity-encoded (no plaintext, incl. JSON-LD).
- OG + Twitter complete; canonical present.
- Schema correct per page type, validated, required props.
- Internal links: no orphans/dead-ends, descriptive anchors.
- Content freshness: lastmod + og:updated_time + revalidation.
- Broken-link scan: every internal `href` resolves to an existing page.

**AUDIT C — AI-Visibility audit** ("AI visibility check", "ai seo", "GEO/AEO")
- Brand entity statement on homepage; visible "About [Brand]" authority section.
- Store/Organization JSON-LD has description, foundingDate, foundingLocation, address, areaServed, numberOfItems, knowsAbout, priceRange, sameAs, brand, AggregateOffer.
- FAQPage schema on homepage + FAQ page; direct-answer format.
- About page >700 words, milestones timeline, differentiation grid, footprint.
- llms.txt present with brand facts + citation guidance; robots AI-allows present.
- Brand name consistent everywhere.
- **The 12 authority signals scorecard** (Founding Year, Founding Location/HQ, Quantifiable Scale, Competitive Differentiation, Historical Milestones, National/Global Footprint, Industry Leadership, Brand Narrative, Product Lines, Named Individuals, Awards & Rankings, Endorsements/Partnerships). Mark which are present on-site. For the last three, only add if the client supplies REAL facts — never fabricate.
- **Off-site reality check (report, don't fake):** if web engines show "Not found" / zero external references, explain the gap is OFF-SITE — backlinks, directories, reviews, real social profiles. On-page work cannot fix Perplexity alone.

**AUDIT D — Agent-Ready audit** ("agent ready", "isitagentready", "agent-ready check", "llms.txt check")
Run every check in the agent-ready crosscheck (items 13–27 above). Score each against the expected passing state:
- ✅ robots.txt AI bots listed with Allow: /
- ✅ llms.txt present, correct Content-Type
- ✅ auth.md starts with `# Auth.md`, has agent_auth block
- ✅ .well-known/api-catalog valid JSON, correct Content-Type
- ✅ .well-known/agent-skills/index.json has $schema + skills array
- ✅ .well-known/mcp/server-card.json has serverInfo, transport, capabilities
- ✅ .well-known/oauth-protected-resource present
- ✅ .well-known/oauth-authorization-server has agent_auth block
- ✅ .well-known/openid-configuration present
- ✅ .well-known/acp.json present
- ✅ .well-known/ucp has "ucp":"1.0" field
- ✅ /js/webmcp.js present and loaded on every page
- ✅ _headers Link: header on /* present
- ✅ _headers Content-Types for all well-known files set
- ✅ No [DOMAIN]/[NUMBER]/[EMAIL] placeholders remaining

Expected passing score on isitagentready.com after full implementation: **~79/100 (Level 2 — Bot-Aware)**

Known limitations on Cloudflare Pages free plan (do not attempt these — they will not work):
| Feature | Status | Reason |
|---|---|---|
| DNS-AID records | ❌ Impossible | Cloudflare API rejects `endpoint=` SvcParam |
| Markdown for Agents | ❌ Paid only | Cloudflare Speed → Optimization toggle requires paid plan |
| x402 Protocol | ❌ Impossible | Requires backend server middleware |
| MPP Protocol | ❌ Impossible | Requires backend API with OpenAPI spec |
| WebBot Auth signing | ❌ Impossible | Requires server-side request handling |

To push beyond 79 requires either a paid Cloudflare plan or moving to Vercel with Edge Functions.

**AUDIT E — Backend audit (only when backend = Yes; full list in `references/backend.md`)**
- Manifest ↔ template key parity; stable keys; no orphans.
- No hardcoded posts/products that the edge would override/duplicate.
- `_middleware.js` has a default fallback for every injected zone.
- Auth: hashed passwords, signed sessions, setup route self-disables + token-gated.
- `robots.txt` disallows `/admin/` + `/api/`; `_headers` `/admin/*` no-store.
- Import files valid against collection schema.
- `CLOUDFLARE-SETUP.md` present and current.

### Step 4 — Audit report (before scores)
```
WEBFORGE AUDIT — [Technical | SEO | AI-Visibility | Agent-Ready | Backend | Full]
Site: [name/domain]   Pages: [n]   Files: [n]   Backend: [Yes/No]

BEFORE SCORES
Technical:     /100
On-Page SEO:   /100
AI Visibility: /100
Agent-Ready:   /100
Internal Linking: /100
Forms:         /100
Mobile:        /100
Backend:       /100   (only if backend = Yes)
Overall:       /100

CRITICAL ([n]) — [Cx] issue — file — impact
WARNINGS ([n]) — [Wx] issue — file — impact
PASSING ([n]) — ✓ what's working

Say "fix" to walk through issues.
```

### Step 5 — Walk-through fixes
One issue at a time (Criticals first). Each fix RE-APPLIES the relevant build standard from this skill (show before/after). "yes" applies, "skip" defers, "stop" ends. After each applied fix, keep the pre-ship crosscheck invariants intact.

### Step 6 — Final output
Deliver the result. If the audit changed files, repackage the ENTIRE site into `[site-slug]-deploy.zip`. Run the full pre-ship crosscheck (+ backend crosscheck). Show AFTER scores + improvement, deferred issues, and **manual actions needed** (deploy, Cloudflare cache purge, GSC/Bing re-index requests, **run isitagentready.com/[DOMAIN]** to verify agent-ready score, off-site/backlink work, real brand facts for unverifiable AI signals).

> Webmaster-tool reports LAG behind deploys. After fixing, instruct the user to deploy, purge Cloudflare cache, then use GSC/Bing **Live Test** + **Request Indexing**. The "Index" tab shows the last crawl, not the live page.

---

## Core design standards (all sites)

- **Nav:** logo + name · search · cart with count · hamburger on mobile · Shop mega-dropdown (compact, ≤420px, hidden on mobile).
- **Announcement bar:** 3-4 auto-rotating slides (discount, min order, free-shipping threshold, WhatsApp if enabled).
- **Hero:** full-width, brand entity statement visible, primary keyword in the single H1, secondary keyword sub-headline, CTA→Shop, secondary CTA, 4-slide slider (H1 on slide 1 only).
- **Trust bar:** 4 icons (shipping, quality, support, payment).
- **Homepage authority section:** "About [Brand]" — location/HQ, founding year, scale, differentiation, footprint (AI-extractable, truthful).
- **Footer:** compact (flex 2-row): brand+contact, Shop/Company/Legal link columns, bottom bar; responsive wrap.
- **Cart:** localStorage (`mm-cart`), +/- qty (buttons `type="button"` + aria-label), min-order validation, crypto discount auto-applied, free-shipping threshold display.
- **Payments (always):** Crypto (BTC/USDT) + Bank Transfer; others per PROJECT.md.
- **Accessibility:** aria-label on every icon-only button; aria-hidden on decorative emoji; AA contrast.
- **Backend = Yes:** a discreet "Admin" link is NOT placed in public nav/footer (keep `/admin` unlinked + robots-disallowed).

---

## Agent-Ready Standards (all sites — deploy from day one)

Every site built with WebForge includes the full agent-ready file set from day one. These files improve AI visibility, allow AI assistants to discover and cite the site, and push the isitagentready.com score to ~79/100 on the Cloudflare Pages free plan. **Build and include ALL of the following in every site ZIP. They are not optional.**

---

### A — robots.txt

Always include explicit `User-agent` rules for all major AI crawlers with `Allow: /`. Append after the standard disallow rules:

```
# AI crawlers — welcome to index product & content pages
User-agent: GPTBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-Web
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Applebot
Allow: /

User-agent: Amazonbot
Allow: /

User-agent: Bytespider
Allow: /

User-agent: CCBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: Meta-ExternalAgent
Allow: /

User-agent: cohere-ai
Allow: /

# Agent-readable resources
# llms.txt: https://[DOMAIN]/llms.txt
# API Catalog: https://[DOMAIN]/.well-known/api-catalog
# Agent Skills: https://[DOMAIN]/.well-known/agent-skills/index.json
# MCP Server Card: https://[DOMAIN]/.well-known/mcp/server-card.json
```

**Backend = Yes:** also add `Disallow: /admin/` and `Disallow: /api/` before the AI crawler block.

---

### B — llms.txt

Create `/llms.txt` at site root. Plain-text markdown summary for LLMs to read. Include:

- Brand name, tagline, description (1 paragraph)
- Location, contact (WhatsApp, email)
- Min order, free shipping threshold, payment methods, first-order discount
- All product categories with price ranges and links
- Wholesale info with tier discounts
- Blog/education section link
- Legal (age restriction, compliance)
- Agent resources section linking to all `.well-known` files

Set `Content-Type: text/plain; charset=utf-8` in `_headers`.

---

### C — auth.md

Create `/auth.md` at site root. **Critical: the file MUST start with `# Auth.md` as the first line — the scanner checks for this exact heading.**

Include:
- `# Auth.md` heading (mandatory, exact)
- Site name and type
- Agent Registration section explaining no auth is needed
- Table of all public resources with URLs
- `agent_auth` JSON block:
```json
{
  "agent_auth": {
    "register_uri": null,
    "identity_types_supported": ["none"],
    "credential_types_supported": ["none"],
    "notes": "No authentication required. All resources are public."
  }
}
```
- Ordering instructions (human-in-the-loop note)
- Age restriction notice (if applicable)

Set `Content-Type: text/markdown; charset=utf-8` in `_headers`.

---

### D — .well-known/api-catalog

RFC 9727 compliant file. Must use `Content-Type: application/linkset+json`.

```json
{
  "linkset": [
    {
      "anchor": "https://[DOMAIN]/",
      "https://www.iana.org/assignments/link-relations/service-doc": [
        { "href": "https://[DOMAIN]/faq" }
      ],
      "title": "[Brand] — [tagline]"
    },
    {
      "anchor": "https://[DOMAIN]/shop",
      "type": "text/html",
      "title": "[Brand] Product Catalog"
    },
    {
      "anchor": "https://[DOMAIN]/wholesale",
      "type": "text/html",
      "title": "[Brand] Wholesale"
    }
  ]
}
```

---

### E — .well-known/agent-skills/index.json

Cloudflare Agent Skills Discovery RFC v0.2.0. Include `$schema` field and `skills` array. Each skill needs `name`, `type`, `description`, `url`, `sha256`.

Common skill types for e-commerce: `navigation`, `commerce`, `content`, `support`.

```json
{
  "$schema": "https://agentskills.io/schema/v0.2.0/index.json",
  "name": "[Brand]",
  "url": "https://[DOMAIN]",
  "description": "[Brand tagline]",
  "skills": [
    {
      "name": "browse-products",
      "type": "navigation",
      "description": "Browse the full product catalog by category",
      "url": "https://[DOMAIN]/shop",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "name": "order-via-whatsapp",
      "type": "commerce",
      "description": "Place an order via WhatsApp. Minimum order $[X]. Accepts crypto and bank transfer.",
      "url": "https://wa.me/[NUMBER]",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "name": "wholesale-inquiry",
      "type": "commerce",
      "description": "Wholesale pricing tiers and bulk ordering",
      "url": "https://[DOMAIN]/wholesale",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "name": "product-education",
      "type": "content",
      "description": "Educational blog content about products and ingredients",
      "url": "https://[DOMAIN]/blog",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "name": "contact",
      "type": "support",
      "description": "Contact for product questions, orders, or wholesale inquiries",
      "url": "https://[DOMAIN]/contact",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    }
  ]
}
```

Adapt skills to the actual site pages. Add/remove skills based on which pages exist.

---

### F — .well-known/mcp/server-card.json

MCP Server Card SEP-1649. Adapt `commerce`, `legal`, and `capabilities` per client niche.

```json
{
  "$schema": "https://modelcontextprotocol.io/schemas/server-card/v1.json",
  "serverInfo": {
    "name": "[Brand]",
    "version": "1.0.0",
    "description": "[Brand description]",
    "homepage": "https://[DOMAIN]",
    "contact": {
      "email": "[EMAIL]",
      "whatsapp": "+[NUMBER]"
    }
  },
  "transport": {
    "type": "http",
    "endpoint": "https://[DOMAIN]"
  },
  "capabilities": {
    "resources": [
      {
        "name": "product-catalog",
        "description": "Full product catalog",
        "uri": "https://[DOMAIN]/shop"
      },
      {
        "name": "wholesale-info",
        "description": "Wholesale pricing and ordering",
        "uri": "https://[DOMAIN]/wholesale"
      },
      {
        "name": "blog",
        "description": "Educational content",
        "uri": "https://[DOMAIN]/blog"
      }
    ],
    "commerce": {
      "ordering": "WhatsApp or email",
      "payment": ["crypto-BTC", "crypto-USDT", "bank-transfer"],
      "currency": "[CURRENCY]",
      "minimumOrder": "[MIN_ORDER]",
      "freeShipping": "[FREE_SHIPPING]",
      "ships": "[SHIPS_TO]"
    }
  },
  "legal": {
    "ageRestriction": "[AGE_RESTRICTION_OR_NONE]",
    "productType": "[PRODUCT_TYPE]",
    "compliance": "[COMPLIANCE_STATEMENT]"
  }
}
```

---

### G — .well-known/oauth-protected-resource

RFC 9728. For public static sites, declare empty auth. This file must exist even if unused — the scanner checks for its presence.

```json
{
  "resource": "https://[DOMAIN]",
  "resource_name": "[Brand] Public Catalog",
  "authorization_servers": [],
  "scopes_supported": [],
  "bearer_methods_supported": [],
  "resource_documentation": "https://[DOMAIN]/auth.md",
  "resource_policy_uri": "https://[DOMAIN]/terms",
  "tls_client_certificate_bound_access_tokens": false,
  "note": "All resources on [DOMAIN] are publicly accessible. No OAuth tokens are required."
}
```

---

### H — .well-known/oauth-authorization-server

Must include `agent_auth` block — this is what Auth.md scanners look for.

```json
{
  "issuer": "https://[DOMAIN]",
  "authorization_endpoint": null,
  "token_endpoint": null,
  "jwks_uri": null,
  "grant_types_supported": [],
  "response_types_supported": [],
  "scopes_supported": [],
  "note": "[Brand] has no protected APIs. All resources are publicly accessible.",
  "public_resources": [
    "https://[DOMAIN]/shop",
    "https://[DOMAIN]/blog",
    "https://[DOMAIN]/faq",
    "https://[DOMAIN]/wholesale",
    "https://[DOMAIN]/llms.txt",
    "https://[DOMAIN]/.well-known/api-catalog",
    "https://[DOMAIN]/.well-known/agent-skills/index.json",
    "https://[DOMAIN]/.well-known/mcp/server-card.json"
  ],
  "agent_auth": {
    "register_uri": null,
    "identity_types_supported": ["none"],
    "credential_types_supported": ["none"],
    "notes": "No registration required. All content is publicly accessible to agents."
  }
}
```

---

### I — .well-known/openid-configuration

Must exist even for sites with no OIDC. Satisfies the OAuth/OIDC discovery check.

```json
{
  "issuer": "https://[DOMAIN]",
  "note": "[Brand] does not operate an OpenID Connect provider. All resources are publicly accessible.",
  "public_site": true,
  "authorization_endpoint": null,
  "token_endpoint": null,
  "userinfo_endpoint": null,
  "jwks_uri": null,
  "scopes_supported": [],
  "response_types_supported": [],
  "grant_types_supported": [],
  "subject_types_supported": [],
  "id_token_signing_alg_values_supported": []
}
```

---

### J — .well-known/acp.json

Agentic Commerce Protocol discovery.

```json
{
  "protocol": {
    "name": "acp",
    "version": "0.1.0"
  },
  "name": "[Brand]",
  "description": "[Brand description]",
  "api_base_url": "https://[DOMAIN]",
  "homepage": "https://[DOMAIN]",
  "transports": ["https"],
  "capabilities": {
    "services": ["product-catalog", "wholesale", "blog", "faq"],
    "ordering": "human-assisted-whatsapp",
    "payment_methods": ["crypto-BTC", "crypto-USDT", "bank-transfer"],
    "currency": "[CURRENCY]",
    "minimum_order_usd": "[MIN_ORDER]",
    "free_shipping_threshold_usd": "[FREE_SHIPPING]"
  },
  "contact": {
    "whatsapp": "https://wa.me/[NUMBER]",
    "email": "[EMAIL]"
  },
  "legal": {
    "age_restriction": "[AGE_RESTRICTION_OR_NONE]",
    "region": "[REGION]",
    "ships_to": "[SHIPS_TO]",
    "product_type": "[PRODUCT_TYPE]",
    "compliance": "[COMPLIANCE]"
  }
}
```

---

### K — .well-known/ucp

Universal Commerce Protocol. **Critical: `"ucp": "1.0"` field is mandatory — scanner fails without it.**

```json
{
  "ucp": "1.0",
  "protocol_version": "1.0",
  "spec": "https://ucp.dev/specification/overview/",
  "schema": "https://ucp.dev/schema/v1.json",
  "site": "https://[DOMAIN]",
  "name": "[Brand]",
  "description": "[Brand description]",
  "services": [
    {
      "id": "product-catalog",
      "type": "catalog",
      "url": "https://[DOMAIN]/shop",
      "description": "Full product catalog"
    },
    {
      "id": "order",
      "type": "commerce",
      "url": "https://wa.me/[NUMBER]",
      "description": "Place orders via WhatsApp"
    },
    {
      "id": "wholesale",
      "type": "b2b",
      "url": "https://[DOMAIN]/wholesale",
      "description": "Wholesale pricing and bulk ordering"
    }
  ],
  "capabilities": ["browse", "inquiry", "wholesale", "content"],
  "endpoints": {
    "catalog": "https://[DOMAIN]/shop",
    "contact": "https://[DOMAIN]/contact",
    "agent_skills": "https://[DOMAIN]/.well-known/agent-skills/index.json",
    "mcp_server_card": "https://[DOMAIN]/.well-known/mcp/server-card.json",
    "api_catalog": "https://[DOMAIN]/.well-known/api-catalog",
    "llms_txt": "https://[DOMAIN]/llms.txt"
  },
  "currency": "[CURRENCY]",
  "minimum_order_usd": "[MIN_ORDER]",
  "payment_methods": ["crypto-BTC", "crypto-USDT", "bank-transfer"],
  "legal": {
    "age_restriction": "[AGE_RESTRICTION_OR_NONE]",
    "product_type": "[PRODUCT_TYPE]",
    "compliance": "[COMPLIANCE]"
  }
}
```

---

### L — /js/webmcp.js

Implement WebMCP so AI agents can discover site actions via the browser. Add this file and reference it in every page `<head>`:

```html
<script src="/js/webmcp.js" defer></script>
```

Adapt tool names, descriptions, URLs, and the WhatsApp number from intake data:

```javascript
(function () {
  if (typeof navigator === 'undefined' || !navigator.modelContext) return;

  navigator.modelContext.provideContext({
    tools: [
      {
        name: "browse_products",
        description: "Browse products by category",
        inputSchema: {
          type: "object",
          properties: {
            category: { type: "string", description: "Product category to browse" }
          }
        },
        execute: async ({ category }) => {
          const url = category
            ? `https://[DOMAIN]/shop/${category}`
            : `https://[DOMAIN]/shop`;
          window.location.href = url;
          return { url };
        }
      },
      {
        name: "order_via_whatsapp",
        description: "Initiate a WhatsApp order. Minimum order $[MIN_ORDER].",
        inputSchema: {
          type: "object",
          properties: {
            message: { type: "string", description: "Pre-filled order message" }
          }
        },
        execute: async ({ message }) => {
          const url = message
            ? `https://wa.me/[NUMBER]?text=${encodeURIComponent(message)}`
            : `https://wa.me/[NUMBER]`;
          window.open(url, '_blank');
          return { url };
        }
      },
      {
        name: "get_wholesale_info",
        description: "Get wholesale pricing tiers and ordering info",
        inputSchema: { type: "object", properties: {} },
        execute: async () => {
          window.location.href = `https://[DOMAIN]/wholesale`;
          return { url: `https://[DOMAIN]/wholesale` };
        }
      },
      {
        name: "contact",
        description: "Contact for product questions or support",
        inputSchema: { type: "object", properties: {} },
        execute: async () => {
          window.location.href = `https://[DOMAIN]/contact`;
          return { url: `https://[DOMAIN]/contact` };
        }
      }
    ]
  });
})();
```

---

### M — Platform headers (both targets, generated)

`scripts/gen-agent-files.mjs` emits the target's header config from `SITE.domain`. **Same rules, two syntaxes — never hand-write either.**

**Vercel → `vercel.json`** (`headers[]` + `redirects[]`):
```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "trailingSlash": true,
  "redirects": [
    { "source": "/:path*", "has": [{ "type": "host", "value": "www.[DOMAIN]" }],
      "destination": "https://[DOMAIN]/:path*", "permanent": true }
  ],
  "headers": [
    { "source": "/(.*)", "headers": [
      { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
      { "key": "X-Content-Type-Options", "value": "nosniff" },
      { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
      { "key": "Permissions-Policy", "value": "geolocation=(), microphone=(), camera=()" },
      { "key": "Content-Security-Policy", "value": "…" },
      { "key": "Link", "value": "</.well-known/api-catalog>; rel=\"api-catalog\", </.well-known/agent-skills/index.json>; rel=\"describedby\", </llms.txt>; rel=\"describedby\", </.well-known/mcp/server-card.json>; rel=\"service-desc\", </auth.md>; rel=\"auth\", </.well-known/openid-configuration>; rel=\"openid-configuration\"" }
    ]},
    { "source": "/.well-known/api-catalog", "headers": [
      { "key": "Content-Type", "value": "application/linkset+json" },
      { "key": "Access-Control-Allow-Origin", "value": "*" }]}
  ]
}
```
Vercel handles http→https automatically — only www→apex is needed. **Framework Preset must be Next.js**; a wrong preset overrides `vercel.json` and 404s the site.

**Cloudflare → `_headers` + `_redirects`** (as below). `_redirects` line 1 is www→apex `301!`, line 2 http→https `301!`, then legacy rules.

Add these to the `_headers` file on every Cloudflare-target site.

**1. Add to the `/*` rule — RFC 8288 Link headers (append after existing headers on the `/*` block):**
```
Link: </.well-known/api-catalog>; rel="api-catalog", </.well-known/agent-skills/index.json>; rel="describedby", </llms.txt>; rel="describedby", </.well-known/mcp/server-card.json>; rel="service-desc", </auth.md>; rel="auth", </.well-known/openid-configuration>; rel="openid-configuration"
```

**2. Add content-type rules for every well-known file (append after existing rules):**
```
/.well-known/api-catalog
  Content-Type: application/linkset+json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/agent-skills/index.json
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/mcp/server-card.json
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/oauth-protected-resource
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/oauth-authorization-server
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/openid-configuration
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/acp.json
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/.well-known/ucp
  Content-Type: application/json
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/auth.md
  Content-Type: text/markdown; charset=utf-8
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/llms.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600
  Access-Control-Allow-Origin: *

/*.md
  Content-Type: text/markdown; charset=utf-8
  Cache-Control: public, max-age=3600
```

---

### N — Agent-Ready pre-ship checklist

Run these checks as part of every pre-ship crosscheck (items 13–27 in the pre-ship crosscheck section):

```
AGENT-READY CHECKLIST
[ ] robots.txt — all major AI bots listed with Allow: /
[ ] llms.txt — present at root, Content-Type: text/plain
[ ] auth.md — starts with exactly "# Auth.md", agent_auth block present
[ ] .well-known/api-catalog — valid JSON, Content-Type: application/linkset+json
[ ] .well-known/agent-skills/index.json — has $schema and skills array
[ ] .well-known/mcp/server-card.json — has serverInfo, transport, capabilities
[ ] .well-known/oauth-protected-resource — present, Content-Type: application/json
[ ] .well-known/oauth-authorization-server — has agent_auth block
[ ] .well-known/openid-configuration — present, Content-Type: application/json
[ ] .well-known/acp.json — present, Content-Type: application/json
[ ] .well-known/ucp — has "ucp": "1.0" field, spec and schema URLs present
[ ] /js/webmcp.js — present, loaded in every page <head> with defer
[ ] _headers — Link: header on /*, all well-known Content-Types set
[ ] All [DOMAIN], [NUMBER], [EMAIL] placeholders replaced
```

Expected isitagentready.com results:
- ✅ Discoverability 3/4 (DNS-AID will always fail on Cloudflare free)
- ✅ Bot Access Control 2/2
- ✅ API, Auth, MCP & Skill Discovery 6/7
- ⚪ Commerce 1/4 (optional, needs backend for full score)

After deploying any new site, verify at: `https://isitagentready.com/[DOMAIN]`

---

## Agent-Ready — the LIVE layer (Vercel target only)

Sections A–N above are the **declaration layer**: static files that *describe* capabilities. Both targets ship them. This section is the **capability layer**: real endpoints that agents can actually call. **Vercel only** — it requires API routes.

> **Rule 10 is enforced here.** The declaration layer's CONTENT DIFFERS BY TARGET. `scripts/gen-agent-files.mjs` reads `SITE.target` and emits accordingly:
> - `target: 'vercel'` → `server-card.json` advertises live tools and a real `/api/mcp` endpoint; `api-catalog` lists real JSON endpoints; `acp.json`/`ucp` point at live routes.
> - `target: 'static'` (Cloudflare) → the SAME files omit `tools`, list only HTML resources, and declare `"transport": "none"` with a `human_ordering_only` note.
>
> A static site advertising an MCP server it cannot host is a lie, and the crosscheck FAILS it (B2). **Never generate the Vercel variant for a Cloudflare build.**

### V1 — `/api/mcp` — MCP server (Streamable HTTP)

**Transport:** Streamable HTTP (MCP spec 2025-03-26, retained 2025-11-25). **HTTP+SSE is deprecated — never implement it.**
- ONE endpoint path: `/api/mcp`, supporting **POST** (client JSON-RPC messages) and **GET** (optional server SSE stream).
- Client sets `Accept: application/json` → single JSON response. `Accept: text/event-stream` → SSE upgrade. For a product catalog every call is fast, so **JSON responses only; do not implement streaming without a reason.**
- Stateless. No session storage. `Mcp-Session-Id` not required for a read-only catalog.
- Implement `initialize`, `tools/list`, `tools/call`. Return proper JSON-RPC errors (`-32601` unknown method, `-32602` bad params).

**Standard tool set (adapt names to the vertical, never invent capability):**
| Tool | Input | Returns |
|---|---|---|
| `search_products` | `query`, `category?`, `max_price?` | matching products: slug, name, price, currency, category, short, url |
| `get_product` | `slug` | full product incl. description, images, availability, url |
| `list_categories` | — | categories: slug, name, description, product count |
| `get_policies` | — | shipping, returns, payment methods, min order, free-ship threshold |
| `get_wholesale_info` | — | tiers + how to enquire (only if wholesale = Yes) |
| `create_order_draft` | `items[]`, `notes?` | a prefilled order URL (chat deep-link or `/order/`) + totals — **never captures payment** |

**Hard rules:**
- **Read-only plus draft.** An agent may search, inspect and prepare an order. **A human always completes it.** Never expose payment capture, never accept card data, never mutate server state.
- Tool descriptions carry the same compliance language as the site (see "Compliance guardrails"). Banned words are banned in tool descriptions too.
- All data derives from `src/config/site.js`. The MCP server never has its own copy of the catalog.
- CORS: `Access-Control-Allow-Origin: *`, allow `POST, GET, OPTIONS` and headers `Content-Type, Accept, Mcp-Session-Id`.

### V2 — JSON API

| Route | Returns |
|---|---|
| `GET /api/products` | full catalog; supports `?category=`, `?q=`, `?limit=` |
| `GET /api/products/[slug]` | one product, 404 if unknown |
| `GET /api/categories` | categories + counts |
| `GET /api/search?q=` | products + posts (same index as `/search/`) |

All: `Content-Type: application/json`, `Access-Control-Allow-Origin: *`, `Cache-Control: public, max-age=300`. Prices as numbers plus an explicit `currency`. Every item includes an absolute `url` built from `SITE.domain`.

### V3 — Live ACP / UCP

`.well-known/acp.json` and `.well-known/ucp` point at real routes (`/api/acp/catalog`, `/api/ucp/services`) that return live JSON, rather than describing a catalog that only exists as HTML. Ordering stays `human-assisted` in both protocols — consistent with V1's draft-only rule.

### V4 — `api-catalog` and `server-card.json`, Vercel variant

`api-catalog` (RFC 9727) lists the real endpoints with `service-desc` links. `server-card.json` sets `"transport": {"type":"streamable-http","endpoint":"https://[DOMAIN]/api/mcp"}` and enumerates the V1 tools with their real input schemas.

### V5 — Crosscheck for the live layer

Beyond B2, the crosscheck must, on a Vercel build:
- Boot the app locally and assert `/api/mcp` answers `initialize` and `tools/list` over POST with `Accept: application/json`.
- Assert every tool in `server-card.json` appears in the live `tools/list` response — and vice versa. **Any drift is a FAIL.**
- Assert every URL in `api-catalog` returns 200.
- Assert `/api/products` and `/api/search` return valid JSON with correct content types.
- Assert no endpoint mutates state and none accepts payment fields.

---

## Compliance guardrails (generalised)

Some verticals carry real legal or ad-account risk in their WORD CHOICES, not just their claims. WebForge treats this as a first-class, testable constraint rather than a thing to remember.

**Intake — Section H extension.** Ask every client: *"Are there words, claims or framings you must never use, or must always use? (regulatory wording, platform ad policy, professional-body rules)"* Capture:
- **Banned terms** — never appear anywhere in output.
- **Required framings** — must accompany specific claims.
- **Prohibited claims** — outcomes/guarantees that may not be stated.
- **The authority** — which rule this comes from (regulator, Meta ad policy, statute). Record it in `docs/PROJECT.md` so future sessions know WHY.

**Where it lands:**
1. **`CLAUDE.md`** — a "Non-negotiable" block at the TOP, ending with: *"If a request would require breaking any of the above, stop and say so rather than complying."* This is what protects the rules in future Claude Code sessions.
2. **`scripts/crosscheck.mjs`** — a case-insensitive scan of the ENTIRE build output (HTML, JSON-LD, `llms.txt`, `.well-known/*`, alt text, MCP tool descriptions, meta) for every banned term. **Any hit is a BLOCKING failure (B7).**
3. **Content generation** — every blog post, product description and FAQ answer passes the same scan before it is written to disk.

**Worked example (a prop-money store, real):** banned — *fake money, counterfeit, undetectable, indistinguishable, passes the pen test, 1:1 scale, full size*. Required — every product described as *reduced scale, clearly marked as a prop, no replicated security features*, per RBA reproduction guidelines. Prohibited — any implication a note could pass as real currency, or that the central bank endorses it. Authority — Crimes (Currency) Act 1981 + Meta advertising policy.

**Also enforced:** Rule 5 (never fabricate brand facts) is scanned too — no invented statistics, awards, press mentions, named clients, or partnerships. If the client did not state it in intake, it does not appear on the site. Reviews/testimonial schema is emitted ONLY for real, attributable reviews — never generated.

**Note for image-heavy verticals:** compliance applies to imagery, not just words. If a product's whole legal position is "this is not the real thing", stock photos of the real thing contradict it in the first place a reviewer looks. Flag this at intake and at handover.

---

## Multi-image products

```js
{ slug: 'aud-100-prop-notes', name: '…', price: 119,
  images: ['aud-100-prop-notes.webp', 'aud-100-prop-notes-2.webp', 'aud-100-prop-notes-3.webp'] }
```

- **First image is primary** — used on cards, OG tags, and `Product` schema's first `image` entry. A single-image product is just `images: [one]`; never special-case it.
- **`scripts/images.mjs`** processes every image identically (white 4:3 canvas, 96% fill, WebP q86) and names them `<slug>.webp`, `<slug>-2.webp`, …
- **Product page gallery:** primary image large, thumbnails beneath; click/tap swaps; keyboard-navigable (arrow keys, focus ring); **no lightbox library** — a CSS/state swap costs nothing. Only render the gallery when `images.length > 1`.
- **Schema:** `Product.image` takes the full array — Google prefers multiple images and it is one of the few genuinely free rich-result wins.
- **Performance:** primary `loading="eager"` (it is the LCP element), thumbnails and secondary images `loading="lazy"`. Never preload the whole gallery.
- **Crosscheck:** every filename in `images[]` exists in `public/images/`; no orphaned image files; alt text is per-image and descriptive (never "product image 2").

---

## Reference files

| File | Read when |
|---|---|
| `references/intake.md` | Mode 1 intake, updating `docs/PROJECT.md`, **Section H compliance language**, **Section M deploy target**, **Section O backend gate** |
| `references/nextjs.md` | Building or updating any page — App Router patterns, `generateStaticParams`, metadata API, JSON-LD, static-export vs Vercel, the `src/config/site.js` contract |
| `references/pages.md` | Page-by-page build requirements — sections, components, schema per page type, mobile, a11y, and what v8 removed |
| `references/seo.md` | Meta formulas, schema per page type, sitemap/robots, GSC + Bing + IndexNow, validity traps, freshness |
| `references/ai-visibility.md` | GEO/AEO/LLM visibility, the 12 authority signals, llms.txt, off-site reality |
| `references/agent-live.md` | **Vercel live layer** — MCP Streamable HTTP server, JSON API, live ACP/UCP, target-aware `.well-known` generation, live-layer crosscheck |
| `references/vercel.md` | **Vercel target** — `vercel.json`, GitHub workflow, env vars, connecting a domain later, Framework Preset trap, 404 troubleshooting, deploy verification |
| `references/cloudflare.md` | **Cloudflare target** — `_headers`, `_redirects`, ZIP upload, canonicalization, dashboard setup |
| `references/forms.md` | Any form — provider selection, the Web3Forms CORS method, the `resend` route, Turnstile, key-pending fallback, thank-you pages |
| `references/niche-variants.md` | Locale, age gates, disclaimers, payment variants, shop structures — the starting point per vertical |
| `references/backend.md` | **Client backend (backend = Yes → Cloudflare only)** — two-layer model, manifest, functions, admin panel, SEO Score, `schema.sql`, setup guide, import workflow, backend audit |

Every reference is current as of v8. Each opens with a **v8 delta** block where behaviour changed from v5/v6 — read it before the body. **On any conflict, SKILL.md wins.**

---|---|---|
| `references/intake.md` | Mode 1 intake, updating `docs/PROJECT.md`, **Section H compliance language**, **Section O backend intake** | **v8** |
| `references/nextjs.md` | Building or updating any page — App Router patterns, `generateStaticParams`, metadata API, JSON-LD, static-export vs Vercel, the `src/config/site.js` contract | **v8 (new)** |
| `references/vercel.md` | Vercel target — `vercel.json`, GitHub workflow, env vars, connecting a domain later, the Framework Preset trap, 404 troubleshooting, deploy verification | **v8 (new)** |
| `references/agent-live.md` | Vercel live layer — MCP Streamable HTTP server, JSON API, live ACP/UCP, target-aware `.well-known` generation, live-layer crosscheck | **v8 (new)** |
| `references/forms.md` | Any form — provider selection, the Web3Forms CORS method, the `resend` route, Turnstile, key-pending fallback, thank-you pages | **v8** |
| `references/pages.md` | Building or updating any page (content/structure per page type) | ⚠️ v6 — predates Next.js; treat SKILL.md MODE 3 as authoritative on structure |
| `references/seo.md` | Meta, schema, sitemap, robots, GSC tag, freshness, validity traps | ⚠️ v6 — principles still hold; ignore any static-HTML/Cloudflare-specific mechanics |
| `references/ai-visibility.md` | AI/GEO/AEO signals, 12 authority signals, llms.txt, schema, off-site reality | ⚠️ v6 — principles hold; the live layer supersedes its static-only assumptions |
| `references/cloudflare.md` | Cloudflare target — `_headers`, `_redirects`, ZIP upload, canonicalization | ⚠️ v6 — still accurate for the Cloudflare target; ignore any "default platform" framing |
| `references/niche-variants.md` | Locale, compliance, payments, age gate | ⚠️ v6 — see "Compliance guardrails" in SKILL.md for the language-constraint mechanism |
| `references/backend.md` | **Client backend (backend = Yes → Cloudflare target only):** architecture, two-layer model, manifest, functions, admin panel, SEO Score, `schema.sql`, dashboard setup + deploy guide, import workflow, backend audit | ⚠️ v6 — still accurate (backend is unchanged and Cloudflare-only) |

> **On conflict, SKILL.md wins.** Where a ⚠️ v6 reference contradicts SKILL.md v8 — framework, deploy target, forms, images, domain handling, docs layout — SKILL.md is authoritative. Those files were written for the static-HTML/Cloudflare-default era.

---|---|
| `references/intake.md` | Mode 1 intake, updating PROJECT.md, **Section O backend intake** |
| `references/pages.md` | Building or updating any page |
| `references/seo.md` | Meta, schema, sitemap, robots, GSC tag, freshness, validity traps |
| `references/ai-visibility.md` | AI/GEO/AEO signals, 12 authority signals, llms.txt, schema, off-site reality |
| `references/vercel.md` | **Vercel target: `vercel.json`, Next.js config, GitHub workflow, API routes, env vars, domain connection, Framework Preset trap** |
| `references/agent-live.md` | **Vercel live layer: MCP Streamable HTTP server, JSON API, live ACP/UCP, target-aware .well-known generation, live-layer crosscheck** |
| `references/cloudflare.md` | Cloudflare target: `_headers`, `_redirects`, ZIP upload, canonicalization |
| `references/nextjs.md` | **App Router patterns, `generateStaticParams`, metadata API, static export vs Vercel differences, `src/config/site.js` contract** |
| `references/niche-variants.md` | Locale, compliance, payments, age gate |
| `references/forms.md` | Any form — provider selection, the Web3Forms CORS method, the `resend` API route, Turnstile, thank-you pages |
| `references/backend.md` | **OPTIONAL client backend (backend = Yes): architecture, two-layer model, manifest, functions, admin panel, SEO Score, schema.sql, Cloudflare dashboard setup + deploy guide, import workflow, backend audit** |

---

## Working conventions

**Vercel sites — the repo is the project.**
`CLAUDE.md` + `docs/` live in the repo and travel with the code. Work in Claude Code; it auto-reads `CLAUDE.md` every session. Loop: edit `src/config/site.js` → `npm run build` → `npm run crosscheck` → commit → push → Vercel deploys. No ZIP, no upload.

**Cloudflare sites — Claude Project.**
Name: `[Site Name] — [domain or PENDING]`. Attach `docs/PROJECT.md` + `docs/keyword-map.md` (never in the deploy output). Project instructions: "This is the [Site Name] website project. Always read PROJECT.md before any task. keyword-map.md contains all blog topics — use the next unused cluster for each new post. If PROJECT.md shows Backend = Yes, deliver new blog/product content as *-import.json import files (not hardcoded HTML), and read references/backend.md."

**Mode 2 rule:** the docs are in context — read them directly, never ask the user to paste.

**Domain pending:** keep `SITE.domain = 'DOMAIN.com'`. Every file derives from it. To go live: change that ONE line → rebuild → deploy. **Never** instruct a find-and-replace across files.

**CLAUDE.md template (keep it under ~120 lines):**
```markdown
# [Site Name] — project instructions

[One line: what this is, framework, deploy target.]

## Non-negotiable: [compliance area, if any]
[Banned words. Required framing. "If a request would require breaking the above, stop and say so."]

## Architecture
`src/config/site.js` is the single source of truth. Adding one entry to PRODUCTS / CATEGORIES / POSTS
generates the page, route, meta, JSON-LD, sitemap entry and nav links. Never hand-write pages.
Never hand-edit generated files (llms.txt, .well-known/*, vercel.json, _headers) — edit the config.

## Rules
- `npm run build && npm run crosscheck` must pass before every push.
- One <h1> per page. Meta descriptions 110-158 chars. Titles <=60.
- Product images: WebP, white 4:3 frame, via `npm run images`.
- Emails entity-encoded (&#64;) everywhere, including JSON-LD.
- Never commit node_modules/, .next/, out/.

## Live placeholders
[domain / chat / form key / GSC — and what breaks while each is unset]

## Brand facts (only these are true — never invent more)
[founded, ships, order rules, payment, real milestones]
No invented statistics, awards, press, or named clients. Ever.
```
