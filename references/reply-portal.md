# WebForge — Reply Portal reference (v2.0)

Read when Intake **Section P = Yes** — available on **both** deploy targets now. This is the order/enquiry **transactional** system: a passcode-gated admin dashboard that stores orders + enquiries, composes payment-details and reply emails, and mirrors them to WhatsApp. It is **not** the Cloudflare CMS (`references/backend.md`) — different stack, different purpose. The two can coexist on the same Cloudflare site.

**v2.0 delta — this reference was rewritten against the actual production pattern (Peak Pedal, the most advanced live build), not the aspirational design the v1.0 doc described.** What changed:
- Storage is a single Redis/KV **hash per type**, sorted in memory — not a hand-rolled REST client with a separate sorted-set index. Simpler, fewer round trips, and what both live Vercel sites actually run.
- Payment details are entered as **one pasted blob, auto-split into individually-copyable fields** (`parsePaymentDetail`) — works for a bank account, a wallet address, a payment link, or any future rail, in any country, with zero code changes. This is the piece that makes the portal genuinely flexible per country/payment method.
- **Cloudflare now gets the full portal too**, via Pages Functions + KV + a raw-socket SMTP client — previously Section P was Vercel-only.
- **Web3Forms is gone.** Every form/email path — Vercel and Cloudflare — runs on SMTP now. See `references/forms.md`.
- The WhatsApp checkout path is wired end-to-end: it now mints an order number, saves it, and sends the same structured message the email channel does — this was a gap in the reference build, fixed here.
- Language is **JavaScript** (`.js`/`.jsx`), matching the rest of the site — not TypeScript.

The email side (template, the emails, payment-terms copy) lives in `references/forms.md`. This file covers **storage, auth, the admin UI, IDs, payment-detail parsing, and WhatsApp**. Everything brand/location/payment-specific comes from the one `SITE.reply` block documented in `forms.md` — nothing here hardcodes a colour, prefix, currency, location, method, or channel.

---

## 0. Principle — invariant plumbing, config-driven content

Same split as the emails: the **plumbing** (storage access, auth, store shape, admin structure, payment-detail parsing) is invariant across sites and across targets; the **content** (order prefix, currency, locale, channels, accent colour, payment methods, copy) is read from `SITE.reply`. This is what lets the portal be dropped onto — or retrofitted into — any site, in any country, on either deploy target.

**Graceful degradation (every layer, both targets):**
- No SMTP env vars → forms still redirect to thank-you; nothing crashes.
- No storage configured (Redis or KV) → orders/enquiries still **email through**; they just don't appear in the dashboard.
- No `ADMIN_PASSCODE` → `/api/admin/*` (or the Cloudflare Function equivalent) returns 503 (never open).

Requires `smtp` as the form provider on either target — see `references/forms.md`. There is no longer a "static/no-env-var" build that can't run the portal; Cloudflare's Pages Functions give it real server-side code too.

---

## 1. IDs — short, human, collision-resistant, generated client-side

```js
// Zero imports so the identical function runs anywhere it's called from —
// checkout mints the ref client-side before any server round-trip, so the
// same number appears in the WhatsApp message, the dashboard, and the email.
function makeOrderNumber(prefix) {
  const t = Date.now().toString(36).toUpperCase().slice(-4);
  // No 0/O/1/I/L — an operator reading this off a screenshot or over the
  // phone can never confuse a character. crypto.getRandomValues, not
  // Math.random — this is a lookup key, not a security token, but it's free.
  const alphabet = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  const bytes = typeof crypto !== 'undefined' && crypto.getRandomValues
    ? crypto.getRandomValues(new Uint8Array(2))
    : [Math.floor(Math.random() * 256), Math.floor(Math.random() * 256)];
  let r = '';
  for (const b of bytes) r += alphabet[b % alphabet.length];
  return `${prefix}-${t}${r}`;
}
```

- `generateOrderNumber()` → `makeOrderNumber(SITE.reply.orderPrefix)`, called from the checkout component only (client-side, after mount — `Date.now`/`crypto` during SSR would cause a hydration mismatch). Validate incoming refs server-side against `/^{PREFIX}-[A-Z0-9]{4,8}$/`.
- `generateEnquiryId()` → `ENQ-` + 8 lowercase-base36 timestamp+random characters, uppercased. Server-side (Vercel API route / Cloudflare Function) is fine here — enquiries aren't minted at checkout, so there's no hydration concern.

---

## 2. Storage

### Vercel — Upstash Redis (`lib/redis.js`, `lib/orderStore.js`, `lib/enquiryStore.js`)

**`lib/redis.js` — the `@upstash/redis` SDK, not a hand-rolled REST client.**

```js
import { Redis } from '@upstash/redis';

// Checks the common env var names so whichever one Vercel's Storage tab
// assigns just works. FIVE pairs, not four — the fifth is the actual pair
// Vercel generates for an Upstash Redis connection with Custom Environment
// Variable Prefix = "UPSTASH_REDIS": Vercel inserts its own "_KV_" segment,
// so it does NOT match the plain UPSTASH_REDIS_REST_URL pattern despite the
// identical prefix. Confirmed live — do not drop this pair.
const CREDENTIAL_CANDIDATES = [
  ['UPSTASH_REDIS_REST_URL', 'UPSTASH_REDIS_REST_TOKEN'],
  ['KV_REST_API_URL', 'KV_REST_API_TOKEN'],
  ['STORAGE_REST_API_URL', 'STORAGE_REST_API_TOKEN'],
  ['STORAGE_KV_REST_API_URL', 'STORAGE_KV_REST_API_TOKEN'],
  ['UPSTASH_REDIS_KV_REST_API_URL', 'UPSTASH_REDIS_KV_REST_API_TOKEN'],
];

let cached;
export function getRedis() {
  if (cached !== undefined) return cached;
  for (const [urlKey, tokenKey] of CREDENTIAL_CANDIDATES) {
    const url = process.env[urlKey];
    const token = process.env[tokenKey];
    if (url && token) { cached = new Redis({ url, token }); return cached; }
  }
  cached = null;
  return null;
}
export function isStoreConfigured() { return getRedis() !== null; }
```

**Store pattern (orders + enquiries identical) — one hash per type, in-memory sort:**

```js
// lib/orderStore.js
import { getRedis, isStoreConfigured } from './redis';

const KEY = `${SITE.reply.orderPrefix.toLowerCase()}:orders`; // e.g. 'pp:orders' — namespaces multiple sites sharing one DB

export const isOrderStoreConfigured = isStoreConfigured;

export async function saveOrder(order) {
  const redis = getRedis();
  if (!redis) return;
  await redis.hset(KEY, { [order.orderNumber]: JSON.stringify(order) });
}

export async function listOrders() {
  const redis = getRedis();
  if (!redis) return [];
  const all = await redis.hgetall(KEY);
  if (!all) return [];
  return Object.values(all)
    .map((v) => (typeof v === 'string' ? JSON.parse(v) : v))
    .sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime());
}

export async function getOrder(orderNumber) {
  const redis = getRedis();
  if (!redis) return null;
  const raw = await redis.hget(KEY, orderNumber);
  if (!raw) return null;
  return typeof raw === 'string' ? JSON.parse(raw) : raw;
}

// paymentDetails: { methodId, fields:[{label,value}], sentAt } — stored so
// the customer's payment-details link can still render the same fields if
// they come back to it later.
export async function markOrderSent(orderNumber, paymentDetails) {
  const order = await getOrder(orderNumber);
  if (!order) return;
  order.status = 'payment-sent';
  if (paymentDetails) order.paymentDetails = paymentDetails;
  await saveOrder(order);
}

export async function markPaymentConfirmed(orderNumber) {
  const order = await getOrder(orderNumber);
  if (!order) return;
  order.status = 'payment-confirmed';
  await saveOrder(order);
}

export async function deleteOrder(orderNumber) {
  const redis = getRedis();
  if (!redis) return;
  await redis.hdel(KEY, orderNumber);
}
```

`enquiryStore.js` is the identical shape against `{prefix}:enquiries`, with `markEnquiryReplied` (`new → replied`) instead of the two order-status transitions.

**Shapes:** `StoredOrder { orderNumber, customerName, customerEmail, customerPhone, address?, items:[{name,quantity,lineTotal}], amountDue, paymentMethod, notes?, status:'pending'|'payment-sent'|'payment-confirmed', channel:'whatsapp'|'email', createdAt, paymentDetails? }`. `StoredEnquiry { id, type:'contact'|'wholesale', name, email, phone?, message, meta?:Record<string,string>, status:'new'|'replied', createdAt }`.

### Cloudflare — KV (`functions/_shared/orderStore.js`, `functions/_shared/enquiryStore.js`)

Cloudflare Pages Functions run in the Workers runtime — no Upstash REST needed; a KV namespace is bound directly. Same shapes, same status lifecycles, index-array pattern instead of a hash (KV has no native hash type). **Binding name is the fixed constant `REPLY_KV`** on every site — same convention the CMS backend uses for `DB`/`MEDIA` (`references/backend.md`), so the code never hardcodes a per-site name and a retrofit never needs a find-and-replace:

```js
// functions/_shared/orderStore.js
const INDEX_KEY = 'order:index';
const MAX_INDEXED = 500; // oldest fall off the dashboard list, never the KV record itself

export const isOrderStoreConfigured = (env) => Boolean(env.REPLY_KV);

export async function saveOrder(env, order) {
  const kv = env.REPLY_KV;
  if (!kv) return false;
  await kv.put(`order:${order.orderNumber}`, JSON.stringify(order));
  const index = JSON.parse((await kv.get(INDEX_KEY)) || '[]');
  index.unshift({ orderNumber: order.orderNumber, createdAt: order.createdAt });
  if (index.length > MAX_INDEXED) index.length = MAX_INDEXED;
  await kv.put(INDEX_KEY, JSON.stringify(index));
  return true;
}

export async function listOrders(env, limit = 200) {
  const kv = env.REPLY_KV;
  if (!kv) return [];
  const index = JSON.parse((await kv.get(INDEX_KEY)) || '[]');
  const results = await Promise.all(
    index.slice(0, limit).map(async (entry) => {
      const data = await kv.get(`order:${entry.orderNumber}`);
      return data ? JSON.parse(data) : null;
    })
  );
  return results.filter(Boolean);
}

export async function getOrder(env, orderNumber) {
  const kv = env.REPLY_KV;
  if (!kv) return null;
  const data = await kv.get(`order:${orderNumber}`);
  return data ? JSON.parse(data) : null;
}

// paymentDetails: { methodId, fields:[{label,value}], sentAt } — same shape as the Vercel store.
export async function markOrderSent(env, orderNumber, paymentDetails) {
  const order = await getOrder(env, orderNumber);
  if (!order) return;
  order.status = 'payment-sent';
  if (paymentDetails) order.paymentDetails = paymentDetails;
  await env.REPLY_KV.put(`order:${orderNumber}`, JSON.stringify(order)); // direct put — no need to touch the index, key unchanged
}

export async function markPaymentConfirmed(env, orderNumber) {
  const order = await getOrder(env, orderNumber);
  if (!order) return;
  order.status = 'payment-confirmed';
  await env.REPLY_KV.put(`order:${orderNumber}`, JSON.stringify(order));
}

export async function deleteOrder(env, orderNumber) {
  const kv = env.REPLY_KV;
  if (!kv) return;
  await kv.delete(`order:${orderNumber}`);
  const index = JSON.parse((await kv.get(INDEX_KEY)) || '[]');
  await kv.put(INDEX_KEY, JSON.stringify(index.filter((e) => e.orderNumber !== orderNumber)));
}
```

`enquiryStore.js` mirrors this exactly against `enquiry:{id}` / `enquiry:index`, with `markEnquiryReplied` in place of the two order-status functions — same shape as its Vercel counterpart (§2 above).

**Binding is dashboard-only, like every other Cloudflare setting in this workflow** — this is a direct/drag-and-drop ZIP upload, not a Git-connected or `wrangler`-CLI-deployed project, so a `wrangler.toml` in the ZIP is never read. Bind it at Project → Settings → Bindings → Add → KV namespace → variable name **`REPLY_KV`** (matches `env.REPLY_KV` in code exactly — fixed name, every site) → select the namespace created at Storage & Databases → KV → Create. Same mandatory-redeploy rule as every other binding: it only applies to deployments made *after* it's added.

**Why a 500-entry cap here and not on Vercel:** KV's index is one JSON blob rewritten on every write — unbounded growth means an ever-larger read+parse+write on every order. Redis's hash has no such cost, so it isn't capped. Both are one dashboard's worth of history in practice; raise the cap if a site genuinely needs more.

---

## 3. Payment details — the flexible, country/method-agnostic parser

This is the mechanism that makes the portal fit any country, currency, or payment rail without code changes. The admin pastes **one blob** — real bank details, a wallet address, a PayPal.me link, anything — and it's split into individually-copyable fields.

```js
// lib/order.js (Vercel) / functions/_shared/order.js (Cloudflare) — identical logic, same file both targets
const KNOWN_LABELS = [
  'account name', 'account number', 'sort code', 'bank name', 'branch code',
  'routing number', 'beneficiary name', 'beneficiary', 'swift code', 'swift',
  'bic code', 'bic', 'iban', 'wallet address', 'wallet', 'network', 'memo',
  'destination tag', 'tag', 'paypal email', 'paypal.me', 'paypal',
  'payment link', 'reference',
].sort((a, b) => b.length - a.length); // longest phrase wins, e.g. "account number" over "account"

const LABEL_PATTERN = new RegExp(
  `^(${KNOWN_LABELS.map((l) => l.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).join('|')})\\s*[:\\-]?\\s+(.+)$`,
  'i'
);

/**
 * Three tiers, in order:
 *  1. "Label: value" on a line — the general case. ANY label works here,
 *     including ones not in KNOWN_LABELS, so a brand-new payment method
 *     needs zero code changes as long as the admin uses a colon.
 *  2. No colon, but the line starts with a recognised label word
 *     ("Sort code 00-00-00") — split there so only the value gets copied.
 *  3. No colon, no recognised label — the whole line becomes one field
 *     labelled "Detail" (numbered if there's more than one), so a bare
 *     wallet address on its own still gets a working copy button.
 */
export function parsePaymentDetail(text) {
  const lines = String(text || '').split('\n').map((l) => l.trim()).filter(Boolean);
  let unlabeled = 0;
  return lines.map((line) => {
    const colonIdx = line.indexOf(':');
    if (colonIdx > 0 && colonIdx < line.length - 1) {
      return { label: line.slice(0, colonIdx).trim(), value: line.slice(colonIdx + 1).trim() };
    }
    const match = line.match(LABEL_PATTERN);
    if (match) return { label: titleCase(match[1]), value: match[2].trim() };
    unlabeled += 1;
    return { label: unlabeled > 1 ? `Detail ${unlabeled}` : 'Detail', value: line };
  });
}

function titleCase(s) { return s.replace(/\S+/g, (w) => w[0].toUpperCase() + w.slice(1).toLowerCase()); }
```

**How the admin uses it:** paste, e.g.
```
Account name: Peak Pedal Ltd
Sort code: 12-34-56
Account number: 12345678
Reference: PP-K3P9QX
```
→ four fields, each with its own copy button on the customer's payment-details page and in the WhatsApp message. For a crypto wallet it's just as valid to paste `Wallet address: 0xABC…` and `Network: Ethereum`. Nothing about the parser assumes a country, currency, or rail — that flexibility is the entire design.

`money()`, `findMethod()`, `paymentMethodParts()`, `paymentTermsLines()`, `paymentTermsHtml()` live alongside `parsePaymentDetail` in the same file — see `references/forms.md` for the payment-method registry and terms copy. **`money()` reads both `SITE.reply.currency.code/symbol` AND a new `SITE.reply.currency.locale` field** (e.g. `'en-GB'`, `'en-US'`, `'en-AU'` — default `'en-US'` if unset) so number grouping/decimal conventions are also config-driven, not hardcoded to one country.

---

## 4. Auth — passcode (server + client, both targets)

Deliberately simpler than the CMS backend's email/password+sessions — a single shared passcode is right for a one-operator reply desk.

- **Server check** — compares the `X-Admin-Passcode` header to `ADMIN_PASSCODE`. Returns 503 if unset, 401 if wrong, `null` if OK. Call it as the **first line of every admin route/Function**.
  ```js
  // Vercel: lib/adminAuth.js
  export function checkAdminPasscode(request) {
    const expected = process.env.ADMIN_PASSCODE;
    if (!expected) return Response.json({ ok: false, error: 'admin-not-configured' }, { status: 503 });
    const given = request.headers.get('x-admin-passcode') || '';
    if (given !== expected) return Response.json({ ok: false, error: 'unauthorized' }, { status: 401 });
    return null;
  }

  // Cloudflare: functions/_shared/adminAuth.js — same logic, env passed explicitly
  // (Functions don't have a global process.env; env is a per-request argument)
  export function checkAdminPasscode(request, env) {
    const expected = env.ADMIN_PASSCODE;
    if (!expected) return new Response(JSON.stringify({ ok: false, error: 'admin-not-configured' }), { status: 503 });
    const given = request.headers.get('x-admin-passcode') || '';
    if (given !== expected) return new Response(JSON.stringify({ ok: false, error: 'unauthorized' }), { status: 401 });
    return null;
  }
  ```
- **No dedicated `/verify` route.** The client tests a passcode by calling the orders-list endpoint with it and checking the response status — one fewer file, one fewer thing to keep in sync. (`PasscodeGate` below.)
- **Client `components/admin/PasscodeGate.jsx`** — render-prop wrapper: reads/writes the passcode to `localStorage` (namespaced key, e.g. `pp-admin-passcode`; wrapped in try/catch so private browsing still works), shows a lock screen, verifies by fetching `/api/admin/orders` (or the Cloudflare Function equivalent) with the header, then renders `children(passcode, signOut)`. Every admin page is wrapped by `app/admin/layout.jsx` using this gate — pages themselves never re-implement auth.

---

## 5. Admin UI (`app/admin/*`) — identical on both targets

Private operator tool — a **dark** shell is fine here (it is not an email, so the light-shell rule doesn't apply), but its **accent still reads `SITE.reply.brand.primary`** so the portal follows the site theme. `layout.jsx` sets `robots: noindex, nofollow`. **JavaScript/JSX, not TypeScript** — matches the rest of the site.

- **`layout.jsx`** — wraps everything in `PasscodeGate`; renders `AdminNav` (site-name brand link → `/admin/`, nav: Dashboard · Orders · Enquiries, sign-out) + the page content inside an `AdminPasscodeContext.Provider` so pages read the verified passcode via a hook instead of prop-drilling.
- **`page.jsx` (dashboard hub)** — two stat cards (Orders, Enquiries) fetched in parallel, each showing a total + a secondary metric (pending orders / new enquiries), plus a "Recent orders" and "Recent enquiries" list (5 rows, "View all" link if more). This is the easy entry point — `yourdomain.com/admin`.
- **`orders/page.jsx`** — list with status badge (Pending / Payment sent / Payment confirmed) + channel icon (WhatsApp / email); row click → order detail; "Delete all" + per-row delete with confirm.
- **`orders/[id]/page.jsx`** — customer card, payment card, itemised table, notes; "Send/Resend payment details" button → composer; shows a confirmation note once the customer has uploaded a payment screenshot.
- **`enquiries/page.jsx`** — list with filter pills (all / contact / wholesale / new / replied); row click → enquiry detail.
- **`enquiries/[id]/page.jsx`** — contact card, any extra `meta` fields, the message, "Reply" button → composer.
- **`send-payment-email/page.jsx`** — the payment-details composer: payment-method select, a free-text paste box for the real detail (placeholder shows the expected `Label: value` shape), a live preview built from `parsePaymentDetail` + `paymentMethodParts`, and a `WhatsAppSendPanel` (pre-filled `wa.me` link + copy-message fallback button). Sends via the admin send-payment-email endpoint, which also calls `markOrderSent`.
- **`reply-enquiry/page.jsx`** — plain-text reply composer with a live preview; sends via the admin reply-enquiry endpoint, which calls `markEnquiryReplied`.

Routes under `api/admin/*` (Vercel) or `functions/api/admin/*` (Cloudflare): `orders` (+`[id]`), `enquiries` (+`[id]`), `send-payment-email`, `reply-enquiry` — each begins with `checkAdminPasscode`.

---

## 6. Customer-facing payment flow (`app/order/*`) — public, no-auth, order-number-as-token

Two pages, reachable only from links in the payment-details email/WhatsApp message:

- **`order/payment-details/page.jsx`** — `?id={orderNumber}` in the query string. Fetches `GET /api/order/payment-details?id=…` (public route — the order number itself is the access token, same pattern banks and payment processors use), renders the amount due, the opening line, each parsed field as a tap-to-copy `CopyField`, and the closing line. Nothing here is exposed that wasn't already in the email the customer received.
- **`order/confirm-payment/page.jsx`** — `?id={orderNumber}`. A file upload (JPG/PNG/WebP/HEIC, capped at 4MB) + optional note. `POST /api/order/confirm-payment` emails the shop with the screenshot attached and calls `markPaymentConfirmed`. Public, no-auth, never returns order data back to the caller beyond `{ok}`.

`components/CopyField.jsx` — one payment-detail row, `navigator.clipboard.writeText(value)` on click, a 2-second "copied" checkmark state. This is necessary because **email clients strip all JavaScript** — no email provider supports an in-email copy button, on any platform, ever. The email links out to this real page instead; that's not a workaround, it's the only mechanism that exists.

---

## 7. WhatsApp system (`lib/whatsapp.js`) — optional, config-gated, wired end-to-end

Every WhatsApp feature is gated on `SITE.reply.channels.whatsapp` being set; sites without WhatsApp render cleanly with no dead links.

**⭐ Greeting rule — every message a customer originates must open with `Hi {SITE.name},`.** Not a bold label, a literal greeting sentence, as the first line of the pre-filled text. A business running several WebForge sites (or just juggling one WhatsApp number across order requests, payment confirmations, and general chat) needs to know which site an inbound message is about before opening anything else — the greeting is that signal, sitting right in the message the customer is about to send, so it survives even if they edit the rest of the text before pressing send.

```js
export const WA_HEADER = `*${SITE.name}*`; // bold — prepended to ADMIN → customer messages only (brand identity in the reply)

function buildAdminText(body) { // admin -> customer
  const lines = Array.isArray(body) ? body : [body];
  return [WA_HEADER, '', ...lines].join('\n');
}

function waGreeting() { return `Hi ${SITE.name},`; }
function buildCustomerText(body) { // customer -> business — every entry point that lets a
  // customer message the business (order, payment confirmation, the general chat-hub link)
  // routes through this, never buildAdminText.
  const lines = Array.isArray(body) ? body : [body];
  return [waGreeting(), '', ...lines].join('\n');
}

export function waMessageText(body) { return buildAdminText(body); } // admin-side "Copy message" fallback
export function waLinkTo(phone, body) { // admin -> a specific customer
  return `https://wa.me/${toWhatsAppNumber(phone)}?text=${encodeURIComponent(buildAdminText(body))}`;
}

export function waLink(body) { // customer -> the business's own WhatsApp number — always the greeting form
  return `https://wa.me/${toWhatsAppNumber(REPLY.channels.whatsapp)}?text=${encodeURIComponent(buildCustomerText(body))}`;
}
```

- `toWhatsAppNumber(phone)` — normalises to bare digits.
- `waOrderLink(order, customer)` — customer → business, itemised new-order notification, opens `Hi {SITE.name}, New order request {ref} …` via `waLink`. **Must be wired into the checkout's "Order via WhatsApp" button** — see §7a below. A WhatsApp checkout button that's just a bare `wa.me` chat link with no order data and no dashboard record is a broken implementation of this feature, not a valid simplification of it.
- `waPaymentConfirmationLink(ref)` — customer → business, `Hi {SITE.name}, I've completed payment for order {ref}.` via `waLink`.
- `waPaymentDetailsMessage/Link` — admin → customer, via `waLinkTo`/`buildAdminText` (`WA_HEADER`, not the greeting — this direction is the business identifying itself to the customer, not the other way round). Mirrors the payment-details email using the **same** `parsePaymentDetail` fields and `paymentTermsLines()`. **Email/WhatsApp content parity is a rule.**
- **The general chat-hub WhatsApp link channel** (see "Live chat — the chat hub" in `SKILL.md` — not Reply-Portal-specific) gets the same treatment: `https://wa.me/{number}?text=` + URL-encoded `Hi {SITE.name}, ` — a pre-filled, cursor-ready greeting, not a blank chat window. This applies whether or not Section P is enabled; it's a general WhatsApp-UX rule, not a reply-portal feature.

---

## 7a. Checkout — WhatsApp is still a Yes/No intake question; when Yes, it's built properly

**Intake Section E asks "how do customers order: chat / order form / both" — that choice is unchanged, still a plain Yes/No per channel.** What changes is what "Yes" for WhatsApp is allowed to mean: never a bare `wa.me` chat link bolted on as an afterthought. Whichever combination is chosen, the checkout is **one real form** — name, email (**always required, in all three configurations**), phone, delivery address, payment method — never a form-less "just message us" shortcut. The form's *submit button(s)* are what change per configuration, not its fields:

- **Order form only** — one button, "Place order." Saves (`channel:'email'`), sends the shop notification + `orderConfirmationEmail()` to the customer.
- **WhatsApp only** — one button, "Order via WhatsApp." **Still the same full form, still a required email field.** Saves (`channel:'whatsapp'`) through the identical save-and-email call the email path would use, *and* opens `waOrderLink(order, customer)` — greeting-prefixed, itemised, to the business's WhatsApp number. A site with WhatsApp as its *only* checkout channel gets a real order record and a real confirmation email exactly like one with an email-only checkout — "WhatsApp only" describes which button is shown, never a reason to collect less data or skip the email.
- **Both** — two buttons on the same form, "Place order" and "Order via WhatsApp," each doing the above for its own `channel`.

**⭐ The customer confirmation email fires unconditionally, on every configuration that includes WhatsApp — never gated on which button exists, let alone which one was pressed.** A customer ordering via WhatsApp — whether or not an email button even exists on that site — still gets a written email record of their order. Email is the paper trail, WhatsApp is the fast lane; a site offering only the fast lane doesn't get to drop the paper trail too. If `channel==='whatsapp'` ever reaches checkout without a required email field, or short-circuits before `orderConfirmationEmail()` is sent, that is a bug — on any of the three configurations above.

**Ordering, to avoid pop-up blocking:** `window.open(waOrderLink(...))` must fire **synchronously** inside the click handler, before any `await` — pop-up blockers trigger the instant a click handler goes async before calling `window.open`. Immediately after, fire-and-forget the same save-order request the email path uses (`POST /api/contact` with `formName:'order'`, `channel:'whatsapp'`) — that request is what triggers both the shop notification and the customer confirmation email, exactly as it does for the email channel. Clear the cart once the request resolves (or optimistically, matching the email channel's behaviour).

---

## 8. Env vars (canonical — same names on both targets)

- Email: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` (optional — falls back to `SMTP_USER`). See `references/forms.md`.
- Destinations: `CONTACT_EMAIL`, `ORDER_EMAIL`, `WHOLESALE_EMAIL` — each optional, falling back to the matching `FORMS.destinations.{contact|order|wholesale}` config value if unset. Lets an operator redirect where enquiries land without a redeploy.
- Admin: `ADMIN_PASSCODE` (server-only; gates every admin route/Function).
- Storage — **Vercel:** `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN` (or any of the other 4 accepted name pairs in §2 — 5 total). **Cloudflare:** no env var — storage is a KV namespace **binding**, set in Project → Settings → Bindings (dashboard-only — see §2).

**Access differs by runtime, names don't:** Vercel reads `process.env.SMTP_HOST`; Cloudflare Functions read `env.SMTP_HOST` (env is a per-request argument, not a global) — set as encrypted secrets in Project → Settings → Variables and Secrets, same place the D1/R2 secrets go for backend-enabled sites (`references/backend.md`).

All secrets live in the platform dashboard only — never in the repo (crosscheck B5).

---

## 9. Files

### Vercel (Section P = Yes)
```
src/lib/
  redis.js            §2 REST client
  orderStore.js        StoredOrder + save/list/get/markOrderSent/markPaymentConfirmed/delete
  enquiryStore.js       StoredEnquiry + save/list/get/markEnquiryReplied/delete
  adminAuth.js          checkAdminPasscode()
  mailer.js             nodemailer singleton (forms.md)
  order.js              parsePaymentDetail, money, paymentMethodParts, paymentTermsLines/Html (forms.md, §3)
  whatsapp.js            §7
src/utils/emailTemplates.js   per-email builder functions (forms.md)
src/components/CopyField.jsx
src/components/admin/PasscodeGate.jsx, AdminNav.jsx, AdminPasscodeContext.jsx, StatusBadge.jsx, WhatsAppSendPanel.jsx
src/app/admin/            layout, page (hub), orders(+[id]), enquiries(+[id]), send-payment-email, reply-enquiry
src/app/order/            payment-details/page.jsx, confirm-payment/page.jsx
src/app/api/admin/        orders(+[id]), enquiries(+[id]), send-payment-email, reply-enquiry
src/app/api/order/        payment-details/route.js, confirm-payment/route.js
src/app/api/contact/      route.js — ONE handler for order + contact + wholesale (by formName); order
                             creation saves + emails the shop AND the customer, either checkout channel
```

### Cloudflare (Section P = Yes)
```
functions/_shared/
  redis.js → (none — use KV directly, see §2)
  orderStore.js, enquiryStore.js    KV-backed, §2
  adminAuth.js                       §4
  mailer.js                          raw-socket SMTP, forms.md
  order.js                           §3, shared logic with Vercel
  emailTemplate.js                   forms.md
  config.js                          SITE/CONTACT/REPLY — mirrors src/config/site.js's reply block
functions/api/
  contact.js                         ← ONE handler for order + contact + wholesale (by formName), mirrors
                                         src/app/api/contact/route.js exactly — same unification, same rule
                                         (order confirmation email fires on either checkout channel)
  order/payment-details.js, order/confirm-payment.js
  admin/orders.js, admin/orders/[id].js, admin/enquiries.js, admin/enquiries/[id].js
  admin/send-payment-email.js, admin/reply-enquiry.js
functions/_middleware.js             CORS headers for the admin API surface
```
No `wrangler.toml` — this is a direct/drag-and-drop ZIP upload, not a `wrangler`-CLI or Git-connected deploy, so that file is never read. The KV binding and the `nodejs_compat` compatibility flag are both set in the Cloudflare dashboard instead (§2, `references/forms.md`).
The **front-end admin pages are the same Next.js static-exported components as Vercel** (§5) — only the API layer they call differs. Duplicate `app/admin/*` and `app/order/*` unchanged; point their `fetch()` calls at `/api/admin/*` / `/api/order/*`, which Cloudflare Pages routes to the Functions above automatically (same path, no config needed).

---

## 10. Build vs. edit an existing site

- **New build:** scaffold the files above for the chosen target (both, if the site might migrate later — the front-end and `parsePaymentDetail`/`order.js` logic are identical either way) + the `SITE.reply` block; wire the checkout and contact/wholesale forms to save (best-effort) then email.
- **Retrofit existing:** add `SITE.reply`; add the storage layer + `adminAuth` + `PasscodeGate` + `app/admin/*` + the API/Function layer; point existing checkout/contact handlers at the stores (guarded by `isStoreConfigured()` so nothing breaks pre-provisioning); set the admin accent to `reply.brand.primary`. Because save is best-effort and every layer degrades, you can ship the portal before the operator has created the database or set the passcode.

---

## 11. Audit checklist (Section P)

- Every admin route/Function calls `checkAdminPasscode` first; `ADMIN_PASSCODE` never reaches the client bundle.
- Storage layer never crashes when unconfigured; `isStoreConfigured()` gates the UI notice.
- Order ref generated client-side with the unambiguous alphabet; validated server-side; same ref in WhatsApp + dashboard + email.
- `parsePaymentDetail` present and used by both the email composer preview and the customer-facing copy page — never a single unparsed blob.
- Status lifecycles fire (`markOrderSent` on payment email, `markPaymentConfirmed` on screenshot upload, `markEnquiryReplied` on reply).
- WhatsApp checkout actually saves an order and sends the structured `waOrderLink` message — not a bare chat link. WhatsApp features gated on a configured number otherwise.
- **Whatever Section E's checkout config is (order form only / WhatsApp only / both), the checkout form always has all its fields, including a required email** — a "WhatsApp only" site is not an excuse for a shorter form or no email field.
- Every customer-originated WhatsApp message (order, payment confirmation, general chat-hub link) opens with `Hi {SITE.name},` — never the bare `WA_HEADER` bold label, which is admin→customer only.
- **The customer confirmation email sends on every configuration that includes WhatsApp — including WhatsApp-only sites — exactly as it would on an email checkout.** Trace `channel==='whatsapp'` through to `orderConfirmationEmail()` and confirm it isn't short-circuited on any of the three Section E configurations.
- Admin pages `robots:noindex`; accent = `reply.brand.primary`; `/admin` hub reachable and passcode-gated.
- No hardcoded brand/location/currency/prefix/method/locale — all from `SITE.reply`.
- Emails are the **light** shell from `forms.md` (never a dark body — this applies on Cloudflare too).
- No `web3forms` reference anywhere in the build.
