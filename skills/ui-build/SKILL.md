---
name: ui-build
description: Use when adding a reusable UI component to a React Native / web project, building a screen or component from a screenshot, mockup, Figma export, or design reference, or restyling an existing screen to match project conventions. Triggers include "scaffold a component", "new primitive", "create a Badge/Banner/Toggle", "add to the UI kit", "new UI-kit component", "build this screen", "here's a screenshot", "recreate this design", "turn this mockup into a screen", "implement this UI", "match this design", "code this from the image", "restyle this screen", "make this screen match the app".
---

# Build UI in the project's own style

Two entry points, one discipline: whether scaffolding a kit component or
recreating a screenshot, the result is built from the TARGET project's own
primitives and tokens — never a freshly imported styling philosophy.

## Step 0 — Discover THIS project's conventions FIRST (both paths)

Open 2–3 existing components / screens, plus CLAUDE.md / README, and extract:

- **Token / colour seam** — `tokens.*`, `theme.*`, tailwind config, or a
  `colors.*` map; how colour is applied (semantic classes, `useTheme()`,
  StyleSheet constants). Check for a no-raw-hex lint/script and respect it.
- **Variant mechanism** — `cva`, `tailwind-variants`, a props→style map, or
  StyleSheet. Use the same one.
- **Primitives** — typography (`<Text variant>`), money (`<Price>`/`<Amount>`
  with tabular figures + null handling), press (`PressableScale` with the
  house feedback). Compose with them; never re-roll text / money / press.
- **Kit location + barrel** — `components/ui`, the `index.ts` barrel, Storybook.
- **Ref convention** — does the kit `forwardRef`? Match it for pressables/inputs.
- **Verification harness** — Playwright / Detox / Storybook / `expo start --web`
  / test files. This is how you close the loop at the end.
- **CalmMoney-family app?** calmmoney-design already documents the house style —
  load it instead of re-deriving these from samples.
- **Greenfield (nothing to discover)?** Create the seam first — react-native-skill
  for Expo/RN structure, `/impeccable init` for web — then return here.

## Path A — Scaffold a kit component

1. **Placement:** reusable + presentational + no data → shared kit. Composes
   data or domain logic → feature/screen-local. Unsure → feature-local until a
   second consumer appears.
2. File with a header comment stating purpose + any contract; a `<Name>Props`
   interface; variants via the project's variant lib; `forwardRef` if the
   convention calls for it. On web, interactive primitives use the native
   element (`<button>` / `<input>` / `<a>`, never div+onClick) and cover
   hover / `:focus-visible` / disabled.
3. Colour / spacing / type through the project's seam and scale — extend the
   scale once (documented) rather than inlining one-offs.
4. Compose existing primitives. Export from the barrel. Add the project's
   flavour of test / story / render-check.

## Path B — Build from a screenshot

The image is a *spec*, not paint-by-numbers.

1. **Read the image like a spec** — layout regions top→bottom; every text run
   by its ROLE (not size); money values; interactive elements; colour by
   MEANING (surface vs card vs field; primary/secondary/tertiary text;
   semantic accents); spacing rhythm.
2. **Map to the discovered primitives** — text → typography primitive by role;
   money → money primitive; surfaces / pills / taps → kit components; colour →
   token seam (semantic, never sampled hex); sizes → the locked scale.
3. **Missing a primitive?** Scaffold it via Path A rather than inlining a
   bespoke block.
4. **Close the loop** — wire it into a route/screen, run the render harness,
   READ the produced screenshot back, and compare to the source: hierarchy,
   spacing, colour roles, alignment. Iterate. No harness? Set up the simplest
   one that screenshots a route — guessing from code alone is the bug. On web,
   screenshot at a phone width (~375px) AND a desktop width — the mockup shows
   one viewport, not the layout's only size.
5. **Complex screens** — a short multi-lens pass before "done": fidelity vs
   source · convention compliance · contrast/a11y (WCAG on the pairs used,
   ≥44px targets). Fix what any lens flags.

## Verify (both paths)

Run the project's typecheck + lint + tests, and render the result at least
once — read the output, don't assume. Every interactive element exposes
role + name + state to screen readers (RN: `accessibilityRole` /
`accessibilityLabel` / `accessibilityState`; web: a native element or ARIA —
icon-only pressables always need a label) and ships a ≥44pt effective hit
area (padding or `hitSlop`) — both paths, not just complex screens.

## Pitfalls

- Introducing a styling approach the project doesn't use (`dark:` variants,
  inline styles, a new lib) instead of matching the existing one.
- Sampling hex from an image instead of using the token seam.
- Raw hex where the project mandates tokens; forgetting the barrel export.
- A text primitive holding a price (loses tabular figures + null handling).
- A bespoke pressable that drops the house press / reduce-motion feedback.
- A pinned brand background with scheme-reading children (invisible labels in
  the other scheme) — see the mobile-theme-parity skill.
- A full phone screen missing the safe-area wrapper (`SafeAreaView` /
  `useSafeAreaInsets`) or keyboard avoidance on TextInputs — screenshot
  harnesses render chrome-less and keyboard-down, so neither failure shows.
- A repeating list region rebuilt as `ScrollView` + `.map()` instead of the
  project's `FlatList`/`FlashList` primitive — every row mounts at once at
  real data sizes.
- Copying a mockup's placeholder-only inputs literally — fields still need a
  real label (web: `<label>`; RN: `accessibilityLabel` on the `TextInput`).
- A web screen sized with `100vh`, or a bottom bar without
  `env(safe-area-inset-bottom)` — content hides behind mobile browser chrome /
  the home indicator; use `dvh` + safe-area insets.
- iOS `shadow*` styles with no Android `elevation` (or cross-platform
  `boxShadow`) — cards render flat on Android; one platform's render doesn't
  verify the other.
- Fixed heights on text-bearing elements — clips under large Dynamic Type /
  browser zoom; use `minHeight` + padding.
- Business logic in a presentational primitive.
- Declaring done without rendering + eyeballing the output.
