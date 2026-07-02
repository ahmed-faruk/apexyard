<!-- Source: ApexYard · templates/technical-design.md · github.com/me2resh/apexyard · MIT (adapted: static frontend, no API/DB sections) -->

# Technical Design: GLS Bilingual Marketing Site

**Status**: Approved
**Author**: Tech Lead
**Date**: 2026-07-02
**PRD**: [../prds/marketing-site.md](../prds/marketing-site.md)

---

## Overview

### Summary

A single-page, dependency-free bilingual (AR/EN) static site (`index.html` + `styles.css` + `script.js`) that presents Glories Language School's programs and lets a mother submit an admissions-interest form via a client-side `mailto:` fallback (no backend exists yet).

### Goals

- Zero build step, zero third-party dependency, hostable anywhere static files are served.
- Arabic-first, fully mirrored RTL layout with an English toggle, not a bolted-on translation.
- Client-side form that degrades gracefully without a backend.

### Non-Goals

- No API, database, or authentication — none of the "Architecture Review Required" triggers (new service, integration, tech, data model, or perf-critical work) apply, so no Solution Architect review is needed for this design.
- No automated test suite (manual QA pass only, per PRD scope).

---

## Architecture

Trivial static-site architecture; ASCII sketch is sufficient (per template guidance, dedicated C4 templates are overkill here):

```
Browser
  └── index.html (semantic sections, data-i18n attrs)
        ├── styles.css   (mobile-first, CSS custom properties, logical properties for RTL)
        └── script.js
              ├── i18n module      { ar: {...}, en: {...} } keyed by data-i18n id
              │                    toggles document dir/lang, persists choice in localStorage
              ├── nav module       mobile menu toggle, smooth-scroll to anchors
              ├── carousel module  small vanilla-JS rotation for testimonials
              └── form module      client-side validation -> builds mailto: link -> shows
                                   confirmation banner (works even without a mail client)
```

### Data Flow

Registration form: user input → client-side required-field validation → on success, JS constructs a `mailto:gloriesschool@gmail.com?subject=...&body=...` URL from the field values and opens it, and shows an on-page confirmation banner regardless (covers the "no mail client configured" edge case from the PRD). No data is stored or transmitted anywhere else. Comment in `script.js` marks this as the swap-in point for a real backend (e.g. Formspree or a custom endpoint) later.

---

## Implementation Plan

### Files

| File | Purpose |
|------|---------|
| `index.html` | Semantic markup for all sections; both languages' strings via `data-i18n` keys, not duplicated markup |
| `styles.css` | Mobile-first responsive layout; CSS custom properties for palette/spacing; logical properties (`margin-inline-start`, etc.) so RTL mirrors correctly |
| `script.js` | i18n toggle, mobile nav, testimonial rotation, form validation + `mailto:` submission |
| `images/` | Placeholder SVG/CSS-drawn graphics only, named `placeholder-*`, so nothing is broken without real assets |

### Tasks

| # | Task | Ticket | Dependencies |
|---|------|--------|--------------|
| 1 | Semantic markup for all 10 sections (Header/Nav, Hero, About, Programs, Why Us, Gallery, Testimonials, Registration form, Contact & Map, Footer) with `data-i18n` keys | GH-2 | - |
| 2 | Styling: mobile-first layout, palette, RTL-aware logical properties | GH-3 | 1 |
| 3 | JS: i18n toggle + persistence, mobile nav, carousel, form validation + `mailto:` | GH-4 | 1 |
| 4 | Manual QA pass against PRD acceptance criteria | GH-5 | 1, 2, 3 |

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| RTL layout bugs (icons/spacing not mirroring) | Med | Med | Use CSS logical properties throughout instead of left/right; QA pass explicitly checks both directions |
| `mailto:` feels like a dead end on devices with no mail client | Med | Low | On-page confirmation banner shown regardless of whether the mail client opened |
| Placeholder content mistaken for real school info once deployed | Low | Med | Placeholder sections clearly commented in code; PRD open question flags real assets as still needed from the school |

---

## Security Considerations

- [x] No authentication/authorization surface (static site, no accounts)
- [x] No PII stored — form data only leaves the browser via the user's own `mailto:` action
- [x] No third-party scripts/CDNs to introduce supply-chain risk
- [ ] N/A: input validation at a server boundary (no server exists)

---

## Testing Strategy

| Type | Coverage | Notes |
|------|----------|-------|
| Manual QA | All PRD acceptance criteria (US-1, US-2) | Run directly in a browser, mobile + desktop widths, both languages |

---

## Open Questions

| Question | Owner | Status |
|----------|-------|--------|
| Real form backend once available | Tech Lead | Open — `mailto:` is the interim solution |

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Author |
