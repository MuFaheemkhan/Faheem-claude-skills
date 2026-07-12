---
name: mobile-theme-parity
description: Use when a mobile screen looks fine in light mode but breaks in dark mode — invisible labels, near-black inputs on a light surface — or when building/reviewing screens whose background is pinned to a brand colour while children resolve colours via useColorScheme(). Also for other ambient-state mismatches (locale, RTL, dynamic type). Triggers include "dark mode broken", "black input on cream", "labels not visible", "useColorScheme", "scheme lock", "brand pinned background", "theme parity", "fix dark mode", "branded auth screen".
---

# Mobile Theme Parity — the brand-pinned-background trap

A class of bug ubiquitous in React Native / Expo apps that has a "warm" brand voice (auth, onboarding, marketing screens). It compiles, lints, types, passes jest — and ships broken on every dark-mode device. This skill teaches you how to spot it, fix it at the seam (not screen by screen), and verify it.

## The trap

```tsx
// SignInScreen.tsx — looks fine, ships broken in dark mode
<SafeAreaView style={{ backgroundColor: theme.brand.cream }}>
  <Text style={{ color: colors.textSecondary }}>Email</Text>
  <TextInput style={{ backgroundColor: colors.backgroundSecondary }} />
</SafeAreaView>
```

The parent **fixes** the background to a brand colour that does not flip with the OS scheme. The children **read** their colours through `useColorScheme()` (directly or via a `colors.*` token map keyed on scheme). On a dark-mode device the result is:

- background: cream (light) — unchanged, brand colour does not flip
- `colors.backgroundSecondary` for the input: `#1A201A` (near-black) — flipped to dark-mode value
- `colors.textSecondary` for the label: `rgba(255,255,255,0.7)` — flipped to dark-mode value, **invisible on cream**

The author tested in light mode (where everything matches because the dark/light values happen to be tonal cousins of cream), the screen looked right, and shipped. The dark-mode user opens the app and sees a cream screen with black boxes and invisible field labels. **jest cannot render against a colour scheme; eslint and tsc have no opinion about ambient state mismatches.** Only an on-device run in the failing scheme surfaces it.

## The rule

> **A fixed brand background obligates a fixed brand foreground.** Either both flip with the OS or neither does. Mixing them is the bug.

Equivalent framings:
- Every descendant of a brand-pinned surface must resolve its colours against the *same locked scheme* as the parent.
- `useColorScheme()` is correct for screens with a `colors.background` surface, wrong for screens with a `brand.*` surface, unless you wrap the subtree in a scheme-lock.

## How to spot it (audit checklist)

Before writing the fix, find every instance — the trap is usually copy-pasted. Use these greps:

```bash
# 1. Every screen with a brand-pinned background
rg 'backgroundColor:\s*[a-zA-Z.]+brand\.' <src> --type tsx --type ts
rg 'backgroundColor:\s*[a-zA-Z.]+\.(cream|warm|mint|sand|sage|fawn|paper)' <src>

# 2. Every consumer of useColorScheme — these flip with the OS
rg 'useColorScheme\(' <src>

# 3. Hard-coded #FFFFFF / #000000 outside the theme module (HIG anti-pattern)
rg "['\"]#(FFFFFF|000000)['\"]" <src> --type tsx --glob '!**/theme/**'
```

Any file appearing in (1) whose descendants resolve text/fill from `useColorScheme()` (directly or via a scheme-aware token like `colors.text`/`colors.backgroundSecondary`) is a bug site. Expect to find several — the idiom spreads.

A canonical sign: `SettingsScreen.tsx` and `WelcomeScreen.tsx` both have `style={{ backgroundColor: t.brand.cream }}`. The first author chose it; everyone else copied. The bug count is the screen count.

## The fix (three shapes)

Pick **one** per surface. Don't mix.

### Shape A — scheme-lock the subtree (recommended for branded screens)

Extend `ThemeProvider` to accept a `lockedScheme?: ColorSchemeName` prop; `useTheme()` honours it over the OS scheme. Wrap branded surfaces:

```tsx
// theme/ThemeContext.tsx
const ThemeContext = createContext<{ theme: Theme; lockedScheme?: ColorSchemeName }>({ theme: defaultTheme });

export function ThemeProvider({ theme, lockedScheme, children }) {
  const parent = useContext(ThemeContext);
  const value = useMemo(() => ({
    theme: theme ?? parent.theme,
    lockedScheme: lockedScheme ?? parent.lockedScheme, // inner shadows outer; omit to inherit
  }), [theme, lockedScheme, parent.theme, parent.lockedScheme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const { theme, lockedScheme } = useContext(ThemeContext);
  const scheme = lockedScheme ?? useColorScheme() ?? 'light';
  return { ...theme, scheme, colors: theme.palette[scheme] /* ... */ };
}
```

Then at each branded surface:

```tsx
// AuthLayout.tsx, WelcomeScreen.tsx, OnboardingScreen.tsx, splash, marketing...
<ThemeProvider lockedScheme="light">
  <SafeAreaView style={{ backgroundColor: t.brand.cream }}>{children}</SafeAreaView>
</ThemeProvider>
```

This is **a single seam** that fixes every descendant — including third-party components that internally call `useColorScheme()`, because the system hook is overridden in the resolved theme value. Pair with a unit test asserting `useTheme()` honours `lockedScheme` over a spied `useColorScheme()`.

### Shape B — make the background scheme-aware

If the screen does not need to be brand-pinned (e.g. inner app screens, not the auth/marketing tone-setting surfaces), swap the background to a scheme-aware token:

```tsx
- <SafeAreaView style={{ backgroundColor: t.brand.cream }}>
+ <SafeAreaView style={{ backgroundColor: t.colors.background }}>
```

Now the parent flips with the OS, matching every `useColorScheme()`-driven child. Verify the design intent — sometimes "everywhere cream" was deliberate for brand coherence and dropping it is a regression of a different kind.

### Shape C — pin the foreground too (rare)

If brand-pinning is non-negotiable and you do not want to introduce a scheme-lock seam (e.g. a one-screen prototype), explicitly pass brand-rooted colours to every descendant text and fill:

```tsx
<Text style={{ color: theme.brand.primaryDeep }}>Email</Text>
<TextInput style={{ backgroundColor: theme.brand.mint }} />
```

Brittle and easy to undo accidentally; only acceptable for throwaways or single-screen prototypes. Prefer Shape A or B for anything that will grow.

## StatusBar follow-on

Branded surfaces almost always need a `<StatusBar style="dark" />` (Expo) or `barStyle="dark-content"` (RN) because the background is a *light* brand colour regardless of the OS. The OS-driven default flips, which means light-icon status bar on a cream screen — invisible. If RootNavigator owns the StatusBar globally, hardcode `style="dark"` until the entire app is scheme-aware; otherwise declare per-screen.

## Verification — only on-device works

```bash
# Build a dev client (Expo Go cannot run native modules added during the polish)
cd mobile && npx expo run:ios     # or: run:android, or: eas build --profile development

# Run THE EXACT failing scenario:
#   iOS:     Settings → Developer → Dark Appearance → On
#   Android: Quick Settings → Dark mode → On
# Re-open the app and visit every branded screen.
```

A screen passes if input fields, labels, dividers, and placeholders are all legible against the screen background. Failures look like missing text or dark squares on a light surface.

**Things that cannot catch this bug:**
- `pnpm tsc --noEmit` — types know nothing about runtime scheme
- `eslint` — no rule for ambient-state mismatches
- `jest` — does not render to a colour scheme; no headless device
- visual snapshots without an explicit scheme override — they capture light mode and stop

**Things that can:**
- on-device QA in both schemes (the only zero-setup option)
- a jest mock that spies `useColorScheme()` and renders both schemes against a snapshot
- Detox / Maestro flows with a per-test scheme toggle

## Why this gets shipped

Three reinforcing reasons, in priority order:

1. **PRDs typically defer "dark mode parity" to a polish phase.** The phase that builds the first branded screen does not have a dark-mode acceptance criterion. The bug is born and shipped within its own phase's spec.
2. **Copy-paste propagation.** The first author picked `backgroundColor: t.brand.cream` because it matched a Figma frame. Every subsequent screen copied the idiom. The bug count grows with the screen count.
3. **The reference patterns teach the bug.** Apple HIG examples, Expo docs, every "how to do dark mode in React Native" article shows `const scheme = useColorScheme() ?? 'light'`. None of them mention "if your screen background is fixed to a brand colour, this seam will produce a broken result." The author follows the canonical pattern and produces the canonical bug.

When you find the bug, surface all three — the team usually thinks they just missed something and miss the systemic reason next time too.

## Generalization

The same shape — *parent fixes one half of a context-dependent visual pair while descendants resolve the other half from a different ambient source* — bites in other ambient-state pairs. Search for these analogues when auditing:

- **Locale** — parent fixes a number format / currency / date, children read `Intl` against device locale
- **RTL** — parent forces LTR layout, children read `I18nManager.isRTL`
- **Dynamic Type** — parent caps `fontSize` numerically, children scale with `PixelRatio` / system accessibility settings
- **Density** — parent fixes dp values, children use `useWindowDimensions()` against the actual device

The fix shape is the same: a scoped provider that locks the ambient value for the subtree, or make the parent honour the ambient source too. Mixing is always the bug.

## When NOT to scheme-lock

- Inner app screens that *should* feel native to the device (Discover, Settings, list views): use scheme-aware `colors.background`, do not lock.
- A screen using `colors.background` with `useColorScheme()`-driven children is *not* the bug — that pair flips together.
- A `lockedScheme="dark"` is occasionally correct (e.g. a video player screen, an always-dark hero): same rule, opposite direction.

The trap is specifically *mismatched* fixed/dynamic — not *all* fixed surfaces. Locked scheme is a tool, not a default.

## Minimum-effective audit when invoked

1. Grep `backgroundColor: .*brand\.` to enumerate brand-pinned surfaces.
2. For each match, open the file and confirm whether descendants resolve via `useColorScheme()` (directly or via scheme-aware tokens).
3. Pick Shape A (lock) or Shape B (scheme-aware) per surface; document the pick in PR description.
4. Add or extend a unit test asserting `useTheme()` honours `lockedScheme` (Shape A) — this is the only automated guardrail this class of bug responds to.
5. Add a one-line invariant to the project's status / convention doc: *"Brand-pinned screens scheme-lock to light."*
6. On-device verification in both schemes before merge — the only test that actually proves it.
