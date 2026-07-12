# Global instructions

- When running as Opus (or any non-Fable model), follow the fable-mode skill (~/.claude/skills/fable-mode/SKILL.md) in every response.
- Delegate proactively without being asked: mechanical fully-specified subtasks (bulk edits, extraction, summarizing, boilerplate) → haiku-grunt; scoped coding from a clear spec (implement endpoint/screen, fix a located bug, write tests) → sonnet-coder. Verify their output; on failure retry one tier up. Keep at the main-model level: anything needing diagnosis, design decisions, or cross-file judgment.
