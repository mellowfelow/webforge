# Mobile-native — the fixes behind "feels like an app" vs. "feels like a website"

`pages.md`'s mobile section (overflow guards, 4→2→1 columns, hamburger ≤700px, tap targets ≥44px) and
AUDIT A's "Mobile (release blocker)" bullet are the floor. This file is the detail layer: the specific
platform-level bugs — viewport, touch, scroll, safe areas — that separate a site that merely survives
on a phone from one that feels installed. None of these are animation; for motion see
`design-quality.md` rule 6.

**Hard rule: never declare one of these fixed from Chrome's device-toolbar emulator.** Sticky hover,
the 300ms tap delay, the URL-bar height bug, input zoom, overscroll, safe-area insets, and the
software keyboard are real-hardware behaviors that do not reproduce in emulation. Verify on a real
phone (connect over USB, open the dev server by LAN IP, Safari Web Inspector / `chrome://inspect`) or
say plainly in the audit report which fixes could only be verified from code.

## Symptom table

| Problem | Fix |
|---|---|
| Hover state stuck after a tap | Wrap in `@media (hover: hover) and (pointer: fine)` |
| Gray/blue flash on tap | `-webkit-tap-highlight-color: transparent` globally; give every tappable element its own `:active` state |
| Layout has the wrong height | `100dvh` for app shells/drawers/bottom-pinned UI; `100svh` for heroes (never cut off, no layout shift mid-scroll) — never raw `100vh` for either |
| Page zooms into an input | Input `font-size` ≥16px. **Never** `user-scalable=no` or `maximum-scale=1` — that's an accessibility failure; fix the font size, which is the actual cause |
| Tap feels laggy | `touch-action: manipulation` to kill the 300ms click delay; feedback on `:active`/`pointerdown`, never `click`-only |
| Pull-to-refresh hijacks scroll | `overscroll-behavior: none` on `html, body`; `overscroll-behavior: contain` on inner scrollables (sheets, drawers) |
| Content stops at the notch | `viewport-fit=cover` in the viewport meta + `env(safe-area-inset-*)` padding on fixed headers/bottom bars/sheets |
| Long-press selects button text | `user-select: none` + `-webkit-touch-callout: none` on controls — never on `body` (users copy addresses/order numbers, that's real content) |
| Carousel scrolls the wrong way | `touch-action: pan-y` on a horizontal carousel track; `pan-x` on a vertical sheet handle; `none` only on an element that owns every axis itself |
| Status bar color mismatched | Separate `theme-color` meta per `prefers-color-scheme` |

## Baseline — ship this before the first component

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, interactive-widget=resizes-content" />
<meta name="theme-color" media="(prefers-color-scheme: light)" content="#ffffff" />
<meta name="theme-color" media="(prefers-color-scheme: dark)" content="#0a0a0a" />
```

```css
html {
  -webkit-tap-highlight-color: transparent;
  -webkit-text-size-adjust: 100%; /* no font inflation in landscape */
  overscroll-behavior: none;      /* drop this line if the site is a scrolling document where
                                      pull-to-refresh is actually welcome */
}
input, textarea, select { font-size: 16px; }
button, a, [role="button"] {
  touch-action: manipulation;
  user-select: none;
  -webkit-user-select: none;
}
@media (hover: hover) and (pointer: fine) { /* all :hover rules live here, nowhere else */ }
```

`interactive-widget=resizes-content` makes the software keyboard shrink the layout viewport on
Android Chrome, so `100dvh` and bottom-pinned inputs react to it the way they already do on iOS.

## Never ship — self-check before calling mobile done

| Never | Instead |
|---|---|
| `user-scalable=no` / `maximum-scale=1` | 16px inputs — fix the cause, not the symptom |
| Ungated `:hover` | `@media (hover: hover) and (pointer: fine)` |
| `100vh` for an app shell or bottom-pinned UI | `100dvh` |
| `100vh`/`100dvh` on a marketing hero | `100svh` — stable, never overflows, no layout shift on scroll |
| Press feedback on `click` only | `:active` / `pointerdown` |
| `touchmove` + `preventDefault()` to fight overscroll | `overscroll-behavior` |
| `user-select: none` on `body` | Only on controls, never on real content |
| `env(safe-area-inset-*)` without `viewport-fit=cover` in the meta tag | The value is `0px` without it — add the meta tag |
| One `theme-color` for both light and dark | One per `prefers-color-scheme` |
| User-agent sniffing to detect touch | `(hover)` / `(pointer)` media queries |
| Declaring a mobile fix verified from device emulation | Real hardware, or say plainly which fixes are unverified |

## Stress-testing product/category/cart grids with real data

Product cards, cart rows, and category tiles render real data — names, prices, counts. Demo data is
implicitly chosen to look good (short names, round counts, every field filled); test the realistic
*worst* case instead of a nonsense one (not 5,000-character strings — the actual longest value the
schema/database column allows, or a real naming pattern like a real-world long product name).

| What breaks | Cause | Fix |
|---|---|---|
| Avatar/icon squished into an oval | Flex child shrinking | `flex-shrink: 0` |
| Text overflows instead of wrapping | `min-width: auto` default on a flex/grid child | `min-width: 0` on the text column |
| Long product name/URL runs past the edge | No break opportunities in the string | `overflow-wrap: anywhere` |
| Trailing action (qty stepper, remove button) pushed off-screen | Middle content took all the space | `min-width: 0` on the middle, `flex-shrink: 0` on the action |
| "1 items", "0 item" | Hardcoded plural string | `Intl.PluralRules`, or separate strings per count |
| Prices/counts jitter or columns misalign when updating | Proportional figures | `font-variant-numeric: tabular-nums` |
| Raw `1284`, `NaN`, `undefined` rendered | No number formatting | `Intl.NumberFormat` with `SITE.locale`; guard null |
| Long badge/button label overflows | Fixed-width element | Width from content with `min-width`, never a fixed `width` |

**Truncate, wrap, or clamp — the actual decision, per field, not globally:**
- **Wrap** text the user needs in full: product names, titles on the PDP.
- **Truncate at the end** for secondary metadata where the start carries meaning: category label,
  short description preview.
- **Truncate in the middle** only when items differ at the *end* (file names, long SKUs) — rare in a
  webforge build.
- **Clamp** (`line-clamp: 2`) for multi-line previews in cards, to keep card heights predictable.
- **Never truncate** prices, quantities, or dates — the exact figures the user is there to compare.
