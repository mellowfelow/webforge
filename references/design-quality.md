# WebForge — Design Quality reference (v9.1) — clean, rich, intentionally beautiful

Read when building or auditing **any** page. Technical correctness is not the bar — **visual quality is a build gate.** A site can pass every SEO/accessibility/performance check and still look bare, uneven, or templated. This file makes "clean and genuinely attractive" a non-negotiable, checkable standard, the same way schema and sitemaps are.

> **Why this exists (v9.1).** v9 guaranteed a *technically* excellent site but never pinned down layout uniformity, hero composition, section rhythm, or visual richness — so builds came out sparse and inconsistent. This reference closes that gap. The auto-styling system (archetype detection, CSS tokens, font pairing) picks the *palette and type*; this file governs the *composition and polish*.

The target in one line: **every page should look like it was designed on purpose by someone with taste — rich but uncluttered, uniform, and calm.**

---

## The seven rules (all non-negotiable)

### 1. No bare sections (visual density)
- **No section is plain text on flat white.** Every section carries at least one visual anchor: real imagery, a cohesive icon set, an elevated/bordered card treatment, a subtle background tint/gradient, or a graphic device (rule line, number, shape).
- The **homepage has ≥ 6 visually distinct sections** with *alternating surface treatments* (white → tinted → white, or contained → full-bleed) to create rhythm. Adjacent sections never share the same flat background with no divider.
- Whitespace is generous **but intentional** — emptiness is not whitespace. Balance negative space with a clear visual anchor in every section.

### 2. Grid & tile uniformity (the "clean / orderly" rule)
- **Every grid uses ONE shared tile/card component** with a **single fixed aspect ratio**, images `object-fit: cover`, **equal heights**, and a **consistent gap**. This applies to product cards, category tiles, brand tiles, and blog cards.
- **Category tiles:** one fixed aspect (1:1 or 4:3 — pick one and use it for all), image `cover`, and a consistent label treatment (caption bar or overlay) identical across every tile. Never mixed sizes, never ragged heights, never some-with-images-some-without.
- **Product cards:** keep the existing uniform 4:3 image frame; card body (title, price, CTA) equal height via flex so buttons align across the row.
- One **border-radius token**, one **shadow token**, one **border treatment** — reused everywhere. No ad-hoc per-component radii or shadows.

### 3. Hero composition
- The hero has a **deliberate image treatment**, not a squashed or awkwardly-cropped photo:
  - either a **full-bleed background image** with a contrast overlay/scrim, or
  - a **sized image grid/collage** where each cell is `object-fit: cover` with tuned `object-position` so subjects are actually visible.
- **Minimum hero height** so imagery reads: **≥ 70vh desktop / ≥ 60vh mobile** (or a properly sized image container that fits its content). If product/subject images must show fully, size the hero to them — do not crop the subject out.
- Hero text stays legible over imagery via an overlay/scrim (ties directly to the WCAG AA contrast rule in `accessibility.md`).

### 4. Section rhythm & alignment consistency
- **One content container max-width** and **one horizontal alignment pattern** applied consistently. If the FAQ is centered, "brands we carry," trust bar, and other peer sections use the **same** centering — no section drifts to a different alignment by accident.
- A **reused section-header pattern**: optional eyebrow/kicker + H2 + optional one-line sub, same spacing every time.
- **Consistent vertical rhythm:** one section-padding scale, one heading→body spacing. Sections feel evenly spaced, not cramped in one place and airy in another.

### 5. Depth, colour & type polish
- Use the archetype palette fully: **surfaces are not all pure #fff** — introduce subtle surface tints (e.g. a 2–4% brand or neutral tint) to separate sections and create depth. Pure black-on-white with no tint reads as "unfinished."
- **Type hierarchy is obvious:** display font for headings, readable body font, a clear type scale, comfortable line-height, and a constrained measure (~60–75ch) for body copy.
- **Depth via soft shadows/borders** on cards; **hover states** on every interactive card/tile; cohesive **iconography** (one icon set, one stroke weight) — never mixed icon styles.

### 6. Tasteful motion (within reduced-motion limits)
- Subtle, **non-blocking** micro-interactions only: gentle fade/slide-in on scroll, hover lift on cards, smooth button/link transitions. They add life without shouting.
- **No carousels, no autoplay, no motion that hurts CLS/LCP.** Motion never causes layout shift.
- **All motion is gated behind `prefers-reduced-motion: reduce`** and fully disabled when requested.

### 7. Imagery quality & consistency
- Consistent image treatment across a context: same corner radius, same aspect, `cover` fit — **never stretched or squashed.** Mismatched image shapes in one grid is an automatic fail.
- Provide graceful treatment when a real photo is missing (branded gradient/pattern placeholder that still looks intentional), never a broken frame or a raw off-ratio image.

---

## Acceptance bar — what "very attractive and clean" means (checkable)

A build passes visual quality only if **all** are true:
- No section is unstyled text on flat white; every section has a visual anchor.
- Homepage has ≥ 6 distinct sections with alternating surface treatments.
- Every grid is uniform: one aspect ratio, `object-fit: cover`, equal heights, consistent gap.
- Hero imagery shows its subjects clearly; hero meets the min-height (or sized-container) rule; text is legible over it.
- One shared radius token, one shadow token, one spacing scale — used everywhere (no scattered ad-hoc values).
- Section alignment is consistent (peer sections share the FAQ's centering pattern where centered).
- Palette is cohesive with surface tints (not default black-on-white); type hierarchy is clear.
- Interactive tiles/cards have hover states; tasteful scroll/hover motion present and `prefers-reduced-motion`-safe.

---

## Crosscheck — visual quality (static tier, enforced on every build)

These are the concrete, parseable proxies for the acceptance bar (crosscheck item 41):
1. **Grid uniformity:** each grid (category, product, brand, blog) renders through a single shared component; one aspect-ratio utility per grid; images use `object-fit: cover`; no per-tile size overrides.
2. **No-bare-section:** every homepage `<section>` contains at least one of {`<img>`/`SmartImage`, icon element, `.card`/elevated container, tinted/gradient background class}. Flag any section that is only heading + paragraph on the default background.
3. **Token discipline:** border-radius, box-shadow, and section padding come from CSS custom properties / shared utilities — flag scattered inline/ad-hoc px radii or shadows.
4. **Hero rule:** hero has a min-height (or a sized image container) and, if it carries text over imagery, an overlay/scrim element.
5. **Alignment consistency:** peer homepage sections share one container max-width and alignment pattern; flag a section that diverges.
6. **Motion safety:** any animation/transition is wrapped by a `prefers-reduced-motion: reduce` disable block.
7. **Section count/rhythm:** homepage has ≥ 6 sections with ≥ 2 distinct surface treatments alternating.

> Beauty has a subjective edge a static check can't fully capture, so these proxies enforce the *structural* conditions of a clean, rich design. The build still applies taste on top — but it can no longer ship a bare, uneven, or flat page and call it done.
