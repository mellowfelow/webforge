# WebForge — Accessibility reference (v9) — WCAG 2.2 AA

Read when building or auditing any page. Accessibility is a **build standard and an audit gate**, not an afterthought. Target conformance: **WCAG 2.2 Level AA** on every public page, both targets.

> **v9 delta.** v8 shipped a component-level baseline (`aria-label` on icon buttons, focus rings, `aria-live` on cart totals, `prefers-reduced-motion`). That baseline is retained and folded in below. v9 raises the bar to full WCAG 2.2 AA and adds accessibility to both the pre-ship crosscheck (static checks) and MODE 4 AUDIT F (live checks that need rendering).

---

## Why this splits into two tiers

Some accessibility rules are checkable from the built HTML with no browser (alt text present, labels bound, heading order, `lang`, ARIA roles, skip-link present). Others require rendering (colour contrast, visible focus, keyboard tab order, screen-reader name computation). So accessibility is enforced in two places:

- **Tier 1 — static (pre-ship crosscheck, gates the build):** everything parseable from the HTML/JSX output.
- **Tier 2 — live (MODE 4 AUDIT F, post-deploy):** contrast, keyboard operability, focus visibility — run with axe-core against the deployed URL.

A build never ships on a failing Tier-1 accessibility check. Tier-2 findings are reported and fixed, but can't run before a URL exists.

---

## The AA standard — what every page must satisfy

### 1. Perceivable
- **Text alternatives.** Every `<img>` has descriptive `alt` (see `seo.md` → Image SEO). Decorative images use `alt=""` AND `aria-hidden="true"`. Icon-only controls carry an `aria-label`.
- **Colour contrast (AA):** body text ≥ **4.5:1** against its background; large text (≥24px, or ≥18.66px bold) and UI components/graphical objects ≥ **3:1**. Never rely on colour alone to convey meaning (e.g. form errors get an icon or text, not just red).
- **Content reflow:** no horizontal scroll at 320px CSS width (already covered by the mobile overflow guards — accessibility and mobile-first agree here).
- **Text resize:** layout survives 200% zoom without loss of content or function.

### 2. Operable
- **Full keyboard operability.** Every interactive control (nav, mega-dropdown, hamburger, qty stepper, add-to-cart, cart remove, forms, chat hub, accordions) is reachable and usable with keyboard alone. No keyboard traps.
- **Visible focus (2.2 AA — Focus Not Obscured):** a clearly visible focus indicator on every focusable element; the focused element is never fully hidden behind sticky headers/announcement bars. Use `:focus-visible` rings, not `outline:none`.
- **Logical focus order:** DOM order matches visual order; never re-order with positive `tabindex`. Only `tabindex="0"` (adopt) and `tabindex="-1"` (programmatic focus) are allowed.
- **Skip link:** a `Skip to main content` link is the first focusable element, jumping to `<main id="main">`.
- **Target size (2.2 AA — Target Size Minimum):** interactive targets ≥ **24×24px** minimum; WebForge keeps its existing **≥44px** tap-target rule, which already exceeds AA.
- **No motion trap:** `prefers-reduced-motion: reduce` disables non-essential animation (carousels are already banned; honour it for any transition).

### 3. Understandable
- **Language:** `<html lang>` from `SITE.locale` (already in layout).
- **Labels & instructions:** every form control has a programmatically associated `<label>` (or `aria-label`/`aria-labelledby`). Placeholders are never the only label.
- **Error identification & suggestion (AA):** on validation failure, the field gets `aria-invalid="true"`, an error message tied via `aria-describedby`, and the message states how to fix it. Errors are announced (container is `aria-live="polite"`).
- **Consistent navigation:** nav and footer are identical across pages (guaranteed by the shared layout).

### 4. Robust
- **Semantic landmarks:** exactly one `<header>`, one `<main id="main">`, one `<footer>` per page; `<nav>` for navigation blocks. Don't simulate landmarks with `<div>`s.
- **Heading order:** exactly one `<h1>`; no skipped levels (h1→h2→h3). This overlaps the existing single-H1 rule — accessibility extends it to *no skipped levels*, which the crosscheck now enforces.
- **Name, role, value:** custom widgets (accordion via `<details>`, mega-dropdown, qty stepper) expose correct roles/states. Prefer native elements (`<details>`, `<button>`, real `<a href>`) over ARIA re-implementations.

---

## Component accessibility contract (folds in the v8 baseline)

| Component | Requirements |
|---|---|
| `Nav` | `<nav aria-label="Primary">`; hamburger is a `<button aria-expanded>` toggling the menu; mega-dropdown items are real `<a href>`; focus order logical; skip-link precedes it |
| `ChatHub` | expander is a `<button aria-expanded aria-controls>`; each channel is a real `<a href>` with an accessible name; icon-only trigger has `aria-label` |
| `QtyStepper` | −/+ are `<button type="button">` with `aria-label` ("Decrease quantity"/"Increase quantity"); the number input has an associated label; value change announced |
| `WebForm` | labels bound to inputs; `autoComplete` set; `type="email"`/`type="tel"`; errors via `aria-invalid` + `aria-describedby`; submit is `<button type="submit">`; success/failure in an `aria-live` region |
| `ProductCard` | image `alt` set; add-to-cart is a `<button>`; price and name are text, not image; card is not a nested-interactive trap (no `<a>` wrapping a `<button>`) |
| `Breadcrumbs` | `<nav aria-label="Breadcrumb">` with an ordered list; current page marked `aria-current="page"`; mirrors `BreadcrumbList` schema exactly |
| `Footer` | `<footer>` landmark; link columns are lists; entity-encoded email is a real `mailto:` link |
| Cart | totals in `aria-live="polite"`; remove is a labelled `<button>`; min-order block communicates the disabled reason in text, not colour alone |
| Accordions (FAQ) | native `<details>/<summary>` — keyboard-operable for free; never a div+JS reimplementation |

---

## Static build rules (Tier 1 — enforced by crosscheck)

Add these to the shared layout / components so they are true on every page:
- Skip-link as first focusable element → `<main id="main">`.
- One `<h1>`, no skipped heading levels.
- Every `<img>` has `alt` (empty + `aria-hidden` if decorative).
- Every form control has an associated label; every icon-only button has `aria-label`.
- Landmarks present: one `header`/`main`/`footer`; `nav` labelled.
- No positive `tabindex`; no `outline:none` without a `:focus-visible` replacement.
- `prefers-reduced-motion` block present in global CSS.

## Live audit rules (Tier 2 — AUDIT F, needs the deployed URL)

Run axe-core (or Lighthouse Accessibility) against the deployed URL and assert:
- Zero critical/serious violations.
- Contrast ≥ 4.5:1 (text) / 3:1 (large text + UI) everywhere.
- Full keyboard walk: every control reachable and operable; focus always visible and never obscured by the sticky header; no traps.
- Lighthouse Accessibility ≥ 95 (see `references/performance.md` for the unified live gate).

---

## What accessibility does NOT allow the build to do

- No `outline: none` without a visible `:focus-visible` replacement.
- No colour-only status (errors, availability, active state) without a text/icon partner.
- No `<div onClick>` for anything that should be a `<button>`/`<a>`.
- No positive `tabindex`, no ARIA that contradicts a native element's built-in semantics.
- No auto-playing motion that ignores `prefers-reduced-motion`.
