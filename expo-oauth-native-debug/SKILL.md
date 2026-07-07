---
name: expo-oauth-native-debug
description: Use when native social sign-in (Google/Apple/OAuth/SSO) in a Clerk + Expo / React Native app fails to complete — the browser opens, the user picks an account, and the app returns to the login screen instead of signing in — or when a re-login after logout bounces while the first login worked. Also the playbook for debugging ANY Expo/RN issue on a live Android emulator via adb. Triggers include "OAuth bounces to login", "sign in returns to login screen", "Continue with Google does nothing", "startOAuthFlow dismiss", "authSessionResult dismiss", "redirect url not authorized", "needs_identifier", "Clerk Expo OAuth", "useSSO / useOAuth redirect", "re-login fails after logout", "debug the app on the emulator", "read the app logs with adb".
---

# Native OAuth (Clerk + Expo) sign-in bounce — and how to debug it on-device

A class of bug where `startOAuthFlow`/`startSSOFlow` opens the browser, the user authenticates, and the app silently returns to the login screen with no session. It compiles, types, lints, and often works on the *first* login — then fails on re-login after logout. Only an on-device run with logs surfaces the real cause. This skill is the checklist of causes (in priority order) plus the `adb` loop that makes it observable.

## The #1 cause: bare-scheme redirect fails the Android return match

```ts
// BROKEN — bare scheme, no path
const redirectUrl = AuthSession.makeRedirectUri();          // -> "calmmoney://"
// FIXED — scheme + PATH (Clerk's documented native form)
const redirectUrl = AuthSession.makeRedirectUri({ scheme: 'calmmoney', path: 'sso-callback' });
```

On Android the return leg comes back as `calmmoney:` / `calmmoney:///` and expo-web-browser can't prefix-match a bare `calmmoney://`, so it closes the auth session with **`authSessionResult.type: 'dismiss'`**, `createdSessionId: ''`, `signIn.status: 'needs_identifier'` — no session, silent bounce. A distinct path makes the match unambiguous. First login can slip through on fresh task/cookie state; after logout it fails consistently.

## The catch: the pathed redirect must be allowlisted in Clerk

The redirect fix is **two-part** — code alone throws
`"The current redirect url … does not match an authorized redirect URI for this instance."`

> Add the exact URI (e.g. `calmmoney://sso-callback`) in **Clerk Dashboard → Native applications → "Allowlist for mobile SSO redirect."** This is infra, not in the repo, and **every Clerk instance the app points at (dev + prod) needs it.**

## Read the result object — it names the cause

Instrument `runOAuth` and log the raw result before branching:

| Signal | Meaning | Fix |
|---|---|---|
| `authSessionResult.type: 'dismiss'` | redirect not captured | scheme+path redirect (above) |
| `signIn.status: 'needs_identifier'`, `createdSessionId: ''` | no OAuth code came back | same — redirect match failure |
| throws `session_exists` | stale Clerk session (single-session mode) | `await signOut()` then retry `start()` once |
| error `"does not match an authorized redirect URI"` | redirect not allowlisted | add it in Clerk Native applications |
| `type: 'cancel'` | user backed out | nothing — do not show an error |

## Dead ends — do NOT waste a device round-trip on these

- **`launchMode`** — `singleTask` is Expo's default AND the value Expo's AuthSession docs require. `dismiss` is a redirect-*match* failure, not a task-routing one. Changing launchMode needs a rebuild and won't help. (A return intent logging `result code=2` = START_TASK_TO_FRONT — the existing task resumed; nothing was lost to a "new task".)
- **Browser warm-up** (`WebBrowser.warmUpAsync`) — good Android practice, harmless to keep, but it does not change redirect matching. Not the fix.
- **"Nested session" recovery** (`signIn?.createdSessionId` fallback) — there is no hidden session when the flow dismisses. Don't add speculative recovery for a session that never exists.
- `useOAuth` is deprecated in clerk-expo v2 in favour of `useSSO`, but it still works — a migration is not required to fix this.

## The on-device debugging loop (this is what made it solvable)

Reason from evidence, not pattern-matching. The Metro terminal (which the user sees, not you) gets `console.log`; `adb` is your channel. The core loop:

```bash
adb devices                                   # confirm the emulator (e.g. emulator-5554)
adb logcat -c                                 # clear the buffer before a repro
# ... user performs the repro (you can't drive Google's account picker blind) ...
adb logcat -d | grep -a "ReactNativeJS"       # JS console.* lands under this tag
adb exec-out screencap -p > screen.png        # READ the screen yourself — error banners, which screen
```

Guaranteeing your code edit is actually loaded (a disconnected Metro HMR socket = the app runs a stale bundle → total log silence, which looks like "nothing happened"):

```bash
adb shell am force-stop <package>
adb shell monkey -p <package> -c android.intent.category.LAUNCHER 1   # cold start re-fetches the bundle
timeout 40 adb logcat | grep -m1 -a "isLoading done"                  # wait for boot, then clear + repro
```

Temporarily instrument the failing function with `if (__DEV__) console.log('[oauth] result', {...})` logging the raw result/error, run the repro, read it via logcat, then **strip the debug logs** before committing. Confirm success from an observable you control (a nav log flipping to `isAuthenticated = true`, or a screencap of the authenticated screen) — not from "it should work now."

## Discipline

This bug ate several wrong guesses (nested-session, warm-up, launchMode) before instrumentation revealed `dismiss` + `needs_identifier`. The lesson: when a native auth flow fails, **log the raw result object first**; the `type`/`status`/error-code fields tell you which row of the table above you're in, and each has a different fix. Verify the fix by exercising it on the device, not by asserting it.
