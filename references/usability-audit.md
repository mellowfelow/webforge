# Usability & cognitive-load audit — the `Usability: /100` row

Read when running AUDIT A (Technical audit, `SKILL.md` Mode 4) on an existing site. This is a
**usability/cognition axis**, not a visual-polish, SEO, or performance one — `design-quality.md`
covers polish, this file covers "can a real user actually use this screen without friction." It
produces the `Usability: /100` row in the standard audit report template (`SKILL.md` lines ~878–901),
scored the same way as every other category already there.

---

## 1. Nielsen's 10 usability heuristics (0–4 each)

Score every heuristic honestly — a 4 means genuinely excellent, not "good enough." Most real
interfaces land 20–32 out of a possible 40.

| # | Heuristic | What to check |
|---|---|---|
| 1 | Visibility of system status | Loading indicators, action confirmations, progress indicators, current-location nav, inline form validation |
| 2 | Match system/real world | Familiar terms (no unexplained jargon), logical info order, recognizable icons/metaphors |
| 3 | User control and freedom | Undo/redo, cancel buttons, clear path back, easy way to clear filters/selections, escape from multi-step flows |
| 4 | Consistency and standards | Same terminology everywhere, same action = same result everywhere, platform conventions, visual consistency |
| 5 | Error prevention | Confirmation before destructive actions (cart remove, order submit), constraints on invalid input, smart defaults, autosave/draft recovery |
| 6 | Recognition over recall | Visible options (not buried in menus), contextual help, recent items/autocomplete, labeled icons |
| 7 | Flexibility and efficiency | Keyboard shortcuts, customization, bulk actions — **score n/a on marketing/portfolio pages** where this doesn't apply |
| 8 | Aesthetic/minimalist design | Only necessary info visible, clear hierarchy, purposeful color/emphasis, no decorative clutter |
| 9 | Error recovery | Plain-language errors, specific problem identification, actionable recovery suggestions, errors shown near the problem |
| 10 | Help and documentation | Searchable help, contextual tooltips, task-focused organization — **score n/a on marketing/portfolio pages** where this doesn't apply |

**Normalize to /100 for the report row:** sum the applicable heuristics' scores, divide by the
applicable max (40, or 32 if heuristics 7 and/or 10 are n/a on a marketing page), multiply by 100,
round to the nearest whole number. A webforge build (mostly marketing/shop pages) will usually score
against the 32-point applicable max; the backend CMS/dashboard (`backend.md`) scores against the full
40.

Tag every individual issue found with severity:
- **P0** — blocking, fix immediately
- **P1** — major, fix before release
- **P2** — minor, fix next pass
- **P3** — polish, fix if time permits

Borderline-severity tie-breaker: "would a user contact support about this?" — if yes, it's at least P1.

---

## 2. Cognitive load assessment

**The working-memory rule:** humans hold roughly ≤4 items in working memory at once (Miller's Law, as
revised by Cowan 2001). At any single decision point on the page, count the distinct
options/actions/pieces of info a user must weigh simultaneously: **≤4 is fine, 5–7 is pushing it**
(group the options or progressively disclose them), **8+ means users will skip, misclick, or
abandon.**

Applied to webforge's own page types:
- **Nav menus:** keep top-level items to a small, scannable count — group anything beyond that under
  a dropdown rather than adding more top-level links.
- **Product action buttons (PDP):** 1 primary action (Add to cart), 1–2 secondary (wishlist, compare)
  — anything else goes in a menu, not another button in the row.
- **Category/shop filter chips:** group or progressively disclose past ~4–5 visible options; don't
  render every filter value as a flat row of chips.

**8-item checklist** — count failures: 0–1 = good, 2–3 = address soon, 4+ = critical:
1. **Single focus:** can the primary task be completed without competing distractions?
2. **Chunking:** is info grouped in ≤4-item digestible groups?
3. **Grouping:** are related items visually grouped (proximity/borders/shared background)?
4. **Visual hierarchy:** is what matters most immediately obvious?
5. **One thing at a time:** can the user focus on one decision before the next?
6. **Minimal choices:** ≤4 visible options at any single decision point?
7. **Working memory:** does the current screen require remembering info from a previous one?
8. **Progressive disclosure:** is complexity revealed only when needed, not all at once?

Three types of load, useful for framing a fix: **intrinsic** (inherent to the task itself — structure
it, don't try to eliminate it), **extraneous** (caused by bad design — eliminate ruthlessly, it's pure
waste), **germane** (the effort of learning a pattern — support it with consistency and feedback;
this is the *good* kind, don't flag it as a problem).

---

## 3. Five personas for walking through the primary user flow

Walk the primary action for a page through the 2–3 relevant personas below and report **specific**
red flags — named elements and interactions that fail, not generic concerns ("the checkout is
confusing" is not a finding; "the discount-code field has no visible label and sits directly under the
qty stepper with no gap, read as the same control" is).

| Persona | Profile | What to check |
|---|---|---|
| **Alex** (Power User) | Expert, hates hand-holding, will find shortcuts or leave | Core task completable in under 60s? Keyboard shortcuts? Skippable onboarding? Esc dismisses modals? |
| **Jordan** (First-Timer) | Never used this type of product, needs guidance, will abandon rather than figure it out | First action clear within 5s? Icons labeled, not icon-only? Contextual help at decision points? Jargon-free copy? |
| **Sam** (Accessibility-dependent) | Screen reader/keyboard-only, needs 4.5:1+ contrast, may zoom to 200% | Keyboard-only flow completable end to end? Visible focus indicators? Meaningful alt text? State changes announced? |
| **Riley** (Stress Tester) | Pushes past the happy path: edge cases, unexpected input, backwards nav, refresh mid-flow | Empty-state and 1000-item states both handled? Graceful error recovery? State preserved on refresh? |
| **Casey** (Distracted Mobile) | One-handed on the go, interrupted often, impatient, possibly slow connection | Primary actions reachable in the thumb zone (lower half of screen)? State preserved on interruption? Touch targets ≥44×44px? |

**Persona selection by webforge page type:**

| webforge page | Primary personas to walk |
|---|---|
| Home / landing | Jordan, Riley, Casey |
| Shop / category listing | Casey, Riley, Jordan |
| Product (PDP) | Casey, Riley, Jordan |
| Cart / order | Casey, Riley, Jordan |
| CMS / client backend (`backend.md`, Cloudflare-only) | Alex, Sam |

---

## Reporting

Fold the findings into the standard audit report (`SKILL.md`'s report template) as:
- One **`Usability: /100`** row, computed per section 1 above.
- A short findings list under it, each item tagged with its severity (P0–P3) and, where relevant,
  which persona walkthrough surfaced it.
- Cognitive-load failures (section 2) and heuristic violations (section 1) that point at the same root
  cause should be reported once, not duplicated as two separate findings.
