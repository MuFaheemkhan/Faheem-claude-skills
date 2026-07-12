---
name: react-native-skill
description: Use when starting or structuring a React Native / Expo app, choosing its stack or state management, or hardening it for release — security, performance, store submission. Triggers include "new mobile app", "Expo project setup", "project structure", "state management", "secure token storage", "slow list", "app performance", "app store submission", "EAS build", "mobile deployment". For a single component or screen use ui-build; for dark-mode/theming bugs use mobile-theme-parity; for the sage house style use calmmoney-design.
---

# React Native / Expo Discipline

Production-grade React Native + TypeScript apps: structure, stack, security,
and performance. Visual design lives elsewhere — the calmmoney-design skill
holds the house design system; ui-build holds component construction.

**Dependency rule:** never hand-pin Expo package versions. Use
`npx expo install <pkg>` so Expo resolves peer-compatible versions for the
SDK in use.

## The Genius Panel

For complex decisions, convene a mental panel before implementation:

1. **The UX Designer** — user flows, Apple HIG compliance, accessibility
2. **The Performance Engineer** — render optimization, memory, bundle size, 60fps
3. **The Architect** — code structure, state management, navigation, modularity
4. **The Platform Specialist** — iOS/Android specifics, native modules, parity
5. **The Pragmatist** — timeline, maintenance burden, team capabilities

```
🏛️ GENIUS PANEL CONVENED: [Topic]

UX DESIGNER: [...]
PERFORMANCE: [...]
ARCHITECT: [...]
PLATFORM: [...]
PRAGMATIST: [...]

CONSENSUS: [Agreed approach with rationale]
```

## Task Decomposition

Never tackle large features as monoliths. Split into tasks that are
independently testable, small (1–4 hours), clearly "done", and ordered by
dependency:

```
❌ BAD: "Build onboarding flow"
✅ GOOD: container+nav → slide component → pagination dots → button bar →
         content slides → gestures → haptics → persistence → tests
```

## Project Structure

```
src/
├── app/            # Entry, providers, navigation (RootNavigator, tabs, types)
├── components/     # Shared UI
│   ├── ui/         # Primitives: Button/, Text/, Input/, Card/ (each with
│   │               #   Component.tsx, .styles.ts, .test.tsx, index.ts)
│   ├── forms/  ├── feedback/  └── layout/
├── features/       # Feature modules: auth/, home/, settings/
│   └── <feature>/  #   screens/, components/, hooks/, services/, types.ts
├── hooks/          # Shared hooks (useDebounce, useKeyboard, useAppState)
├── services/       # api/ (client, interceptors, endpoints/), storage/, analytics/
├── store/          # Global state (useAuthStore, useSettingsStore)
├── theme/          # Design tokens (see calmmoney-design for the house system)
├── utils/          # Pure functions (format, validation, platform)
├── constants/  └── types/
```

## Code Quality

TypeScript strict mode with all the extras:

```json
{
  "compilerOptions": {
    "strict": true, "noImplicitAny": true, "strictNullChecks": true,
    "noUnusedLocals": true, "noUnusedParameters": true,
    "noImplicitReturns": true, "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true
  }
}
```

Type props explicitly; model async state as a discriminated union:

```typescript
type AsyncState<T> =
  | { status: 'idle' } | { status: 'loading' }
  | { status: 'success'; data: T } | { status: 'error'; error: Error };
```

Error handling (ApiError class, global ErrorBoundary with logging/analytics):
see `references/error-handling.md`.

## Mobile Security Checklist

Mobile apps are not browsers — the binary lives on a device the user (or
attacker) controls. Apply to every feature touching auth, PII, or money.

**Secret / token storage:**
- [ ] Never store JWTs or refresh tokens in AsyncStorage — plaintext on disk
- [ ] Use `expo-secure-store` (Keychain / Keystore) for tokens, session IDs, keys
- [ ] Rotate refresh tokens on use; invalidate the old one server-side
- [ ] Wipe secrets on logout

```typescript
// services/storage/secureStorage.ts
import * as SecureStore from 'expo-secure-store';

export const secureStorage = {
  async setToken(key: 'access' | 'refresh', value: string) {
    await SecureStore.setItemAsync(key, value, {
      keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
    });
  },
  getToken: (key: 'access' | 'refresh') => SecureStore.getItemAsync(key),
  clearAll: async () => {
    await Promise.all([
      SecureStore.deleteItemAsync('access'),
      SecureStore.deleteItemAsync('refresh'),
    ]);
  },
};
```

**Network:**
- [ ] HTTPS only — reject `http://` in production builds
- [ ] Consider certificate pinning for high-value APIs
- [ ] Strip tokens/PII from error logs and analytics events

**Deep links & URL schemes:**
- [ ] Validate every deep-link parameter — untrusted user input
- [ ] Universal Links / App Links, not just custom schemes (scheme hijacking)
- [ ] Don't navigate to untrusted URLs from deep-link params

**Device integrity (high-value actions):**
- [ ] App Attest (iOS) / Play Integrity (Android) before sensitive operations
- [ ] Jailbreak/root detection if the threat model requires it (`jail-monkey`)
- [ ] Biometric re-auth (`expo-local-authentication`) for sensitive operations

**Build hardening:**
- [ ] Hermes engine — bytecode, harder to reverse
- [ ] Strip source maps from production bundles; upload Hermes source map to
      the crash reporter separately
- [ ] No dev/test API keys in `app.json` extra — use EAS Secrets
- [ ] `android:allowBackup="false"` to prevent ADB data extraction

**PII handling:**
- [ ] `secureTextEntry` + `textContentType` on sensitive inputs
- [ ] Blur/exclude sensitive screens from the app-switcher preview
- [ ] Clear clipboard after copying account numbers / OTP

## Performance Checklist

**Rendering:**
- [ ] `React.memo()` for expensive components; `useMemo` for expensive calcs
- [ ] `useCallback` for handlers passed as props; no inline styles/functions in render
- [ ] Proper `keyExtractor` in lists

**Lists:**
- [ ] `FlatList`/`FlashList`, never `ScrollView`, for lists
- [ ] Always set `estimatedItemSize` on FlashList — omitting it silently drops
      to the slow path
- [ ] `getItemLayout` for fixed-height items; `removeClippedSubviews` for long lists
- [ ] Phone-first: don't chase FlashList web-only layout gaps; verify on iOS/Android

**Server state & loading:**
- [ ] Fetch via TanStack Query, not `useState`/`useEffect` chains (dedup, SWR
      cache, retry/backoff for free) — see `references/performance-patterns.md`
- [ ] Layout-matched skeletons on first load; spinners only for inline actions

**Images:**
- [ ] `expo-image` / `FastImage` for caching; explicit dimensions (no layout shift)
- [ ] Appropriate resolution per device; progressive loading for large images

**Animations:**
- [ ] Reanimated on the UI thread (`useAnimatedStyle`, `withSpring`/`withTiming`)
- [ ] Avoid animating layout properties when possible

**Memory:**
- [ ] Clean up subscriptions in `useEffect` cleanup; cancel requests on unmount
- [ ] `useFocusEffect` for screen-specific logic

## Technology Stack

- **Core:** current Expo SDK + React Native, TypeScript, React Navigation
- **State:** Zustand (global), TanStack Query (server), React Hook Form (forms)
- **UI/animation:** Reanimated, Gesture Handler, expo-haptics, expo-blur
- **Dev:** ESLint + Prettier, Jest + Testing Library, React Native DevTools
  (RN 0.76+). Flipper was removed from RN core — don't recommend it for new
  projects; Reactotron is a solid third-party alternative.

## Deployment Readiness

Environment config per stage (dev/staging/prod API URLs, log gating). Store
checklist: icons, splash, privacy policy URL, screenshots, accessibility
labels, deep linking, push notifications, analytics, crash reporting
(Crashlytics needs a dev/EAS build, not Expo Go), OTA updates (EAS Update).

## Quick Commands

```bash
npx create-expo-app@latest myapp -t expo-template-blank-typescript
npx expo install react-native-reanimated react-native-gesture-handler
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install expo-haptics expo-blur expo-image
npx expo start          # development
npm test                # tests
eas build --platform ios|android
eas submit --platform ios|android
```

## References

- `references/performance-patterns.md` — optimization techniques and profiling
- `references/error-handling.md` — error boundaries, API errors, logging
