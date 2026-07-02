---
name: vertical-slice
description: Build a feature as a complete vertical slice across a layered codebase, following the project's OWN conventions rather than a generic template. Use when adding or extending a feature that spans multiple layers — a new resource, endpoint, screen, or data flow from storage to UI. Triggers include "add a feature", "new endpoint", "new screen", "full stack", "vertical slice", "wire up end to end", "scaffold a resource". Encodes the discipline of mirroring the nearest existing slice and wiring every cross-cutting concern (auth/scoping, authorization, validation, audit/logging, cache invalidation, error handling, tests) that compiles fine when missing. Employs a Genius Panel methodology.
---

# Vertical Slice Builder

Build a feature the way *this* codebase already builds features — same layers, same names, same side-channels — instead of importing a generic scaffold that fights the existing conventions. A feature is rarely one file; it's a slice cutting through storage → data access → business logic → transport → client state → UI, plus the cross-cutting concerns each layer owes the others.

> This skill is about *fit and completeness*, not framework tutorials. For "how does library X work," use that ecosystem's docs/skill. For "how do *we* add a thing here," use this.

## Core Principle: Copy the Nearest Sibling

Before writing anything, find the most similar feature that already exists and **mirror its structure**. A mature codebase has a house style — layer names, file layout, error types, test shape. The closest sibling is the spec.

1. Identify the slice's layers in this project (e.g. model/entity → repository/DAO → service/use-case → controller/endpoint → client API → state/query hook → view).
2. Open a sibling that spans all of them.
3. Make your new code look like it — if it doesn't, you've diverged; justify it or fix it.

This beats any generic template because it inherits decisions already made and reviewed: naming, validation placement, auth seam, error contract.

## Core Principle: The Slice Includes the Side-Channels

A slice that returns the right data on the happy path can still be wrong. The expensive defects live in the cross-cutting wiring that *compiles fine when absent*:

- **Auth & scoping** — is the resource correctly scoped to a tenant / user / workspace? Is the protected route actually protected?
- **Authorization** — can the wrong role mutate it? Read-only users must be blocked at the layer that owns the rule, not just hidden in the UI.
- **Validation** — inputs validated at the boundary, with the project's existing validation mechanism.
- **Audit / logging / events** — does this project emit domain events or audit entries on create/update/delete? Match siblings.
- **Cache / state invalidation** — after a write, what reads go stale? Invalidate every dependent view (lists, detail, dashboards, counts).
- **Error handling** — every mutation path surfaces failure the project's way (toast, error boundary, problem+json), not a raw throw.
- **Tests** — at minimum: auth/authz failure, the happy path, and the primary error path.

## The Genius Panel

Convene before scaffolding a non-trivial feature:

```
🏛️ GENIUS PANEL CONVENED: [Feature]

ARCHITECT: Which existing slice is the closest sibling? New resource
  (entity + migration) or extension of an existing one? Which layers does
  it touch?
SECURITY: How is this scoped (tenant/user/role)? Where is the auth seam,
  and where does authorization actually get enforced?
DATA: What reads go stale on write — which caches/queries invalidate?
  Does anything need an offline/optimistic path? Any new index?
INTERFACE: What does the client/UI owe — loading, empty, error states?
  Does a write surface respect role and connectivity?
PRAGMATIST: Smallest slice that ships value. What's deferrable?

CONSENSUS: [sibling to copy + the side-channels this slice must wire]
```

## Workflow

1. **Locate conventions** — read the closest sibling slice top to bottom; note its layers, error types, test layout, and how it registers itself (router include, DI container, route table, export barrel).
2. **Panel** for anything non-trivial.
3. **Build inward-out or storage-up**, one layer at a time, mirroring the sibling: data model (+ migration if needed — consider a migration-discipline pass) → data access → business logic + authorization → transport → register the route/handler → client API → state/query → UI with loading/empty/error.
4. **Wire the side-channels** from the checklist above.
5. **Test** the auth, happy, and error paths the project's way.
6. **Self-review** against the sibling — does anything look out of place?

## The Completeness Checklist

- [ ] Mirrors the nearest sibling's structure and naming.
- [ ] Protected routes are actually protected; resource correctly scoped.
- [ ] Writes enforce authorization at the owning layer (not just UI).
- [ ] Inputs validated at the boundary.
- [ ] Audit/events emitted on CRUD if the project does that.
- [ ] Handler/route registered where the project registers them.
- [ ] Client write-paths invalidate every stale read.
- [ ] UI handles loading / empty / error.
- [ ] Tests cover authz failure + happy path + primary error.
- [ ] Lint + type-check clean.

## When NOT to use this skill

- A single-file change with no cross-layer impact — just make the edit.
- Framework/library learning questions with no project convention at stake — use that ecosystem's reference.
- Pure schema/migration work — use a migration-discipline skill.
- Verifying an already-built slice before a PR — use a pre-PR audit skill.
