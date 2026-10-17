# WebForge — Forms & Email reference (v11.0)

Read whenever building or auditing any form, email send, or intake Section L/P.

**v11 delta — Web3Forms is deleted entirely.** SMTP is now the **only** form/email transport, on **both** deploy targets. `resend` remains available as an opt-in, verified-domain option on Vercel. Cloudflare gets real SMTP via Pages Functions + a raw-socket client (Cloudflare's Workers runtime has no `net`/`tls`, so nodemailer can't run there — see "Provider: `smtp` (Cloudflare)" below). The email-builder API section was rewritten to match what's actually live in production: **per-email builder functions sharing HTML primitives**, not a generic `buildEmailHtml(rows[])` call — the "invariant row API" this file previously described was never how the real sites work.

---

## Provider selection

One constant in `src/config/site.js` selects the provider:

```js
export const FORMS = {
  provider: 'smtp',                   // 'smtp' | 'resend'
  smtpFrom: 'noreply@yourdomain.com', // fallback; SMTP_FROM env var overrides
  resendFrom: '',                     // only for provider: 'resend'; REQUIRES verified domain
  turnstileSiteKey: '',               // optional Cloudflare Turnstile
  destinations: { contact: '', order: '', wholesale: '' }, // fallback when the matching env var is unset
}
```

| | `smtp` | `resend` |
|---|---|---|
| **Default?** | **Yes — both targets** | No |
| Targets | Vercel + Cloudflare (Pages Functions) | Vercel only |
| Needs a domain? | No (any SMTP server) | **Yes** (verified DNS) |
| Sends from | Your SMTP account | Your domain |
| Env vars | `SMTP_HOST/PORT/USER/PASS` + `SMTP_FROM` (§ below) | `RESEND_API_KEY` |
| When unconfigured | `{sent:false}` — never crashes | server error |
| HTML emails | ✅ Full branded HTML | ✅ Full HTML |
| Reply Portal | ✅ Required | ✅ |

**Decision guide:**
- New site, either target → **`smtp`** (set SMTP creds in the platform dashboard; site still works via WhatsApp/chat fallback if not set)
- First-party domain email, Vercel, domain live → `resend` (opt-in only, never the default)

There is no longer a "no env var" fallback provider. If SMTP credentials aren't available yet at build time, ship with `smtp` selected and state at handover that submissions won't be emailed until the credentials are set — the site degrades to WhatsApp/chat, it doesn't fail.

---

## Provider: `smtp` (default, Vercel) — nodemailer

### `lib/mailer.js` — nodemailer lazy singleton

Never throws. Returns `{sent:false, reason:'not-configured'}` gracefully when env vars are absent — the API route responds 503, and the UI falls back to WhatsApp/phone.

```js
import nodemailer from 'nodemailer';

export function isMailerConfigured() {
  return Boolean(process.env.SMTP_HOST && process.env.SMTP_USER && process.env.SMTP_PASS);
}

function transporter() {
  const port = Number(process.env.SMTP_PORT || 587);
  return nodemailer.createTransport({
    host: process.env.SMTP_HOST,
    port,
    secure: port === 465,
    auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS },
  });
}

export async function sendMail({ to, subject, html, text, replyTo, from, attachments }) {
  if (!isMailerConfigured()) return { sent: false, reason: 'not-configured' };
  await transporter().sendMail({ from, to, replyTo, subject, text, html, ...(attachments ? { attachments } : {}) });
  return { sent: true };
}
```

### Required env vars (Vercel dashboard only — never in the repo)

```
SMTP_HOST=smtp.zoho.com          # or smtp.gmail.com, mail.privateemail.com, etc.
SMTP_PORT=465                    # 465 = SSL (recommended), 587 = TLS/STARTTLS
SMTP_USER=you@yourdomain.com     # sending address (login credential)
SMTP_PASS=app-specific-password  # use app passwords / DKIM passwords, not the main account password
SMTP_FROM=noreply@yourdomain.com # optional — "From" address shown to recipients; falls back to SMTP_USER
```

**Zoho SMTP note:** use port 465 (SSL), not 587. Generate an app-specific password in Zoho → Security → App Passwords.

### Destination-email env vars (canonical — both targets)

```
CONTACT_EMAIL=hello@yourdomain.com      # optional — falls back to FORMS.destinations.contact
ORDER_EMAIL=orders@yourdomain.com       # optional — falls back to FORMS.destinations.order
WHOLESALE_EMAIL=trade@yourdomain.com    # optional — falls back to FORMS.destinations.wholesale
```

Reading destination addresses from env vars (with a config fallback so nothing breaks if unset) lets an operator redirect where enquiries land — split a busy inbox, hand wholesale to a different person — without a redeploy or a code change.

### Email deliverability — DNS checklist (document in `.env.example`)

These are DNS records, not code. Without them, emails land in spam:

```
SPF:   TXT @          "v=spf1 include:zoho.com ~all"
                       (replace with your provider's SPF include value)
DKIM:  CNAME/TXT records — copy from your SMTP provider's dashboard
DMARC: TXT _dmarc     "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com"
```

**This is the real spam fix.** Headers, from names, and subject lines help but are secondary. Warn the client at handover.

---

## Provider: `smtp` (Cloudflare) — raw-socket client, same env vars

Cloudflare's Workers/Pages runtime has no Node `net`/`tls` module, so nodemailer cannot run in a Pages Function. `functions/_shared/mailer.js` speaks SMTP directly over a raw TCP socket via the `cloudflare:sockets` API — same protocol, same env var names, same `sendMail()` shape, different transport:

```js
import { connect } from 'cloudflare:sockets';

async function readResponse(reader) {
  let result = '';
  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    result += new TextDecoder().decode(value);
    const lines = result.split('\r\n').filter(Boolean);
    const last = lines[lines.length - 1];
    // A final SMTP response line is "NNN " (digit-digit-digit-space) — a
    // multi-line response uses "NNN-" for every line but the last. Waiting
    // for the space form is what lets this loop terminate correctly on
    // multi-line EHLO replies instead of returning after the first line.
    if (last && /^\d{3} /.test(last)) break;
  }
  return result.trim();
}

async function smtpCommand(writer, reader, cmd) {
  await writer.write(new TextEncoder().encode(cmd + '\r\n'));
  return readResponse(reader);
}

function buildMimeMessage({ from, to, subject, text, html, replyTo }) {
  const boundary = 'b' + Date.now().toString(36) + Math.random().toString(36).slice(2);
  const msgId = `<${Date.now()}.${Math.random().toString(36).slice(2)}@${from.split('@')[1] || 'localhost'}>`;
  let headers = `From: ${from}\r\nTo: ${to}\r\n`;
  if (replyTo) headers += `Reply-To: ${replyTo}\r\n`;
  headers += `Subject: ${subject}\r\nMessage-ID: ${msgId}\r\nMIME-Version: 1.0\r\n`;
  headers += `Content-Type: multipart/alternative; boundary="${boundary}"\r\n`;
  headers += `Date: ${new Date().toUTCString().replace('GMT', '+0000')}\r\n`;
  return (
    headers +
    `\r\n--${boundary}\r\nContent-Type: text/plain; charset=utf-8\r\n\r\n${text || ''}\r\n` +
    `--${boundary}\r\nContent-Type: text/html; charset=utf-8\r\n\r\n${html || ''}\r\n` +
    `--${boundary}--\r\n`
  );
}

// One SMTP session, N messages — cheaper than reconnecting per email. Used by
// checkout, which always sends a shop notification AND a customer receipt
// (see "WhatsApp checkout — one form, two channels, one guaranteed email"
// in reply-portal.md — both are sent in this single batch call).
export async function sendMailBatch(env, messages) {
  const user = env.SMTP_USER, pass = env.SMTP_PASS, host = env.SMTP_HOST || 'smtp.zoho.com';
  const from = env.SMTP_FROM || user;
  if (!user || !pass) return messages.map(() => ({ sent: false, reason: 'not-configured' }));

  const socket = connect({ hostname: host, port: Number(env.SMTP_PORT || 465) }, { secureTransport: 'on' });
  const writer = socket.writable.getWriter();
  const reader = socket.readable.getReader();
  const results = [];
  try {
    const greeting = await readResponse(reader);
    if (!greeting.startsWith('220')) throw new Error('Bad greeting: ' + greeting);

    let r = await smtpCommand(writer, reader, `EHLO ${host}`);
    if (!r.startsWith('250')) throw new Error('EHLO failed: ' + r);

    r = await smtpCommand(writer, reader, `AUTH PLAIN ${btoa('\0' + user + '\0' + pass)}`);
    if (!r.startsWith('235')) throw new Error('AUTH failed: ' + r);

    for (const msg of messages) {
      try {
        r = await smtpCommand(writer, reader, `MAIL FROM:<${user}>`);
        if (!r.startsWith('250')) throw new Error('MAIL FROM failed: ' + r);
        r = await smtpCommand(writer, reader, `RCPT TO:<${msg.to}>`);
        if (!r.startsWith('250')) throw new Error('RCPT TO failed: ' + r);
        r = await smtpCommand(writer, reader, 'DATA');
        if (!r.startsWith('354')) throw new Error('DATA failed: ' + r);
        const mime = buildMimeMessage({ from, ...msg });
        await writer.write(new TextEncoder().encode(mime + '\r\n.\r\n'));
        r = await readResponse(reader);
        if (!r.startsWith('250')) throw new Error('Message rejected: ' + r);
        results.push({ sent: true });
      } catch (err) {
        results.push({ sent: false, reason: err.message });
      }
    }
    await smtpCommand(writer, reader, 'QUIT');
  } catch (err) {
    while (results.length < messages.length) results.push({ sent: false, reason: err.message });
  } finally {
    try { writer.releaseLock(); } catch {}
    try { reader.releaseLock(); } catch {}
    socket.close();
  }
  return results;
}

export async function sendMail(env, opts) {
  const [result] = await sendMailBatch(env, [opts]);
  return result;
}

export function isMailerConfigured(env) {
  return Boolean(env.SMTP_HOST && env.SMTP_USER && env.SMTP_PASS);
}
```

Every call site takes `env` as an explicit first argument (Functions have no global `process.env` — `env` is per-request). This is at full parity with `lib/mailer.js` — same graceful-degradation contract (`{sent:false, reason:'not-configured'}`, never throws), same env var names, same batch-send-for-checkout usage.

**Env vars — set in Cloudflare Pages → Project → Settings → Variables and Secrets (tick Encrypt on each), same place the D1/R2 secrets go for backend-enabled sites:** `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` — **identical names to Vercel.** The socket API additionally needs the `nodejs_compat` compatibility flag, set in Settings → Functions → Compatibility Flags (both Production and Preview) — **not** a `wrangler.toml` file, which a drag-and-drop ZIP upload never reads. See `references/cloudflare.md` → "Forms & Reply Portal."

---

## `utils/emailTemplates.js` — the branded HTML email builder (LIGHT shell, theme-driven)

**One function per email type, sharing a small set of HTML primitives** — `shell()`, `field()`, `divider`, `callout()`, `button()`, `label()`. This is what's actually live on both reference sites; there is no generic `buildEmailHtml(rows[])` call to reach for — building a new email means writing a new function that composes the primitives, the way the five documented below do.

Table-based inline CSS throughout — required for Outlook/Gmail/Zoho compatibility. No external CSS, no `<style>` block, no web fonts.

**⭐ Light-theme mandate (the #1 email rule — non-negotiable).** Build every transactional email as a **light** email: white card body, a **dark brand header band**, near-black text, the brand colour used only as an **accent**. Never a dark-background body.

Why: dark-background emails are **force-inverted** by Zoho Mail and Gmail dark mode — the white header flips dark, the dark body flips cream — and **no** HTML opt-out prevents it (`color-scheme: light only`, `supported-color-schemes`, Gmail `u + .body` overrides, `@media (prefers-color-scheme: dark)`, `!important`, and MSO/VML conditionals were all tried and all failed in Zoho). Dark-mode readers leave **light** emails alone.

**Two invariants + one config source.** The primitives (structure, spacing, type) never change per site. Everything brand/location/payment-specific comes from one block, `SITE.reply`, so the same code **builds or retrofits any site**:

```js
// src/config/site.js — single source of truth for the reply system
export const REPLY = {
  brand: { primary: '#B91C1C', headerDark: '#141010' }, // primary = accent; headerDark = header band
  currency: { code: 'AUD', symbol: '$', locale: 'en-AU' }, // locale drives number grouping/decimals — default 'en-US' if unset
  channels: { email: 'orders@site.com', whatsapp: '+61480811308' }, // whatsapp optional
  headerTagline: 'Craft Butcher · Alexandria, Sydney',
  dispatchLine: 'Cold-chain refrigerated delivery across NSW, once payment clears.',
  paymentMethods: [ /* registry — see "Payment-method registry" below */ ],
}
```
`orderPrefix` lives alongside `REPLY` (or as its own top-level export) and also namespaces the Redis/KV key (`references/reply-portal.md` §2).

**Primitives (`shell`, `label`, `field`, `divider`, `callout`, `button`) — keep identical across every site:**
- `shell({eyebrow, title, meta, body})` — page `#F2F4F0` → centred white card `max-width:600` `border-radius:12` → dark header band (`eyebrow` 11px uppercase accent, `title` 22px white bold, `meta` 12px muted) → `3px` accent rule → body (the email-specific content) → light footer with `SITE.name · SITE.domain`.
- `label(text)` — 10px uppercase bold muted, used above every field value.
- `field(label, valueHtml, marginBottom)` — a labelled row: `label()` + a 14px value line.
- `divider` — one hairline `<tr>`.
- `callout(innerHtml)` — a tinted panel with a left accent border, used for the payment-terms bullet list.
- `button(href, text)` — solid dark-header-coloured pill button with an arrow.
- `escapeHtml(s)` — HTML-entity-escape; every dynamic string goes through this before interpolation.

Fixed light neutrals (never brand-driven): page `#F2F4F0`, card `#FFFFFF`, ink `#1B2320`, soft `#6A746E`, faint `#9AA39D`, rule `#E2E5DF`. Only `reply.brand.primary` (accent) and `reply.brand.headerDark` (header band + button colour) vary per site.

### Outbound email types (Section P = Yes)

Five email types, each its own exported function, all built from the primitives above, all light, all accent-from-config:

| Email | Function | Trigger | To |
|---|---|---|---|
| Order notification | (part of the checkout handler's own HTML, not `emailTemplates.js` — see below) | Customer checks out — **either channel** | Admin — "Reply in Dashboard →" button |
| Order confirmation | `orderConfirmationEmail()` | Customer checks out — **either channel** | Customer — "watch for payment-details email" only, **no bank details** |
| Contact/enquiry notification | (checkout handler's own HTML) | Contact/wholesale form submit | Admin — "Reply in Dashboard →" button |
| Payment details | `paymentDetailsEmail()` | Admin sends | Customer — amount, opening line, each parsed field, closing line, payment-terms callout, three CTAs |
| Enquiry reply | `enquiryReplyEmail()` | Admin replies | Customer — the admin's typed reply |

The order/contact **notification-to-admin** emails are commonly built inline in the API route/Function that handles the submission (they need the raw submitted fields, which vary per form), rather than as a fifth exported function — both are fine; keep them light-shell and accent-consistent either way.

**⭐ "Either channel" is load-bearing, not decorative.** `orderConfirmationEmail()` sends on a WhatsApp checkout exactly as it does on an email checkout — the checkout form collects the customer's email regardless of which submit button they press, and both buttons route through the same save-and-notify call (`channel` differs, everything else doesn't). Never gate this email on `channel === 'email'`. See `references/reply-portal.md` §7a for the full checkout flow.

### The payment-details email (`paymentDetailsEmail()`)

```js
export function paymentDetailsEmail({ orderNumber, amountDue, customerName, opening, closing, fields = [] }) {
  const detailsUrl = `https://${SITE.domain}/order/payment-details/?id=${encodeURIComponent(orderNumber)}`;
  // body: Order + Amount due (highlighted) + divider + opening line + one field()
  // row per parsed {label,value} (monospace) + closing line + a "View & Copy
  // Payment Details" button (→ detailsUrl) + a callout() with paymentTermsHtml()
  // + three buttons: "I've Paid — Upload Confirmation", "Confirm via WhatsApp",
  // "Reply to us" (mailto).
  return { subject: `Payment details for order ${orderNumber} — ${money(amountDue)} due`, text, html: shell({...}) };
}
```

Each field renders on its own monospace line **in the email** for manual copying — email clients strip all JavaScript, so a real click-to-copy button cannot exist inside an email on any platform. The email links through to `/order/payment-details/` (`references/reply-portal.md` §6), a live page with real copy buttons — the same pattern banks and payment processors use for exactly this reason.

### Payment-method registry (`SITE.reply.paymentMethods`)

Payment methods are **data, not code** — so any site, in any country, on any rail, defines its own. Each entry:

```js
{
  id, label,
  opening,  // template string, tokens {amount} {ref} — the sentence before the pasted details
  closing,  // template string — the sentence after the pasted details
}
```

The **variable detail itself — account numbers, a wallet address, a PayPal.me link — is never in this config.** It's real per-order/per-site secret information, entered by the admin at send time via `parsePaymentDetail` (`references/reply-portal.md` §3), never fabricated or hardcoded. `paymentMethodParts(methodId, amount, ref)` resolves `{opening, closing}` with the tokens filled in; `findMethod(methodId)` looks up the registry entry.

### The payment-terms message (`lib/order.js` — one source, every renderer)

Standing terms appended to every payment-details email **and** WhatsApp message — `paymentTermsLines()` (plain text) and `paymentTermsHtml()` (HTML `<li>`s) render the **same** content, so the two can never drift apart:

1. `This order is confirmed once payment is received — it is not yet final.`
2. `Use your order number — {ref} — as the payment reference.`
3. `{reply.dispatchLine}` — the logistics line (config — e.g. dispatch timing, delivery method).

Add a compliance/legal line here per-vertical when Intake Section H requires one (e.g. "Adult signature (21+) is required at delivery for every alcohol order.") — appended as a fourth line, still config-driven, never hardcoded into the function for one site and left out for others.

### Email env vars (canonical set — same names both targets)

`SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` (optional) — platform dashboard only, never in the repo. `sendMail()`/`sendMailBatch()` returns `{sent:false}` when any required var is missing so forms still redirect to the thank-you page. Deliverability still needs SPF/DKIM/DMARC DNS (above).

### Build vs. edit an existing site

- **New build:** scaffold `utils/emailTemplates.js` (primitives + the per-email functions), `lib/order.js`, `lib/whatsapp.js`, the API routes/Functions, and the `SITE.reply` block from intake.
- **Retrofit existing:** (a) add the `SITE.reply` block; (b) map the site's existing brand colour → `reply.brand.primary`, pick a `headerDark`; (c) replace any existing email template with the invariant light-shell primitives, keeping the same exported function names so call sites don't change; (d) move all inline copy (prefix, currency, locale, tagline, dispatch line, methods) into `SITE.reply`; (e) add storage/passcode if absent but keep graceful degradation.

**Thank-you page rule:** `/thank-you-order/` MUST only say "watch for the follow-up email with payment details." It must NOT show bank details, deadlines, or WhatsApp screenshot requests. Those come only after the admin sends payment details via the portal. Showing them prematurely creates confusion.

---

## Provider: `resend` (Vercel only, verified domain)

```js
// src/app/api/contact/route.js
import { Resend } from 'resend'
export async function POST(req) {
  const body = await req.json()
  if (body.botcheck) return Response.json({ success: true }) // honeypot: pretend success
  if (!body.email || !body.message) return Response.json({ success: false }, { status: 400 })
  const resend = new Resend(process.env.RESEND_API_KEY)
  await resend.emails.send({
    from: FORMS.resendFrom,    // must be on a VERIFIED domain
    to: process.env.ORDER_EMAIL || FORMS.destinations.order,
    replyTo: body.email,
    subject: body.subject,
    text: formatBody(body),
  })
  return Response.json({ success: true })
}
```

Client posts JSON to `/api/contact` — same-origin, no CORS constraints. `RESEND_API_KEY` in Vercel env vars only. **Never in the repo** (crosscheck B5).

**Crosscheck B3:** `provider: 'resend'` + static target = FAIL. `output: 'export'` drops the API route on a Next-export-only Cloudflare build — this doesn't apply once Pages Functions are in use, but `resend` still isn't offered on Cloudflare (Functions use the raw-socket `smtp` mailer instead).

---

## Turnstile (optional, all providers)

Free, privacy-friendly, no puzzles. Honeypot alone stops only naive bots.
- Widget in the form; CSP `script-src` + `frame-src` → `https://challenges.cloudflare.com`
- Verify the token server-side (route or Function) against Cloudflare's `siteverify` endpoint **before** sending
- Never render a visible "I'm not a robot" checkbox — use the invisible or managed mode

---

## All providers — shared rules

- Three forms → three thank-you pages: `/thank-you-contact/`, `/thank-you-order/`, `/thank-you-wholesale/`
- All thank-you pages: `noindex, follow`, excluded from sitemap
- Every form: `e.preventDefault()`, disabled/"Sending…" state while submitting, error path with entity-encoded email + chat link
- Labels tied to inputs, `autoComplete` set, 44px minimum tap targets, correct `type` attributes
- Order form: prefill from cart, offer chat deep-link with order pre-composed
- **Entity-encode emails everywhere** — `&#64;` not `@` — in HTML, JSON-LD, and plain text
- **Never log SMTP credentials** in any API route or Function

## Crosscheck (items 1, B3, B5)

- **Item 1:** API route/Function exists, returns 503 (not crash) when unconfigured, sends correctly when configured. Thank-you pages exist, correct redirects, for all three forms.
- **B3:** `resend` provider on a static (non-Functions) target → FAIL.
- **B5:** `SMTP_PASS`, `RESEND_API_KEY`, `ADMIN_PASSCODE` in tracked files → FAIL.
- **B11:** any reference to `web3forms`, `WEB3FORMS_KEY`, or `api.web3forms.com` anywhere in the build → FAIL. The provider is retired.
