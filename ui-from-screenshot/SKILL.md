---
name: ui-from-screenshot
description: Build a React Native / Expo (or web) UI from a screenshot, mockup, Figma export, or reference image — using the TARGET project's own primitives, token seam, and verification harness, then rendering it back to compare against the source. Use whenever a visual reference is the spec. Triggers include "build this screen", "make a component from this image", "here's a screenshot", "recreate this design", "turn this mockup into a screen", "implement this UI", "match this design", "code this from the image". Discovers the project's typography primitive, colour-token seam, press primitive, and render/screenshot harness FIRST, maps the image to those (never raw hex), then closes the loop by rendering and reading the screenshot back.
---

# Build UI from a screenshot (any RN / web project)

A screenshot is a *spec*, not paint-by-numbers. A good result matches the
reference AND obeys the target project's conventions, so it themes, scales, and
stays consistent. Hand-rolling pixels straight from the image is the failure mode.

## Step 0 — Discover THIS project's conventions FIRST

Never assume. Find, before writing anything:

- **Token / colour seam** — a `tokens.*`, `theme.*`, tailwind config, or `colors.*`
  map. How is colour applied: semantic classes? a `useTheme()` hook? StyleSheet
  constants? Whatever it is, use it — not raw hex. Check for a no-raw-hex
  lint/script and respect it.
- **Typography primitive** — a `<Text variant=…>` / `<Typography>` / `<AppText>`
  with a locked scale, or bare RN `<Text>` + a fontSize scale. Reuse it.
- **Money / number primitive** — many apps have a `<Price>`/`<Currency>`/`<Amount>`
  with tabular figures + null handling. Find it; route values through it.
- **Press primitive** — a `PressableScale`/`Touchable`/`Pressable` wrapper with
  the house press feedback. Reuse it.
- **Component kit + barrel** — `components/ui`, a barrel `index.ts`, a Storybook.
- **Verification harness** — Playwright / Detox / Storybook / `expo start --web`.
  This is how you close the loop in Step 4.

Read the project's CLAUDE.md / README / a couple of existing screens to absorb the
patterns. Match what exists; do not import a different styling philosophy.

## Step 1 — Read the image like a spec

Inventory, explicitly: layout regions top→bottom; every text run by its ROLE (not
size); money values; interactive elements; colour by MEANING (surface vs card vs
field; primary/secondary/tertiary text; semantic accents); spacing rhythm.

## Step 2 — Map to the discovered primitives

Text → the typography primitive (by role). Money → the money primitive. Surfaces /
pills / taps → the kit's components. Colour → the token seam (semantic), never
sampled hex. Sizes → the locked scale (extend it once + document if a step is
genuinely missing).

## Step 3 — Missing a primitive? Scaffold it

Use the `ui-scaffold` skill to add it in the project's own pattern, rather than
inlining a bespoke block.

## Step 4 — Close the loop (this is what makes screenshot→code work)

Render it and look. Wire it into a route/screen, run the project's render harness,
**read the produced screenshot back**, and compare to the source: hierarchy,
spacing, colour roles, alignment. Iterate. If the project has no render harness,
set up the simplest one that screenshots a route (e.g. Playwright against the web
target) — guessing from code alone is the bug. Then run the project's
typecheck / lint / test.

## Step 5 (complex screens) — Genius Panel

Before "done", a short multi-lens pass: fidelity vs source · convention compliance
· contrast/a11y (WCAG on the pairs used, ≥44px targets). Fix what any lens flags.

## Pitfalls

- Sampling hex from the image instead of using the token seam (breaks theming /
  trips a no-raw-hex guard).
- Importing a styling approach the project doesn't use (`dark:` variants, inline
  styles, a new lib) instead of matching the existing one.
- A text primitive holding a price (loses tabular figures + null handling).
- A pinned brand background with scheme-reading children (invisible labels in the
  other scheme).
- Declaring done without rendering + eyeballing the screenshot.
