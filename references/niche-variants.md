# Niche Variants Reference (v8)

> **v8 delta — read first.** The locale, age-gate, disclaimer, payment and shop-structure patterns below are still authoritative. What changed:
>
> - **Compliance language is now a first-class, TESTABLE mechanism** — not a per-niche memory. Intake Section H asks every client for: banned terms, required framings, prohibited claims, **and the authority behind each** (regulator / platform ad policy / statute). These land in `CLAUDE.md` (so Claude Code sessions inherit them) and in a **blocking crosscheck scan (B7)** across the entire build output — HTML, JSON-LD, llms.txt, `.well-known/*`, alt text, and MCP tool descriptions. See SKILL.md → "Compliance guardrails".
> - **Use the patterns below as the STARTING POINT for a vertical, then ask.** Never assume a niche's constraints from this file alone — regulations change and clients have their own counsel.
> - **Record the authority, always.** Future sessions need to know WHY a rule exists or they will rationalise around it.
> - **Compliance applies to imagery too.** If a product's whole legal position is "this is not the real thing", stock photos of the real thing contradict it in the first place an ad reviewer looks. Flag at intake, not at handover.
> - **Payments/chat:** live chat is now a multi-channel hub (link channels unlimited, max ONE widget) — see SKILL.md → "Live chat". Payment display rules below are unchanged.

---

Use this file whenever PROJECT.md contains any of:
- A location / locale field (any country)
- Age gate = Yes
- Compliance disclaimer (any)
- Finance/Pay in 4 = Yes
- Cookie/GDPR banner = Yes
- Non-standard payment methods

---

## Locale — worldwide, resolved on intake

There is no hardcoded country list. The client provides their location during intake and Claude resolves all locale settings automatically from that information.

### How it works

When the client states their country or location, Claude derives and applies all of the following before building anything:

```
Country name:              [from client input]
ISO country code:          [2-letter ISO — resolved by Claude]
Currency name + symbol:    [resolved by Claude]
Currency ISO code:         [resolved by Claude]
Date format:               [DD/MM/YYYY or MM/DD/YYYY — resolved by Claude]
Phone dial code + format:  [resolved by Claude]
Address format:            [resolved by Claude — country postal convention]
Tax type:                  [VAT / GST / Sales Tax / None — resolved by Claude]
Tax display:               [inc. tax / ex. tax / not applicable]
Meta locale code:          [e.g. en_NG, fr_FR, pt_BR — resolved by Claude]
Primary language:          [resolved — flag RTL if Arabic, Hebrew, etc.]
Privacy / data law:        [resolved by Claude — named legislation]
Cookie consent required:   [Yes / No — resolved from that country's legal regime]
Cookie policy page needed: [Yes / No]
Governing law for Terms:   [country + court system — resolved by Claude]
GSC target country:        [set to client's country]
```

Claude resolves every field using its knowledge of that country. No guessing — if a field is uncertain (e.g. a newly enacted law, a niche tax rule), Claude flags it explicitly to the client before building:

> "I've applied standard [Country] locale settings. Please confirm: [specific uncertain item] before I finalize."

### Special cases Claude handles automatically

**RTL languages** (Arabic, Hebrew, Persian, Urdu, etc.):
Add `dir="rtl"` to `<html>` tag. Flip all layout directions. Mirror navigation. Test at 375px RTL viewport.

**Multi-language countries** (Canada, Switzerland, Belgium, etc.):
Ask client which language the site is in. Set meta locale accordingly (e.g. `en_CA` vs `fr_CA`). Note in PROJECT.md.

**Non-Latin scripts** (Arabic, Chinese, Cyrillic, etc.):
Use a web-safe font stack or Google Font that supports the script. Never use a Latin-only font for non-Latin content.

**Strict cookie consent regimes** (EU GDPR, UK GDPR, Brazil LGPD, South Africa POPIA, etc.):
Cookie banner is mandatory. Analytics must NOT fire before consent. Cookie policy page required at `/cookie-policy/`. See GDPR section below for banner code.

**Countries with no formal data protection law:**
Note in Privacy Policy that the site follows international best practices. No cookie banner required but still recommended.

**Multi-region sites** (client ships internationally):
Use the client's home country as the primary locale. Note all served regions in the Shipping Policy. Add hreflang tags only if site has separate language versions. Use one currency only per site unless explicitly building a multi-currency store.

---

## Age gate variants

### Hemp / Cannabis — 21+ US (Mellow Fellow pattern)
No modal needed if site has clear 21+ disclaimer. Add to footer:
```html
<p class="disclaimer">For adults 21+ only · Hemp-derived products · 
Not FDA evaluated · Use responsibly</p>
```
Add to every page meta:
```html
<meta name="rating" content="adult">
```

### Nicotine — 21+ with modal (Fifty Bar pattern)
Full-screen age gate modal on first visit. Store confirmation in sessionStorage.
```html
<div id="age-gate" class="age-gate-overlay">
  <div class="age-gate-box">
    <div class="age-gate-logo">[Logo]</div>
    <h2>Age Verification</h2>
    <p>★ ★ ★ ★ ★</p>
    <p><strong>This website contains products with nicotine.<br>
    Nicotine is an addictive chemical.</strong></p>
    <p>You must be <strong>21 years or older</strong> to enter.</p>
    <button id="age-confirm" onclick="confirmAge()">I AM 21 OR OLDER — ENTER</button>
    <button id="age-deny" onclick="denyAge()">I AM UNDER 21 — EXIT</button>
    <p class="age-gate-legal">By entering you confirm you are of legal age 
    to purchase vaping products in your jurisdiction.</p>
  </div>
</div>
```
```javascript
function confirmAge() {
  sessionStorage.setItem('ageVerified', 'true');
  document.getElementById('age-gate').style.display = 'none';
}
function denyAge() {
  window.location.href = 'https://www.google.com';
}
window.addEventListener('DOMContentLoaded', () => {
  if (!sessionStorage.getItem('ageVerified')) {
    document.getElementById('age-gate').style.display = 'flex';
  }
});
```
Add to every page footer:
```html
<p class="nicotine-warning">WARNING: This product contains nicotine. 
Nicotine is an addictive chemical. For adult use only. Must be 21+ to purchase.</p>
```

### 18+ (general)
Simplified modal or footer notice only:
```html
<p class="disclaimer">Must be 18 or over to purchase. 
Age verification may be required on delivery.</p>
```

---

## Compliance disclaimers (add to footer + relevant pages)

### Hemp / Farm Bill (Mellow Fellow)
```
Hemp-derived products · Farm Bill compliant · Not FDA evaluated · 
For adults 21+ only · Use responsibly
```

### Nicotine (Fifty Bar)
```
WARNING: This product contains nicotine. Nicotine is an addictive chemical. 
For adult use only.
```

### Off-road vehicles (VoltTrail)
```
Most vehicles sold are designed for off-road, private land use only and are not 
road legal without appropriate type approval and DVLA registration. Always check 
local laws before use.
```
Add prominent warning box on product pages for off-road items:
```html
<div class="compliance-warning">
  ⚠ Off-Road Use Only: This vehicle is designed for off-road, private land use 
  only. It is not road legal in the UK without type approval registration, DVLA 
  registration, insurance, and appropriate licence.
</div>
```

### Supplements / Health products
```
These statements have not been evaluated by the Food and Drug Administration. 
This product is not intended to diagnose, treat, cure, or prevent any disease. 
For adults only.
```

---

## Payment method variants

### Crypto (all sites — default)
```
Accepted: BTC, USDT (add others per client)
Discount: 10% auto-applied (or via code — see PROJECT.md)
Display: Show wallet address + QR code placeholder at checkout
Instructions: "Send exact amount to address above. Include order number in memo."
```
Wallet address placeholder in code:
```javascript
const walletAddresses = {
  BTC: 'YOUR_BTC_ADDRESS_HERE',
  USDT: 'YOUR_USDT_TRC20_ADDRESS_HERE'
};
```

### Apple Pay (Fifty Bar pattern)
```
Display: Apple Pay button (Apple Pay JS SDK or Square)
Note in checkout: "Available on Safari/iOS only"
```

### Cash App / Chime (Fifty Bar pattern)
```
Display: Username/cashtag placeholder
Instructions: "Send to $[cashtag]. Include order number in note."
```
```javascript
const cashAppTag = '$YOUR_CASHTAG_HERE';
const chimeTag = '$YOUR_CHIME_TAG_HERE';
```

### Square Credit Card
```
Integration: Square Payment Link (generate in Square dashboard)
Display: Button → opens Square hosted payment page
Note: "Powered by Square — secure card processing"
```

### Pay in 4 — Manual installments (VoltTrail pattern)
Installment breakdown display:
```javascript
function calculatePayIn4(total) {
  const installment = (total / 4).toFixed(2);
  return {
    payment1: `£${installment} — Today`,
    payment2: `£${installment} — Week 4`,
    payment3: `£${installment} — Week 8`,
    payment4: `£${installment} — Week 12`
  };
}
```
Required disclaimer:
```
Pay in 4 lets you split your purchase into 4 equal, interest-free installments. 
The first payment is taken at checkout. Remaining payments are billed every 4 weeks. 
No interest, no fees. Subject to status. Age 18+ required. Terms apply.
```

### Klarna / Clearpay (UK/EU)
```
Display: Klarna badge + "Pay in 3" messaging
Integration: Add Klarna On-Site Messaging script (from Klarna dashboard)
Note: Requires Klarna merchant account
```

### Bank Transfer (all sites)
```
Display: Account details (fill from PROJECT.md or leave as placeholder)
Instructions: "Transfer exact amount. Use order number as reference."
Processing note: "Order held until payment clears (1–3 business days)."
```

---

## GDPR / Cookie banner (UK + EU sites)

Add to every page, fires on first visit:
```html
<div id="cookie-banner" class="cookie-banner">
  <p>We use cookies to improve your experience and serve relevant ads. 
  By continuing, you agree to our 
  <a href="/cookie-policy/">Cookie Policy</a>. 
  <a href="/privacy/">Privacy Policy</a></p>
  <button onclick="acceptCookies()">Accept Cookies</button>
</div>
```
```javascript
function acceptCookies() {
  localStorage.setItem('cookiesAccepted', 'true');
  document.getElementById('cookie-banner').style.display = 'none';
  // Initialize analytics only after consent
  initAnalytics();
}
window.addEventListener('DOMContentLoaded', () => {
  if (!localStorage.getItem('cookiesAccepted')) {
    document.getElementById('cookie-banner').style.display = 'flex';
  } else {
    initAnalytics();
  }
});
```
Analytics must NOT load before cookie consent on UK/EU sites.

---

## Niche-specific shop structures

### Type A — Category → Subcategory → Product (Mellow Fellow)
Nav mega dropdown:
```
Shop
├── Disposable Vapes
│   ├── All Disposable Vapes
│   ├── Live Resin
│   ├── Spinners
│   └── [subcategories...]
├── Vape Cartridges
│   └── [subcategories...]
├── Edibles
│   └── [subcategories...]
└── [categories...]
```
URL structure: `/shop/[category]/[subcategory]/`
Breadcrumb: Home > Shop > [Category] > [Subcategory] > [Product]

### Type B — Series/Collection → Product (Fifty Bar)
Nav mega dropdown:
```
Shop
├── All Flavors
├── Original Series — 6,500 Puffs
├── White Series — 6,500 Puffs
├── Black Series — 20,000 Puffs
└── [collections...]
```
URL structure: `/shop/[series]/`
Breadcrumb: Home > Shop > [Series] > [Product]

### Type C — Category → Brand → Model (VoltTrail)
Nav mega dropdown:
```
Electric Dirt Bikes
├── All Dirt Bikes
├── By Rider: Adult / Kids / Road Legal
└── By Brand: Sur-Ron / Talaria / Stark Varg / ...

Electric Quads
├── All Quads
├── By Rider: Adult / Kids
└── By Brand: Segway / Eco Rider / ...
```
URL structure: `/[top-category]/` · `/brands/[brand]/` · `/[top-category]/[brand]/[model]/`
Brand pages: full brand hub with brand story + product grid

---

## Analytics setup (all sites)

Add after cookie consent fires (or unconditionally for US sites without GDPR):

```html
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
<!-- End GTM — replace GTM-XXXXXXX with client's container ID -->
```

GTM container ID: add to PROJECT.md after client provides it.
GA4 Measurement ID: configure inside GTM, not directly in code.

Conversion events to track (configure in GTM):
- `add_to_cart` — fires on Add to Cart click
- `begin_checkout` — fires when checkout form is opened
- `purchase` — fires on order confirmation
- `whatsapp_click` — fires on WhatsApp button click
- `crypto_selected` — fires when crypto payment method selected
