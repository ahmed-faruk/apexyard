<!-- Source: ApexYard · templates/technical-design.md · github.com/me2resh/apexyard · MIT -->

# Design Spec: UX/UI Revamp — Design Tokens, Dark Mode, Material 3 Theming

**Status**: In Review
**Author**: UI Designer (Nour)
**Date**: 2026-07-02
**PRD**: ahmed-faruk/todo-app#17

---

## Overview

### Summary

Replace stock Material 3 defaults and inlined magic numbers in `todo_app` with a small, named token set (spacing, radius, color roles, type scale) sourced from Flutter/Material's own `ColorScheme`/`TextTheme`/`ThemeData` mechanisms — no third-party token package, no framework change. Adds proper dark mode support (currently entirely missing).

### Goals

- Every spacing/radius/color value in `lib/presentation/todo_screen.dart` traces to a named token, not a magic number
- Dark mode via `ThemeMode.system` + a real dark `ColorScheme`
- Component-level theming (`AppBarTheme`, `ListTileThemeData`, `SegmentedButtonThemeData`, `FilledButtonThemeData`, `CheckboxThemeData`) centralized in one place, not scattered per-widget overrides

### Non-Goals

- No new screens, no IA/flow changes (Iman/UX Designer not needed for this ticket)
- No departure from Material 3 as the design language
- No illustration assets — empty states get an icon, not custom artwork
- No golden/screenshot tests (see rationale in the parent ticket)

---

## Design Token Specification Format

### Spacing

Flutter has no native spacing-token primitive, so this introduces `AppSpacing` (`lib/theme/design_tokens.dart`) — plain `static const double` fields, the idiomatic Flutter equivalent of CSS custom properties:

```dart
class AppSpacing {
  static const xs = 4.0;   // tight inline gaps (icon-to-text)
  static const sm = 8.0;   // existing add-todo row internal gap
  static const md = 12.0;  // existing default padding (add-todo row, filter row)
  static const lg = 16.0;  // screen-edge padding, list-tile content padding
  static const xl = 24.0;  // empty-state icon-to-text gap
  static const xxl = 32.0; // reserved — not used in this pass, kept for future screens
}
```

Usage:
- `xs`/`sm`: inline icon/button gaps
- `md`: existing row padding (add-todo, filter segmented button) — matches current behavior exactly, this pass documents the value rather than changing it
- `lg`: list-tile content padding (currently default `ListTile` padding, ~16px — now explicit via `ListTileThemeData`)
- `xl`: empty-state icon-to-message gap (new — empty states currently have no icon)

### Radius

```dart
class AppRadius {
  static const sm = 8.0;   // TextField / SegmentedButton corner radius
  static const md = 12.0;  // FilledButton corner radius (M3 default is close to this; making it explicit)
}
```

### Color: Roles (via `ColorScheme.fromSeed`)

No raw hex values — both light and dark schemes derive from the existing seed (`Colors.indigo`), per M3's algorithmic role generation. This spec maps roles to usage, not values (the seed algorithm computes actual hex per-brightness):

- `colorScheme.surface` — `Scaffold` background
- `colorScheme.surfaceContainerHighest` — empty-state icon container background (subtle, not `surface`-flat)
- `colorScheme.outline` / `colorScheme.outlineVariant` — `TextField` border, any future `Divider`
- `colorScheme.primary` / `colorScheme.onPrimary` — `FilledButton`, active `SegmentedButton` segment, drag-handle icon accent
- `colorScheme.error` — delete `IconButton` (currently unstyled default red; routed explicitly)
- `colorScheme.onSurfaceVariant` — empty-state icon + message text (de-emphasized, not full-contrast `onSurface`)

Usage:
- Every color reference in `todo_screen.dart` should read `Theme.of(context).colorScheme.<role>`, never a raw `Colors.*` constant, except the seed itself (`Colors.indigo`, unchanged, lives only in `app_theme.dart`)

### Type Scale

Reuses Material 3's own `TextTheme` slots — no new named tokens:

| Slot | Usage |
|---|---|
| `titleLarge` | AppBar title ("ToDo") |
| `titleMedium` | Todo item title text |
| `bodyMedium` | Empty-state message, secondary text |
| `labelLarge` | `SegmentedButton`/`FilledButton` labels |

Completed-todo strikethrough is a **computed variant**, not a new token: `titleMedium.copyWith(decoration: TextDecoration.lineThrough, color: colorScheme.onSurfaceVariant)`.

### Elevation

**Deliberately not expanded.** This is a flat, single-list screen with no cards/modals/dialogs needing elevation tiers. M3's `AppBar` defaults (`elevation: 0`, `scrolledUnderElevation: 1`) are already correct — do not introduce a 3-tier elevation system for one screen. Flagging this explicitly so it isn't read as a gap during review.

### Dark Mode

`ColorScheme.fromSeed(seedColor: Colors.indigo, brightness: Brightness.dark)` for the dark variant. The only two non-stock composed styles need a manual contrast check (everything else inherits M3's pre-vetted role pairings):
1. Completed-todo strikethrough color (`onSurfaceVariant` on `surface`) — passes AA at both brightness levels per M3's seed-generation contrast guarantees.
2. Delete-icon color (`error` on `surface`) — same guarantee applies; M3's `error` role is contrast-checked against `surface` by construction.

---

## Component Specification Format

### Component: AppBar

- Background: `colorScheme.surface` (M3 default, explicit via `AppBarTheme`)
- Elevation: `0`, `scrolledUnderElevation: 1` (unchanged from M3 default, made explicit)
- Title style: `titleLarge`

### Component: List Tile (todo row)

- Content padding: `EdgeInsets.symmetric(horizontal: AppSpacing.lg)` via `ListTileThemeData` (centralized, replaces any per-tile override)
- Title style: `titleMedium`, strikethrough variant when completed (see Type Scale above)
- Icon color (delete button): `colorScheme.error`
- Icon color (drag handle): `colorScheme.primary` (subtle accent, signals interactivity without overpowering the row)

### Component: SegmentedButton (filter)

- Selected segment: `colorScheme.primary`/`colorScheme.onPrimary`
- Label style: `labelLarge`
- Corner radius: `AppRadius.sm`

### Component: FilledButton (Add)

- Background: `colorScheme.primary`
- Label style: `labelLarge`
- Corner radius: `AppRadius.md`

### Component: Empty State (new structure)

- **Variants**: "No todos yet." (all), "No active todos." (active filter), "No completed todos." (completed filter) — copy unchanged from current implementation
- **Structure**: `Column` — `Icon` (size 48, `colorScheme.onSurfaceVariant`) → `SizedBox(height: AppSpacing.xl)` → existing `Text` widget (`bodyMedium`, `onSurfaceVariant`)
- **Icon choice per variant**:
  - All-empty: `Icons.checklist_outlined`
  - Active-empty: `Icons.task_alt_outlined`
  - Completed-empty: `Icons.task_alt_outlined` (or a distinct glyph if Frontend Engineer finds one that reads more clearly at implementation time — non-blocking choice)
- **Constraint (binding on implementation)**: the icon must be added as a **sibling** to the existing `Text` widget, not a wrapper that changes the widget's position in the render tree — this keeps `find.text('No todos yet.')` etc. valid in `test/presentation/todo_screen_test.dart` without requiring test changes

### States

Not applicable beyond Flutter/Material's built-in state layers (hover/pressed/focused/disabled) — M3 components handle these automatically via `ColorScheme`; no custom state styling introduced in this pass.

---

## Architecture

No new architectural boundary — `lib/theme/` is a new leaf directory with zero dependencies on domain/data/application layers (pure presentation-layer concern, consistent with clean-architecture rules already followed elsewhere in this codebase).

```
lib/theme/design_tokens.dart   (AppSpacing, AppRadius — no Flutter Material import needed beyond types)
lib/theme/app_theme.dart       (AppTheme.light(), AppTheme.dark() — builds ThemeData from tokens)
lib/main.dart                  (wires theme/darkTheme/themeMode)
lib/presentation/todo_screen.dart  (consumes Theme.of(context), AppSpacing/AppRadius — no direct token duplication)
```

---

## Implementation Plan

### Tasks

| # | Task | Estimate |
|---|------|----------|
| 1 | `.claude/project-config.json` — `.ui_paths` gate fix (first commit) | 0.1h |
| 2 | `lib/theme/design_tokens.dart` | 0.3h |
| 3 | `lib/theme/app_theme.dart` (light + dark `ThemeData`) | 1h |
| 4 | `lib/main.dart` wiring | 0.1h |
| 5 | `lib/presentation/todo_screen.dart` — token references + empty-state icons | 1h |
| 6 | Verify existing tests still pass; add lightweight dark-mode smoke assertion if desired | 0.5h |

**Total Estimate**: ~3h

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Empty-state icon wrapper breaks existing `find.text(...)` widget test finders | Med (if implemented carelessly) | Low (test failure, caught by CI) | Binding constraint above: icon must be a sibling, not a wrapper |
| Dark mode contrast issue on the two composed (non-stock) color usages | Low | Low (cosmetic) | Manual spot-check during QA phase; M3's seed algorithm contrast-guarantees the underlying roles |
| `.ui_paths` fix accidentally over-broad (matches generated `.g.dart` files, unnecessarily gating those PRs too) | Low | Low (extra review friction, not a correctness bug) | Accepted trade-off — `.g.dart` files are regenerated from `.dart` source changes anyway, so gating them is redundant-but-harmless, not wrong |

---

## Testing Strategy

| Type | Coverage | Notes |
|------|----------|-------|
| Existing widget/unit tests | Must remain green unchanged | No color/padding pixel assertions exist today; `ThemeData` changes are test-safe by construction |
| New (optional) | `AppTheme.dark().brightness == Brightness.dark` smoke assertion | Lightweight, non-golden — recommended over screenshot tests for this pass |
| Golden/screenshot | **Not introduced** | Maintenance overhead (font-rendering platform variance) disproportionate to a first token pass with no prior visual baseline to protect |

---

## Open Questions

None — token set, color-role mapping, and the empty-state sibling-widget constraint were resolved during spec authoring.

---

## Addendum v2: 2026 Mobile Design Trends

**Status**: In Review
**Author**: UI Designer (Nour)
**Date**: 2026-07-02
**Trigger**: user feedback that v1 (tokens + dark mode, same palette/layout) read as visually unchanged; direction expanded to apply current mobile-design-trend patterns, researched via web search.

### Research sources

- Calm, minimal, low-visual-noise UX — [UXPilot: 9 Mobile App Design Trends for 2026](https://uxpilot.ai/blogs/mobile-app-design-trends), [Envato: UX/UI design trends for 2026](https://elements.envato.com/learn/ux-ui-design-trends)
- Bottom-anchored primary actions (thumb-reach) — [Muzli: What's Changing in Mobile App Design](https://muz.li/blog/whats-changing-in-mobile-app-design-ui-patterns-that-matter-in-2026/)
- Card-based blocks, soft/organic "squircle" corner radii — [Midrocket: UI Design Trends for 2026](https://midrocket.com/en/guides/ui-design-trends-2026/), [WriterDock: Bento Grids & Beyond](https://writerdock.in/blog/bento-grids-and-beyond-7-ui-trends-dominating-web-design-2026)
- Subtle glassmorphism, used for hierarchy not decoration — [Lucky Graphics: UI Design Trends 2026](https://lucky.graphics/learn/ui-design-trends-2026/)
- Microinteractions as baseline expectation — [UXPilot: 9 Mobile App Design Trends for 2026](https://uxpilot.ai/blogs/mobile-app-design-trends)

### Decision

Keep the v1 token/dark-mode foundation (`ColorScheme.fromSeed`, indigo brand seed, `AppSpacing`/`AppRadius`) — it's still correct infrastructure regardless of visual style — and layer the following on top, translated to concrete, Flutter-feasible, single-screen-appropriate changes (no bento grids, no kinetic typography, no heavy glass blur — those don't fit a single-list utility app):

### New tokens

Add to `AppRadius` (`lib/theme/design_tokens.dart`):
- `lg = 20.0` — todo-card corner radius ("squircle"-ish via a large consistent radius)
- `pill = 999.0` — fully-rounded controls (segmented button, add-bar button/field)

### Component: Bottom Add-Todo Bar (new — replaces the top add-todo row)

- Position: pinned above the keyboard/safe-area at the bottom of the screen, not in the top `Column`
- Structure: unchanged widgets (`TextField` + `FilledButton`), same `AppSpacing.md`/`sm` padding/gap — only the position changes
- Shape: `TextField` border and `FilledButton` both use `AppRadius.pill`
- Rationale: this is the most frequent user action; 2026 mobile-nav trend places frequent actions within thumb reach at the screen bottom. Filter control (infrequent, contextual) stays at the top — not moved, avoids scope creep into a bottom-sheet filter pattern

### Component: Todo Card (replaces flat `ListTile` row styling)

- Shape: `RoundedRectangleBorder(borderRadius: BorderRadius.circular(AppRadius.lg))`
- Background: `colorScheme.surfaceContainer` (tonal, distinct from `colorScheme.surface` screen background) — this is the "subtle glassmorphism-adjacent" depth cue without introducing actual blur/translucency, keeping legibility and avoiding the "style over substance" failure mode the research flags for over-applied glassmorphism
- Spacing: `AppSpacing.sm` vertical gap between cards (visible separation, not edge-to-edge dividers)
- Content: unchanged (checkbox, title, delete icon, drag handle) — only the container changes

### Component: Pill Controls

- `SegmentedButton` (filter) and the bottom bar's `FilledButton`/`TextField`: `AppRadius.pill` corner radius via `ButtonStyle.shape`/`InputDecoration.border`

### Microinteractions (scoped to what's cheap and load-bearing, not decorative excess)

1. **Checkbox toggle**: brief scale animation (`AnimatedScale`, ~150ms) + `HapticFeedback.lightImpact()` on toggle
2. **Completed-todo dim**: `AnimatedOpacity` to ~0.6 alongside the existing strikethrough (both signal completion, opacity adds a felt transition instead of an instant flip)
3. **New-todo entry**: fade+slide-in for a newly added card, implemented per-item (keyed `TweenAnimationBuilder`, not a full `AnimatedList` — avoids fighting `ReorderableListView`'s own animation machinery)
4. **Delete**: fade-out before removal where feasible within the existing `ListView`/`ReorderableListView` structure

### Explicitly out of scope for this addendum

- Bento-grid layout — wrong shape for a linear todo list
- Kinetic/animated typography — not appropriate for a utility app's task titles
- Heavy glass blur (`BackdropFilter`) — legibility risk flagged directly in the research; tonal `surfaceContainer` achieves the depth cue without it
- AI personalization, voice/biometric auth — out of scope for a local single-user app with no auth surface

### Accessibility check

Card `surfaceContainer` backgrounds must be spot-checked for contrast against `onSurface` text in both light and dark — same manual-check approach as v1's contrast spot-check, since `surfaceContainer` is one of M3's own contrast-vetted roles (not a custom color), so risk is low. Tap targets: cards must remain ≥48dp tall (checkbox + padding already ensures this, unaffected by the shape/background change).

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| UI Designer | Nour | 2026-07-02 | Author (v1 + v2 addendum) |
| Head of Design | Maha | 2026-07-02 | Approved direction (v1 + v2) — pending final `/approve-design` on the implementation PR |
