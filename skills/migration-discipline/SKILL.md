---
name: migration-discipline
description: Use whenever a model or database schema changes, a table or column is added, renamed, or dropped, a data backfill is needed, or a migration won't apply. Triggers include "migration", "schema change", "add a column", "rename column", "new table", "autogenerate migration", "data migration", "backfill", "migration failed", "alembic", "prisma migrate", "rails migration", "django migrate", "flyway", "knex".
---

# Migration Discipline

Get a schema or data change into a database without losing data, breaking a fresh deploy, or locking a production table. Migrations are append-only history that runs unattended on environments you can't watch — so the discipline is front-loaded: review before apply, make it reversible, make it safe on every shape of database it will meet.

Tool-agnostic (Alembic, Prisma, Rails, Django, Flyway, Knex, etc.); the principles hold across all of them.

## Core Principle: Run It Where the Real Config Lives

Generate and apply migrations in the **same environment that owns the DB connection, settings, and dependencies** — typically the app's container/venv/runtime, not a stray host shell. A migration generated against the wrong config drifts from reality.

- If dependencies are baked into an image/lockfile, a migration needing a new dep requires rebuilding/syncing that environment first — installing into a running container is often wiped on restart.
- Use the project's invocation (its task runner, `exec` into the service, the documented command). Match how CI/deploy runs it.

## Core Principle: Autogeneration Is a Draft, You Decide

Auto-generated migrations are a *starting point*, never the final answer. Always open the generated file and confirm before applying:

- It captured the change you intended — and **only** that (no stray drops from model drift, no churn from enums / server defaults / JSON columns the differ misreads).
- **A rename was not emitted as drop-then-add** — that silently destroys the column's data. Hand-edit to a real rename/alter.
- The **down/reverse** path is correct, not just the generated guess.
- No accidental data loss (narrowing a type, dropping a "renamed" column, `NOT NULL` without a default on a populated table).

## Core Principle: Reversible & Forward-Safe

- Write the **down migration** and round-trip it once locally (down, then up) before committing. If down throws, fix it now.
- One logical change per migration — easier to review, revert, and bisect.
- For large/production tables, prefer **expand/contract** (a.k.a. parallel change) to avoid downtime and long locks:
  1. *Expand*: add the new nullable column/table; deploy code that writes both.
  2. *Backfill*: migrate data in batches.
  3. *Contract*: switch reads, then drop the old column in a later migration.
- Know your engine's locking behavior (adding a `NOT NULL` column with a default, adding an index, rewriting a table) before running it on a big table; use the non-blocking variant where the engine offers one.

## Core Principle: Data Migrations Must Be Safe on Every DB Shape

A data migration runs on databases with different contents — a brand-new empty one (CI, a fresh deploy) and populated ones (staging, prod). It must do the right thing on all of them:

- **No-op cleanly on empty.** A backfill over an empty table is a natural no-op — good. A migration that *assumes rows exist* errors on fresh setup.
- **Idempotent / re-runnable** where possible — guard with conditions (`WHERE …`, existence checks), don't blind-write.
- **One-time backfills tied to historical data** (e.g. grandfathering existing accounts) are correct on populated DBs and meaningless on fresh ones — document that, and don't bake them into the baseline/initial schema where they'd run pointlessly or wrongly. If the migration history is ever squashed/baselined, such data steps are intentionally left out; record where the original SQL lives for snapshot restores.
- Heavy backfills belong in their **own** migration (or an out-of-band batched job), separate from the schema change.

## The Genius Panel

```
🏛️ GENIUS PANEL CONVENED: [The schema/data change]

THE DBA: Additive (safe) or destructive (careful)? Nullable-with-default vs
  NOT NULL needing a backfill first? Index for the new query path? Will it
  lock a big table on prod?
THE MIGRATION REVIEWER: Did autogeneration over/under-reach? Is a rename
  masquerading as drop+add (data loss)? Is the reverse path real?
THE RELEASE ENGINEER: Runs in the right environment? New dep → rebuild, not
  restart? Does deploy apply migrations automatically? Zero-downtime needed
  (expand/contract)?
THE DATA-MIGRATION SPECIALIST: Is there a backfill? Correct on an EMPTY db
  AND a populated one? Idempotent? Batched if large?
THE PRAGMATIST: Smallest reversible step. Split schema and backfill if the
  backfill is heavy.

CONSENSUS: [migration plan, rebuild-or-not, behavior on empty vs populated]
```

## Workflow

1. **Change the model/schema** in code; ensure the migration tool can see it.
2. **New dependency for the migration?** Add it and rebuild/sync the environment first.
3. **Generate** the migration via the project's command (or hand-write if the tool can't express it).
4. **Review the generated file** against the principles above; hand-edit renames, false diffs, and the reverse path.
5. **Data step?** Add it here (small) or as a separate migration/job (heavy); make it empty-DB-safe and idempotent.
6. **Apply locally and verify** the resulting schema (`current`/`status`/inspect).
7. **Round-trip the reverse** locally (down then up).
8. **Run the test suite** — and remember many suites build schema from models, not from the migration file, so the local up/down round-trip is what actually validates the migration itself.
9. **Commit** the migration file with the code change. One logical change per migration.

## Common Failures → Cause

| Symptom | Cause |
|---|---|
| Missing-module/dependency error during migration | new dep added but environment not rebuilt/synced |
| Generator drops a table/column you didn't touch | model not registered, or enum/default/JSON false diff — review & hand-edit |
| Rename lost all the column's data | emitted as drop+add — use a real rename/alter |
| Works locally, fails on fresh deploy | data migration assumed existing rows — make it empty-safe |
| Migration hangs / times out on prod | long lock on a large table — use expand/contract or the non-blocking variant |
| "Can't locate revision" / broken chain | migration chained off a removed/archived revision after a squash/baseline |

## When NOT to use this skill

- Building the feature around the schema change — use a vertical-slice skill (it points here for the migration step).
- A one-off manual data fix on a live DB that won't be versioned — but consider whether it *should* be a migration for reproducibility.
- Restoring a pre-squash/baseline snapshot — that's the special replay-the-archived-data-SQL case, not a normal forward migration.
