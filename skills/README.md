# Faheem's Claude Skills

This repository contains custom agent skills and tools for Claude.

## Included Skills

- **calmmoney-design**: The CalmMoney house design system (sage palette, dual-font, motion) shared across the family apps.
- **drunk-claude**: A fun personality module. (third-party)
- **feature-gating**: Designing and managing feature flags/gates.
- **harden**: Adversarial validation pipeline for a claim, change, or conclusion (`/harden <claim>`).
- **impeccable**: Production-grade frontend UI iteration and styling helper. (third-party)
- **migration-discipline**: Database and code migration protocols.
- **mobile-theme-parity**: Catching brand-pinned-background dark-mode breakage.
- **pre-pr-audit**: Compliance checks and auditing before PR submission.
- **qt-ui-discipline**: Standard patterns and checks for Qt-based user interfaces.
- **react-native-skill**: React Native / Expo app structure, security, and performance discipline.
- **scraper-discipline**: Best practices and rules for web scraping pipelines.
- **ui-build**: Scaffold a kit component or build UI from a screenshot, in the project's own style (merger of ui-scaffold + ui-from-screenshot).
- **vertical-slice**: Creating complete end-to-end features (vertical slices).

All of them can be invoked manually — any skill in ~/.claude/skills is callable as /name. There's no separate "manual" flag; what decides automatic firing is the description, since that's what I read when deciding whether a skill matches the task. So the real split is by how each description is scoped:

  Auto-trigger (description matches tasks/symptoms, fires without you naming it):

  
         Skill                          Fireswhen                                                                                                                                                                                                    
  │ vertical-slice           │ you ask to add a feature, endpoint, or screen spanning layers              │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ feature-gating           │ paywall, feature flag, quota, tier work                                    │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ migration-discipline     │ any schema change, backfill, or failing migration                          │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ pre-pr-audit             │ "ready for PR", "before I merge", "did I wire this up"                     │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ mobile-theme-parity      │ dark-mode breakage symptoms, brand-pinned backgrounds                      │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ qt-ui-discipline         │ Qt work — freezes, QThread, adding I/O to a widget, pre-PR in a Qt app     │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ ui-build                 │ scaffolding a component or building UI from a screenshot                   │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ react-native-skill       │ RN/Expo app setup, structure, security/perf hardening, store submission    │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ scraper-discipline       │ scraper failures, hidden-API probing, suspected throttling                 │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ calmmoney-design         │ working in a CalmMoney-family app, or you say "sage theme" / "house style" │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────────────┤
  │ impeccable (third-party) │ almost any frontend design/polish request — its triggers are very broad    │
  └──────────────────────────┴────────────────────────────────────────────────────────────────────────────┘

  Manual by design (description gates on you explicitly asking):

  - harden — only when you invoke /harden or explicitly ask for adversarial validation. We scoped it that way deliberately: it's an expensive rigor escalation you opt into.
  - drunk-claude (third-party) — a personality mode; it only makes sense when you summon it (/drunk-claude 0.7 --mood chaotic).

  Removed:

  - fable-mode (deleted 2026-08-01) — it tried to be always-on behavior inside a conditionally-loaded container, and about two thirds of it restated the base system prompt. The parts that were genuinely additive (outcome-first ordering, prose register, final-message completeness,
  verify-this-turn, evidence before state-changing commands) now live inline in ~/.claude/CLAUDE.md, where they load every session without a skill invocation.

  One practical note: auto-triggering is judgment, not regex — a skill with matching triggers should fire, but naming it (/pre-pr-audit) is the guarantee. If you ever notice one of the auto set failing to fire when it should, that's a description bug worth fixing — tell me the prompt that missed and I'll  
  widen its triggers.