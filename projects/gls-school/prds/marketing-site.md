<!-- Source: ApexYard · templates/prd.md · github.com/me2resh/apexyard · MIT -->

# PRD: GLS Bilingual Marketing & Admissions Site

**Status**: Approved
**Author**: Product Manager
**Created**: 2026-07-02
**Last Updated**: 2026-07-02

---

## Overview

### Problem Statement

Glories Language School (GLS), a K-12 private school in Sheikh Zayed / 6th October City, has no web presence beyond a sparsely-filled Facebook page. Mothers researching schools for their children have no dedicated place to learn about GLS's programs, values, and stages, or to register interest — they either don't find the school or leave without a way to follow up.

### Target User

**Primary**: Mothers of children aged Kindergarten through Secondary, evaluating schools in the Sheikh Zayed / 6th October area, browsing primarily on mobile, comfortable reading Arabic first and English second.
**Secondary**: English-speaking or bilingual parents/relatives who prefer the English version.

### Goals

1. Present GLS's programs, mission, and facts clearly enough that a mother can decide to inquire, in both Arabic and English.
2. Let a mother submit an admissions-interest registration (parent + child details) directly from the site.
3. Ship as a static, dependency-free site that can be hosted anywhere at zero cost (GitHub Pages).

### Non-Goals (Out of Scope)

- Backend/CMS, database, or authenticated portal
- Online payments or tuition processing
- Real photography, logo, or verified testimonials (placeholders only, clearly swappable)
- Multi-page routing/build tooling (single page is sufficient for this scope)

### Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
| Bilingual correctness | 100% of visible strings translate and layout mirrors (RTL/LTR) with no toggle | Manual QA pass |
| Mobile usability | All sections usable at 360px width | Manual QA pass |
| Form completion | A mother can fill and submit the registration form, producing a pre-filled email to the school | Manual QA pass |

---

## User Stories

### US-1: Learn about the school in my language
> As a mother browsing in Arabic, I want to read about GLS's programs and values in Arabic by default, so that I don't have to fight an English-first site.

**Acceptance Criteria**:

- [ ] Page loads with Arabic text and RTL layout by default
- [ ] A visible toggle switches to English/LTR and back, persisting across visits
- [ ] No mixed-direction or untranslated strings after toggling

### US-2: Register interest for my child
> As a mother who likes what she sees, I want to submit my and my child's details, so that the school follows up with me.

**Acceptance Criteria**:

- [ ] Form requires parent name, child name, child age/grade, and a phone or email
- [ ] Submitting with missing required fields shows inline validation, not a silent failure
- [ ] Successful submission opens a pre-filled email to the school with the entered details

### Edge Cases

| Scenario | Expected Behavior |
|----------|-------------------|
| Visitor on a narrow (360px) mobile screen | All sections readable, nav collapses to a mobile menu, no horizontal scroll |
| Visitor toggles language mid-form-fill | Entered values are preserved; labels/placeholders re-render in the new language |
| Visitor's device has no mail client configured | `mailto:` link still opens the OS "no app" fallback; form still shows a visible confirmation state so the visit isn't a dead end |

---

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-1 | Single-page site with Header/Nav, Hero, About, Programs, Why Choose Us, Gallery, Testimonials, Registration form, Contact & Map, Footer | Must | |
| FR-2 | AR/EN language toggle flips `dir`/`lang` and all visible text, persisted in `localStorage` | Must | |
| FR-3 | Registration form with client-side validation, submits via pre-filled `mailto:` | Must | No backend exists yet |
| FR-4 | Contact section with click-to-call phone, email, WhatsApp link, and an address-based Google Maps iframe | Should | |
| FR-5 | Placeholder gallery/testimonials clearly marked as placeholder in code (easy find/replace) | Should | |

**Priority Key**: Must (required for launch) | Should (important) | Could (nice to have)

### Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | No external framework/CDN dependency | 0 render-blocking third-party JS |
| Accessibility | Semantic HTML, sufficient color contrast | WCAG AA-ish (manual check, no automated audit in this scope) |
| Compatibility | Works with zero build step, opened directly as a file | Loads correctly via `file://` and any static host |

---

## Design

### User Flow

```
[Landing on Hero]
    |
    v
[Scrolls / navs through About -> Programs -> Why Us -> Gallery -> Testimonials]
    |
    v
[Reaches Registration form]
    |
    +---> [Fills required fields, submits] --> [mailto: opens, pre-filled] --> [Confirmation state shown]
    |
    +---> [Leaves fields blank, submits] --> [Inline validation errors shown]
```

### Wireframes / Mockups

None — see the Journey Preview (Phase 1.5) in `projects/gls-school/journeys/marketing-site.md` for the flow description; no dedicated visual mockups given the small scope.

---

## Technical Notes

### Dependencies

| Dependency | Type | Status | Owner |
|------------|------|--------|-------|
| Real logo/photos/testimonials from the school | External | Blocked (placeholder used) | School |
| Verified school address for map embed | External | Ready (from public listing) | — |

### Technical Constraints

- No backend/server available at launch — registration form must work client-side only (`mailto:`).
- No build tooling — plain HTML/CSS/JS only.

---

## Launch Plan

### Rollout Strategy

- [x] All users at once (single static page, no phased rollout needed)

---

## Open Questions

| Question | Owner | Status | Resolution |
|----------|-------|--------|------------|
| Real school logo/branding colors | School | Open | Using a placeholder soft blue/yellow palette until provided |
| Preferred form backend once available (Formspree, custom API, etc.) | Tech Lead | Open | `mailto:` is the interim solution; swap-in point documented in code |

---

## Timeline

| Milestone | Target Date | Status |
|-----------|-------------|--------|
| PRD Approved | 2026-07-02 | Done |
| Design Complete | 2026-07-02 | Done |
| Dev Complete | 2026-07-02 | In Progress |
| QA Complete | TBD | Pending |
| Launch | TBD | Pending |

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Product Manager | Claude (as PM) | 2026-07-02 | Author |
| Tech Lead | Claude (as Tech Lead) | 2026-07-02 | Approved |
