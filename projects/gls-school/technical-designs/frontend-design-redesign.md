<!-- Technical design addendum for GH-9, following the frontend-design skill's brainstorm -> critique -> build process -->

# Technical Design Addendum: Frontend-Design Redesign ("The Copybook")

**Status**: Approved
**Author**: Tech Lead / Frontend Engineer
**Date**: 2026-07-02
**Ticket**: [GH-9](https://github.com/ahmed-faruk/gls-school/issues/9)

---

## Brief (per the skill's "ground it in the subject")

**Subject**: Glories Language School — a bilingual (Arabic/English) K-12 school in Sheikh Zayed, 6th October City, Egypt.
**Audience**: mothers evaluating schools for children from Kindergarten through Secondary.
**Page's single job**: convince a mother, in the first screen, that this is a warm, credible, bilingual place for her child — then get her to register interest.

## Self-critique of the prior two passes

The original build (#2-#5) and the 2026 "nature distilled" refresh (#7) both landed on a **generic template look**: flat rounded cards, a soft blue/yellow-then-teal/clay palette with no connection to the subject, and copy full of empty adjectives ("قيم أصيلة وتطور مستمر" / "distinguished team, continuously trained") that could belong to any school website. Nothing in the design or copy was specific to *this* being a bilingual language school. This addendum replaces both.

## Design plan (brainstorm, per the skill)

**Direction: "The Copybook"** — the page is styled like an open bilingual exercise/handwriting-practice book, because that artifact — ruled dotted baselines, a center page-fold, the coral-red teacher's marking pen, torn/creased page edges — is literally what this school's core product (a child learning to read and write in two languages) looks like in physical form. This avoids the three current AI-design defaults (cream+serif+terracotta, near-black+neon, broadsheet/hairline) by pulling every choice from the subject itself rather than a generic "warm school" mood board.

### Color (6 named hex values)

| Token | Hex | Role |
|---|---|---|
| `--ink` | `#24352C` | primary text, headings — chalkboard-dark green, not black |
| `--paper` | `#F3E8D2` | page background — warm ochre-tinted paper, not neutral cream |
| `--paper-alt` | `#EADCC0` | secondary page tone (deeper "page" sections) |
| `--pen-red` | `#C5432B` | signature accent — the marking pen; used sparingly (CTA, corrections/marginalia, the baseline signature) |
| `--chalk-teal` | `#4E8B7C` | secondary accent — chalk dust; used for the program path/timeline |
| `--pencil-gray` | `#5B5347` | muted/secondary text |

### Type (2 display roles + 1 body role)

- **Display (English)**: Fraunces — a warm, ink-trap variable serif with real character; used only for headings, restrained.
- **Display (Arabic)**: Amiri — a traditional Naskh calligraphic serif; pairs thematically with Fraunces (both are "handwritten book" faces, not generic Arabic UI fonts like Cairo/Tajawal).
- **Body (both languages)**: system sans stack (unchanged from the base build) — legibility and zero-dependency for the bulk of the reading experience; only the two display faces are loaded externally.
- **Deliberate exception to "zero dependency"**: the base technical design specified no external assets. Loading Fraunces + Amiri (Google Fonts, `font-display: swap`, subset to Latin+Arabic) is a considered trade for visual distinctiveness — both are static file loads with no JS/tracking, and body copy still renders instantly on system fonts while the display faces swap in.

### Layout concept

- **Hero**: two-panel "open book" composition with a soft center-fold shadow between the text panel and an illustrative panel; the headline sits on an animated baseline that draws from a dotted (tracing) line into a solid stroke on load — see Signature below.
- **Section rhythm**: dotted rule lines (like copybook baselines) replace hard borders between sections, reinforcing the artifact.
- **Programs (KG -> Primary -> Prep -> Secondary)**: a drawn connecting path (SVG, `stroke-dashoffset` animated on scroll-into-view) links the four stage cards as a real progression — justified use of a sequential device per the skill's guidance (numbering/ordering only when the content is truly a sequence, which grade progression is).
- **Cards**: a folded-corner detail (CSS `clip-path`) instead of a plain shadow box, reinforcing "page" material.
- **Testimonials**: restyled as marginalia — a handwritten-style annotation in `--pen-red` beside the quote, as if a teacher wrote a note in the book's margin.

### Signature element

**The confidence baseline** — the hero headline sits on a line that animates from a dotted "tracing" pattern to a solid stroke once, on page load. It is the one memorable, ownable moment: it literalizes a child's handwriting/language-learning journey (tracing -> confident writing), it's specific to a language school (not generic to "any school"), and it's the only large orchestrated animation on the page — everything else is quiet by comparison, per the skill's "spend your boldness in one place."

## Motion budget

1. Hero baseline: dotted -> solid draw, once, on load.
2. Programs path: SVG line draws in once, on scroll-into-view.
3. `prefers-reduced-motion: reduce` disables both — content/lines render in their final state immediately, nothing is gated on the animation completing.

No other animation (no per-card hover lift, no scroll-fade on every section) — the prior GH-7 pass added scroll-reveal to every section, which is exactly the "scattered effects" the skill warns reads as AI-generated; this pass removes that in favor of the two orchestrated moments above.

## Content rewrite approach

Per the skill's writing guidance: concrete over vague, active voice, no filler. Example changes:

- Hero headline: from generic "مستقبل ابنك يبدأ من هنا" / "Your child's future starts here" → "من أول حرف، إلى أول قصة" / "From the first letter to the first story" — ties directly to language learning, not swappable with any school's headline.
- Why-us copy: from empty adjectives ("معلمون مؤهلون... فريق تدريس متميز") → concrete, specific claims a mother can verify or picture (e.g. teachers training together across grade levels so the school "sounds like one school" from KG to Secondary).
- All facts that are real and sourced (founded 2016, Sheikh Zayed/6th October, KG-Secondary, contact info) are unchanged — this is a copywriting-quality pass, not new fact-finding; anything not independently verifiable stays clearly framed as illustrative/placeholder in code comments.

## Files touched

- `styles.css` — full token/typography/layout rebuild per the above (supersedes GH-3 and GH-7's palette work).
- `index.html` — copy rewrite (both `data-i18n` source strings), hero/programs/testimonials structural changes for the copybook layout, font `<link>` tags.
- `script.js` — replaces GH-7's per-section `initScrollReveal()` with two targeted animations (`initHeroBaseline()`, `initProgramsPath()`); i18n dictionary strings updated to match the new copy; form/nav/carousel logic unchanged.

## Verification

- Design-review/Polish pass: screenshots at mobile/tablet/desktop, both languages/directions, checked against this plan.
- Re-run static checks: HTML parse, `node -c script.js`, i18n key parity, WCAG AA contrast on the new palette.
- Confirm reduced-motion and no-JS fallbacks per the motion budget above.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Author |
