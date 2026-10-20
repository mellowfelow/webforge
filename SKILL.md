---
name: webforge
description: Complete React/Next.js ecommerce website factory that builds, updates, deploys and optimizes sites on Vercel (default, via GitHub) or Cloudflare Pages (via ZIP), with a reply portal + order dashboard ON BY DEFAULT on both targets (skip only if the client declines), and an OPTIONAL client backend (CMS, Cloudflare-only). Uses SMTP as the only email transport (nodemailer on Vercel, a raw-socket SMTP client via Cloudflare Pages Functions on Cloudflare) — Web3Forms is retired. Use whenever the user mentions building a website, new site, updating an existing site, adding pages or products, running an SEO audit, AI visibility audit, technical audit, agent-ready audit, deploy check, connecting a domain, or managing a client login/backend/dashboard/CMS/reply portal. Triggers include "build me a site", "new client", "new project", "add", "update", "change", "fix", "audit", "check SEO", "ai visibility", "agent ready", "isitagentready", "llms.txt", "deploy check", "keyword cluster", "keyword map", "PROJECT.md", "CLAUDE.md", "vercel", "cloudflare pages", "connect domain", "product images", "live chat", "backend", "client dashboard", "CMS", "reply portal", "admin portal", "order dashboard", "whatsapp reply", "payment details", or any reference to a managed client site. Always read this skill before doing anything.
---

# WebForge v11.1 — React/Next.js Ecommerce Site Factory (dual-target: Vercel + Cloudflare, agent-ready + AA-accessible + design-polished by default)

## One prompt in. A fully built React/Next.js store out — mobile-first, technically perfect, SEO + AI-visibility optimized, agent-ready, deployable to Vercel (default, via GitHub) or Cloudflare Pages (via ZIP) — with a reply portal + order dashboard on by default on both targets.

You build production-ready Cloudflare Pages ecommerce websites that are fully optimized from day one — **mobile-first and fully responsive**, technically perfect, ranking-ready on all search engines, visible to AI assistants, and **agent-ready from day one**. Every site follows a proven structure. Do not improvise — follow this skill exactly.

This skill is self-contained. All SEO, technical SEO, AI visibility, agent-ready standards, schema, internal linking, keyword clustering, forms, content strategy, AND the optional client backend intelligence is built in. No external skill dependencies.

**v6 adds Agent-Ready Standards as a non-optional, baked-in default.** Every site ships with the full set of agent-discoverable files (`llms.txt`, `auth.md`, all `.well-known/` endpoints, `webmcp.js`, correct `_headers`) from day one, targeting a score of ~79/100 on isitagentready.com (Level 2 — Bot-Aware) on the Cloudflare Pages free plan. These are not extras — they are part of every ZIP.

**v9 adds the full Technical-SEO layer as a baked-in default:** split + image sitemaps, duplicate-content detection, an inverse-noindex audit, AVIF output, full **WCAG 2.2 AA accessibility** (`references/accessibility.md`), and a measured **Core Web Vitals / Lighthouse 95+ gate** (`references/performance.md`). Audit mode (MODE 4) now runs in two tiers — **static** (from the ZIP, gates the build) and **live** (post-deploy, new AUDIT F for performance + accessibility). Image optimisation is target-conditional: Vercel uses `next/image` + AVIF; Cloudflare keeps the pre-optimised pipeline, now emitting AVIF too.

**v9.1 adds the Visual Design Quality layer as a build gate** (`references/design-quality.md`): grid/tile uniformity, deliberate hero composition, section rhythm + alignment consistency, cohesive depth/colour/type, tasteful reduced-motion-safe micro-interactions, and a hard **no-bare-section** rule. Technical correctness alone no longer ships — a build must also be clean and genuinely attractive (crosscheck item 46).

**v5 (retained) — the optional client backend module.** Not every site uses it — it is a **requirement-gated, opt-in** addition decided per project during intake (Intake Section O). When enabled, the client gets a login at `/cms` where they manage content, blog posts, products, media, and SEO details — all stored in Cloudflare D1 + R2, all on Cloudflare Pages, no external host. **`/cms`, not `/admin`** — `/admin` belongs to the Reply Portal (Section P, default-Yes on every site now), and the two must never collide. Full detail in `references/backend.md`.

**v11.1 adds a live Rich Results Test pass to AUDIT F.** The pre-ship crosscheck's JSON-LD validity check only confirms schema *parses* — it can't confirm Google actually treats it as rich-result-eligible, since a block can be valid JSON and still miss a field Google requires for eligibility specifically. AUDIT F now runs Google's Rich Results Test live (via the in-app browser, no scriptable API for this tool) against one representative URL per distinct schema type shipped. See "AUDIT F" below and `references/seo.md` § JSON-LD validity.

**v11 makes the Reply Portal the default on every site, on both deploy targets (Intake Section P — default Yes, skip only if declined).** Orders and enquiries are stored in Upstash Redis (Vercel) or KV (Cloudflare, via Pages Functions), a passcode-gated admin dashboard appears at `/admin/` (hub, orders list, enquiry list, payment-details composer, reply-enquiry composer), payment details are pasted once and auto-split into individually-copyable fields (`parsePaymentDetail` — fits any country/payment rail with zero code changes), WhatsApp pre-filled reply links mirror the same fields, and a public `/order/payment-details/` page lets the customer tap-to-copy each one. Both WhatsApp and email checkout channels save to the dashboard — WhatsApp checkout is wired end-to-end, never a bare chat link. **Web3Forms is deleted entirely; SMTP is the only default form/email transport, on both targets** (Cloudflare via a raw-socket SMTP client in a Pages Function, since its runtime has no nodemailer support). All emails are the **light** shell (white body, dark brand header band, brand as accent only — never a dark body, which Zoho/Gmail dark mode force-invert); every brand/location/currency/payment specific comes from one `SITE.reply` config block. See Section P, "Reply Portal system", `references/forms.md` (email) and `references/reply-portal.md` (storage/auth/admin/WhatsApp, both targets).

**v4 (retained) — the hard-won production lessons** from live client builds: mobile-first overflow rules, flexible product-image handling, the JSON-LD validity traps, and three first-class audit modes (Technical, SEO, AI-Visibility) that now include Agent-Ready as a fourth sub-audit. (The Web3Forms CORS method this section used to document is retired — see "Forms" below.)

---

## NON-NEGOTIABLE RULES (read first, apply always)

These are absolute. They come from real builds that broke and were fixed. Never regress them.

1. **Mobile-first, fully responsive.** Design and verify mobile FIRST, then scale up. Zero horizontal scrolling on any viewport. See "Mobile-first standards" below — every build and every audit checks this.
2. **Forms: SMTP only, on both deploy targets. Web3Forms is retired.** One constant in `src/config/site.js` selects the provider (`smtp` default, or opt-in `resend` on Vercel with a verified domain). `lib/mailer.js` (Vercel, nodemailer) or `functions/_shared/mailer.js` (Cloudflare, raw-socket SMTP via `cloudflare:sockets`) — same `SMTP_HOST/PORT/USER/PASS` + `SMTP_FROM` env vars either way, lazy singleton, returns `{sent:false}` when unconfigured (never crashes). See "Forms — SMTP only, both targets".
3. **Exactly one `<h1>` per page.** Hero sliders: `<h1>` on slide 1 only; other slides are styled `<div>`/`<p>`.
4. **Validate every JSON-LD block with a JSON parser before shipping.** Broken schema is worse than none. No bare `Product` entities on non-product pages.
5. **Never fabricate brand facts — but always preserve the client's REAL ones.** Do not INVENT founders, awards, partnerships, revenue, statistics or reviews that do not exist. But real content the client provides — or that already exists on the site you are migrating (reviews, ratings, awards, testimonials, named clients, milestones) — is legitimate: carry it over faithfully and record its source in `docs/PROJECT.md`. This rule bans invention, never real content. Migrating a site with real reviews must PRESERVE those reviews, not strip them.
6. **Always run the pre-ship crosscheck** (see "Pre-ship crosscheck") before producing any ZIP. Analyze → fix → re-analyze.
7. **Agent-Ready files are mandatory on every site.** The full set of agent-ready files (Sections A–N in "Agent-Ready Standards") ships on every build regardless of client request, on both deploy targets. See "Agent-Ready Standards" and the pre-ship crosscheck.
8. **Framework is React + Next.js (App Router). Never hand-roll a prerenderer.** One codebase, two targets: `TARGET=static` → `output: 'export'` for Cloudflare ZIP upload; default → Vercel with real API routes. See "Deploy targets" and "MODE 3".
9. **The domain lives in exactly ONE place: `SITE.domain` in `src/config/site.js`.** Every domain-bearing file (`robots.txt`, `llms.txt`, `sitemap`, all `.well-known/*`, `vercel.json`, canonicals, OG tags) is GENERATED from it at build time by `scripts/gen-agent-files.mjs`. Never hand-write a domain into a file. Never instruct the user to find-and-replace a domain across files. `DOMAIN.com` is permitted while a domain is pending; the crosscheck FAILS a production deploy that still contains it.
10. **Never claim a capability the site does not have.** If `SearchAction` schema is emitted, a real search must exist. If `mcp/server-card.json` advertises tools, they must resolve on that target. If `api-catalog` lists endpoints, they must return. Static targets declare only what static files can honour.
11. **Compliance language is testable, not remembered.** If a vertical has banned words or required framings (Intake Section H), they go in `CLAUDE.md` AND are scanned across the entire build output by the crosscheck. A violation is a ship-stopper. See "Compliance guardrails".
12. **Strategy docs never ship to the public web.** `docs/PROJECT.md`, `docs/keyword-map.md`, `docs/blog-plan.md`, and (when keyword-engine ran) `docs/taxonomy-map.md` + `docs/starter-product-list.csv` + `docs/faq-bank.md` + `docs/product-gaps.md` all live in `docs/` — never in `public/`, never in the deploy output. A publicly downloadable keyword strategy hands competitors the whole plan.
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
| Forms / email | `smtp` (default) via `/api/contact`, or opt-in `resend` | `smtp` via Pages Functions (`functions/api/contact.js`) — same env vars, raw-socket client instead of nodemailer |
| API routes | **Yes** — real Next.js API endpoints | No (static export) — but **Pages Functions** (`functions/`) run alongside it, see below |
| Agent ceiling | Higher — live MCP server, real product/search APIs, real ACP/UCP endpoints | ~79 (isitagentready free-plan cap — accepted, do not chase) |
| Reply Portal | **Yes** — default (Section P), Upstash Redis, passcode-gated `/admin/` | **Yes** — default (Section P), Cloudflare KV via Pages Functions, passcode-gated `/admin/` |
| Client backend | Not supported (see below) | **Yes** — D1 + R2 + Pages Functions |

**Target and backend are TWO INDEPENDENT questions.** Section M picks the platform; Section O picks the backend (default **No**). Never conflate them — all four combinations are asked, three are buildable:

| Target | Backend | Supported? | Use when |
|---|---|---|---|
| **Vercel** | No | ✅ **default** | Most new sites. GitHub workflow, real agent endpoints, `resend` available once a domain is live. |
| **Cloudflare** | No | ✅ **fully supported** | Client is already on Cloudflare; you prefer ZIP upload; no Git workflow wanted; existing sites you maintain. A static Next.js export — no D1, no R2, no CMS dashboard. (It does ship `functions/` for forms + the Reply Portal, Section P — that's a separate, always-present mechanism, not the opt-in CMS backend. See "Deploy targets" table above.) |
| **Cloudflare** | Yes | ✅ | Client needs to edit content themselves at `/cms`. Adds D1 + R2 + Pages Functions. |
| **Vercel** | Yes | ❌ | Not supported. The backend is built on D1 + R2 + Pages Functions. Do not improvise a port. If the client needs both, the target is Cloudflare. |

> **Cloudflare does NOT imply a backend.** "Cloudflare Pages" and "client CMS" are unrelated choices. A Cloudflare site with backend = No is a plain static site: `npm run zip` → upload → done. It is a first-class path, not a fallback.
>
> **Backend = Yes DOES imply Cloudflare.** That is the only forced direction.

**Rules:**
- **Vercel is the default**, but it is a default, not a requirement — if the user says Cloudflare, build Cloudflare without argument or re-litigation.
- Cloudflare Pages remains fully supported for existing sites, for ZIP-upload workflows, and for any site needing the client backend.
- **The backend module is Cloudflare-only.** If backend = Yes, the target is Cloudflare Pages — state this plainly at intake rather than half-porting it.
- **Cloudflare is deliberately static for pages.** Its ~79 agent score is a free-plan ceiling, not a build defect. `functions/` exists for forms + the Reply Portal (real correctness requirements) and, when backend = Yes, the CMS — never add it purely to chase agent-readiness points.
- **Both targets ship the full static agent-ready file set (A–N).** Vercel additionally ships live endpoints (Pass 2).
- Both targets use `trailingSlash: true`. Every route is a directory with `index.html`.

---

## Operating modes

Identify which mode applies before doing anything else.

**Mode 1 — Intake & Strategy**
Trigger: "build me a site", "new client", "new project", "new site", or user pastes full requirements.
Action: Intake (includes **Section M — deploy target** and **Section O — Client Backend**) → check for an existing keyword-engine session (Step 0) → brand entity statement → keyword strategy → content strategy → `CLAUDE.md` + `docs/PROJECT.md` + `docs/keyword-map.md` + `docs/blog-plan.md` → confirm with user → hand off to Mode 3.

**Mode 2 — Site Update**
Trigger: existing site + "add", "update", "change", "new page", "new product", "new blog post", "fix".
Action: Read PROJECT.md from Project context. **New blog post:** take the next `Pending` row from `docs/blog-plan.md`, flip it to `Built` once delivered. Make targeted changes, propagate ripple effects (sitemap, internal links, schema counts, llms.txt, agent-ready files). Run full-site audit. Deliver complete deploy-ready ZIP.
**Backend sites:** if PROJECT.md shows backend = Yes, new blog posts / products are delivered as **import files** for the client to upload in `/cms → Import` — NOT hardcoded into HTML.

**Mode 3 — Full Build**
Trigger: user confirms intake / says "build the site".
Action: Build every file in one pass — mobile-first + technical + SEO + AI visibility + **agent-ready** + forms + **the Reply Portal** (default on every build now, both targets — same weight as the shop pages: not indexed, but no sales without it) + deploy config. **If backend = Yes:** also generate the `functions/`, `cms/`, `cms-manifest.json`, `schema.sql`, and `CLOUDFLARE-SETUP.md`. Pre-ship crosscheck (including the Reply Portal crosscheck and the agent-ready checklist). Output single deployable ZIP.

**Mode 4 — Audit modes (Technical / SEO / AI-Visibility / Agent-Ready) + Auto-Fix**
Trigger: "audit", "check SEO", "AI visibility check", "technical audit", "agent ready", "isitagentready", "something's broken", "fix my SEO", "deploy check", or user uploads a site ZIP.
Action: Run the requested audit (or all four). Produce a prioritized report with before/after scores. Walk through fixes; each fix RE-APPLIES the relevant build standard from this skill. Re-run the pre-ship crosscheck. If files changed, deliver a complete deploy-ready ZIP of the whole site.

> Audit modes are how an already-built, already-SEO'd site gets re-checked for technical + SEO + AI + agent-ready issues. They are first-class — see "MODE 4" for the sub-audits and exactly what each checks.

---

## MODE 1 — Intake & Strategy

**Step 0 — Check for an existing keyword-engine session.** Before Section J or Step 3 below, check whether `docs/keyword-map.md` (and its siblings `blog-plan.md`/`faq-bank.md`/`product-gaps.md`) already exist for this project — keyword-engine may have already run. **If they exist, they win:** skip Section J's own keyword collection and skip Step 3's own clustering entirely; attach and read keyword-engine's docs instead (see the doc table in "After intake is complete" → Step 5, and `references/intake.md` § J). Only run Section J + Step 3 as WebForge's own lightweight fallback when no keyword-engine docs exist for this project.

**Same check, before Sections I and K.** Also check whether `docs/taxonomy-map.md` / `docs/starter-product-list.csv` exist — produced by keyword-engine's Phase 0, a separate (and separately optional) step from the keyword docs above. Unlike Section J, these never skip the section outright — they **pre-fill** it: present the taxonomy tree / starter product list for the client to confirm or adjust, instead of asking from scratch. Section K still collects photos separately regardless, since Phase 0 never produces those.

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
- **E — Checkout & Payment:** WhatsApp checkout Y/N, email/order-form checkout Y/N (independent, plain Yes/No each — but whichever is Yes gets the full form: required email, phone, address, payment method, and an unconditional confirmation email, never a shortcut version — see "Reply Portal system" → WhatsApp), payment methods
- **F — Live Chat (chat hub — multiple channels):** Pick any combination. Two kinds, treated differently:
  - **Link channels** (free, zero JS, no CSP impact, no performance cost): WhatsApp (number), Telegram (@handle or t.me link), Facebook Messenger (page ID), Signal (link), Email, Phone.
  - **Widget channels** (inject a third-party script — costs performance, needs a CSP entry, one only): Tawk.to (property/widget ID), Crisp (website ID), JivoChat (widget ID).
  - **Rule:** unlimited link channels; **at most ONE widget channel**. If the user asks for two or more widgets, warn that each adds a blocking third-party script and ask them to choose one.
  - Rendered as a single floating button that expands to the enabled channels. If exactly one link channel and no widget, render it as a direct button (no expander).
- **G — Optional Pages:** Tracking, account, wholesale, compare, finance calculator. **Plus, if the catalog supports them:** Brand pages (y/n — 3+ brands also triggers a Brand Hub aggregator; under 3, individual pages only), Best Sellers (y/n), New Arrivals (y/n), Sale/Deals (y/n). These four are the conditional page types keyword-engine's Guarantee Flag system is built around — say no to all if the catalog doesn't support them.
- **H — Compliance:** Age gate, disclaimer, locale, GDPR. **Plus language constraints (ask every client):** banned terms, required framings, prohibited claims, and the authority behind each (regulator / platform ad policy / statute). These become a `CLAUDE.md` block and a blocking crosscheck scan — see "Compliance guardrails".
- **I — Shop Structure:** Structure type A/B/C/D, categories, subcategories. **First check whether a keyword-engine session already produced `docs/taxonomy-map.md` for this project** — if so, present its category→subcategory tree for confirmation instead of asking from scratch.
- **J — Keywords:** Primary keyword, 10-20 secondary keywords. **Skip this section if a keyword-engine session already produced `docs/keyword-map.md` for this project (see Step 0 above)** — attach its docs instead of collecting this.
- **K — Initial Products:** featured products with name, price, category, description, badge, **image(s)**. **First check whether a keyword-engine session already produced `docs/starter-product-list.csv` for this project** — if so, present its category/subcategory/product/brand/confidence rows for confirmation instead of asking from scratch (photos are still collected separately either way). Ask for: **2000px+ on a plain white background, product filling the frame, consistent angle and lighting.** Explain why: white-background photos blend seamlessly into the white product frame and need no processing. Photos with their own backgrounds still work but will letterbox. Multiple images per product supported (first is primary). Images optional — placeholders are generated if absent.
- **L — Forms / email transport:** Two providers — **Web3Forms is retired:**
  - `smtp` — **default, both targets.** Branded HTML emails. Vercel: nodemailer (`lib/mailer.js`). Cloudflare: a raw-socket SMTP client run from a Pages Function (`functions/_shared/mailer.js`) — same env vars, same behaviour. Requires `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, and optionally `SMTP_FROM`. Without them, `sendMail()` returns `{sent:false}` — site still works via WhatsApp/chat fallback.
  - `resend` — Vercel only, opt-in, first-party sending, **requires a verified domain**.
  Collect: destination emails (contact/order/wholesale — each optionally its own env var, `CONTACT_EMAIL`/`ORDER_EMAIL`/`WHOLESALE_EMAIL`, falling back to a config default), and the SMTP credentials or "pending". Optionally: Cloudflare Turnstile site key for spam protection. See `references/forms.md`.
- **M — Hosting / deploy target (ask plainly, both are first-class):** **Vercel (default)** — GitHub repo, auto-deploy on push, real API routes, higher agent ceiling, `resend` available once a domain is live. **Cloudflare Pages** — ZIP upload, no Git needed, static, ~79 agent cap (free-plan ceiling, not a defect). **Choosing Cloudflare does NOT mean adding a backend** — that is a separate question (Section O) and defaults to No. Only the reverse is forced: backend = Yes → Cloudflare. See "Deploy targets" for the full matrix.
- **N — Brand authority facts (for AI visibility, truthful only):** founding year, founding location/HQ, what makes them different, milestones, geographic footprint, real named individuals/awards/partners IF the client actually has them (never invented). **If a named individual actually writes/reviews the blog content**, also collect name/role/LinkedIn — feeds Article schema's optional `Person`-author path (`references/seo.md`); absent that, Article stays Organization-authored.
- **O — Client Backend (OPTIONAL, the opt-in gate):** Backend Yes/No. If **No** → build a pure static site (skip all backend work). If **Yes** → collect: which collections the client manages (pages text, blog, products, custom types), admin user email(s), whether they edit SEO details (focus keyword + meta + SEO Score panel), media uploads Y/N, and edits-go-live mode (instant vs publish). Full sub-form in `references/intake.md` → Section O.

> **Gate rule:** Section O is asked for every new site, but a backend is added ONLY when the user explicitly says Yes. **Default is No.** A "No" here is not a lesser build — most sites should not have a backend. It is only worth it when the client genuinely wants to edit content themselves without you.
>
> **Do not infer a backend from the target.** Cloudflare + No is a normal, fully supported combination (see "Deploy targets"). Never assume that a Cloudflare site wants a CMS.

- **P — Reply Portal (default Yes, both targets — skip only if declined):** Reply Portal Yes/No. **Default Yes.** When **Yes**, the site gets:
  - Order/enquiry store — Upstash Redis on Vercel, Cloudflare KV via Pages Functions on Cloudflare (`references/reply-portal.md` §2)
  - Passcode-gated admin dashboard: `/admin` hub (live counts) + `/admin/orders/` + `/admin/enquiries/`
  - Payment-details composer at `/admin/send-payment-email/` — paste the real detail once, `parsePaymentDetail` auto-splits it into individually-copyable fields (any country, any payment rail, zero code changes) + WhatsApp reply panel
  - Enquiry-reply composer at `/admin/reply-enquiry/`
  - A public `/order/payment-details/` page where the customer taps each field to copy it, and `/order/confirm-payment/` to upload a payment screenshot
  - WhatsApp channel tagging on orders — both checkout channels (WA + email) save to the dashboard; WhatsApp checkout mints and saves a real order, never a bare chat link
  - Payment terms displayed consistently: `paymentTermsLines()` → WA text, email HTML, and the composer preview always match
  - Order notification email to admin includes "Reply in Dashboard →" CTA; confirmation to customer says "watch for payment-details email" — NOT premature payment instructions

  **Collect when Yes:**
  - `ADMIN_PASSCODE` — server-only, gated via `X-Admin-Passcode` header; never exposed to the browser
  - **Vercel:** Upstash Redis REST URL + Token (Vercel Storage tab → "Create Database"; the client checks 5 common prefix variants automatically). **Cloudflare:** a KV namespace bound in Pages → Settings → Bindings (`references/cloudflare.md`).
  - WhatsApp number (for WA checkout + admin WA reply panel)

  **Mandatory theme + light-email rule (applies even when portal = No for emails):** all outbound HTML emails are **light** (white card, dark brand header band, near-black text, brand as accent only) — never a dark-background body, which Zoho/Gmail dark mode force-invert (see `references/forms.md`) — **this applies on Cloudflare too.** Brand/location/currency/payment specifics come from the one `SITE.reply` block in `config/site.js`; the accent = `reply.brand.primary`, header band = `reply.brand.headerDark`. Admin pages read their accent from the same config so the portal follows the site theme.

  **Section P requires:** `smtp` form provider (or `resend` on Vercel). No longer requires Vercel — Cloudflare runs it via Pages Functions.

  > Gate rule: Section P is asked for every site, either target. **Default is Yes** — build it unless the client explicitly declines. Skip only for sites genuinely without orders/enquiries to manage.

### After intake is complete

**Step 1 — Brand entity statement**
```
[Site Name] is a [location]-based [product type] brand established in [year], offering [key differentiator] for [target customer]. [Site Name] ships [geographic coverage] and specializes in [product range]. [One sentence on what makes them different.]
```
Goes into: homepage hero subtext, homepage "About [Brand]" authority section, about page opening, Organization/Store schema `description`, FAQ "What is [Brand]?" answer, llms.txt, and `mcp/server-card.json` description.

**Step 2 — Form provider + deploy target + reply portal confirmation.** Defaults: Vercel + SMTP + Reply Portal **Yes**. State the consequences plainly: SMTP gives branded first-party emails (Zoho/Gmail/any SMTP) on either target — needs credentials in the platform dashboard, site still works via WhatsApp/chat fallback without them; `resend` gives first-party sending on Vercel but needs a verified domain. Reply Portal is on by default on both targets now — it adds `/admin/` order/enquiry dashboard + the SMTP requirement; only turn it off if the client says they don't want it. If backend = Yes, target is Cloudflare Pages. Confirm with user.

**Step 3 — Keyword strategy.** **Skip this step if Step 0 found an existing keyword-engine session** — use its `keyword-map.md`/`blog-plan.md`/`faq-bank.md`/`product-gaps.md` as-is instead. Otherwise (no keyword-engine docs for this project): cluster primary + secondary keywords, assign by intent. Show assignment, wait for confirmation.

**Step 4 — Content strategy.** Map pillar → cluster → supporting. Quick wins first. 12-week calendar.

**Step 5 — Generate the docs** (templates in `references/intake.md`). `CLAUDE.md` + `docs/PROJECT.md` + `docs/keyword-map.md` + `docs/blog-plan.md` always; `docs/taxonomy-map.md` + `docs/starter-product-list.csv` + `docs/faq-bank.md` + `docs/product-gaps.md` only when a keyword-engine session produced them. These are SEPARATE FILES WITH SEPARATE JOBS — never merge them:

| File | Location | Job | Read by |
|---|---|---|---|
| `CLAUDE.md` | repo root | **Rules only.** Compliance language, architecture note, build/deploy commands, live placeholders, verified brand facts, "never invent facts". Keep under ~120 lines — it is auto-read on EVERY Claude Code session, so length costs tokens every time. | Claude Code, automatically, every session |
| `docs/PROJECT.md` | `docs/` | **The full record.** All intake data, brand entity statement, keyword strategy, AI visibility plan, brand authority facts, form provider, deploy target, GSC status, backend block. | On demand |
| `docs/taxonomy-map.md` (if keyword-engine's Phase 0 ran) | `docs/` | Category→subcategory tree, top brands/products with confidence tags, price ranges. Answers intake Section I instead of asking from scratch. | At intake, Section I |
| `docs/starter-product-list.csv` (if keyword-engine's Phase 0 ran) | `docs/` | Flat category/subcategory/product/brand/confidence list. Answers intake Section K instead of asking from scratch. | At intake, Section K |
| `docs/keyword-map.md` | `docs/` | **Keyword → page assignments**, exclusions. Page-level keyword targets only — the blog queue lives in `blog-plan.md`, not here. | When writing content |
| `docs/blog-plan.md` | `docs/` | **The blog build queue.** Every planned post with a `Pending`/`Built` Status field — take the next `Pending` row for each new post, flip it to `Built` once published. Generated by WebForge's own Step 4 when no keyword-engine session ran; generated more richly by keyword-engine (full briefs, schema, 12-week calendar) when it did. | When writing a new blog post |
| `docs/faq-bank.md` (if keyword-engine ran) | `docs/` | Homepage + FAQ-page question content, pre-written answers, schema blocks. | When building the FAQ section/page |
| `docs/product-gaps.md` (if keyword-engine ran) | `docs/` | Which product/category pages to create, priority-ranked. | When deciding new product/category pages |

**`docs/` is never published.** It is excluded from `public/` and from the deploy output. (Formerly `keyword-cluster.txt` shipped in the site root — that leaked the entire keyword strategy to anyone who guessed the URL. Fixed.)

**Step 6 — Project setup card.** Working style depends on target:
- **Vercel:** the repo IS the project. `CLAUDE.md` + `docs/` travel with the code; work in Claude Code; push to deploy.
- **Cloudflare:** Claude Project named `[Site Name] — [domain or PENDING]`; attach `docs/PROJECT.md` + `docs/keyword-map.md` + `docs/blog-plan.md` (+ `docs/taxonomy-map.md` + `docs/starter-product-list.csv` + `docs/faq-bank.md` + `docs/product-gaps.md` if keyword-engine produced them).

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
│   │   ├── brands/page.jsx          ← Brand Hub, only if 3+ brands stocked
│   │   ├── brands/[slug]/page.jsx   ← individual brand pages, if Section G enabled
│   │   ├── best-sellers/page.jsx    ← if Section G enabled
│   │   ├── new-arrivals/page.jsx    ← if Section G enabled
│   │   ├── sale/page.jsx            ← if Section G enabled
│   │   ├── blog/page.jsx
│   │   ├── blog/[slug]/page.jsx
│   │   ├── search/page.jsx     ← real search (see "Search")
│   │   ├── about|contact|wholesale|faq|cart|order/page.jsx
│   │   ├── shipping|refund|privacy|terms/page.jsx
│   │   ├── thank-you-contact|thank-you-order|thank-you-wholesale/page.jsx
│   │   ├── checkout/page.jsx       ← order number minted client-side; WA + email both save to store
│   │   ├── thank-you-order/page.jsx ← "watch for payment-details email" (NO premature instructions)
│   │   ├── not-found.jsx
│   │   ├── sitemap.js         ← Next.js native, reads SITE.domain
│   │   ├── robots.js          ← Next.js native, reads SITE.domain; disallows /admin/ (+ /cms/ too if backend = Yes)
│   │   ├── admin/             ← REPLY PORTAL (Section P — default Yes)
│   │   │   ├── layout.jsx             ← PasscodeGate wrapper, AdminNav, robots:noindex
│   │   │   ├── page.jsx               ← hub: live order/enquiry counts, recent lists
│   │   │   ├── orders/page.jsx        ← list: status/channel badge, delete
│   │   │   ├── orders/[id]/page.jsx   ← detail: customer/payment/items, send-payment-details link
│   │   │   ├── enquiries/page.jsx     ← list: filter pills (all/contact/wholesale/new/replied)
│   │   │   ├── enquiries/[id]/page.jsx ← detail: contact/message, reply link
│   │   │   ├── send-payment-email/page.jsx  ← paste detail → parsePaymentDetail preview → send; WA panel
│   │   │   └── reply-enquiry/page.jsx ← compose reply with live preview
│   │   ├── order/
│   │   │   ├── payment-details/page.jsx  ← public, ?id= — parsed fields as CopyField, tap to copy
│   │   │   └── confirm-payment/page.jsx  ← public, ?id= — screenshot upload
│   │   └── api/               ← VERCEL ONLY (omitted from static export — Cloudflare mirrors this via functions/, see below)
│   │       ├── contact/route.js       ← ONE handler for order + contact + wholesale (by `formName`) — checkout's
│   │       │                            "Place order" AND "Order via WhatsApp" buttons both POST here; saves the
│   │       │                            order/enquiry, sends the shop notification, and — for orders, EITHER
│   │       │                            channel, unconditionally — orderConfirmationEmail() to the customer
│   │       ├── order/payment-details/route.js ← GET parsed fields for the public copy page
│   │       ├── order/confirm-payment/route.js ← POST screenshot → email + markPaymentConfirmed
│   │       └── admin/                 ← REPLY PORTAL API (Section P = Yes)
│   │           ├── orders/route.js          ← GET list
│   │           ├── orders/[id]/route.js     ← GET, DELETE, PATCH (mark sent)
│   │           ├── enquiries/route.js       ← GET list
│   │           ├── enquiries/[id]/route.js  ← GET, DELETE
│   │           ├── reply-enquiry/route.js   ← POST send reply email
│   │           └── send-payment-email/route.js ← POST send payment details email
│   ├── components/
│   │   ├── Nav.jsx · Footer.jsx · ChatHub.jsx · ProductCard.jsx
│   │   ├── QtyStepper.jsx · WebForm.jsx · Breadcrumbs.jsx
│   │   ├── CopyField.jsx        ← one payment-detail row, click-to-copy, 2s "copied" state
│   │   └── admin/               ← REPLY PORTAL components (Section P = Yes)
│   │       ├── PasscodeGate.jsx        ← passcode entry gate (shared by all /admin/* pages)
│   │       ├── AdminNav.jsx            ← Dashboard · Orders · Enquiries nav + sign out
│   │       ├── AdminPasscodeContext.jsx ← passcode context, avoids prop-drilling into every page
│   │       ├── StatusBadge.jsx         ← order/enquiry status pill
│   │       └── WhatsAppSendPanel.jsx   ← pre-filled WA link + copy-message fallback
│   ├── lib/                     ← REPLY PORTAL lib (Section P = Yes)
│   │   ├── mailer.js            ← SMTP nodemailer lazy singleton; {sent:false} when unconfigured
│   │   ├── redis.js             ← @upstash/redis client; checks 5 credential prefix variants
│   │   ├── orderStore.js        ← StoredOrder (channel:'whatsapp'|'email'); one hash, in-memory sort
│   │   ├── enquiryStore.js      ← StoredEnquiry (type, status:'new'|'replied'); same pattern
│   │   ├── adminAuth.js         ← X-Admin-Passcode header check (server-only)
│   │   ├── order.js             ← parsePaymentDetail(), money(), paymentMethodParts(), paymentTermsLines/Html()
│   │   └── whatsapp.js          ← waLink, waLinkTo, waOrderLink, waPaymentDetailsMessage, WA_HEADER
│   ├── utils/emailTemplates.js  ← shell/field/callout/button primitives + one function per email type
│   └── styles/globals.css
├── functions/                   ← CLOUDFLARE ONLY — Pages Functions, mirrors src/app/api/** above
│   ├── _shared/                 ← redis.js has no Cloudflare equivalent (KV used directly); orderStore.js,
│   │   │                          enquiryStore.js, adminAuth.js, mailer.js (raw-socket SMTP), order.js,
│   │   │                          emailTemplate.js, config.js — same logic as src/lib/* above, KV/env-bound
│   ├── api/                     ← same paths as src/app/api/** — contact.js (order+contact+wholesale, unified,
│   │   │                          same as Vercel), order/payment-details.js, order/confirm-payment.js,
│   │   │                          admin/orders.js(+[id]), admin/enquiries.js(+[id]),
│   │   │                          admin/send-payment-email.js, admin/reply-enquiry.js
│   └── _middleware.js           ← CORS headers for the admin API surface
                                    (NO wrangler.toml — direct/drag-and-drop ZIP upload never reads it;
                                     KV binding + nodejs_compat flag are set in the dashboard instead, see
                                     references/cloudflare.md → "Forms & Reply Portal")
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
│   ├── keyword-map.md
│   ├── blog-plan.md
│   ├── faq-bank.md            ← only if a keyword-engine session ran
│   └── product-gaps.md        ← only if a keyword-engine session ran
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
  // v9: target-conditional images. Vercel optimises at runtime (AVIF+WebP, responsive).
  // Cloudflare static export can't run the optimiser, so it serves pre-optimised files
  // from scripts/images.mjs (which now emits AVIF + WebP). See references/performance.md.
  images: isStatic
    ? { unoptimized: true }
    : { formats: ['image/avif', 'image/webp'] },
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

`scripts/images.mjs`, for each photo in `assets/product-photos/` (accept `.jpg/.jpeg/.png/.webp/.avif/.tiff` sources):
1. **Trim** the uniform background border — **with a guard**: keep the trim only if neither dimension collapses below ~12% of the original (protects photos where the subject reaches an edge). This is what removes the built-in whitespace that makes a product float tiny in the frame.
2. **Scale the trimmed product to fill ~90% of a WHITE 4:3 1600×1200 canvas, ENLARGING small or whitespace-heavy sources** (`kernel:'lanczos3'`, no `withoutEnlargement`). This is the uniformity trick — a low-res or heavily-padded source now fills the card like every other product instead of sitting small in a sea of white.
3. **Sharpen only when upscaling** (`.sharpen({sigma:1})` when the fill scale > ~1.1) to recover crispness lost to enlargement; leave already-large photos untouched so they don't get halos.
4. **Adaptive quality (stay under the 150 KB budget):** encode to a **buffer**, stepping WebP quality down from ~88 (floor ~40) and AVIF from ~62 (floor ~40) only until the result is under ~145 KB, then **write the chosen buffer once**. (Repeated `.toFile()` to the same path throws EINVAL/UNKNOWN on Windows under AV scanning — write once, and wrap the write in a short retry with backoff.) Output `<slug>.webp` + `<slug>.avif`; multiple images → `<slug>-2.*`, etc. Cloudflare serves via `<picture>`; Vercel `next/image` negotiates. See `references/performance.md` → image handling.
5. **Emit a contact sheet** (`docs/_contact-sheet.png`) and **show it to the user for approval before shipping**. Also report any source whose trimmed short side is < ~500 px — those had to be upscaled and are the reshoot candidates (list them for the client).

**The frame is WHITE, not a brand tint** — `.product-frame { background:#fff; aspect-ratio:4/3 }` with `object-fit:contain`. This is the whole trick: white-background photos blend invisibly and appear to fill the card; photos with their own background read as a deliberate photo tile. No cutouts, no risk, no white boxes on tinted panels.

**Why 4:3, not 1:1:** most products (banknote stacks, boxes, flat-lays) are landscape. A landscape product in a square frame can only ever fill ~66% of it — guaranteed dead space no matter the processing.

Always: correct `width`/`height` (or `next/image` intrinsic sizing) matching the canvas ratio (CLS = 0), descriptive alt, `loading="lazy"` (eager for the LCP image). Pages call the SmartImage wrapper (Vercel → `next/image`; static → `<picture>` AVIF+WebP), never a raw `<img>` — see `references/nextjs.md`.

**Crosscheck asserts:** every product has an image; every asset is ≥1.6× its rendered CSS size; no image under-fills its frame by more than 45%; **both AVIF and WebP present for each image**; markup uses SmartImage (never a raw `<img>`).

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
- **Backend = Yes:** the `/cms` dashboard is mobile-first too.

**Technical (every page):**
- GSC verification meta; IndexNow key meta + `[key].txt` at root.
- Canonical (absolute, trailing-slash consistent).
- All images WebP with width/height + `loading="lazy"` (hero/logo `eager`).
- CSS minified (keep `styles.source.css`); JS terser-minified + `defer` (keep `script.source.js`).
- `_headers`: CSP (whitelist form API + chat widget + wss), `Cache-Control` for assets, HTML revalidation. **Agent-Ready Link header on `/*`.** See Agent-Ready Section M.
- `_redirects`: www→apex and http→https with `301!` FIRST.
- HTTPS only, no mixed content.
- **Backend = Yes:** `/cms/*` must NOT be cached (`Cache-Control: no-store`); CSP `connect-src` includes `'self'`.

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
- **No duplicate body content:** product descriptions and category intros are unique — no boilerplate reused across items (crosscheck hashes and compares them). Titles/descriptions/H1 are already unique.
- **No unintended noindex/nofollow:** only the intended noindex pages (thank-you, not-found) carry it; every indexable page is index,follow, and no important internal link is `rel="nofollow"`.
- **Sitemaps are split + image-aware:** a sitemap index references child sitemaps (pages, products, categories, blog); product entries carry `<image:image>`. Generated by `app/sitemap.js` via `generateSitemaps()` — see `references/seo.md`.

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
- FAQPage schema on homepage + FAQ page; direct-answer format — homepage FAQ 40–55 words, FAQ page up to 65 words, never exceeding 65.
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

**Accessibility (WCAG 2.2 AA — every page; full spec in `references/accessibility.md`):**
- Skip-link → `<main id="main">`; one `<h1>`, no skipped heading levels; semantic landmarks (`header`/`main`/`footer`/labelled `nav`).
- Every image has `alt` (decorative = `alt=""` + `aria-hidden`); every form control has a bound label; every icon-only button has `aria-label`.
- Full keyboard operability, visible `:focus-visible` never obscured by the sticky header, logical focus order, no positive `tabindex`, no `outline:none` without a replacement.
- Colour contrast ≥ 4.5:1 (text) / 3:1 (large text + UI); never colour-only status. Form errors use `aria-invalid` + `aria-describedby` in an `aria-live` region.
- `prefers-reduced-motion` honoured. Static parts gate the build; contrast + keyboard are verified live in AUDIT F.

**Performance (Core Web Vitals — both targets; full spec in `references/performance.md`):**
- Structural (gates the build): AVIF+WebP, correct dimensions (CLS=0), lazy/eager, minified CSS/JS + `defer`, no render-blocking resources, route-split bundles, homepage < 1.5MB.
- **Fonts: self-host the woff2 files, never a Google Fonts `@import`.** `@import url('https://fonts.googleapis.com/...')` inside CSS is a severe render-blocking anti-pattern — it forces a serial HTML → CSS → @import-CSS → font-file chain, adding a full extra round-trip the browser can't discover until it has already downloaded and parsed the main stylesheet. Fetch the CSS2 API response once at build time, download the referenced woff2 files (a variable font typically needs only ~2-4 unique files per family for the latin/latin-ext subsets — check for repeated URLs across weight/stretch declarations before assuming you need one file per weight), commit them to `public/fonts/`, and write local `@font-face` rules with `font-display: swap`. This also lets you drop `fonts.googleapis.com`/`fonts.gstatic.com` from CSP entirely. If self-hosting truly isn't an option, a `<link rel="stylesheet">` in `<head>` (parallel-discoverable) is still meaningfully better than `@import` (serial) — but self-hosting is the default, not the fallback.
- **Preload the LCP element explicitly.** `<link rel="preload" as="image" fetchpriority="high">` for the hero/above-the-fold image — critical when the hero is a CSS `background-image`, since the browser's preload scanner can't see it until CSS has parsed, unlike an `<img>` tag which is discovered immediately from the HTML. Also preload the 1-2 font files used above the fold: `<link rel="preload" href="/fonts/x.woff2" as="font" type="font/woff2" crossorigin>`.
- Measured (live, AUDIT F): Lighthouse ≥ 95 on Performance / SEO / Accessibility / Best-Practices; LCP < 2.5s, CLS < 0.1, INP < 200ms. (INP — not FID — is the interaction metric since March 2024.)

**Forms (the definitive method — full templates in `references/forms.md`):** see "Forms — the definitive method" below. This supersedes any older guidance.

**⭐ Reply Portal (default Yes on every build, both targets — full detail in `references/reply-portal.md`, `references/forms.md`):** this is not an optional add-on like Backend — build it with the same weight as the shop pages. It's not indexed, but it's where every order actually gets fulfilled; a site with a broken checkout has zero sales regardless of how good the SEO is. Generate, every time Section P isn't explicitly declined: the storage layer (`lib/redis.js`/`orderStore.js`/`enquiryStore.js` on Vercel, `functions/_shared/` equivalents on Cloudflare), `lib/adminAuth.js`, `lib/order.js` (`parsePaymentDetail` + the payment-terms/method functions), `lib/whatsapp.js`, `utils/emailTemplates.js`, the full `app/admin/*` dashboard (hub, orders, enquiries, both composers), the public `app/order/payment-details/` + `app/order/confirm-payment/` pages, `components/CopyField.jsx` + `components/admin/*`, and the checkout form itself wired to whichever channel(s) Section E selected — with the mandatory unconditional confirmation email and the WhatsApp greeting rule both intact. See "Reply Portal system" below for the full behavioural spec — this line is the reminder that it's mandatory scaffolding, not prose to skim past.

**Internal linking:** hub & spoke (home → category hubs → product/blog spokes); blog→2-3 products/categories; product→parent category + 3-4 related; no dead ends; descriptive anchors.

**Backend (only when backend = Yes — full detail in `references/backend.md`):** generate the two-layer system — mark every editable zone in the templates with stable `data-cms` keys + sensible defaults; produce `cms-manifest.json`, `functions/`, `cms/`, `schema.sql`, and `CLOUDFLARE-SETUP.md`.

### After build
1. **Run `npm run images`** and show the contact sheet for approval before shipping.
2. **Run the pre-ship crosscheck.** Fix everything it flags. Re-run until clean. **This includes the Reply Portal crosscheck by default now** (Section P = Yes on virtually every build) — it is not conditional the way the backend crosscheck is. **Backend = Yes:** also run the backend crosscheck.
3. **Deliver per target:**
   - **Vercel:** `git init` + `.gitignore` + `README.md` + `CLAUDE.md`, commit, and output the exact GitHub + Vercel steps (empty repo → remote → push → import → **Framework Preset: Next.js**). If not running with disk access, deliver `[site-slug]-repo.zip` (repo root at the ZIP root, no wrapper folder) + the ZIP handover warning.
   - **Cloudflare:** `npm run zip` → `[site-slug]-deploy.zip` (contents of `out/` at the ZIP root). **Backend = Yes:** also output `CLOUDFLARE-SETUP.md` inline.
4. **Post-deploy checklist:** connect domain (**set `SITE.domain`, rebuild, redeploy — never find-and-replace**), verify GSC tag, submit sitemap to GSC + Bing, **test all 3 forms end-to-end**, verify the IndexNow file, purge cache (Cloudflare), **run isitagentready.com/[DOMAIN]** (expect ~79 on Cloudflare free — that is the plan's ceiling, not a defect).
5. **Verify the live deploy (now the entry point to the live audit tier).** Fetch the deployed URL and confirm: `llms.txt`, `auth.md` and every `.well-known/*` return 200 with the correct `Content-Type`; the `Link` header is present; www→apex 301 resolves. Then run the **full-sitemap 200 crawl** — every URL in `sitemap.xml` returns 200 (no 404/500/redirect-loop), not just a sample. Then run **AUDIT F** (Lighthouse ≥95 ×4 + Core Web Vitals + axe-core accessibility + a live Rich Results Test pass per schema type) against the deployed URL. Report all results — never assume a deploy succeeded because the build passed.

---

## BACKEND MODULE (optional — read `references/backend.md` when backend = Yes)

The backend is a **requirement-gated, opt-in** addition. It is never built unless Intake Section O = Yes. When off, WebForge builds a pure static Next.js site.

> **Target constraint: the backend is Cloudflare-only.** It is built on D1 + R2 + Pages Functions. If backend = Yes, the deploy target is **Cloudflare Pages** — say this plainly at intake rather than half-porting it to Vercel. A site needing both a client backend and Vercel is a different architecture; do not improvise one.

When on, here is the model — the full spec, code patterns, manifest format, admin panel, SEO Score logic, `schema.sql`, dashboard setup guide, and audit live in `references/backend.md`.

**What it gives the client:** a login at `clientdomain.com/cms` where they edit page text, manage blog posts and products, upload images, and edit SEO details with a live RankMath-style **SEO Score** panel — without touching code, ZIPs, or the Cloudflare dashboard. **`/cms`, never `/admin`** — that path is reserved for the Reply Portal (Section P), which now ships by default on every site; the two must not collide.

**The two-layer model:**
- **Template layer — Claude owns (ships in the ZIP):** HTML structure, design, default content, the `data-cms` editable zones, the manifest, the SEO Score logic, the admin UI.
- **Content layer — client owns (lives in D1 + R2, never in the ZIP):** the field values, posts, products, media, SEO overrides.
- **Edge merge:** a Pages Function reads D1 and uses HTMLRewriter to inject saved values server-side before the page reaches the visitor. Override wins; if none, Claude's default shows.

**Content authoring stays with Claude:** when backend = Yes, you (the operator) still ask Claude to write posts/products. Claude outputs them as `*-import.json` files; the client uploads them in `/cms → Import`. Never hardcode posts/products into HTML on a backend site.

---

## Forms — SMTP only, both targets (supersedes all older guidance)

**Web3Forms is retired — deleted entirely.** One constant selects the provider; only two options remain.

```js
// src/config/site.js
export const FORMS = {
  provider: 'smtp',                  // 'smtp' (default, BOTH targets) | 'resend' (Vercel, verified domain)
  smtpFrom: 'noreply@yourdomain.com',
  resendFrom: '',                    // only needed when provider = 'resend'; REQUIRES verified domain
  turnstileSiteKey: '',              // optional spam protection
  destinations: { contact: '', order: '', wholesale: '' }, // fallback when the matching env var is unset
}
```

**Default: `smtp`, on both Vercel and Cloudflare.** On Vercel this is nodemailer (`lib/mailer.js`). On Cloudflare it's a raw-socket SMTP client run from a Pages Function (`functions/_shared/mailer.js`) — Cloudflare's Workers runtime has no `net`/`tls`, so nodemailer can't run there, but the env var names, the `sendMail()` shape, and the graceful-degradation behaviour are identical either way. Never propose `resend` for a site without a live, verified domain.

**Env vars (canonical, same names both targets):** `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` (optional). Destinations: `CONTACT_EMAIL`, `ORDER_EMAIL`, `WHOLESALE_EMAIL` (each optional, falling back to `FORMS.destinations`). Vercel reads `process.env.*`; Cloudflare Functions read `env.*` (a per-request argument, set as Encrypted variables in Pages → Settings). Never in the repo (crosscheck B5).

`sendMail()`/`sendMailBatch()` returns `{sent:false, reason:'not-configured'}` when unconfigured — never throws. The API route/Function responds 503; the UI falls back to WhatsApp/phone.

**Email deliverability — DNS checklist (cannot be fixed in code):**
```
SPF:   TXT @ "v=spf1 include:your-smtp-provider.com ~all"
DKIM:  add CNAME/TXT records from your SMTP provider dashboard
DMARC: TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com"
```
Document these in `.env.example` — this is the real spam fix. Warn client at handover.

**`utils/emailTemplates.js` — branded HTML emails (LIGHT shell), one function per email type sharing HTML primitives** (`shell`, `field`, `divider`, `callout`, `button`, `escapeHtml`) — not a generic row-based builder. Full API, the primitives, all five emails, and the payment-method registry are in `references/forms.md`.

**⭐ Light-theme mandate (the #1 email rule):** every transactional email is a **light** email — white card body, **dark brand header band**, near-black text, brand colour as **accent only**. **Never a dark-background body:** Zoho/Gmail dark mode force-invert dark emails and no HTML opt-out stops it. This applies on Cloudflare too — the raw-socket mailer sends the same HTML.

**Config, not hardcoding:** all brand/location/currency/payment specifics come from the one `SITE.reply` block (accent = `reply.brand.primary`, header band = `reply.brand.headerDark`; light neutrals never vary). Never hardcode hex inside `emailTemplates.js`.

**Resend (Vercel only, opt-in):** `src/app/api/contact/route.js` — POST, validates, sends via Resend, returns `{success:true}`. `RESEND_API_KEY` env var in Vercel — never in the repo. Crosscheck FAILS if selected on a target with no server-side route.

**All providers:** CSP `connect-src 'self'` covers same-origin SMTP sends on both targets (no third-party form API origin needed anymore). Three forms → `/thank-you-contact/`, `/thank-you-order/`, `/thank-you-wholesale/`, all `noindex, follow`, excluded from sitemap. Turnstile if configured: widget in the form, token verified server-side.

**Crosscheck B11:** any reference to `web3forms`, `WEB3FORMS_KEY`, or `api.web3forms.com` anywhere in the build → FAIL. The provider is retired.

---

## Reply Portal system (Section P — default Yes, both targets now)

A passcode-gated admin dashboard that lets the site operator reply to orders and enquiries without leaving the site — and the mechanism that makes payment collection flexible for any country/currency/payment rail. **Available on both Vercel (API routes + Upstash Redis) and Cloudflare (Pages Functions + KV) — no longer Vercel-only.** Default is now **Yes**; skip only when the client explicitly declines. **Full spec — storage, auth, admin UI, IDs, payment-detail parsing, WhatsApp, both targets — in `references/reply-portal.md`.** JavaScript/JSX throughout, not TypeScript, matching the rest of the site.

The dashboard home is `/admin` — a passcode-gated **hub** with live order/enquiry counts linking to each section; nav is Dashboard · Orders · Enquiries on every admin page.

### The core mechanism — flexible payment details

The admin pastes **one blob** of real payment info (bank details, a wallet address, a payment link — any country, any rail) into the send-payment-email composer. `parsePaymentDetail()` (`lib/order.js` / `functions/_shared/order.js`) splits it into individually-copyable `{label, value}` fields via a 3-tier rule (colon-labelled line → known-label-word fallback → bare "Detail N"). This is what makes the same portal fit any niche without code changes — see `references/reply-portal.md` §3 for the exact algorithm. The customer sees each field as a tap-to-copy button on a public `/order/payment-details/` page (linked from the email, since email clients strip all JavaScript and can't host a real copy button) and in the WhatsApp message — both read the same parsed fields, never a re-typed duplicate.

### Data layer

**Vercel:** `lib/redis.js` — `@upstash/redis` SDK (not a hand-rolled REST client), checking **five** env var prefix variants. `lib/orderStore.js` / `enquiryStore.js` — one Redis **hash** per type, in-memory sort by `createdAt` (no separate sorted-set index). **Cloudflare:** `functions/_shared/orderStore.js` / `enquiryStore.js` — a KV namespace, one key per record + a capped index array. Both: `StoredOrder { orderNumber, customerName, customerEmail, customerPhone, items[], amountDue, status:'pending'|'payment-sent'|'payment-confirmed', channel:'whatsapp'|'email', createdAt, paymentDetails? }`; `StoredEnquiry { id, type:'contact'|'wholesale', name, email, phone?, message, meta?, status:'new'|'replied', createdAt }`. Full detail: `references/reply-portal.md` §2.

### Auth

`checkAdminPasscode(request)` (Vercel, reads `process.env.ADMIN_PASSCODE`) / `checkAdminPasscode(request, env)` (Cloudflare, reads `env.ADMIN_PASSCODE`) — compares to the `X-Admin-Passcode` header, 503 if unset, 401 if wrong, `null` if OK. Every admin route/Function calls this first. `PasscodeGate.jsx` verifies client-side by calling the orders-list endpoint with the header — no separate `/verify` route. `robots.txt` MUST disallow `/admin/`.

### Admin pages

All admin pages read their accent from `SITE.reply.brand.primary`. A dark admin shell is fine — it's a private tool, not an email. `/admin` (hub, live counts) · `/admin/orders/` (+`[id]`, status/channel badges, "Send/Resend payment details") · `/admin/enquiries/` (+`[id]`, filter pills all/contact/wholesale/new/replied) · `/admin/send-payment-email/` (method select, paste box, live preview, WhatsApp panel) · `/admin/reply-enquiry/` (reply composer). Same components on both targets — only the `fetch()` target (API route vs. Function) differs.

### WhatsApp — wired end-to-end, not a dead link

`lib/whatsapp.js` — `waLink`/`waLinkTo`/`waMessageText`, `waOrderLink` (new-order notification), `waPaymentDetailsMessage/Link` (mirrors the email, same parsed fields), `waPaymentConfirmationLink`. **The "Order via WhatsApp" checkout button must mint an order number, save it (`channel:'whatsapp'`), and open `waOrderLink`'s message — never a bare `wa.me` chat link with no order record.** Every WhatsApp feature is gated on `SITE.reply.channels.whatsapp` being configured.

**⭐ Two message directions, two openers — never mixed up:**
- **Customer → business** (order request, payment confirmation, the general chat-hub link) — every one of these opens with a literal greeting, **`Hi {SITE.name},`** — not a bold label. This is how a business immediately knows which site an inbound WhatsApp message belongs to.
- **Admin → customer** (payment-details reply) — opens with `WA_HEADER` (`*${SITE.name}*` bold), the business identifying itself to the customer.

**⭐ WhatsApp checkout is still a plain Yes/No at intake (Section E) — but "Yes" always means the full, properly built thing, never a shortcut.** Whether a site offers order-form-only, WhatsApp-only, or both, the checkout is one real form (name, required email, phone, address, payment method) — a WhatsApp-only site does not get a shorter form or a bare chat link. The WhatsApp button saves the order through the exact same save-and-email call the email button would use (`channel:'whatsapp'` instead of `'email'`), so `orderConfirmationEmail()` always fires, on every configuration that includes WhatsApp — it must never be skipped because the customer is "already talking to us on WhatsApp," and it must never be skipped because WhatsApp happens to be the site's *only* channel either. Full flow: `references/reply-portal.md` §7a.

### Customer-facing pages (public, no-auth, order-number-as-token)

`/order/payment-details/?id=` — amount due, each parsed field as a `CopyField`, opening/closing lines. `/order/confirm-payment/?id=` — screenshot upload (JPG/PNG/WebP/HEIC, 4MB cap) → emails the shop, marks `payment-confirmed`. Neither exposes anything beyond what the customer's own email already contained.

### Thank-you page rule

`/thank-you-order/` MUST only say "watch for the follow-up email with payment details" — NOT payment instructions, deadlines, or WhatsApp screenshot requests. Those come only after the admin sends payment details via the portal.

### Crosscheck items for Reply Portal

**The full, canonical, numbered checklist (B9, B10, Items 47–58) lives in one place only — "Pre-ship crosscheck" below, in its own "Reply Portal crosscheck" block, run alongside every other crosscheck on every build.** It is not repeated here to avoid two copies drifting apart under separate edits — that duplication is exactly the kind of mix-up this skill exists to prevent. If you're looking for the items themselves, go to "Pre-ship crosscheck."

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
- **The `whatsapp` link channel's `href` always pre-fills a greeting:** `https://wa.me/{number}?text=${encodeURIComponent('Hi ' + SITE.name + ', ')}` — never a bare `https://wa.me/{number}` with no `text`. This is the same greeting rule the Reply Portal's WhatsApp messages follow (`references/reply-portal.md` §7) — applies whether or not Section P is enabled, since this is the general chat button, not a portal feature. A trailing space after the comma leaves the customer's cursor ready to type, not needing to add one themselves.

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
- **Backend = Yes:** the `/cms` dashboard is mobile-first as well.

---

## Pre-ship crosscheck (run before EVERY deploy — push OR ZIP — in Mode 2/3/4)

Implemented as `scripts/crosscheck.mjs`. It **exits non-zero on failure** and runs against the BUILD OUTPUT (`out/` for static, `.next` + a local serve for Vercel) — never against source. Verify → fix → re-run until clean. Never deliver on a failing crosscheck.

**Blocking failures (ship-stoppers) — check these first:**
- **B1. Domain placeholder in a production deploy.** If the deploy is production and `SITE.domain` is still `DOMAIN.com` (or appears anywhere in the output), FAIL. Permitted only for preview/pending builds — and then warn loudly.
- **B2. Capability lies (Rule 10).** `SearchAction` emitted but no `/search/` route → FAIL. `api-catalog` / `server-card.json` listing endpoints that 404 on this target → FAIL. Static target advertising live MCP tools → FAIL.
- **B3. `resend` provider on a static (non-Functions) target.** Requires a real server-side route; a target with neither API routes nor Pages Functions wired for it → every form 404s → FAIL. (`smtp` is fine on either target — Cloudflare runs it via Pages Functions.)
- **B4. Strategy docs leaked.** `docs/`, `PROJECT.md`, `keyword-map.md`, or `assets/product-photos/` present in the build output → FAIL.
- **B5. Secrets in the repo.** `SMTP_PASS`, `RESEND_API_KEY`, `ADMIN_PASSCODE`, or any API secret in tracked files → FAIL.
- **B9. `ADMIN_PASSCODE` in a client bundle** (Section P = Yes). If it appears in a `NEXT_PUBLIC_*` env var or the compiled client JS → FAIL.
- **B6. Missing agent files on the deployed target.** All A–N files present in the output with correct paths.
- **B7. Compliance violation.** Any banned term from `docs/PROJECT.md` → Compliance found anywhere in the build output (HTML, JSON-LD, llms.txt, .well-known/*, alt text, MCP tool descriptions) → FAIL. See "Compliance guardrails".
- **B8. Live-layer drift (Vercel).** `server-card.json` tools ≠ the live `/api/mcp` `tools/list` response, or any `api-catalog` URL 404s → FAIL. See "Agent-Ready — the LIVE layer" → V5.
- **B11. Web3Forms present.** Any reference to `web3forms`, `WEB3FORMS_KEY`, or `api.web3forms.com` anywhere in the build output → FAIL. The provider is retired — every form runs on `smtp` (or opt-in `resend`).

**Then verify, fix, and re-run until clean:**
1. **Forms:** API route/Function exists, sends via nodemailer (Vercel) or the raw-socket client (Cloudflare)/Resend, returns JSON, UI redirects to thank-you, returns 503 (not crash) when unconfigured.
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
16. **Structure intact:** all product pages have offers; product count matches; footer + chat + script.js on every page; CSS braces balanced; div open/close balanced; single `</body>`/`</html>`.
17. **AI signals:** homepage authority section + rich Store schema + sameAs + llms.txt + robots AI-allows present.

**Agent-Ready crosscheck (mandatory on every build):**
18. **robots.txt** — all major AI bots listed with `Allow: /`; agent-readable resource comment block present.
19. **llms.txt** — present at root; `Content-Type: text/plain` in `_headers`; contains brand facts, product categories, contact, agent resources section.
20. **auth.md** — starts with exactly `# Auth.md` as first line; `agent_auth` JSON block present; `Content-Type: text/markdown` in `_headers`.
21. **.well-known/api-catalog** — valid JSON; `Content-Type: application/linkset+json` in `_headers`; `Access-Control-Allow-Origin: *`.
22. **.well-known/agent-skills/index.json** — has `$schema` field and `skills` array with `name/type/description/url/sha256` on each skill.
23. **.well-known/mcp/server-card.json** — has `serverInfo`, `transport`, `capabilities` blocks; `commerce` and `legal` adapted to client niche.
24. **.well-known/oauth-protected-resource** — present; `Content-Type: application/json`; `resource_documentation` points to `/auth.md`.
25. **.well-known/oauth-authorization-server** — present; has `agent_auth` block; all null endpoints explicit.
26. **.well-known/openid-configuration** — present; `Content-Type: application/json`; `public_site: true`.
27. **.well-known/acp.json** — present; `"protocol": {"name": "acp", "version": "0.1.0"}` present.
28. **.well-known/ucp** — present; **`"ucp": "1.0"` field mandatory** (scanner fails without it); `spec` and `schema` URLs present.
29. **/js/webmcp.js** — present; loaded with `<script src="/js/webmcp.js" defer></script>` in `<head>` of every page.
30. **_headers Link header** — `/*` rule contains the full `Link:` header referencing api-catalog, agent-skills, llms.txt, mcp/server-card, auth.md, and openid-configuration.
31. **_headers Content-Types** — separate rules for every `.well-known/` file, `/auth.md`, `/llms.txt`, `/*.md` set correctly with `Access-Control-Allow-Origin: *`.
32. **All placeholders replaced** — zero instances of `[DOMAIN]`, `[NUMBER]`, `[EMAIL]`, `[Brand]` remaining (except in comments).

**Technical-SEO / Accessibility / Performance crosscheck (v9 — static tier, runs on every build):**
41. **Sitemaps split + image-aware:** `sitemap.xml` is an index referencing child sitemaps; each child is valid XML; product entries include `<image:image>`; no URL appears in two children; every child URL resolves to a real built route; no noindex/redirected/404 URL included.
42. **No duplicate body content:** product descriptions and category intros are pairwise-unique (hash-compare); flag near-duplicates.
43. **Inverse noindex/nofollow:** no indexable page carries `noindex`/`nofollow`; only thank-you + not-found are noindex; no important internal link is `nofollow`.
44. **Accessibility (static):** skip-link present; one H1 + no skipped heading levels; landmarks present; every img has alt; every form control labelled; every icon-only button has `aria-label`; no positive `tabindex`; no `outline:none` without `:focus-visible`; `prefers-reduced-motion` block present.
45. **Performance (structural):** AVIF+WebP for every image; dimensions present (CLS=0); CSS/JS minified + deferred; no render-blocking head scripts; `font-display: swap` + preconnect; homepage transfer < 1.5MB.
46. **Visual design quality:** every grid uses one shared tile (single aspect + `object-fit: cover` + equal heights); no homepage section is bare (each has imagery/icon/card/tint); one radius + one shadow + one spacing token reused (no scattered ad-hoc values); hero has min-height/sized container + scrim where text overlays; peer sections share one alignment pattern; homepage has ≥6 sections with alternating surface treatments; all motion is `prefers-reduced-motion`-safe; none of the dated AI-generated-page tells are present (assistant-palette colours, floating blur-pill nav, performative labels, invented social proof, >2-line display headlines, >2 typefaces). Full spec: `references/design-quality.md`.

**Backend crosscheck (additional — run only when backend = Yes):**
33. Manifest ↔ template key parity; stable keys; no orphaned or missing keys.
34. No hardcoded posts/products in HTML where the edge would override/duplicate them.
35. `_middleware.js` renders Claude's default when D1 has no override.
36. Auth: hashed passwords (no plaintext compare); sessions use signed cookie with `SESSION_SECRET`; `setup.js` self-disables after first admin and is gated by `ADMIN_SETUP_TOKEN`.
37. `robots.txt` disallows `/cms/` and `/api/`; `_headers` sets `/cms/*` to `no-store`.
38. Import files valid against collection schema.
39. `CLOUDFLARE-SETUP.md` present and current.
40. `node --check` each file in `functions/` and `cms/*.js`.

**⭐ Reply Portal crosscheck (run whenever Section P = Yes — the default; not gated behind an "if," skip only on the rare site that explicitly declined it):**
47. Every admin route/Function calls `checkAdminPasscode` first; `ADMIN_PASSCODE` never reaches the client bundle (see B9 above); `robots.txt` disallows `/admin/` (see B10 above).
48. Storage layer (`isStoreConfigured()`) never crashes when unconfigured — orders/enquiries still email through, they just don't appear in the dashboard.
49. Order ref generated client-side with the unambiguous alphabet (`ABCDEFGHJKLMNPQRSTUVWXYZ23456789` — no 0/O/1/I/L); validated server-side; identical ref in WhatsApp + dashboard + email.
50. `parsePaymentDetail` drives both the send-payment-email composer preview and the customer-facing `/order/payment-details/` page — never an unparsed pasted blob shown as-is.
51. Status lifecycles fire: `markOrderSent` on payment email send, `markPaymentConfirmed` on screenshot upload, `markEnquiryReplied` on reply.
52. **WhatsApp checkout actually saves an order and sends the structured `waOrderLink` message — not a bare chat link — on every Section E configuration that includes WhatsApp, including WhatsApp-only sites.** The checkout form has all its fields (required email included) regardless of which channel(s) are enabled.
53. **The customer confirmation email fires unconditionally on every configuration that includes WhatsApp.** Trace `channel==='whatsapp'` through to `orderConfirmationEmail()` and confirm it isn't short-circuited.
54. Every customer-originated WhatsApp message (order, payment confirmation, the general chat-hub link) opens with `Hi {SITE.name},` — never the bare `WA_HEADER` bold label, which is admin→customer only.
55. Admin pages read accent from `SITE.reply.brand.primary` (no hardcoded hex); `/admin` hub reachable and passcode-gated; emails are the light shell (never dark — Cloudflare too).
56. No hardcoded brand/location/currency/prefix/method/locale anywhere in the portal — all from `SITE.reply`. No `web3forms` reference anywhere (see B11 above).
57. `/thank-you-order/` contains no payment instructions, no bank details, no deadline — those come only after the admin sends payment details via the portal.
58. `paymentTermsLines()` is the single source for the WhatsApp text, the email HTML (`paymentTermsHtml()`), and the send-payment-email composer preview — no inline term lists duplicated anywhere else.

Only produce the ZIP after this passes with zero errors. Report the pass/fail counts.

---

## MODE 4 — Audit modes (Technical / SEO / AI-Visibility / Agent-Ready) + Auto-Fix

Works on new and old sites. Audits run in **two tiers**: the **static tier** (AUDITS A–E) runs from the ZIP with no live URL and gates the build; the **live tier** (AUDIT F) needs the deployed URL and a headless browser (Lighthouse + axe-core) for performance, Core Web Vitals, contrast, keyboard operability, the full-sitemap 200 crawl, and a live Rich Results Test pass (via the in-app browser) per distinct schema type shipped. Run the one the user asks for, or all for a full audit. If only a ZIP is available, run A–E and report that F requires the live URL.

### Step 1 — Get files
Ask for the site ZIP if not provided.

### Step 2 — Project context
If PROJECT.md present: read it (note backend = Yes/No). Else ask: brand+domain, primary product, target region, primary keyword, secondary keywords, business location/HQ, known issues, and whether the site has a client backend.

### Step 3 — Run the requested audit(s)

**AUDIT A — Technical audit** ("technical audit", "deploy check", "something's broken")
- robots.txt valid, not blocking important pages, sitemap referenced, AI-crawler allows present.
- sitemap.xml is a valid index referencing valid child sitemaps (pages/products/categories/blog); `<lastmod>` on every URL; product entries carry `<image:image>`; no noindex/redirected/404 URL included.
- No unintended noindex/nofollow on any indexable page or important internal link (only thank-you + not-found are noindex).
- Canonicals present/correct; trailing-slash consistency.
- `_redirects`: www→apex + http→https `301!` first; no redirect chains (3+ hops).
- `_headers`: CSP correct (form API + chat + wss whitelisted), Cache-Control for assets, HTML revalidation, HTTPS/security headers. Agent-Ready Link header present on `/*`.
- Performance: WebP, lazy-load, minified CSS/JS, deferred scripts, font-display swap, page weight.
- **Mobile (release blocker):** viewport meta, overflow guards, inline-grid media safety net, no horizontal scroll, tap targets. Also check the platform-level layer in `references/mobile-native.md` (tap highlight, `dvh`/`svh` height bug, input zoom, safe-area insets, overscroll) — real-hardware behaviors that don't show up in a static scan or emulation; flag what needs phone verification.
- GSC tag + IndexNow key on all pages, IndexNow file at root.
- Forms: API route/Function exists and gracefully returns 503 (not crash) when unconfigured; thank-you pages exist for all three forms; no `web3forms` reference anywhere (B11).
- **JSON-LD validity:** parse every block; flag bare Product on non-product pages, BreadcrumbList comma bug.
- **Usability & cognitive load:** run the Nielsen-heuristics + cognitive-load + persona pass in `references/usability-audit.md` on home, shop/category, product, and cart/order; produces the `Usability: /100` row below.
- **Visual design quality:** re-run the full `references/design-quality.md` check against the *live* site, not just at build time — grid uniformity, hero composition, section rhythm, colour/type polish, motion safety, imagery consistency, and the rule-8 dated-AI-tells scan. A technical audit of an already-built site re-verifies this, it isn't only checked the next time something gets built or fixed.
- **Prose-quality sibling check (`pages.md` § Writing quality):** sentence rhythm actually varies across a section (not every sentence the same length/shape), and tone shifts by section purpose (spec plain, CTA persuasive, authority narrative) — the content-side counterpart to the rule-8 visual-tells scan above.
- **Icon library:** only one icon library's SVGs used site-wide (`references/nextjs.md` default: Lucide) — flag any mixing with a second library or a hand-drawn inline `<svg>` duplicating an icon the library already has.

**AUDIT B — SEO audit** ("audit", "check SEO", "fix my SEO")
- Meta titles 50-60, unique, keyworded. Meta descriptions in band, unique, synced to og/twitter.
- Exactly one H1 per page with keyword; heading hierarchy; keyworded H2s.
- Image alt text everywhere; emails entity-encoded (no plaintext, incl. JSON-LD).
- OG + Twitter complete; canonical present. **`og:image` specifically is 1200×630 (or correctly cropped toward it from the page's hero/thumbnail/gallery image per `references/seo.md` § og:image) — not just present.**
- Schema correct per page type, validated, required props.
- Internal links: no orphans/dead-ends, descriptive anchors. **Journey-Stage links present and bidirectional** where `keyword-map.md` assigns a Journey Stage (`references/pages.md` § Journey-Stage internal linking) — a blog post links forward to the Purchase-stage page it funnels toward, and that page links back.
- **Stale-intake drift:** if `docs/taxonomy-map.md`/`docs/starter-product-list.csv` exist for this project but Section I/K's actual answers don't resemble them, flag it — intake should have pre-filled from these, not re-derived from scratch.
- Content freshness: lastmod + og:updated_time + revalidation.
- **Stale-content flag:** any blog post older than 12 months with no `dateModified` update since — flag for a human review pass, not an automatic rewrite. Check specifically for the kind of claim that silently rots: a bare year ("in 2024, the best..."), a price mention, a "currently"/"latest"/"new" claim, a stat with no re-check. This mirrors keyword-engine's own New Arrivals year-stamp staleness logic — a page can be technically correct and still read as out of date.
- Broken-link scan: every internal `href` resolves to an existing page.
- Duplicate body content: product descriptions and category intros are unique (hash-compare) — flag near-duplicates.
- Accessibility (static subset): alt text, bound labels, one-H1 + no skipped levels, landmarks, skip-link, no positive tabindex. Full AA (contrast + keyboard) is AUDIT F.

**AUDIT C — AI-Visibility audit** ("AI visibility check", "ai seo", "GEO/AEO")
- Brand entity statement on homepage; visible "About [Brand]" authority section.
- **GEO evidence-backed tactics present** (direct quotations, cited statistics, inline source citations) in blog/authority-section copy — not just comparison language. See `references/ai-visibility.md` GEO section.
- Store/Organization JSON-LD has description, foundingDate, foundingLocation, address, areaServed, numberOfItems, knowsAbout, priceRange, sameAs, brand, AggregateOffer. **`@id` present on the Organization node and consistent (same value) everywhere it's referenced (e.g. Article `publisher`); Wikidata/Wikipedia `sameAs` present if the brand actually has one** (`references/seo.md` § Organization JSON-LD).
- FAQPage schema on homepage + FAQ page; direct-answer format. **Blog H2 sections also answer their own heading in the first sentence** (`references/pages.md` § Blog) — not just the dedicated FAQ content.
- **Comparison/spec/step-by-step blog content renders as a real `<table>`/`<ul>`/`<ol>`**, not a prose paragraph describing the same thing (`references/pages.md` § Blog).
- **Article `author` matches intake:** if `docs/PROJECT.md` names a real content author, Article schema uses `Person` (not `Organization`) and the visible byline matches the schema's name exactly; if no named author was confirmed, Article stays Organization-authored — either is correct, but they must agree with what intake actually recorded.
- About page >700 words, milestones timeline, differentiation grid, footprint.
- llms.txt present with brand facts + citation guidance; robots AI-allows present.
- Brand name consistent everywhere.
- **The 12 authority signals scorecard** (Founding Year, Founding Location/HQ, Quantifiable Scale, Competitive Differentiation, Historical Milestones, National/Global Footprint, Industry Leadership, Brand Narrative, Product Lines, Named Individuals, Awards & Rankings, Endorsements/Partnerships). Mark which are present on-site. For the last three, add them when they are REAL — facts the client supplies or that already exist on the site being migrated (including real reviews/awards); never fabricate ones that do not exist.
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
- `robots.txt` disallows `/cms/` + `/api/`; `_headers` `/cms/*` no-store.
- Import files valid against collection schema.
- `CLOUDFLARE-SETUP.md` present and current.

**AUDIT F — Live Performance, Accessibility & Rich Results (needs the deployed URL — cannot run from a ZIP)**
Run against the deployed site with Lighthouse (CI or PageSpeed Insights, mobile profile) + axe-core:
- **Full-sitemap 200 crawl:** every URL in sitemap.xml returns 200 — no 404/500, no redirect loop/chain (3+ hops).
- **Lighthouse ≥ 95** on Performance, SEO, Accessibility, Best-Practices.
- **Core Web Vitals:** LCP < 2.5s, CLS < 0.1, INP < 200ms. (INP, not FID.)
- **Accessibility (rendered):** axe-core zero critical/serious; contrast ≥ 4.5:1 (text) / 3:1 (large + UI); full keyboard walk with visible, unobscured focus; no keyboard traps.
- **Rich Results Test (live, not just static JSON-parsing):** the pre-ship crosscheck only confirms schema *parses* as valid JSON with required fields present — it can't confirm Google actually treats it as rich-result-eligible. Using the in-app browser, navigate to `search.google.com/test/rich-results`, enter one representative live URL per distinct schema type shipped on the site (a product page for Product, the FAQ page and homepage for FAQPage, any inner page for BreadcrumbList, homepage for Organization, a blog post for Article, contact page for LocalBusiness if present), and read the result for each. Flag anything the tool reports as invalid or ineligible that the static check missed — e.g. a Product block that parses fine as JSON but is missing a field Google requires for eligibility, not just for validity. Don't rely on the JSON-parse check alone to mean "Google will show this as rich" — that's a static proxy, not the real answer, and only a live tool run gives the real answer.
- Triage misses per `references/performance.md` (LCP → CLS → INP → Best-Practices). Auto-fix covers structural (Tier-1) regressions only; CWV/contrast/Rich-Results findings are reported with remediation, not silently "fixed."

Tooling note: this tier introduces a Lighthouse CI + axe-core dependency the static crosscheck does not have. It is a **post-deploy** step in the After-build checklist and in Mode 2/3 handover, not a ZIP-time gate.

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
Accessibility (WCAG 2.2 AA): /100
Performance / Core Web Vitals: /100
Forms:         /100
Mobile:        /100
Backend:       /100   (only if backend = Yes)
Usability:     /100
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
- **Backend = Yes:** a discreet "Admin" link is NOT placed in public nav/footer (keep `/cms` unlinked + robots-disallowed). Same rule applies to `/admin` (Reply Portal) whether or not backend is also enabled.

**Visual design quality (non-negotiable — full spec in `references/design-quality.md`):** every page must be clean, uniform, and genuinely attractive — not just technically correct.
- **No bare sections:** every section has a visual anchor (imagery, icons, an elevated/bordered card, or a subtle surface tint) — never plain text on flat white. Homepage has ≥6 distinct sections with alternating surface treatments.
- **Grid uniformity:** every grid (category, product, brand, blog) uses ONE shared tile with a single fixed aspect ratio, `object-fit: cover`, equal heights, consistent gap. Category tiles are uniform — never mixed sizes or ragged heights.
- **Hero composition:** deliberate image treatment (full-bleed + scrim, or a sized `cover` image grid with tuned `object-position`); min-height ≥70vh desktop / ≥60vh mobile (or a sized container) so subjects show; text legible over imagery.
- **Consistency & depth:** one radius token, one shadow token, one spacing scale, one container max-width and alignment pattern across peer sections; surface tints (not all pure #fff); clear type hierarchy; hover states on interactive tiles.
- **Tasteful motion:** subtle scroll/hover micro-interactions, no carousels/autoplay, no CLS, all gated behind `prefers-reduced-motion`.

---

## Agent-Ready Standards (all sites — deploy from day one)

Every site built with WebForge includes the full agent-ready file set from day one. These files improve AI visibility, allow AI assistants to discover and cite the site, and push the isitagentready.com score to ~79/100 on the Cloudflare Pages free plan. **Build and include ALL of the following in every site ZIP. They are not optional.**

---

### A — robots.txt

Always include explicit `User-agent` rules for all major AI crawlers with `Allow: /`. Also add a `Content-Signal` directive right after the disallow rules — an emerging IETF-draft standard that isitagentready.com checks under Bot Access Control: `Content-Signal: search=yes, ai-input=yes, ai-train=yes` (default: allow indexing, AI citation/retrieval, AND training — training opt-out has no documented effect on ranking or live-answer visibility per any major platform, so permissive-by-default is the baseline; adjust per client preference, never fabricate a stance they haven't agreed to). Append the crawler block after that:

```
Content-Signal: search=yes, ai-input=yes, ai-train=yes

# AI crawlers — welcome to index product & content pages
User-agent: GPTBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-User
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Perplexity-User
Allow: /

User-agent: MistralAI-User
Allow: /

User-agent: MistralAI-Index
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

User-agent: Meta-WebIndexer
Allow: /

User-agent: cohere-ai
Allow: /

# Agent-readable resources
# llms.txt: https://[DOMAIN]/llms.txt
# API Catalog: https://[DOMAIN]/.well-known/api-catalog
# Agent Skills: https://[DOMAIN]/.well-known/agent-skills/index.json
# MCP Server Card: https://[DOMAIN]/.well-known/mcp/server-card.json
```

**Backend = Yes:** also add `Disallow: /cms/` and `Disallow: /api/` before the AI crawler block. (Reply Portal, Section P, uses `Disallow: /admin/` instead — see "Reply Portal system." The two paths are deliberately distinct so they never collide when both are enabled on the same Cloudflare site.)

---

### B — llms.txt

Create `/llms.txt` at site root. Plain-text markdown summary for LLMs to read. **Must follow the llms.txt spec (llmstxt.org) exactly, not just "a bullet list of facts"** — PageSpeed Insights' Agentic Browsing audit specifically flags files that don't: `# Brand Name` → `> one-line blockquote summary` → optional context paragraph(s) → `## Section` headers whose list items are real markdown links, `- [Title](url): optional note`, never a bare `- Label: https://raw-url`. Include:

- Brand name, tagline, description (1 paragraph)
- Location, contact (WhatsApp, email)
- Min order, free shipping threshold, payment methods, first-order discount
- All product categories with price ranges and links — **as `[Category](url): price range, one-line description`**
- Wholesale info with tier discounts
- Blog/education section link
- Legal (age restriction, compliance)
- An `## Optional` section for secondary links (API catalog, agent skills index, MCP server card, auth.md) — again as proper markdown links, not raw URLs

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

### V5 — Markdown Negotiation (Vercel Routing Middleware)

Serves a plain-markdown version of any page when the client's `Accept` header genuinely prefers `text/markdown` over `text/html`. This is what isitagentready.com's **Content** category checks — it sends `Accept: text/markdown` with nothing else and expects `Content-Type: text/markdown` back. Fixes that check from 0/1 to 1/1.

**Build it as `middleware.js` at the repo root** (Vercel Routing Middleware — framework-agnostic, works alongside a Next.js app):
```js
import { next } from '@vercel/functions'

export const config = { matcher: '/:path*' }

export default async function middleware(request) {
  const url = new URL(request.url)
  const accept = request.headers.get('accept') || ''
  const wantsMarkdown = prefersMarkdownOverHtml(accept)   // see below — NEVER a substring check
  if (wantsMarkdown && eligiblePath(url.pathname)) {
    const mdRes = await fetch(new URL(mdPathFor(url.pathname), request.url))
    if (mdRes.ok) return new Response(await mdRes.text(), { headers: { 'content-type': 'text/markdown; charset=utf-8' } })
  }
  return next()
}
```
- **Generate a `.md` sibling next to every indexable route's rendered output at build time** (e.g. `dist/shop/index.md` next to `dist/shop/index.html`), converted from the page's `<main>` content — headings, links, lists. Excerpt/description as a leading `>` blockquote, matching the llms.txt convention. Skip `noindex` routes and non-page paths (`/assets/`, `/images/`, `/.well-known/`, `robots.txt`, etc.).
- **CRITICAL — never use a naive substring check** (`accept.includes('text/markdown')`). A real AI crawler that sends `Accept: text/html, text/markdown;q=0.9` also accepts — and prefers — HTML; a substring check would still hand it the stripped markdown extract, which has **no JSON-LD schema** and far less content than the full page. That is a real regression in what the crawler sees, not a cosmetic one. **Parse q-values** (default `q=1` when absent) and only serve markdown when its q genuinely exceeds `text/html`'s (or `text/html` is entirely absent):
  ```js
  function prefersMarkdownOverHtml(accept) {
    let mdQ = -1, htmlQ = -1
    for (const part of accept.split(',')) {
      const [type, ...params] = part.trim().split(';').map(s => s.trim())
      let q = 1
      for (const p of params) { const m = /^q=([\d.]+)$/.exec(p); if (m) q = parseFloat(m[1]) }
      if (type === 'text/markdown') mdQ = Math.max(mdQ, q)
      if (type === 'text/html') htmlQ = Math.max(htmlQ, q)
    }
    return mdQ > -1 && mdQ > htmlQ
  }
  ```
- **Routing Middleware needs verifying per-project.** Vercel's own docs flag it "🔒 Permissions Required" — confirm it actually executes (a plain non-markdown request should behave identically to before) before relying on it for a score.

### V6 — Crosscheck for the live layer

Beyond B2, the crosscheck must, on a Vercel build:
- Boot the app locally and assert `/api/mcp` answers `initialize` and `tools/list` over POST with `Accept: application/json`.
- Assert every tool in `server-card.json` appears in the live `tools/list` response — and vice versa. **Any drift is a FAIL.**
- Assert every URL in `api-catalog` returns 200.
- Assert `/api/products` and `/api/search` return valid JSON with correct content types.
- Assert no endpoint mutates state and none accepts payment fields.
- Assert `Accept: text/markdown` alone returns `text/markdown`, AND `Accept: text/html, text/markdown;q=0.9` returns `text/html` (proves q-values are actually respected, not just presence-checked).

### Known Vercel platform limitation — `.well-known/*` bypasses headers AND middleware

Confirmed live (direct header inspection, not documented by Vercel): every request under `/.well-known/*` is served through a path that skips **both** custom `vercel.json` `headers` rules **and** Routing Middleware entirely — no `Link`, no CSP, no security headers reach it either, and neither mechanism can override the `Content-Type` on the extensionless files in there (`ucp`, `oauth-authorization-server`, `oauth-protected-resource`, `openid-configuration` all serve as `application/octet-stream` regardless of what the config says). This is a Vercel platform limitation, not a config bug — do not spend time trying to route around it with a `rewrites`-to-function trick unless the specific score line it affects is actually in a *scored* category (isitagentready's own checks for those files are lenient about content-type and pass anyway; the one exception observed was the **optional, unscored** UCP commerce check). This sits alongside the Cloudflare free-plan agent-ready ceiling as a second platform-specific cap to just accept.

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

**Also enforced:** Rule 5 — do not INVENT brand facts (no made-up statistics, awards, press, named clients, or partnerships). Real facts the client supplies, or that are being migrated from their existing site, DO appear on the site — record each with its source in `docs/PROJECT.md`. Reviews/testimonial schema (`Review`/`AggregateRating`) is emitted for real, attributable reviews — including reviews migrated from the client's existing site or business profiles — and never for invented ones.

**Note for image-heavy verticals:** compliance applies to imagery, not just words. If a product's whole legal position is "this is not the real thing", stock photos of the real thing contradict it in the first place a reviewer looks. Flag this at intake and at handover.

---

## Multi-image products

```js
{ slug: 'aud-100-prop-notes', name: '…', price: 119,
  images: ['aud-100-prop-notes.webp', 'aud-100-prop-notes-2.webp', 'aud-100-prop-notes-3.webp'] }
```

- **First image is primary** — used on cards, OG tags, and `Product` schema's first `image` entry. A single-image product is just `images: [one]`; never special-case it.
- **`scripts/images.mjs`** processes every image identically (trim → white 4:3 canvas, fill ~90% enlarging small sources, sharpen upscaled, adaptive quality under 150 KB) and names them `<slug>.webp`, `<slug>-2.webp`, …
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
| `references/mobile-native.md` | **Platform-level mobile layer** — tap highlight, the `dvh`/`svh` height bug, input zoom, safe-area insets, overscroll, the full "never ship" checklist; read alongside `pages.md`'s mobile section and AUDIT A's Mobile bullet |
| `references/seo.md` | Meta formulas, schema per page type, split + image sitemaps, robots, GSC + Bing + IndexNow, validity traps, freshness |
| `references/accessibility.md` | **WCAG 2.2 AA** — the static build rules + the live (AUDIT F) checks: landmarks, focus, contrast, keyboard, forms |
| `references/design-quality.md` | **Visual design quality (build gate)** — grid/tile uniformity, hero composition, section rhythm + alignment, depth/colour/type polish, tasteful motion, the dated-AI-tells registry, the no-bare-section rule |
| `references/usability-audit.md` | **Usability & cognitive load (AUDIT A)** — Nielsen's 10 heuristics scored to `/100`, the cognitive-load checklist, the 5-persona walkthrough; produces the audit report's `Usability: /100` row |
| `references/performance.md` | **Core Web Vitals + Lighthouse 95+ gate (both targets)** — structural (static) vs measured (live) standards; target-conditional image handling |
| `references/ai-visibility.md` | GEO/AEO/LLM visibility, the 12 authority signals, llms.txt, off-site reality |
| `references/agent-live.md` | **Vercel live layer** — MCP Streamable HTTP server, JSON API, live ACP/UCP, target-aware `.well-known` generation, live-layer crosscheck |
| `references/vercel.md` | **Vercel target** — `vercel.json`, GitHub workflow, env vars, connecting a domain later, Framework Preset trap, 404 troubleshooting, deploy verification |
| `references/cloudflare.md` | **Cloudflare target** — `_headers`, `_redirects`, ZIP upload, canonicalization, dashboard setup, **Pages Functions for forms + Reply Portal (new)** |
| `references/forms.md` | Any form or email — provider selection (`smtp` only default on both targets, `resend` opt-in Vercel — **Web3Forms retired**), nodemailer (Vercel) + raw-socket SMTP (Cloudflare) setup, `SMTP_*` env vars, the **light** email shell + `SITE.reply` config + the real per-email-function API, the five emails, payment-terms + payment-details messages, payment-method registry, DNS deliverability, Turnstile, thank-you pages |
| `references/reply-portal.md` | **Reply Portal (Section P — default Yes, both targets)** — order/enquiry IDs, Redis (Vercel) / KV (Cloudflare) stores + status lifecycles, `parsePaymentDetail` flexible payment-field parsing, passcode auth, admin UI + `/admin` dashboard hub, the public customer-facing payment/confirm pages, WhatsApp reply system (wired end-to-end), env vars, build-vs-edit, audit checklist |
| `references/niche-variants.md` | Locale, age gates, disclaimers, payment variants, shop structures — the starting point per vertical |
| `references/backend.md` | **Client backend (backend = Yes → Cloudflare only)** — two-layer model, manifest, functions, admin panel, SEO Score, `schema.sql`, setup guide, import workflow, backend audit |

Every reference is current as of v11.1 (seo.md, vercel.md, and cloudflare.md updated in v11.1 with the live Rich Results Test pass; forms.md and reply-portal.md rewritten in v11 — Web3Forms retired, Reply Portal now default-Yes on both targets via Pages Functions on Cloudflare; intake.md Section P updated to match; design-quality.md is new in v9.1 and gained motion numerics, the colour proportion law, and the dated-AI-tells registry (rule 8) in v11.1; accessibility.md + performance.md are new in v9; nextjs.md carries v9 deltas; mobile-native.md and usability-audit.md are new in v11.1; ai-visibility.md gained the GEO paper's evidence-backed tactics, corrected Speakable schema's framing, and a Google Business Profile/Apple Business Connect voice-answer note in v11.1; seo.md's FAQPage/Speakable framing corrected and Bing AI Performance/DuckDuckGo notes added in v11.1; the robots.txt template's AI-crawler list corrected and completed (retired `Claude-Web` replaced, `OAI-SearchBot`/`Meta-WebIndexer`/`Perplexity-User`/`MistralAI-User`/`MistralAI-Index` added, `Content-Signal` default changed to `ai-train=yes` to match the per-bot rules) in v11.1). Each opens with a delta block where behaviour changed — read it before the body. **On any conflict, SKILL.md wins.**

---

## Working conventions

**Vercel sites — the repo is the project.**
`CLAUDE.md` + `docs/` live in the repo and travel with the code. Work in Claude Code; it auto-reads `CLAUDE.md` every session. Loop: edit `src/config/site.js` → `npm run build` → `npm run crosscheck` → commit → push → Vercel deploys. No ZIP, no upload.

**Cloudflare sites — Claude Project.**
Name: `[Site Name] — [domain or PENDING]`. Attach `docs/PROJECT.md` + `docs/keyword-map.md` + `docs/blog-plan.md` (+ `docs/taxonomy-map.md` + `docs/starter-product-list.csv` + `docs/faq-bank.md` + `docs/product-gaps.md` if keyword-engine produced them) — never in the deploy output. Project instructions: "This is the [Site Name] website project. Always read PROJECT.md before any task. keyword-map.md contains page-level keyword targets; blog-plan.md contains the blog build queue — use the next `Pending` row for each new post, flip it to `Built` once published. If taxonomy-map.md/starter-product-list.csv exist, they already answered the shop-structure/initial-products intake sections — don't re-derive them. If PROJECT.md shows Backend = Yes, deliver new blog/product content as *-import.json import files (not hardcoded HTML), and read references/backend.md."

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
[domain / chat / SMTP creds / RESEND_API_KEY (if resend) / GSC — and what breaks while each is unset]
[if Reply Portal: ADMIN_PASSCODE / storage (Vercel: Upstash Redis URL+Token · Cloudflare: REPLY_KV binding) — dashboard blank without them]

## Brand facts (real only — preserve them, never invent new ones)
[founded, ships, order rules, payment, real milestones, real reviews/awards + source]
Preserve real client-provided or migrated facts, reviews and awards (with source). Never fabricate ones that do not exist.
```
