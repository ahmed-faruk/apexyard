<!-- Source: ApexYard · templates/technical-design.md · github.com/me2resh/apexyard · MIT -->

# Technical Design: Reorder Todos (Drag-to-Reorder)

**Status**: In Review
**Author**: Tech Lead (Hisham)
**Date**: 2026-07-02
**PRD**: ahmed-faruk/todo-app#13

---

## Overview

### Summary

Add manual drag-to-reorder for todos in the Flutter app. Requires a Drift schema migration (new `sortOrder` column) since the app currently has no persisted manual ordering — this is the app's first-ever schema change (`schemaVersion` has been `1` since inception, with no `MigrationStrategy` defined).

### Goals

- Users can drag-reorder todos in the unfiltered ("All") list view
- Order persists across app relaunch
- Existing users see no visual reshuffle on upgrade (backfill preserves current `createdAt desc` order)

### Non-Goals

- Reordering within a filtered (Active/Completed) view
- Custom sort modes (alphabetical, due-date, etc.)

---

## Domain Model

### Entities

```
Todo
├── id: int
├── title: String
├── isCompleted: bool
├── createdAt: DateTime
├── sortOrder: int          <- NEW
└── Methods:
    └── copyWith(...)       <- extended to include sortOrder
```

No new value objects or domain events — `sortOrder` is a plain field, consistent with the existing entity's flat shape.

---

## Architecture

This is a same-shape addition to the existing clean-architecture layering (domain → data → application → presentation); no new components or boundaries. ASCII fallback (trivial design, no new topology):

```
presentation/todo_screen.dart (ReorderableListView, filter-gated)
        │ calls
application/providers.dart (_TodoNotifier.reorder)
        │ calls
domain/todo_repository.dart (TodoRepository.reorder — new interface method)
        │ implemented by
data/drift_todo_repository.dart (transactional bulk sortOrder write)
        │ persists to
data/app_database.dart (TodoItems.sortOrder column, schemaVersion 1→2)
```

### Data Flow

`ReorderableListView.onReorder` fires with (oldIndex, newIndex) for the currently displayed list → UI computes the full new id order → `_TodoNotifier.reorder(orderedIds)` → repository writes sequential `sortOrder` values in one transaction → `watchAll()` stream (already subscribed by `todoListProvider`) re-emits in the new order → UI updates. No separate DFD needed; this is a single local read/write loop, no trust boundary crossed (fully local SQLite, no network/auth surface).

---

## API Design

N/A — local-only Flutter/Drift app, no network API surface for this feature.

---

## Data Model

### Database Schema (Drift `TodoItems` table)

| Field | Type | Key | Purpose |
|-------|------|-----|---------|
| id | INTEGER | Primary (autoincrement) | Unique identifier (unchanged) |
| title | TEXT | - | Todo text (unchanged) |
| isCompleted | BOOLEAN | - | Completion state (unchanged) |
| createdAt | DATETIME | - | Creation timestamp (unchanged) |
| **sortOrder** | **INTEGER** | - | **NEW.** Manual position, ascending. Default `0`; backfilled on migration. |

### Access Patterns

| Access Pattern | Query |
|----------------|-------|
| List all, in display order | `SELECT * FROM todo_items ORDER BY sort_order ASC` (was `created_at DESC`) |
| Insert new todo at top | `sortOrder = min(sortOrder) - 1`, computed in a transaction |
| Reorder | Transactional bulk `UPDATE todo_items SET sort_order = ? WHERE id = ?` for each id in the new order (0..n) |

### Migration (v1 → v2)

This is the app's **first-ever** Drift `MigrationStrategy`. Full detail (rollback plan, backfill approach) is owned by the migration ticket + migration AgDR (`/migration`), created as a follow-up to this design per the SDLC's Migration sub-workflow — not duplicated here. Summary for review purposes:

- `onCreate`: `m.createAll()` (unchanged for fresh installs)
- `onUpgrade` (`from < 2`): `addColumn(todoItems, todoItems.sortOrder)`, then backfill existing rows' `sortOrder` from their current `createdAt DESC` order, in a transaction, so no user-visible reshuffle happens on upgrade

---

## Implementation Plan

### Tasks

| # | Task | Estimate | Dependencies |
|---|------|----------|--------------|
| 1 | Migration ticket + AgDR (`/migration`) | 0.5h | - |
| 2 | Domain: add `sortOrder` to `Todo`, add `reorder()` to `TodoRepository` | 0.5h | - |
| 3 | Data: schema migration (`app_database.dart`) | 1.5h | 1 |
| 4 | Data: repository impl (`watchAll` reorder, `create` insert-at-top, `reorder()`) | 1.5h | 2, 3 |
| 5 | Application: `_TodoNotifier.reorder()` | 0.25h | 4 |
| 6 | Presentation: `ReorderableListView`, keying, filter-gating, drag handle | 2h | 5 |
| 7 | Tests (domain, data incl. migration, application, widget) | 3h | 2–6 |

**Total Estimate**: ~9.25h

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Migration backfill runs on wrong/stale order, users see a one-time reshuffle | Low | Low (cosmetic, local-only data) | Backfill explicitly orders by `createdAt DESC` (today's display order) before assigning `sortOrder`; covered by a migration test |
| `ReorderableListView` drag conflicts with existing tap-to-edit / checkbox / delete gestures | Med | Low (UX annoyance) | Explicit drag-handle icon via `ReorderableDragStartListener`, `buildDefaultDragHandles: false` |
| Reordering a filtered subset produces ambiguous global order | Med (if not addressed) | Med (data confusion) | Explicitly out of scope — drag handles hidden outside "All" filter |

---

## Security Considerations

N/A — local-only SQLite data, no network/auth surface, no PII beyond what's already stored (todo titles).

---

## Testing Strategy

| Type | Coverage | Notes |
|------|----------|-------|
| Unit (domain) | `Todo` construction/copyWith/equality incl. `sortOrder` | `test/domain/todo_test.dart` |
| Integration (data) | `reorder()`, `create()` insert-at-top, v1→v2 migration backfill, all against real in-memory Drift db (existing no-mock convention) | `test/data/drift_todo_repository_test.dart` |
| Integration (application) | `_TodoNotifier.reorder()` via `ProviderContainer` | `test/application/providers_test.dart` |
| Widget | Drag gesture reorders + persists; drag handles absent when filter != All | `test/presentation/todo_screen_test.dart` (must call existing `_drain(tester)` helper) |

---

## Open Questions

None — scope, migration approach, and UI/filter interaction were resolved during design.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Tech Lead | Hisham | 2026-07-02 | Author |
| Solution Architect | Tariq | | Pending — required (schema/data-model change) |
