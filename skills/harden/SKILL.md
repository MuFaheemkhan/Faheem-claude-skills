---
name: harden
description: Use when the user explicitly invokes /harden or asks for adversarial validation of a specific claim, change, or conclusion. Not for routine tasks — this is a deliberate, expensive rigor escalation the user opts into.
---

# Harden

Adversarially validate one claim, change, or conclusion. Every stage produces
quoted evidence or an explicit "skipped because X" — a stage with no quoted
output did not happen.

**Refutation is success.** Finding the flaw is the point. Never soften a
negative result into "mostly works."

## The pipeline

Run the stages in order. Each has an observable exit criterion.

### 1. Scope
State the exact claim being hardened as one falsifiable sentence
("`parseDate` handles all RFC 3339 timestamps", "this migration is safe on
populated tables"). Vague target → pin it down with the user's words before
proceeding. Everything below tests *this sentence*. Record the artifact's
identity (path + timestamp or hash); if it changes mid-run, that is itself a
finding — report it and state which version each stage exercised.

### 2. Research
Check the claim against primary sources: official docs (context7 / WebFetch),
the actual installed dependency source, the real schema — never memory.
**Exit:** quote the specific line that supports or contradicts the claim, with
its source.

### 3. Validate
Execute the real thing — the code path, the test, the command, the query.
**Exit:** paste actual output. "Should work" or reasoning-only means this
stage did not happen.

### 4. Audit
Inspect the artifact for defects. For a code diff, invoke `/code-review`
rather than re-implementing review. For a claim or document, check each
load-bearing statement against stage-2 sources.
**Exit:** findings list, or "none found" plus what was checked.

### 5. Cross-audit
Re-verify the stage-3 conclusion by a second **independent** method — one that
would still catch the error if the first method were itself wrong: a different
test harness, manual derivation from source, a different tool, a different
dataset. Re-running the same check twice is not independence.
**Exit:** both methods' outputs quoted, agreeing or disagreeing.

### 6. Adversarial test
Try to break it. Construct counter-examples aimed at the claim: boundary
values, empty/huge/malformed input, concurrency, the case the author plainly
didn't think about. Spend real attempts — three distinct attack angles
minimum.
**Exit:** each attempt listed with its actual result, including the ones that
failed to break anything.

### 7. Enhance (conditional)
Only if the user asked for fixes — default is report-only; never mutate the
artifact under review unprompted. When fixing: apply the fix, then give the
fixed code its own stage-6 pass with fresh attack angles aimed at the new
code. Re-running the tests that caught the original defect is not enough — a
fix can introduce a defect the old tests structurally cannot see. If fixing
is out of scope, the verdict reports the defect — do not quietly absorb it.

## Verdict

Final message leads with one of:
- **CONFIRMED** — survived all stages; evidence quoted per stage.
- **REFUTED** — broken; the counter-example or failing output shown first.
- **UNCERTAIN** — a stage couldn't run; name it and what's needed.

Then per-stage evidence, in order, for readers who want it.

## Red flags

| Thought | Reality |
|---|---|
| "Both checks agree" (same method twice) | Not independent. Redo stage 5 with a genuinely different method. |
| "The oracle agrees with the code" (oracle built from the code's own formula) | Not independent. Build the reference from the stage-2 source, not from the implementation. |
| "My fix passed the old tests" | Old tests aim at the old bug. New code gets new attacks (stage 7). |
| "Should work" / no pasted output | Stage 3 didn't happen. Run it. |
| Quietly skipping a stage | Say why in the verdict, or do it. |
| "Found a bug but the claim mostly holds" | That's REFUTED (or fixed-then-re-verified). Not "mostly". |
