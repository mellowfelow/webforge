# webforge

Complete React/Next.js ecommerce website factory — builds, updates, deploys, and audits sites on
Vercel (default, via GitHub) or Cloudflare Pages (via ZIP), with a reply portal + order dashboard on
by default and an optional client CMS backend. See [SKILL.md](SKILL.md) for the full build process,
and `references/` for the detailed spec per area (pages, SEO, accessibility, performance, design
quality, forms, backend, deploy targets, niche variants).

## Installing this skill in Claude Code

Claude Code discovers a skill from a folder under `~/.claude/skills/` whose **folder name matches the
`name:` field in `SKILL.md`'s frontmatter** (`webforge`). To install:

1. Copy this entire folder (including `SKILL.md` and the whole `references/` directory) to
   `~/.claude/skills/webforge/`.
2. No dependencies to install — this is a pure instruction skill, no scripts or packages required.
3. Claude Code picks it up automatically; invoke it by asking to build, update, or audit a site, or
   any of the trigger phrases listed in `SKILL.md`'s frontmatter description (e.g. "build me a site",
   "technical audit", "check SEO", "ai visibility", "deploy check").

This repo can also just be shared/cloned as-is — the install step is only about *where* Claude Code
looks, not about the files themselves.

## What you need before using it

- A GitHub account (for the default Vercel deploy path) or a Cloudflare account (for the Cloudflare
  Pages ZIP path) — the skill asks for deploy-target preference during intake.
- No API keys are required to build a site. Optional integrations (live chat, analytics, payment
  processors) are configured per-client during intake if the client wants them.

## Structure

```
webforge/
├── SKILL.md              ← the build process, all modes (intake, build, audits), crosscheck
└── references/
    ├── pages.md           ← every page type's exact spec
    ├── mobile-native.md   ← platform-level mobile layer (tap highlight, dvh/svh, safe areas, overscroll)
    ├── design-quality.md  ← the visual-quality build gate (grid uniformity, motion, colour, tells)
    ├── usability-audit.md ← Nielsen heuristics, cognitive load, 5-persona walkthrough (AUDIT A)
    ├── accessibility.md   ← WCAG 2.2 AA — static + live checks
    ├── performance.md     ← Core Web Vitals, image handling
    ├── seo.md              ← meta/schema formulas, sitemaps, image SEO
    ├── nextjs.md           ← App Router patterns, components
    ├── forms.md            ← contact/order forms, email templates
    ├── backend.md          ← optional client CMS (Cloudflare-only)
    ├── reply-portal.md     ← order dashboard + reply portal
    ├── intake.md           ← the client intake questionnaire
    ├── niche-variants.md   ← locale, age-gate, compliance, payment variants per niche
    ├── vercel.md / cloudflare.md ← deploy-target specifics
    ├── ai-visibility.md    ← GEO/AEO — brand entity signals, schema for AI visibility
    └── agent-live.md       ← MCP server / agent-ready integration
```
