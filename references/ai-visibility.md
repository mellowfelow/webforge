# AI Visibility Reference — GEO, AEO, LLM Visibility, Semantic SEO, Structured Data

> **v8 delta — read first.** Everything below still holds. Two additions:
>
> **1. Declaration vs capability.** This file (and Agent-Ready Sections A–N) describe the DECLARATION layer — static files that *describe* what a site offers. On a **Vercel** target there is now a CAPABILITY layer too: a real MCP server at `/api/mcp`, real JSON APIs, live ACP/UCP. See `references/agent-live.md`. A static site advertising an MCP server it cannot host is a lie and fails crosscheck B2. **Cloudflare targets ship the declaration layer only** and cap ~79 on isitagentready's free plan — that is the plan's ceiling, not a defect. Do not chase it.
>
> **2. Do not fabricate — but do preserve real content.** The ban is on INVENTION, not on real content. Inventing statistics, awards, press or client names that do not exist is a ship-stopper (a fabricated fact is a lie that outranks). But real facts the client provides, or that already exist on the site being migrated — reviews, ratings, awards, testimonials, named clients — are legitimate and MUST be carried over faithfully, with `Review`/`AggregateRating` schema where the reviews are real. Record each real authority fact and its source in `docs/PROJECT.md`. When migrating a site, never strip its real reviews/awards to satisfy this rule — preserve them.
>
> Off-site reality still decides most of this. No amount of on-site schema substitutes for real mentions, real profiles, real citations. Say so to clients rather than overselling the file set.

---

Apply all standards in this file during Mode 3 (Full Build) and Mode 4 (Audit + Fix). These signals determine whether a brand appears when users ask AI assistants (ChatGPT, Claude, Gemini, Perplexity) about products, services, or recommendations in the brand's niche.

---

## The 5 types of AI SEO — what they are and what to build

### 1. GEO — Generative Engine Optimization
Making content structured and authoritative enough that AI assistants quote or cite the brand when answering questions in the niche.

**What to build:**
- Brand entity statement in the first paragraph of homepage — one clear, factual sentence about who the brand is, what they sell, who they serve, where they ship
- Concrete, specific factual statements AI can extract and cite (avoid vague marketing language)
- Statistics and specific claims where available ("ships within 24 hours", "100+ products", "serving customers since [year]")
- Comparison language where natural: "Unlike [category generic], [Brand] offers [differentiator]"
- Case study and results language in blog content where available

**Evidence-backed tactics (GEO paper, arXiv:2311.09735):** work at least one of these into blog/authority-section copy wherever natural —
- **Direct quotations** (expert quotes, review excerpts, customer statements) — largest measured effect
- **Explicit statistics with a named source** ("68% of X, per [source]") — not vague claims
- **Cite sources inline** when referencing a claim/stat
- **Fluent, natural phrasing over keyword density** — keyword stuffing measurably *hurts* AI-visibility, not just looks spammy

These tactics help a smaller/newer site's content disproportionately *more* than an already-dominant competitor's — worth emphasizing to clients who feel they're competing against bigger incumbents.

**Build checklist:**
- [ ] Brand entity statement in homepage hero subtext (first 200 chars of body copy)
- [ ] Concrete factual claims on homepage (shipping speed, product count, founding year)
- [ ] About page contains specific, factual brand information (not just marketing)
- [ ] Blog content contains definitive statements, not hedging language
- [ ] Blog/authority-section copy includes at least one evidence-backed GEO tactic (quotation, cited statistic, or inline source citation) — not just comparison language

---

### 2. AEO — Answer Engine Optimization
Optimizing to appear as a direct answer in Google AI Overviews, ChatGPT, and Perplexity when users ask questions in the niche.

**What to build:**
- FAQ content written in direct-answer format: question as H2/H3, answer in first sentence (40-60 words), expand after
- "What is [Brand]?" question MUST appear on homepage FAQ section AND FAQ page
- "Where does [Brand] ship?" question on FAQ page — clear geographic answer
- "Is [Brand] legit/trusted?" question — answer with trust signals
- Questions formatted to match how real users ask AI assistants
- Short, extractable answer in first sentence — do not bury the answer

**Build checklist:**
- [ ] FAQ page has minimum 8 questions in direct-answer format
- [ ] Homepage FAQ section has minimum 4 questions including "What is [Brand]?"
- [ ] Every FAQ answer: direct answer in first sentence, 40-60 words ideal
- [ ] Questions cover: what the brand is, shipping, payments, returns, product quality, ordering process
- [ ] FAQPage JSON-LD on both FAQ page and homepage

---

### 3. LLM Visibility — Brand Mentions in AI
Making sure when someone asks an AI "recommend a [product type] brand", the brand gets mentioned.

**What to build:**
- Brand name used consistently — exact same spelling and capitalisation on every page, in every schema, in every meta tag, in every heading
- Strong about page: founding story, mission, product range, shipping coverage, contact info — minimum 300 words
- Organization schema with full entity information (see Structured Data section below)
- Social proof signals in content: review counts, customer mentions, trust indicators
- Content that directly answers "recommend a [product type] for [use case]" style queries

**Build checklist:**
- [ ] Brand name identical on every page — check homepage, nav, footer, schema, meta titles
- [ ] About page minimum 300 words with: founding story, mission, product range, geographic coverage, contact
- [ ] Organization schema includes: name, url, logo, description, knowsAbout, areaServed, sameAs
- [ ] Trust signals present on homepage: reviews/ratings reference, shipping promise, quality claim
- [ ] At least one page that answers "[Brand] review" or "[Brand] vs [generic category]" style query

---

### 4. Semantic SEO — Topical Authority
Building content that covers a topic so thoroughly that both Google AND AI systems recognise the brand as an authority.

**What to build:**
- Blog content that covers every major question in the niche (use keyword-cluster.txt)
- Topic clusters: each pillar topic has a hub page + 3-5 cluster articles all interlinked
- Internal links between related content using descriptive, keyword-rich anchor text
- Glossary or definitions section if the niche has technical terminology
- Content depth: not just keywords but comprehensive topic coverage

**Build checklist:**
- [ ] keyword-cluster.txt organizes all remaining keywords into topic clusters with content type and URL
- [ ] Blog index page clearly organizes posts by topic cluster
- [ ] Each blog post links to 2-3 related products or category pages
- [ ] Each category page links to relevant blog posts ("Learn more about [topic]")
- [ ] No orphan blog posts — every post linked from blog index and at least one other page

---

### 5. Structured Data for AI — Schema Signals
Schema markup is a direct information feed to AI systems. Beyond basic schema, specific fields improve AI understanding and citation.

**Enhanced Organization JSON-LD (homepage + about page):**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "[Site Name]",
  "url": "https://[domain]/",
  "logo": "https://[domain]/images/logo.webp",
  "description": "[Brand entity statement — same text used in homepage hero]",
  "foundingDate": "[year if known]",
  "areaServed": "[country or US/UK/Worldwide]",
  "knowsAbout": [
    "[primary keyword]",
    "[secondary keyword 1]",
    "[secondary keyword 2]",
    "[product category 1]",
    "[product category 2]"
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

**Speakable schema (homepage + about page):**
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
Add class `brand-statement` to the element containing the brand entity statement on homepage.
Add class `about-intro` to the opening paragraph of the about page.

**Enhanced Article JSON-LD for blog posts:**
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
    "@type": "Organization",
    "name": "[Site Name]",
    "logo": {
      "@type": "ImageObject",
      "url": "https://[domain]/images/logo.webp"
    }
  },
  "datePublished": "[ISO date]",
  "dateModified": "[ISO date]",
  "about": {
    "@type": "Thing",
    "name": "[primary topic of the article]"
  },
  "mentions": [
    {
      "@type": "Thing",
      "name": "[related topic or product mentioned]"
    }
  ]
}
```

---

## AI Visibility audit checklist (Mode 4)

Run every item. Mark ✓ pass / ✗ fail / ⚠ needs review.

**GEO signals**
- [ ] Brand entity statement in first 200 chars of homepage body copy
- [ ] Concrete factual claims on homepage (not just marketing language)
- [ ] About page contains specific, verifiable brand information
- [ ] Comparison or differentiator language present ("Unlike generic [category], [Brand]...")

**AEO signals**
- [ ] "What is [Brand]?" FAQ on homepage and FAQ page
- [ ] "Where does [Brand] ship?" FAQ on FAQ page
- [ ] All FAQ answers: direct answer in first sentence, 40-60 words
- [ ] FAQPage JSON-LD on homepage and FAQ page
- [ ] Minimum 8 questions on FAQ page, 4 on homepage FAQ section

**LLM visibility signals**
- [ ] Brand name consistent spelling/capitalisation across all pages
- [ ] About page minimum 300 words
- [ ] About page covers: founding story, mission, product range, shipping, contact
- [ ] Organization schema has: description, knowsAbout, areaServed, sameAs

**Semantic SEO signals**
- [ ] keyword-cluster.txt organized by topic cluster
- [ ] Blog posts interlink with related products/categories
- [ ] No orphan blog posts
- [ ] Category pages link to relevant blog content

**Structured data signals**
- [ ] Organization JSON-LD has description, foundingDate, areaServed, knowsAbout, sameAs
- [ ] Speakable schema on homepage and about page
- [ ] brand-statement CSS class on homepage entity statement element
- [ ] about-intro CSS class on about page opening paragraph
- [ ] Article schema includes about and mentions fields on blog posts
- [ ] All schema URLs are absolute, no empty fields

---

## AI visibility scoring (Mode 4)

| Score | Meaning |
|---|---|
| 90-100 | Excellent — brand well-positioned for AI citation |
| 70-89 | Good — minor signals missing, easy wins available |
| 50-69 | Needs work — key signals absent, AI unlikely to cite brand |
| Below 50 | Critical — brand invisible to AI assistants |

Score breakdown:
- GEO signals present: +25 points
- AEO signals present: +25 points
- LLM visibility signals present: +25 points
- Structured data signals present: +25 points

---

## The 12 authority signals (GEO scorecard) — v4 addition

AI-visibility tools (e.g. ogoing.ai) score brands on 12 signals. Publish TRUTHFUL, extractable content + schema for each you legitimately can:

ON-SITE, add truthfully:
1. Founding Year
2. Founding Location / HQ (engines explicitly flag "no HQ address found on homepage")
3. Quantifiable Scale (X products, N categories, strain count)
4. Competitive Differentiation (explicit "what makes us different")
5. Historical Milestones (year-by-year timeline)
6. National / Global Footprint (e.g. ships to all 50 states)
7. Industry Leadership (positioning since founding year)
8. Brand Narrative
9. Product Lines / Brands

Publish when REAL (client-supplied, or already on the site being migrated) — never invent:
10. Named Individuals (founders/executives)
11. Awards & Rankings
12. Endorsements / Partnerships

If #10-12 exist (client-provided or on the migrated site), publish them with their source. If they genuinely do not exist, omit them and report as "needs real client input" — never invent them. Inventing #10-12 = misinformation AI will repeat = brand risk.

### Schema for AI (homepage Store/Organization)
@type ["Store","Organization"], description (brand entity statement), foundingDate, foundingLocation (Place), address (PostalAddress region+country), areaServed, numberOfItems, knowsAbout[], priceRange, brand, makesOffer (AggregateOffer — NEVER a bare itemOffered Product), sameAs[] (official domain + socials). Plus a visible "About [Brand]" authority section on the homepage and a milestones timeline + differentiation grid on the About page (>700 words).

### llms.txt
Include: brand summary, "Brand Facts (cite these)" block, Competitive Differentiation, Historical Milestones, key pages, and explicit Citation Guidance for "best / most trusted / who do you recommend" category queries.

### Off-site reality check (Perplexity / live-web engines)
Perplexity (Sonar Pro) and other live-web engines use real-time web search, NOT on-page content. They return "Not found" / 0 when there are no EXTERNAL references to the domain. On-page SEO + schema CANNOT move them alone. The fix is OFF-SITE and must be reported as a manual action:
- Google Business Profile + real business-directory listings
- Real social profiles (IG, X, etc.) that link to the site, matching schema sameAs
- The official brand domain linking to the store (two-way sameAs)
- Genuine reviews / mentions on category directories

Claiming and keeping Google Business Profile and Apple Business Connect listings current (both free, self-service) also feeds voice-assistant "near me" answers on Google Assistant/Gemini and Siri — not just Maps/local-pack results.

If a backlink tool shows a huge link count from very few referring domains + Authority Score near 0 + no organic traffic + a traffic graph that spiked then crashed: that is inherited spam/PBN from a prior domain owner, not real authority. Check GSC → Security & Manual Actions BEFORE any disavow. Google ignores most spam links automatically; disavow only on a manual action. Prioritize building REAL signals over disavowing.

Scores update on each engine's re-crawl cycle (days–weeks), not instantly. On-site signals are the controllable lever.
