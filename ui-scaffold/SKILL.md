---
name: ui-scaffold
description: Scaffold a new UI component in any React Native / web project, matching that project's OWN component pattern — its variant library, token seam, primitive set, ref convention, barrel export, and test/verify harness. Use when adding a reusable UI piece. Triggers include "scaffold a component", "add a component to the kit", "new primitive", "create a Badge/Banner/Toggle/etc", "new UI-kit component", "make a reusable X". Discovers the existing convention first (cva? tailwind-variants? StyleSheet?), then generates a component that fits in rather than introducing a new style.
---

# Scaffold a UI component (any project)

## Step 0 — Learn the project's component pattern FIRST

Open 2–3 existing components in the kit and extract the recipe:

- **Where** kit pieces live (`components/ui`, `components/common`, …) and the
  **barrel** they export from.
- **Variant mechanism** — `cva`, `tailwind-variants`, a props→style map, or
  StyleSheet. Use the same one.
- **Token / colour seam** — semantic classes, a `useTheme()` hook, a `colors.*`
  map. Apply colour through it; never raw hex if the project forbids it (check for
  a no-raw-hex lint/script).
- **Primitives** — the typography (`<Text variant>`), money (`<Price>`), and press
  (`<PressableScale>`) components to compose with.
- **Ref convention** — does it `forwardRef`? Match it for pressables/inputs.
- **Tests / render harness** — a `*.test.tsx`, Storybook story, or render-smoke to
  add alongside.

## Decide: shared primitive vs feature component

Reusable + presentational + no data → the shared kit. Composes data + domain logic
→ a feature/screen-local component. When unsure, keep it feature-local until a
second consumer appears.

## Recipe

1. File with a header comment stating purpose + any contract.
2. `<Name>Props` interface; variants via the project's variant lib; `forwardRef`
   if the convention calls for it.
3. Colour / spacing / type via the project's seam and scale — extend the scale
   once (documented) rather than inlining one-offs.
4. Compose existing primitives; don't re-roll text / money / press.
5. Export from the barrel.
6. Add the project's flavour of test / story / render-check.

## Verify

Run the project's typecheck + lint + tests, and render the component (Storybook /
web / smoke) at least once — read the output, don't assume.

## Pitfalls

- Introducing a new styling approach instead of matching the existing one.
- Forgetting the barrel export.
- Raw hex where the project mandates tokens.
- A bespoke pressable that drops the house press / reduce-motion feedback.
- Business logic in a presentational primitive.
