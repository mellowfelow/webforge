# WebForge Backend Module (OPTIONAL — read only when backend = Yes) (v8)

> **v8 delta — read first.** The backend is **unchanged** and this file remains authoritative. Three framing notes:
>
> - **The backend is Cloudflare-only.** It is built on D1 + R2 + Pages Functions. **backend = Yes forces the deploy target to Cloudflare Pages** — say this plainly at intake rather than discovering it at build time. Vercel + backend is NOT supported; do not improvise a port to Postgres/Blob.
> - **The reverse is NOT true.** Cloudflare + backend = No is a normal, fully supported, first-class build (a plain static export). Never infer a backend from the target. Default is No, and most sites should stay No — a backend is only worth it when the client genuinely wants to edit content themselves.
> - **The site layer is now a Next.js static export**, not hand-written HTML. The two-layer model is unaffected: Claude still owns the template layer, the client still owns the content layer in D1 + R2, and the edge merge still injects saved values before the page reaches the visitor. `data-cms` keys are still permanent IDs. Where this file says "the ZIP", read: the ZIP produced by `npm run zip` from `out/`.

---

The client backend is an **opt-in, requirement-gated** addition (Intake Section O). When the user does NOT request it, build a pure static site exactly as v4 — this file does not apply. When the user DOES request it, this file is the complete spec.

Everything here runs **inside one Cloudflare Pages project**, on the operator's personal Cloudflare account, dashboard-only (no command line required by the operator). No external host, no PHP, no MySQL, no WordPress.

---

## 0. What this is (and isn't)

**Is:** a native, Cloudflare-only CMS that gives the client a login at `clientdomain.com/admin` to manage content, blog posts, products, media, and SEO details — without touching code, ZIP files, or the Cloudflare dashboard. The site stays Claude-editable at the same time.

**Isn't WordPress / RankMath:** Cloudflare Pages is static — no PHP, no MySQL, no WordPress, so real RankMath cannot run and there is nothing to "read RankMath settings from." Instead WebForge **replicates** the experience: content in **D1** (Cloudflare's serverless SQL), media in **R2** (object storage), logic in **Pages Functions**, and a native **SEO Score panel** that runs the same heuristic checks RankMath is known for. The client gets the familiar green/orange/red score and checklist; under the hood it's all Cloudflare.

---

## 1. The two-layer model (the core principle — never break it)

The reason "client edits to the extreme AND Claude still edits" works without collisions: two layers own different things.

| Layer | Owner | Lives in | Contains |
|---|---|---|---|
| **Template** | Claude | the deploy ZIP | HTML structure, design, default content, `data-cms` editable zones, `cms-manifest.json`, SEO Score logic, admin UI |
| **Content** | the client | D1 + R2 | field values, blog posts, products, uploaded media, SEO overrides |

**Edge merge:** a Pages Function (`functions/_middleware.js`) runs on every request, reads this site's values from D1, and uses **HTMLRewriter** to inject them into the marked zones **server-side** before the HTML reaches the visitor (so it's fully SEO-safe — content is in the response, not painted in by JS).

**The one rule that prevents all collisions — override wins, else default:**
- If the client has saved a value for a zone → the edge injects it.
- If not → Claude's baked-in default (in the ZIP) shows through.

Consequences (state these to the operator):
- **Redeploying never wipes client content** — content is in D1, not the files you replace.
- **Client edits never break the design** — they edit field *values*, not structure/markup.
- **A backend-free client works on the same codebase** — with no D1 overrides, every zone just renders its default; the site is a normal static site.
- **Field keys are permanent IDs** — once `hero.title` ships, never rename it, or you orphan the client's saved value. Add new keys freely; don't rename or reuse old ones.

---

## 2. Editable zones + the manifest

**Mark editable zones in templates** with a stable `data-cms` key and a sensible default as the actual content:
```html
<h1 data-cms="home.hero.title">Premium [Product] Delivered Fast</h1>
<p data-cms="home.hero.subtext">[Brand] is a [location]-based ... </p>
<meta name="description" data-cms="seo.home.description" content="Default meta description ~150 chars...">
<title data-cms="seo.home.title">Default Title 50-60 chars</title>
```
Rules: keys are `dot.namespaced` and **stable forever**; defaults are real, shippable content (the site must look finished with zero client edits); SEO zones (`title`, `meta description`, canonical, OG) are editable too so the client controls SEO.

**`cms-manifest.json`** is the contract between Claude and the admin panel. The admin dashboard is auto-built from it — add a field here + a matching `data-cms` zone, and the client's dashboard grows a control automatically. Shape:
```json
{
  "site": "client-slug",
  "fields": [
    { "key": "home.hero.title", "label": "Homepage headline", "type": "text", "page": "/", "seoRole": "h1", "default": "Premium ... Delivered Fast" },
    { "key": "seo.home.title", "label": "Home SEO title", "type": "text", "page": "/", "seoRole": "title", "max": 60, "default": "..." },
    { "key": "seo.home.description", "label": "Home meta description", "type": "textarea", "page": "/", "seoRole": "metaDescription", "max": 160, "default": "..." }
  ],
  "collections": {
    "posts": {
      "label": "Blog posts", "route": "/blog/[slug]/",
      "schema": {
        "slug": "string!unique", "title": "string!", "excerpt": "string",
        "body": "richtext!", "image": "media", "focusKeyword": "string",
        "metaTitle": "string", "metaDescription": "string",
        "published": "bool", "date": "date!"
      }
    },
    "products": {
      "label": "Products", "route": "/product/[slug]/",
      "schema": {
        "slug": "string!unique", "name": "string!", "price": "number!",
        "currency": "string!", "category": "string!", "description": "richtext!",
        "image": "media", "badge": "string", "availability": "string",
        "focusKeyword": "string", "metaTitle": "string", "metaDescription": "string"
      }
    }
  }
}
```
Field types: `text`, `textarea`, `richtext` (safe-HTML editor), `media` (R2 image), `number`, `bool`, `date`, `string`. `!` = required, `unique` = unique. Only include the collections the client asked for in Section O.

---

## 3. Files to generate (when backend = Yes)

```
functions/
  _middleware.js        Edge merge: read D1 for this site, HTMLRewriter-inject values into data-cms zones; render D1-served posts/products at their routes; emit matching JSON-LD; default-fallback for every zone.
  api/
    login.js            POST {email,password} → verify HASHED password (Web Crypto PBKDF2/SHA-256) → set signed session cookie (HttpOnly, Secure, SameSite=Lax) using SESSION_SECRET. Rate-limit attempts.
    logout.js           Clears the session cookie.
    content.js          GET (load all field values + collections for the dashboard) / POST (save a field value or upsert a post/product). Auth required (valid session).
    upload.js           POST multipart → store image in R2 → return its URL/key. Auth required. Validate type/size.
    import.js           POST a *-import.json body → validate against manifest collection schema → bulk upsert into D1 (slug = key, last-write-wins). Auth required. Returns counts + any skipped rows.
    seo-score.js        POST {content, focusKeyword, title, metaDescription, url} → returns 0-100 score + per-check pass/warn/fail + snippet preview. Pure heuristic; no external calls.
    setup.js            GET/POST guarded by ?token=ADMIN_SETUP_TOKEN → one-time: create first admin (email + hashed password) → then self-disable (refuses once a user row exists).
admin/
  index.html            Login screen (posts to /api/login). Mobile-first.
  dashboard.html        Auto-built from cms-manifest.json: Content (page fields), Blog, Products, Media, SEO (per page/item: focus keyword + meta + live SEO Score panel + snippet preview), Import (upload *-import.json). Mobile-first, single-column on phones.
  admin.js              Fetches manifest + content from /api/*, renders controls, autosaves, runs SEO Score live, handles uploads + import.
  admin.css             Admin styling (separate from public styles.css).
cms-manifest.json       (section 2)
schema.sql              (section 4)
CLOUDFLARE-SETUP.md     (section 6 — the dashboard setup + deploy guide, tailored to this site)
```
Plus: public templates carry `data-cms` marks; `robots.txt` gets `Disallow: /admin/` + `Disallow: /api/`; `_headers` sets `/admin/*` → `Cache-Control: no-store`.

**Stack constraints to respect (Cloudflare Pages Functions):** CPU ~10ms/request free (50ms paid) — keep edge work light (D1 read + HTMLRewriter is fine; no heavy compute); subrequest limits apply; use the D1 and R2 **bindings** (`env.DB`, `env.MEDIA`) — never connection strings. Set `compatibility_date` recent; enable `nodejs_compat` only if a function needs Node APIs (prefer Web Crypto, which needs no flag).

---

## 4. schema.sql (D1 tables)

```sql
-- Field values (the override layer for data-cms zones)
CREATE TABLE IF NOT EXISTS content (
  key   TEXT PRIMARY KEY,         -- matches a data-cms key
  value TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

-- Blog posts
CREATE TABLE IF NOT EXISTS posts (
  slug TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  excerpt TEXT,
  body TEXT NOT NULL,
  image TEXT,
  focus_keyword TEXT,
  meta_title TEXT,
  meta_description TEXT,
  published INTEGER NOT NULL DEFAULT 0,
  date TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

-- Products
CREATE TABLE IF NOT EXISTS products (
  slug TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  price REAL NOT NULL,
  currency TEXT NOT NULL,
  category TEXT NOT NULL,
  description TEXT NOT NULL,
  image TEXT,
  badge TEXT,
  availability TEXT DEFAULT 'InStock',
  focus_keyword TEXT,
  meta_title TEXT,
  meta_description TEXT,
  updated_at TEXT NOT NULL
);

-- Media registry (R2 objects)
CREATE TABLE IF NOT EXISTS media (
  key TEXT PRIMARY KEY,           -- R2 object key
  url TEXT NOT NULL,
  alt TEXT,
  uploaded_at TEXT NOT NULL
);

-- Admin users (passwords HASHED, never plaintext)
CREATE TABLE IF NOT EXISTS users (
  email TEXT PRIMARY KEY,
  password_hash TEXT NOT NULL,
  salt TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'admin',
  created_at TEXT NOT NULL
);

-- Sessions (or use stateless signed cookies; table optional for revocation)
CREATE TABLE IF NOT EXISTS sessions (
  id TEXT PRIMARY KEY,
  email TEXT NOT NULL,
  created_at TEXT NOT NULL,
  expires_at TEXT NOT NULL
);
```
Only create the collection tables the client actually uses (drop `products` if they don't sell, etc.).

---

## 5. The SEO Score panel (the RankMath replica)

Pure heuristic, computed in `seo-score.js` and/or client-side in `admin.js` — no WordPress, no external API. For a given {content/body, focusKeyword, title, metaDescription, url}, score 0-100 with traffic-light checks:

- Focus keyword in SEO title — and ideally near the start
- Focus keyword in meta description
- Focus keyword in the URL slug
- Focus keyword in the first paragraph of the body
- Focus keyword in at least one subheading (H2/H3)
- Keyword density in a sane band (~0.5-2.5%) — flag stuffing
- SEO title length 50-60 chars
- Meta description length ~120-160 (target ~150)
- Content length above a minimum (e.g. ≥300 words for posts)
- At least one internal link + ideally one external link
- All images have alt text (and ideally one alt contains the keyword)
- Readability (avg sentence length, paragraph length) — light touch
- **Google snippet preview**: render the title + URL + meta as Google would, desktop + mobile

Output shape: `{ score: 0-100, checks: [{id,label,status:'pass|warn|fail',hint}], snippet:{title,url,description} }`. Show it live as the client types, exactly like the RankMath editor sidebar.

> Audit note: since there is no real RankMath on a static site, never tell the operator to "check RankMath settings." The SEO Score panel IS the equivalent; audit it against the checks above.

---

## 6. CLOUDFLARE-SETUP.md — the dashboard setup + deploy guide (REQUIRED OUTPUT, dashboard-only)

Every backend build MUST output this, tailored with the site's project name, DB name, and bucket name. It is 100% Cloudflare dashboard — no command line. Generate it like this (fill the [bracketed] names):

```markdown
# Cloudflare Setup — [Site Name] (backend-enabled)

Do these once. After this, every future update is just re-uploading the site folder (Step 8 redo). Your database, bucket, bindings, secrets, and all client content persist across redeploys.

ORDER MATTERS: create the database + bucket first, create the Pages project by uploading, then add bindings + secrets, then REDEPLOY (bindings only take effect on deployments made after you add them).

## 1. Create the database
Sidebar → Storage & Databases → D1 SQL Database → Create. Name it `[site-slug]-db`. Create.

## 2. Build the tables
Open `[site-slug]-db` → Console tab. Open the generated `schema.sql`, copy ALL of it, paste into the console, Execute. Confirm tables appear under the Tables tab.

## 3. Create the media bucket
Sidebar → R2 Object Storage → Create bucket. Name it `[site-slug]-media`. Create.

## 4. Create the Pages project (upload)
Workers & Pages → Create → Pages → Upload assets. Project name `[site-slug]`. Drag in the WHOLE generated site folder (it includes the `functions/` and `admin/` folders — don't upload only the HTML). Deploy. The static site is now live; backend not wired yet.

## 5. Attach the bindings
Project → Settings → Bindings → Add (do this twice):
- D1 database → Variable name `DB` (exactly) → select `[site-slug]-db`.
- R2 bucket → Variable name `MEDIA` (exactly) → select `[site-slug]-media`.
(The function code references `DB` and `MEDIA` — names must match exactly.)

## 6. Add the two secrets
Project → Settings → Variables and Secrets. Add two, tick Encrypt on each:
- `SESSION_SECRET` — paste a long random string (signs login cookies).
- `ADMIN_SETUP_TOKEN` — paste another long random string (protects one-time admin creation).

## 7. Redeploy (activates bindings + secrets)
Deployments tab → latest deployment → ⋯ → Retry deployment (or Create new deployment and re-upload the same folder). Without this the functions can't see the database.

## 8. Create the client's login (once)
Visit `[site-slug].pages.dev/admin/setup?token=YOUR_ADMIN_SETUP_TOKEN` once. Set the client's email + password. It writes the hashed credentials and disables itself. The client now logs in at `/admin`.

## 9. Connect the domain
Project → Settings → Custom domains → Set up a domain → enter `[domain]`. Cloudflare adds the DNS. Login then lives at `[domain]/admin`.

## Verify it worked
1) Load the site — defaults render, content is in the HTML source (SEO-safe).
2) Go to /admin, log in.
3) Change one field, reload the public page — your edit shows live.

## Future updates
Every later change = Deployments → Create new deployment → re-upload the site folder. DB, bucket, bindings, secrets, and all client content stay intact.

If a step about secrets says the project isn't found, do Step 4 first, then Steps 5-7.
```
Labels in the Cloudflare sidebar shift occasionally; if "Storage & Databases" isn't exact, look for D1 and R2 under the storage grouping.

---

## 7. Adding content with Claude (the import workflow — dashboard-only)

On a backend site, content is DATA, not markup. When the operator asks Claude to add blog posts or products:

1. Claude generates an **import file** — `blog-import.json` or `products-import.json` — an array of records matching the collection schema in `cms-manifest.json`:
   ```json
   [
     { "slug":"how-to-choose", "title":"How to Choose", "excerpt":"...", "body":"<p>...</p>",
       "image":"", "focusKeyword":"choose X", "metaTitle":"...", "metaDescription":"...",
       "published":true, "date":"2026-06-22" }
   ]
   ```
2. The operator/client loads it via the admin panel: **log into `/admin` → Import → upload the .json → Import**. `api/import.js` validates each record against the schema and upserts into D1 (slug is the key; re-importing the same slug updates it). Items go live immediately and are fully editable in the dashboard afterward.
3. Claude also adds the new public URLs to `sitemap.xml` (generated from the import slugs) and refreshes `<lastmod>` — these ship in the next site ZIP so the pages are indexable.

**Fallback (only if the operator prefers):** Claude can emit `seed.sql` (INSERT statements) to paste into the D1 Console (same place as Step 2). The Import button is the primary, no-SQL path.

**Never** hardcode posts/products into static HTML on a backend site — the edge layer serves them from D1, so hardcoded copies would duplicate or be overridden.

---

## 8. Backend intake (Section O detail)

Ask only after confirming backend = Yes:
- **Collections to manage:** page text (always), Blog (Y/N), Products (Y/N), custom types (name + fields, e.g. "Team", "Testimonials").
- **SEO editing:** can the client edit focus keyword + meta + see the SEO Score panel? (Y/N — usually Yes.)
- **Media uploads:** Y/N (enables R2 + upload.js).
- **Admin users:** email(s) for the login(s).
- **Edits go live:** Instant (edge reads D1 live — simplest, recommended) vs Publish (client clicks Publish to push). Default Instant unless the client wants a draft/publish gate.

Record all of this in PROJECT.md under a `## Backend` block so Mode 2/4 know the site is backend-enabled and which collections exist.

---

## 9. Backend audit checklist (Mode 4 — Audit D)

- Manifest ↔ template: every `data-cms` key in templates is in the manifest and vice versa; keys are unchanged from any prior manifest (stable IDs); no orphans.
- No hardcoded posts/products that the edge would override or duplicate.
- `_middleware.js`: a default fallback exists for every injected zone; edge-emitted JSON-LD parses and follows the per-page-type schema rules; no heavy compute (CPU-budget safe).
- Auth: `login.js` compares a HASH (no plaintext); sessions signed with `SESSION_SECRET` (HttpOnly/Secure/SameSite); `setup.js` self-disables once a user exists and is `ADMIN_SETUP_TOKEN`-gated.
- Indexing: `robots.txt` disallows `/admin/` + `/api/`; `_headers` sets `/admin/*` to `no-store`.
- Import files: valid JSON; match the collection schema (required fields present, types correct, unique slugs).
- `CLOUDFLARE-SETUP.md`: present, names this site's project/DB/bucket, includes both secrets and the post-binding redeploy step.
- `node --check` passes on every file in `functions/` and `admin/*.js`.

---

## 10. Direction-of-travel note (informational)

Cloudflare is steering NEW full-stack projects toward Workers; as of 2026 Workers reached feature parity with Pages for static assets, SSR, and custom domains, and new platform features tend to land on Workers first. There is no forced-migration deadline, and everything this module uses (D1, R2, KV bindings, Pages Functions) is fully supported on Pages today. Pages Functions already run on Workers infrastructure, so the function code is portable if a migration is ever desired. No action needed now — keep the dashboard-only Pages workflow.
