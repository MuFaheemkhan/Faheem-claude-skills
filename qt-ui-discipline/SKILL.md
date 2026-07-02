---
name: qt-ui-discipline
description: Audit and fix UI-thread blocking, thread-lifecycle, and async-staleness hazards in any Qt desktop app (PySide6/PySide2/PyQt6/PyQt5). Use before opening a PR in a Qt project, after adding any subprocess/network/file-I/O call to a widget, when adding a QThread/QProcess, or when a Qt GUI stutters or freezes. Triggers include "app freezes", "GUI hangs", "audit UI thread", "add a worker thread", "QThread", "QProcess", "PySide", "PyQt".
---

# Qt UI Discipline Audit

Bug class this skill targets: blocking or unsafe concurrency on the Qt UI
thread. These compile fine, work in the demo, and freeze in production when
a device/network/disk is slow.

## Step 0 — Find the project's async seam

Before auditing, identify how THIS project moves work off the UI thread.
Look for (in order): a `workers`/`tasks`/`async_utils` module wrapping
`QThreadPool`/`QRunnable`; `QThread` subclasses; `QtConcurrent`;
`qasync`/`asyncio` integration; bare `threading.Thread` + signal bridges.

- If a seam exists: every fix below must use it, not a new mechanism.
- If none exists: create ONE small helper (QThreadPool + QRunnable + a
  QObject signal bridge that delivers results on the GUI thread) and route
  everything through it. Never invent a second pattern alongside an existing one.

## Rule 1 — No blocking calls on the UI thread

Anything that can take >50ms — `subprocess.run`, urllib/requests, socket I/O,
large file reads/writes, `time.sleep` — must not run in a slot, event handler,
or widget `__init__`.

Audit:
```
grep -rn "subprocess\.\|urllib\|requests\.\|time\.sleep\|\.sleep(" <src> --include="*.py" -l
```
For each hit in a file that imports QtWidgets: the call must be inside a
closure handed to the async seam, or inside `QThread.run`/`QRunnable.run`.
Pure-function modules with no Qt imports are fine — they're the payload,
not the caller; verify their call sites instead.

Also forbidden on the UI thread:
- `waitForStarted` / `waitForFinished` / `waitForReadyRead` on QProcess —
  use the `started` / `finished` / `errorOccurred` / `readyRead*` signals.
- Synchronous dialogs inside timer ticks or stream handlers.
- `time.sleep` anywhere Qt-connected — use `QTimer.singleShot`.

## Rule 2 — Threads end cooperatively, never `QThread.terminate()`

`terminate()` kills mid-statement: half-written files, corrupt archives,
held locks. Pattern: a `_cancelled` flag set by `cancel()`, checked at each
loop iteration / between chunks. Audit: `grep -rn "\.terminate()" <src>` —
only `QProcess.terminate()` (a process, not a thread) is acceptable, and it
should be followed by a delayed `kill()` fallback.

## Rule 3 — Restartable async operations drop stale results

Any refresh/scan/fetch that can be re-triggered while a previous run is in
flight needs a generation counter: increment `self._gen` when starting,
capture the value in the callback closure, compare on arrival and discard
mismatches. Symptom when missing: a slow old result overwrites a newer one.

For periodic capture (QTimer + worker), also guard re-entry: skip the tick
if the previous capture hasn't returned (`_busy` flag).

## Rule 4 — Programmatically injected state is strip-then-add

When code injects a line/flag/entry into user-editable state (extra-args
text, config lists), it must remove all entries with the same key prefix
before appending the current value. Append-if-missing leaves stale values
behind when the setting changes.

## Rule 5 — Unbounded growth has a cap; hot paths are debounced

- Log views: `QPlainTextEdit.setMaximumBlockCount(N)`.
- Disk writes or expensive recomputes driven by `textChanged`/`valueChanged`:
  coalesce through a single-shot `QTimer` (300–500ms) instead of per-event.
- Collections keyed by external identities (serials, paths): verify removal
  on disconnect/close, not just insertion.

## Rule 6 — Subprocess data exchange avoids temp-file round-trips

Prefer capturing bytes directly from stdout (`capture_output=True`, binary
mode) over write-to-remote → pull → delete dances. Fewer round-trips, no
cleanup path to forget, nothing left behind on error.

## Verify

Boot the app's main window headlessly: show it, drive a few interactions
(page switches, a refresh) via `QTimer.singleShot`, quit on a timer after a
few seconds, assert exit 0. The window must appear BEFORE slow results
arrive — if first paint waits on a device/network scan, Rule 1 is violated
in startup code. Never use blocking input or modal-only paths in the test;
the session is non-interactive.
