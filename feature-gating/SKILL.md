---
name: feature-gating
description: Add or modify an entitlement gate — paywall, plan tier, feature flag, or quota — correctly and in one seam. Use when introducing a premium wall, gating a feature behind a plan/role/entitlement, enforcing a usage limit, or touching tier logic. Triggers include "paywall", "premium gate", "gate behind a plan", "feature flag", "entitlement", "usage limit", "quota", "tier check", "free vs paid". Encodes the single-seam rule, the owner/subject-resolution trap for shared resources (check who the resource belongs to, not always the caller), a consistent machine-readable error contract, defense-in-depth for raceable limits, and clean client-side wall wiring. Employs a Genius Panel methodology.
---

# Feature Gating

Add an entitlement gate (paywall / plan tier / flag / quota) that is **correct, centralized, and honest**. Gating is business logic and correctness logic at once: the wrong check either leaks a paid feature or wrongly blocks a customer who *should* have access. Most gating bugs come from scattering the decision across the code and from resolving the wrong subject.

## Core Principle: One Seam for the Decision

There must be exactly **one** place that answers "is this allowed?" — a single service/module/function the rest of the code calls. Never inline `if user.plan == 'pro'` at call sites.

- Centralizing keeps the rule (active status, expiry, grace period, flag rollout) in one place; when billing or flags change, one file changes.
- It makes the gate testable in isolation and auditable ("show me everything gated").
- If a seam already exists, extend it — do not start a second one.

## Core Principle: Resolve the Right Subject

The classic defect: gating a **shared/owned resource** against the *caller's* entitlement instead of the resource's. Pick deliberately:

| The feature is… | Check entitlement of… |
|---|---|
| Personal to the caller | the caller |
| Owned by a team/space/org/account | the **owner / billing subject** of that resource |
| Shared, where access flows from an owner | the owner — so collaborators inherit it |

> ⚠️ Checking the caller for an owned resource passes happy-path tests but wrongly blocks a free collaborator inside a paid owner's team. For anything tied to a shared subject, resolve that subject's entitlement, not the caller's. Resolving a missing subject should fail closed/not-found in an enumeration-safe way, not silently return "denied".

## Core Principle: Gate Honestly (the product rules)

1. **Never gate the core habit/loop.** The action users come for stays free; the wall goes one step beyond, where the user has already invested.
2. **Never gate data export / portability.** Users own their data; charging to get it out is hostile (and store-policy risky for paid apps). Gate *presentation* (styled reports), not the raw data.
3. **Walls fire at the natural wall**, with copy explaining what the user just hit and what unlocking does — not a generic "Go Pro" screen.
4. **Gate the conversion moment, not the adoption moment.** The free tier should be genuinely useful so the wall lands after value, not before it.

## Core Principle: Honest, Machine-Readable Failures

A gate must fail with a **distinct, machine-readable code** the client can branch on — not a generic 403 indistinguishable from a real permission error, and not a 500.

- Server: a dedicated status + code (e.g. HTTP 402 + `PREMIUM_REQUIRED`, or `FEATURE_LOCKED` / `QUOTA_EXCEEDED`). Keep "not entitled" distinct from "not authorized" (role) and "not found".
- Client: branch on the **code**, not the HTTP status, and route to the right wall/upsell.

## The Genius Panel

```
🏛️ GENIUS PANEL CONVENED: [The gate]

PRODUCT: Does this gate the core loop or one step beyond it? Is it data
  export? (If yes to either core-loop/export — reconsider.) Where's the
  natural wall?
SECURITY: Personal or owned-resource? If owned, resolve the OWNER/billing
  subject, not the caller. Any race (concurrent action near a limit)?
BACKEND: Whole-endpoint gate (a guard/dependency) or mid-flow check (call
  the seam inside the use-case)? What's the error code?
CLIENT: Which wall/upsell fires here? Does the client branch on the error
  CODE, not the status? Is entitlement state read from the source of truth,
  never set optimistically by the client?
PRAGMATIST: Single check or defense in depth? (A raceable limit needs a
  UX check AND a server-authoritative check under a lock/transaction.)

CONSENSUS: [subject, seam, where the check sits, error code, client wall]
```

## Recipe

**Whole-endpoint gate** — attach a guard/middleware/dependency that rejects with the entitlement code before the handler runs. Use when the endpoint exists only for entitled users.

**Mid-flow gate** — call the single seam inside the use-case and raise the entitlement error when it returns false. Use when the endpoint stays usable but one action is gated.

**Count/usage limits** — read the current count, compare to the limit, reject on overflow. Two rules:
- **Never gate bootstrap paths** the product guarantees (the first/default resource must always succeed).
- **Raceable limits need defense in depth:** a pre-check for UX *plus* a server-authoritative re-check inside a transaction/lock so concurrent requests can't both slip past.

**Client wall** — fire the upsell at the action the user attempted (submit, tap, limit-hit), with reason-specific copy. Read entitlement from the authoritative source (server/billing SDK), never set it client-side.

## Verification

- [ ] Decision goes through the single seam (no inline tier checks at call sites).
- [ ] Owned/shared resources resolve the owner/subject, not the caller.
- [ ] Not gating the core loop or data export.
- [ ] Distinct machine-readable error code; client branches on code, not status.
- [ ] Bootstrap/default paths bypass the gate.
- [ ] Raceable limits have a transactional/locked authoritative re-check.
- [ ] Tests: unentitled blocked, entitled allowed, and (for shared) a free member of an entitled owner allowed.

## When NOT to use this skill

- Building the non-gating parts of the feature — use a vertical-slice skill.
- Role/permission authorization (who-can-do-what within a tier) — that's a different axis from entitlement; keep the two error codes distinct.
- Billing-provider/store operator configuration (dashboards, products, webhooks setup) — that's ops, not code.
