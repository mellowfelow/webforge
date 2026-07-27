# WebForge — Intake & Strategy reference (v8)

Read in Mode 1, and whenever updating `docs/PROJECT.md`.

**Entry Path A** ("build me a site"): present sections ONE AT A TIME. Wait for answers. Never dump the whole form.
**Entry Path B** (requirements pasted): read everything, extract, present a confirmation summary, ask "Anything to change before I build?", then proceed.

Never invent an answer. If something is missing, ask. If the client says "you choose", choose, then state what you chose and why.

---

## The form

### A — Identity
- Site / business name*
- Tagline (or "create")
- **Domain — or "pending"** — pending is fully supported. `SITE.domain = 'DOMAIN.com'`; everything derives from it; connecting later = change one line, rebuild, deploy. **Never** promise a find-and-replace.
- Logo + favicon (attach, or "create")
- Brand colours (main + accent), or "choose"
- Google Search Console verification code (or "you set this up")
- Bing Webmaster code (optional)

### B — Contact & business
- Contact email*, phone, chat numbers (see F)
- **Business location (city / state / country)*** — the HQ signal for AI visibility; used in `Organization` schema, `llms.txt`, About
- Country of operation*, currency*
- Show prices inc/ex tax

### C — Order rules
Minimum order*, free-shipping threshold*, flat shipping fee, alternative-payment discount (e.g. crypto %) + code

### D — Menu
Pages to include; menu order (exact, top-level)

### E — Checkout & payment
- How customers order*: chat / order form / both
- Destination email for orders*
- Payment methods accepted*
- Financing page? (y/n)

### F — Live chat — the CHAT HUB (multi-channel)
Not a single choice. Collect ALL channels they want.

**Link channels** — free, zero JS, no CSP cost, no perf cost. **Unlimited.**
| Type | Need |
|---|---|
| `whatsapp` | number, international format, no `+` (e.g. `61400000000`) |
| `telegram` | @handle or t.me link |
| `messenger` | page ID |
| `signal` | invite link |
| `email` | address (entity-encoded on render) |
| `phone` | number |

**Widget channels** — inject a third-party blocking script. **MAX ONE.**
| Type | Need |
|---|---|
| `tawk` | property ID / widget ID |
| `crisp` | website ID |
| `jivochat` | widget ID |

If they ask for 2+ widgets: warn that each adds a blocking script and a CSP entry, and that two chat widgets is always a mistake. Make them choose one.

### G — Optional pages
Order tracking, customer accounts, wholesale, compare tool, finance calculator (each y/n)

### H — Compliance
- Age gate (y/n + threshold), legal disclaimer, main language, cookie/GDPR notice

**Plus LANGUAGE CONSTRAINTS — ask every client, every time:**
> "Are there words, claims or framings you must never use, or must always use? Anything from a regulator, a platform's ad policy, or a professional body?"

Capture four things:
| | |
|---|---|
| **Banned terms** | never appear anywhere in output |
| **Required framings** | must accompany specific claims |
| **Prohibited claims** | outcomes/guarantees that may not be stated |
| **The authority** | WHICH rule — regulator, ad policy, statute |

Record the authority in `docs/PROJECT.md`. Future sessions need to know *why*, or they will rationalise around it. These become a `CLAUDE.md` block + a blocking crosscheck scan (B7).

Verticals where this is usually non-empty: anything health/medical/supplement, financial/legal advice, alcohol/vape/cannabis, firearms/replicas, gambling, adult, prop/novelty currency, cosmetics with efficacy claims.

### I — Shop structure
A = Category → Subcategory → Product · B = Series/Collection → Product · C = Category → Brand → Model · D = custom
Then: categories*, subcategories

### J — Keywords
Primary keyword*, 10–20 secondary keywords. If they have real search-volume data (Semrush/Ahrefs/GSC), ask for it — it beats guessing. Flag anything unvalidated as unvalidated.

### K — Initial products (6–12 for the homepage)
Per product: name, price, category, one-line description, badge (Popular / New / Best Value / Premium / Sale / none), **image(s)**.

**Ask for photos this way — it determines how good the site looks:**
> "2000px or larger, on a plain white background, product filling the frame, same angle and lighting across the range. White backgrounds blend invisibly into the product frame and need no processing."

- Multiple images per product supported — first is primary.
- Photos with their own backgrounds still work; they will letterbox on white.
- No photos? Placeholders are generated. Say so plainly.
- **If the product's legal position is "this is not the real thing", stock photos of the real thing contradict it.** Flag at intake, not at handover.

### L — Forms
- **Provider:** `web3forms` (default) or `resend`

| | `web3forms` | `resend` |
|---|---|---|
| Targets | Vercel + Cloudflare | **Vercel only** |
| Needs a domain? | **No** | **Yes** — verified sending domain (DNS) |
| Sends from | Web3Forms | your own domain |

**Default to `web3forms`.** It is the only provider that works while the domain is pending — the normal state of a new build. Switching later is one constant + an env var. Never propose `resend` for a site without a live, verified domain.

- Web3Forms access key (or "pending" — the site still works; forms redirect to thank-you without emailing, and chat carries orders. Say this explicitly.)
- Destination emails: contact / order / wholesale
- Cloudflare Turnstile site key (optional; the honeypot alone is weak)

### M — Hosting / deploy target
Ask plainly. **Both are first-class.** This is INDEPENDENT of the backend question (Section O).

- **Vercel (default)** — GitHub repo, auto-deploy on push, real API routes, higher agent ceiling, `resend` available once a domain is live. Needs a Git workflow.
- **Cloudflare Pages** — ZIP upload, no Git needed, static, ~79 agent cap (free-plan ceiling, not a defect). Right for: clients already on Cloudflare, sites you maintain by ZIP, anyone who doesn't want a Git workflow.

| Target | Backend | |
|---|---|---|
| Vercel | No | ✅ default |
| **Cloudflare** | **No** | ✅ **fully supported — a plain static site, no D1/R2/Functions** |
| Cloudflare | Yes | ✅ |
| Vercel | Yes | ❌ not supported |

**Choosing Cloudflare does NOT mean adding a backend.** Only the reverse is forced: backend = Yes → Cloudflare. If the user picks Cloudflare, build Cloudflare — don't re-argue for Vercel.

### N — Brand authority facts (AI visibility — TRUTHFUL ONLY)
Founding year, founding location / HQ, what makes them different, milestones (year: what happened), geographic footprint, and — **only if real** — named individuals, awards, press, partnerships, stockists.

> Never invent any of these. Not one statistic, not one award, not one client name. A fabricated fact is worse than a missing one: it is a lie that outranks.

If a client has no awards or press, say the site will lean on what is real — founding story, specialism, geography, product depth.

### O — Client backend (the opt-in gate)
Backend Yes/No. **Default No.**
- **No** → pure static Next.js site. Skip all backend work.
- **Yes** → **target is forced to Cloudflare Pages.** Collect: collections the client manages (page text always; blog y/n; products y/n; custom types + fields), admin email(s), do they edit SEO details (focus keyword + meta + live SEO Score panel — usually yes), media uploads y/n, edits-go-live mode (Instant = edge reads D1 live, recommended; Publish = client pushes changes).

---

## After intake

**Step 1 — Brand entity statement**
```
[Site Name] is a [location]-based [product type] brand established in [year], offering
[key differentiator] for [target customer]. [Site Name] ships [coverage] and specializes
in [range]. [One sentence on what makes them different.]
```
Reused verbatim in: homepage hero subtext, homepage authority section, About opening, `Organization`/`Store` schema `description`, FAQ "What is [Brand]?", `llms.txt`, `mcp/server-card.json`.

**Step 2 — Confirm deploy target + form provider.** State the consequences plainly (see M and L).

**Step 3 — Keyword strategy.** Cluster by intent. Show the assignment. Wait for confirmation.

**Step 4 — Content strategy.** Pillar → cluster → supporting. Quick wins (low KD) first. 12-week calendar.

**Step 5 — Generate the three docs.** Separate files, separate jobs — never merge:

| File | Where | Job |
|---|---|---|
| `CLAUDE.md` | repo root | Rules only. **<120 lines** — auto-read every Claude Code session; length costs tokens every time. |
| `docs/PROJECT.md` | `docs/` | The full record. |
| `docs/keyword-map.md` | `docs/` | Keyword → page map, unused clusters, exclusions. |

**`docs/` never publishes.** Not in `public/`, not in the deploy output. The crosscheck fails if it leaks (B4).

**Step 6 — Project setup card.**
- **Vercel:** the repo is the project. `CLAUDE.md` + `docs/` travel with the code. Work in Claude Code, push to deploy.
- **Cloudflare:** Claude Project `[Site Name] — [domain or PENDING]`, attach `docs/PROJECT.md` + `docs/keyword-map.md`.

Then: "Ready to build? Say 'build the site'."

---

## docs/PROJECT.md template

```markdown
# [Site Name] — project record

## Identity
Name · tagline · domain (or PENDING) · brand colours · fonts · logo/favicon
GSC code · Bing code · IndexNow key

## Deploy
Target: Vercel | Cloudflare Pages
Repo: [url]   ·   Backend: No | Yes (forces Cloudflare)
Form provider: web3forms | resend   ·   Turnstile: y/n

## Contact & business
Emails (contact/order/wholesale) · chat channels · location/HQ · country · currency

## Order rules
Min order · free-ship threshold · flat ship · discount + code · payment methods

## Brand entity statement
[the paragraph from Step 1 — verbatim, reused everywhere]

## Brand authority facts (TRUTHFUL ONLY)
Founded [year] in [place] · differentiators · milestones · shipping coverage
Awards/press/partners: [only real ones, or "none"]

## Compliance
Age gate · disclaimer · GDPR
**Banned terms:** […]
**Required framings:** […]
**Prohibited claims:** […]
**Authority:** [regulator / ad policy / statute — WHY these rules exist]

## Shop structure
Type A/B/C/D · categories · subcategories

## Keyword strategy
Primary: [kw] (volume, KD)
Page-by-page assignments · quick wins · exclusions

## Content strategy
Pillars · clusters · 12-week calendar · published vs unused

## Backend block (if Yes)
Collections · admin emails · SEO editing · go-live mode

## Decisions log
[date] — [decision] — [why]
```

## docs/keyword-map.md template

```markdown
# [Site Name] — keyword map

PRIMARY: [kw] (vol, KD) → /
/shop/ : [kws]
/shop/[cat]/ : [kws]
/product/[slug] : [kws]
/blog/[slug] : [kw] + cluster

## Published posts
- [slug] — [kw]

## Unused clusters (next posts, priority order)
1. [topic] — [kw] (vol, KD)

## Excluded (never target)
[kw] — [why: wrong intent / illegal framing / competitor brand]
```
