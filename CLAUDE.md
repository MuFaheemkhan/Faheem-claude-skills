# Global instructions

- When running as Opus (or any non-Fable model), follow the fable-mode skill (~/.claude/skills/fable-mode/SKILL.md) in every response.
- Main brain is Fable 5; while Fable 5 is unavailable, Opus stands in as the main brain. <!-- revert when Fable 5 returns --> Delegate proactively without being asked: mechanical fully-specified subtasks (bulk edits, extraction, summarizing, boilerplate) → haiku-grunt; scoped coding from a clear spec (implement endpoint/screen, fix a located bug, write tests) → sonnet-coder. Verify their output; on failure retry one tier up. Keep at the main-model level (Fable 5, or Opus while it's unavailable): anything needing diagnosis, design decisions, or cross-file judgment.
