<!-- Technical design addendum for GH-13 -->

# Technical Design Addendum: Warm Organic + Motion

**Status**: Approved
**Author**: Tech Lead / Frontend Engineer
**Date**: 2026-07-02
**Ticket**: [GH-13](https://github.com/ahmed-faruk/gls-school/issues/13) · PR [#14](https://github.com/ahmed-faruk/gls-school/pull/14)

---

## Brief

The GH-11 simplification was structurally sound but read as lifeless: one muted accent,
no animation, plain system fonts. Stakeholder ask: bring the site to life with visual
effects, animation, and color; use trend-current Google CDN fonts covering both Arabic
and English; and pick a design concept that fits a K-12 school and its parent audience.

## Direction (pivoted mid-ticket)

The ticket originally specified a bento grid (2026's dominant UI pattern). Stakeholder
feedback rejected bento as a SaaS/product-marketing pattern that doesn't fit a school.
Replaced with the **warm organic** education style — the pattern real award-winning
school sites use (soft nature-inspired shapes, warmth/community-first): floating ambient
blob shapes, wavy SVG section dividers, and friendly rounded cards with colored organic
icon badges. Pivot recorded as a comment on GH-13.

## Typography (Google Fonts CDN)

- **El Messiri** (display) — warm, rounded, geometric; real character in both Arabic and
  Latin scripts.
- **Cairo** (body) — clean bilingual workhorse; Egypt-connected typeface, a
  subject-grounded fit for a Cairo-area school.
- Both families natively ship Arabic + Latin, so the visual character survives the
  language toggle instead of feeling like two different sites. `font-display: swap`.

## Color

Vibrant coral / sky / mint / yellow over the GH-11 base. Applied through gradients on
fact cards, icon badges, gallery placeholders, and hero blobs. Fact-card gradients are
deliberately darker than the badge/blob hues because those cards carry small white text —
the lightest gradient stop itself passes WCAG AA (#cc4428 = 4.75, #33739e = 5.14,
#25805b = 4.86 vs white). Lighter hues restricted to decorative aria-hidden elements.

## Motion budget (all gated behind prefers-reduced-motion)

1. Hero staggered entrance (eyebrow → headline → sub → CTA, 110ms steps, CSS keyframes)
2. Ambient gradient backdrop drift in the hero (26s loop, low opacity)
3. Floating blob drift (18s loop)
4. Stat count-up on the founded-year figure (JS tween, IntersectionObserver-triggered)
5. Card stagger-in reveal per `.reveal-group` (IntersectionObserver + 90ms per-card delay)
6. Hover lift on fact/feature cards, nav underline draw-in, header scroll shadow,
   testimonial crossfade, button press scale-down

No-JS safety: `.reveal` hiding is scoped under `html.js` (class set synchronously in
`<head>`), so content is fully visible when JS never runs.

## Bugs caught by the screenshot loop before shipping

- **Reduced-motion cascade**: `html.js .reveal` outranked the reduced-motion override,
  leaving cards invisible under `prefers-reduced-motion` with JS on. Fixed by matching
  specificity inside the media query. Caught via Playwright `reducedMotion: 'reduce'`
  emulation.
- **Contrast**: white small text on the original gradient stops failed AA
  (coral 2.81, sky 2.62, mint 2.32). Gradients darkened as above.
- Full-page screenshots show `.reveal` elements at opacity 0 — that's a capture artifact
  (full-page capture doesn't scroll, so IntersectionObserver never fires); verified real
  behavior with `scrollIntoViewIfNeeded` + wait.

## Verification

Headless-Chromium screenshots: 1440px AR + EN (via real `#lang-toggle` click), 375px AR,
why-us stagger on real scroll, reduced-motion emulation, two-frame entrance-completion
check. Static: HTML parse, JS syntax, i18n parity (74/74), CSS brace balance, WCAG AA on
all text-bearing pairs. Remaining manual: 768px spot-check, form/nav/map functional pass.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Author |
