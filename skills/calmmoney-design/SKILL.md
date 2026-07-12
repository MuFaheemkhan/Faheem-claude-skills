---
name: calmmoney-design
description: Use when building or styling UI for a CalmMoney-family app (BugetPlanner, coffeeHunter, Halal_guide, Tradie-101, or any app using the calmmoney-ui kit), or when the user asks for the "CalmMoney look", "sage green theme", "house style", or "Serene Wealth" aesthetic. Not for generic mobile work — use ui-build or react-native-skill for that.
---

# CalmMoney house design system

The sage-green, Apple-HIG, "Serene Wealth" aesthetic shared across the
CalmMoney-family apps.

## Source of truth — never re-declare tokens

1. **The `calmmoney-ui` package** (`D:\projects\active\calmmoney-ui`) — the
   extracted UI kit: themed components, Reanimated micro-interactions, a
   configurable palette. Prefer installing/vendoring it.
2. **Apps that predate the kit** (BugetPlanner, coffeeHunter, Halal_guide):
   `mobile/src/theme/{colors,typography,spacing,shadows}.ts` is that app's
   source of truth. Import from it; never inline hex in screens.

If a value below disagrees with the app's theme module, the theme module wins.

## Brand essentials

- **Palette** — sage primary `#8BA888` (light `#A8C4A5`, dark `#6B8B69`, deep
  `#4A6B48`); surfaces mint `#E8F0E8`, cream `#FAF9F7`, warm `#F5F0EB`;
  accents gold `#D4A574`, coral `#E8A598`, lavender `#B8A8C8`; income
  `#4CAF7A`, expense `#E07A5F`.
- **Typography** — dual-font: Fraunces (serif) for display/titles/currency,
  Plus Jakarta Sans for body/UI. Currency renders in the display face, large
  (42/50, semibold 28/36 for small).
- **Spacing** — 4px grid; screens 20 horizontal / 24 vertical; touch targets
  ≥44px, inputs and primary buttons 52px; radius scale 4→24 plus full.
- **Shadows** — soft, low opacity (0.04–0.16), platform-aware card shadow.
- **Motion** — Reanimated springs: gentle (cards/modals), snappy (buttons —
  press scale 0.97–0.98 + light haptic), bouncy (playful), smooth (subtle).
  Entrances fadeInUp / scaleIn with 50–120ms stagger.
- **Principles** — Apple HIG clarity / deference / depth, plus calm premium
  restraint: the UI defers to content.

## Component patterns

Reference implementations — Button with haptic press-scale, Card, FloatingTabBar
with centre FAB, glass-effect dashboard header — live in
`references/apple-design-patterns.md` and in each app's
`mobile/src/components/ui/`. Compose those; don't re-roll them.

## Rules

- Apply colour through the theme module / kit seam, never raw hex in screens.
- Brand-pinned surfaces (cream/warm backgrounds) must scheme-lock their
  subtree — run the mobile-theme-parity skill before shipping a branded screen.
- Never hand-pin Expo package versions; use `npx expo install <pkg>`.
