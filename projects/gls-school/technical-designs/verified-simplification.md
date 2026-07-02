<!-- Technical design addendum for GH-11 -->

# Technical Design Addendum: Verified Simplification

**Status**: Approved
**Author**: Tech Lead / Frontend Engineer
**Date**: 2026-07-02
**Ticket**: [GH-11](https://github.com/ahmed-faruk/gls-school/issues/11)

---

## Root cause of three bad passes

The base build (#6), the palette refresh (#8), and the "Copybook" redesign (#10) were all
judged bad by the user with no way to pin down specifics — because none of them were ever
actually rendered and looked at. The Chrome extension never connected in any session, so
verification was limited to static checks (HTML parses, JS syntax is valid, i18n keys
match, contrast ratios compute correctly). None of that catches what a page actually looks
like: flat color rhythm, amateurish illustration, broken text wrapping.

## Fix

Installed headless Chromium directly (`npx playwright install chromium`, ~100MB, one-time,
local to this environment) so the page can be screenshotted and inspected like a real
browser would show it, without depending on the Chrome extension connecting. This should be
the default verification step for any future visual change to this site — a plan or PR that
touches `styles.css`/`index.html` visual output should include real screenshots, not just
static checks.

## What the first screenshot found

- Section backgrounds (`--paper` `#F3E8D2` vs `--paper-alt` `#EADCC0`) were too close in
  tone — the page read as one undifferentiated beige block despite intending distinct
  section "beats."
- The hero SVG (a hand-drawn squiggly line over ruled paper, meant to evoke a copybook)
  looked like unpolished clip-art at real size, not a premium placeholder.
- Overall typographic contrast was too low to establish clear hierarchy at a glance.

## What changed

- Removed: hero SVG illustration, animated dotted-to-solid baseline SVG, animated
  programs-path SVG, folded-corner `clip-path` cards, marginalia-style testimonial border —
  all bespoke devices from the "Copybook" concept, all unverified visually before this pass.
- One confident accent (`--accent: #c5432b`) instead of a multi-hue palette family.
- Clear alternating section rhythm: white / `--surface` (warm tan) / `--surface-dark`
  (near-black green) bands, so each section reads as a distinct beat.
- Larger, bolder system-font type scale (no external font dependency — Fraunces/Amiri
  removed, one less unverified variable).
- Simple solid-color placeholder blocks (hero media, gallery) instead of illustrated SVG.

## Bug found via the new verification loop

A second screenshot (this time at 375px mobile width) caught the header logo text
("مدرسة جلوريز للغات") wrapping to 3 lines and blowing out the header height — invisible in
any static check, obvious in the screenshot. Fixed by hiding the logo text below 520px
(keeping just the circular mark) and adding `white-space: nowrap` + ellipsis above that
breakpoint.

## Verification

Screenshotted at 1440px (Arabic/RTL default, and English/LTR via clicking the actual
`#lang-toggle` button) and 375px (Arabic/RTL, before and after the header fix). Re-ran all
static checks (HTML parse, JS syntax, i18n key parity, CSS brace balance, WCAG AA contrast)
— all pass. Tablet widths (768/1024px) and full functional re-check (form/nav/map) are
flagged as remaining manual steps in `QA-NOTES.md`.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Author |
