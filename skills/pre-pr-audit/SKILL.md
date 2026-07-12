---
name: pre-pr-audit
description: Use AFTER building or changing a feature and BEFORE opening a pull request or merging, to check the change's cross-cutting wiring. Triggers include "ready for PR", "pre-PR check", "did I wire this up", "consistency check", "review my change before I push", "before I merge", "audit this feature", "check for missing wiring".
---

# Pre-PR Audit

Verify a change is **wired through every cross-cutting channel** before it goes to review. Code that type-checks and passes happy-path tests can still ship a route nobody guarded, a write that never invalidates its cache, a handler that was never registered, or a mutation that throws a raw error at the user. These don't fail the build — they fail in production. This audit catches them, at the pre-PR lifecycle moment.

This is *wiring completeness*, distinct from *logic correctness*. Run it alongside a code-review/security-review tool, not instead of one.

## Core Principle: Trust But Verify

A diff that *looks* complete is the most dangerous kind. **Never mark an item ✓ without opening the file and confirming.** Every pass is backed by a `file:line`; every fail by the line that should exist but doesn't. "I added it" is not evidence — the grep hit is.

Two failure directions, both common:
- **Looks present, is wrong** — a guard on the route but the wrong scope; an entitlement check against the caller instead of the owner; a cache key that won't actually invalidate.
- **Looks fine, silently missing** — the new handler exists but isn't registered in the router/route table; the mutation succeeds but never invalidates dependent reads; there's no test at all.

## Establish the Project's Norms First

This skill is generic, so before auditing, learn what "wired" means *here* by reading 1–2 existing, reviewed features. Note the project's conventions for: auth/guards, authorization enforcement layer, error surfacing, cache/state invalidation, route/handler registration, and test layout. Audit the change against *those*, not against a generic ideal.

## The Genius Panel

Each reviewer independently asks "what would *I* bounce this PR for?" against the verified code:

```
🏛️ GENIUS PANEL: [Change under review]

THE BACKEND REVIEWER: Are new/changed endpoints guarded and correctly
  scoped? Is authorization enforced at the owning layer? Is the handler
  actually registered? Are domain events/audit emitted like siblings?
THE SECURITY REVIEWER: Any entitlement/permission check — right subject,
  fails closed, distinct error code? Any injection/secret/PII leak in the
  new path?
THE CLIENT REVIEWER: Do mutations handle the error path the project's way?
  Do writes invalidate every stale read? Loading/empty/error states present?
THE TEST REVIEWER: Does a test exist covering authz failure + happy path +
  primary error? Does the suite actually pass in the project's runner?
THE PRAGMATIST: Which findings are real blockers vs nits? Shortest fix list?

CONSENSUS: [verdict + ordered fix list with file:line]
```

## Audit Checklist

Run each by reading/grepping the actual change. Record evidence; skip items that genuinely don't apply.

### Backend / server
1. **Protected routes guarded** — new/changed endpoints carry the project's auth guard and correct scoping. *Verify the handler signature/decorators.*
2. **Authorization enforced** — mutations check role/ownership at the owning layer, not just hidden in UI. *Grep the use-case/service.*
3. **Entitlement checks correct** — any tier/flag check uses the single seam and resolves the right subject (owner vs caller). *Grep for the seam calls.*
4. **Audit/events** — CRUD emits the events/audit entries siblings do.
5. **Handler registered** — the new route/handler is wired into the router/route table/DI container. *Open that file — top miss.*
6. **Migration present** — model/schema changes have a matching, reversible migration. *Check the migrations dir in `git status`.*
7. **Input validation** at the boundary, the project's way.

### Client / UI
8. **Mutation error handling** — every write surfaces failure the project's way (toast/boundary/problem+json), with graceful bail on expected codes.
9. **Cache/state invalidation** — writes invalidate every dependent read (lists, detail, counts, dashboards). *Open each success handler.*
10. **Query/cache keys** include all scoping dimensions (tenant/workspace/filter) so context switches refetch.
11. **UI states** — loading, empty, and error all handled.
12. **Branch on codes, not raw status**, where the project distinguishes error kinds.

### Hygiene
13. **Tests** exist covering authz failure + happy path + primary error, and **actually pass** in the project's runner/environment.
14. **Lint + type-check clean.**
15. **No secrets, debug logs, or commented-out code** left in the diff.

## Output Format

```markdown
# Pre-PR Audit — <change>

**Audited:** YYYY-MM-DD
**Verdict:** 🔴 NOT READY | 🟡 READY WITH NITS | 🟢 SHIP

## Blockers
Numbered. Each: what's missing/wrong, file:line (or "absent — expected in <file>"), the fix.

## Nits
Non-blocking polish.

## Verified Compliant
Numbered, each with file:line — the checks that genuinely pass.

## Panel Takes
One line each: Backend / Security / Client / Test / Pragmatist.

## Fix List (ordered)
Blockers first, by file.
```

## When NOT to use this skill

- While *building* the change — use a vertical-slice skill, then audit with this.
- Deep logic/correctness or vulnerability review — use a dedicated code-review / security-review tool (complementary; run both).
- Release/store-submission compliance — that's a broader, policy-scoped audit.
