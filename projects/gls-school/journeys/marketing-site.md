# Journey Preview: GLS Marketing Site

**Role**: UX Designer (journey) + UI Designer (visual direction)
**Source PRD**: [../prds/marketing-site.md](../prds/marketing-site.md)

## Journey (single page, top to bottom / nav-jump)

```
Header/Nav (sticky) ── language toggle always reachable
   |
   v
Hero ── tagline + CTA -> jumps to Registration
   |
   v
About ── mission, founded 2016, quick facts
   |
   v
Programs/Stages ── KG / Primary / Preparatory / Secondary cards
   |
   v
Why Choose Us ── teachers, safety, bilingual curriculum, activities, communication
   |
   v
Gallery ── placeholder image grid
   |
   v
Testimonials ── 2-3 placeholder quotes
   |
   v
Registration form
   |
   +--> [empty state] fields blank, submit disabled/validated on submit
   +--> [error state] required field missing -> inline message, focus returns to field
   +--> [success state] mailto: opens + on-page confirmation banner (in case no mail client)
   |
   v
Contact & Map ── phone (click-to-call), email, WhatsApp, address + map iframe
   |
   v
Footer
```

## Visual direction (UI Designer)

- Palette: soft blues + warm yellow accent — friendly, calm, appropriate for a children's school; avoids corporate/cold tones.
- Typography: rounded/friendly sans-serif that has solid Arabic glyph support (e.g. a system font stack covering Arabic + Latin) so both languages feel native, not like an afterthought translation.
- RTL is the default reading direction, not a toggled edge case — spacing, icon direction (arrows, nav chevrons), and card layouts must mirror correctly, not just the text.
- Placeholder imagery (gallery, hero) uses flat CSS/SVG illustrations rather than stock photography, so it's obviously provisional and not mistaken for real school photos.

## Exit

Reviewed against PRD user stories US-1/US-2; no missing states identified beyond what's captured above. Proceed to Phase 2 (Technical Design).
