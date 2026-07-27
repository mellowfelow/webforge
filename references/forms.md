# WebForge — Forms reference (v8)

Read for any form. **Provider-pluggable; one constant switches it.**

```js
export const FORMS = {
  provider: 'web3forms',        // 'web3forms' (default, both targets) | 'resend' (Vercel only)
  web3formsKey: 'YOUR-WEB3FORMS-KEY',
  resendFrom: '',               // orders@yourdomain.com — REQUIRES a verified domain
  turnstileSiteKey: '',
}
```

## Choosing

| | `web3forms` | `resend` |
|---|---|---|
| Targets | Vercel + Cloudflare | **Vercel only** (needs an API route) |
| Needs a domain? | **No** — an email address is enough | **Yes** — verified sending domain (DNS) |
| Sends from | Web3Forms infrastructure | your own domain |
| Setup | paste an access key | API route + `RESEND_API_KEY` + DNS |
| Spam | honeypot + optional Turnstile | server-side validation + Turnstile |

**Default `web3forms`.** It is the only provider that works while the domain is pending — the normal state of a new build. Never propose `resend` for a site without a live, verified domain: Resend can only send to your own address until DNS is verified, so every customer email silently fails.

Switching later: change the constant, add `RESEND_API_KEY` in Vercel, rebuild, push.

---

## Provider: `web3forms` — the exact CORS method

This is not a style preference. Deviate and forms break with "Failed to fetch".

**The form:** NO `action`, NO `method`, NO `redirect` hidden field.

Hidden inputs: `access_key`, `subject`, `from_name`, `botcheck` (honeypot — must stay empty; `display:none`), `replyto` (synced from the email field on `input`, so replies go to the customer).

```javascript
function onSubmit(e) {
  e.preventDefault();
  fetch('https://api.web3forms.com/submit', {
    method: 'POST',
    headers: { 'Accept': 'application/json' },   // Accept ONLY — never Content-Type
    body: new FormData(form)                     // FormData — never JSON.stringify
  })
    .then(r => r.json().then(d => ({ status: r.status, data: d })))
    .then(res => {
      if (res.status === 200 && res.data.success) window.location.href = thankYou;
      else throw new Error((res.data && res.data.message) || 'Submission failed');
    })
    .catch(() => setError(true));   // on-page error + encoded email + chat fallback
}
```

**Why this exact shape:**
| Setup | Result |
|---|---|
| `body: FormData` (no Content-Type) | multipart/form-data = CORS **simple request** → no preflight ✓ |
| header `Accept: application/json` only | Accept is CORS-safelisted → no preflight ✓ |
| header `Content-Type: application/json` | NOT safelisted → preflight OPTIONS → Web3Forms doesn't answer it → "Failed to fetch" ✗ |
| native POST `action="…"` | Web3Forms 301-redirects → CORS error ✗ |
| `redirect="false"` hidden field | conflicts with the JS redirect — **remove it** ✗ |

CSP: `connect-src` must include `https://api.web3forms.com`; `form-action 'self' https://api.web3forms.com`.

### Key-pending fallback — REQUIRED

```js
if (!FORMS.web3formsKey || FORMS.web3formsKey.startsWith('YOUR-')) {
  window.location.href = thankYou;   // never dead-end an order
  return;
}
```
Ordering must work during pre-launch. **State this at handover, every time:** *"Until the Web3Forms key is set, form submissions are not emailed anywhere — the chat hub is the only live order channel."* A silently-discarding order form is worse than no order form.

---

## Provider: `resend` — Vercel only

```js
// src/app/api/contact/route.js
import { Resend } from 'resend'
export async function POST(req) {
  const body = await req.json()
  if (body.botcheck) return Response.json({ success: true })          // honeypot: pretend success
  if (!body.email || !body.message) return Response.json({ success: false, message: 'Missing fields' }, { status: 400 })
  // if Turnstile configured: verify body.token server-side FIRST, reject on failure
  const resend = new Resend(process.env.RESEND_API_KEY)
  await resend.emails.send({
    from: FORMS.resendFrom,          // must be on a VERIFIED domain
    to: SITE.ordersEmail,
    replyTo: body.email,
    subject: body.subject,
    text: formatBody(body),
  })
  return Response.json({ success: true })
}
```
Client posts JSON to `/api/contact` — **same-origin, so none of the CORS constraints above apply** — then redirects to the thank-you page.

`RESEND_API_KEY` lives in Vercel env vars. **Never in the repo** (crosscheck B5).

**Crosscheck B3:** `provider: 'resend'` + static target = FAIL. `output: 'export'` drops the API route, so every form 404s silently.

---

## Turnstile (optional, both providers)

Cloudflare Turnstile, free, privacy-friendly, no puzzles. The honeypot alone stops naive bots only.
- Widget in the form; CSP `script-src` + `frame-src` → `https://challenges.cloudflare.com`.
- `resend`: verify the token server-side against `https://challenges.cloudflare.com/turnstile/v0/siteverify` **before** sending. A token you don't verify is decoration.
- `web3forms`: pass the token through; Web3Forms validates if configured.

---

## All forms, both providers

Three forms → three thank-you pages:
| Form | Redirects to |
|---|---|
| Contact | `/thank-you-contact/` |
| Order | `/thank-you-order/` |
| Wholesale | `/thank-you-wholesale/` |

- All thank-you pages: `noindex, follow`, excluded from the sitemap, disallowed in robots.txt.
- Every form: `e.preventDefault()`, a disabled/"Sending…" state, and an error path that shows an entity-encoded email plus a chat link. **A form whose only failure mode is silence loses the order.**
- Labels tied to inputs, `autoComplete` set, 44px minimum tap targets, `type="email"` / `type="tel"`.
- Order form: prefill from the cart, and offer a chat deep-link with the order pre-composed. Chat converts better than forms on mobile.

## Crosscheck (item 1)
Every form has: `new FormData(form)`, `Accept` header only, no `Content-Type`, no `action`, no `redirect` field, `e.preventDefault()`, a JS redirect to the correct thank-you page, `access_key` + empty `botcheck`, `replyto` sync. Plus B3 (resend on static) and B5 (secrets committed).
