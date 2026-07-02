<!-- Source: ApexYard technical-design pattern, adapted for a styling-only pass (no new architecture) -->

# Technical Design Addendum: 2026 Visual Refresh

**Status**: Approved
**Author**: Tech Lead
**Date**: 2026-07-02
**Ticket**: [GH-7](https://github.com/ahmed-faruk/gls-school/issues/7)
**Base design**: [marketing-site.md](marketing-site.md)

---

## Summary

Restyle the existing single-page GLS site per 2026 web-design trend research (see below), without changing its structure, i18n system, or form logic. Styling/motion-only pass — no new architecture, no new dependencies.

## Trend research basis (web search, 2026-07-02)

- **"Nature distilled" palette** — muted, warm earthy tones (clay/terracotta, sand, sage, deep teal) over the louder neon/dopamine palette variant; better fit for a children's school (calm, trustworthy) than for a consumer brand.
- **Organic shapes / soft gradients** replacing rigid grids and hard geometric fills.
- **Bold storytelling typography** for hero/headline moments.
- **Subtle skeuomorphism** — soft layered shadows, light embossing, gentle gradients for tactile depth on cards/buttons (a refined return, not heavy 2000s-style skeuomorphism).
- **Scroll-triggered motion** — lightweight reveal-on-scroll, respecting `prefers-reduced-motion`.
- **School-site specific**: lead with warmth/community before curriculum details; mobile-first is non-negotiable (67%+ of parents browse on mobile).

## Palette tokens (replaces existing `:root` variables in `styles.css`)

| Token | Old | New | Purpose |
|---|---|---|---|
| `--color-primary` | `#2f6f8f` (flat blue) | deep warm teal | primary actions, links |
| `--color-accent` | `#f4b93f` (flat yellow) | warm clay/terracotta | secondary accents (nav CTA, icons) |
| `--color-surface-alt` | flat pale blue-gray | warm sand | section backgrounds |
| new: `--color-sage` | — | muted sage green | tertiary accent for shapes/blobs |

Contrast checked manually against WCAG AA for text-on-background pairs before shipping.

## Shape & depth approach

- Hero and section-divider backgrounds get one or two large organic blob shapes (inline SVG with soft asymmetric curves, or CSS `border-radius` trickery on a pseudo-element) in the sage/clay tones at low opacity — decorative only, `aria-hidden`.
- Cards (`.card`, `.feature`) and the primary button get a soft layered `box-shadow` + a subtle linear-gradient fill instead of a flat color, for the "refined skeuomorphism" feel.

## Motion approach

- New `initScrollReveal()` in `script.js` using `IntersectionObserver`: on first intersection, adds a `.revealed` class (CSS handles the fade/slide transition). Content has `.reveal` as a CSS class with a `.revealed` end-state — **the pre-observed state must not be `display:none`/`visibility:hidden`**, only `opacity`/`transform`, so content stays present (and readable, e.g. for crawlers or if JS fails to attach) even before the class flips.
- Wrapped in `@media (prefers-reduced-motion: reduce)` to disable the transition entirely (content just appears).

## Content reordering

- Promote the Testimonials section (already the clearest "warmth/community" beat with parent voices) to appear directly after the Hero, ahead of About/Programs — matches the trend guidance to lead with belonging before curriculum. Pure markup move in `index.html`; no copy or i18n key changes.

## Responsiveness checklist (QA, GH-7)

- [ ] 320px, 375px, 768px, 1024px, 1440px — no horizontal scroll, all sections legible
- [ ] Organic shapes/blobs don't overflow or obscure text at narrow widths
- [ ] RTL (Arabic) mirrors correctly at every breakpoint, including blob placement
- [ ] `prefers-reduced-motion: reduce` disables scroll-reveal transitions
- [ ] Content is visible even if JavaScript fails to run (no `display:none` gating)

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Author |
