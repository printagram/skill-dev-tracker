# Changelog

All notable changes to `dev-tracker` are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [3.2.0] — 2026-04-17

### Fixed
- `save progress` is now a hard action, not an offer. Previous wording in "During work" said *"offer to save it"*, which contradicted the hard-trigger list. The trigger now writes immediately without confirmation.
- Anti-fragmentation made explicit: one DEVLOG entry per session, session boundaries defined only by explicit triggers (session start/end, ~12h gap, project switch). A two-hour debugging pause no longer risks splitting the log.

### Added
- **`/dev-tracker context` command** — read-only context restore for returning to a project after a break. Outputs last session, active decisions, open and recurring blockers, any in-progress marker, plus a prose "where you left off" summary. Does not write an in-progress marker (that's `session start`).
- **Session title guidance** — explicit guidance on naming entries after a feature, problem, or decision. Lists concrete anti-patterns ("Work session", "Updates").
- **Recurring blockers detection** — at session end, scan the last ~5 entries for repeated blockers and flag them as `[recurring since YYYY-MM-DD]`. Three or more recurrences → suggest a `D-###` entry for the root cause.
- **Anti-overlogging rule for DECISIONS.md** — if none of the four positive criteria is clearly satisfied, do not write. Mention in DEVLOG and offer to formalize instead. Rationale captured: under-logging is recoverable via audit; over-logging silently degrades the file.

### Changed
- "During work" section restructured into three named subsections: Hard save, Soft save, Anti-fragmentation.

### Out of scope (deferred to separate skill)
- Git integration with auto-generated commit messages from DEVLOG entries. Valuable idea but scope would dilute dev-tracker's clear boundary. Candidate for a separate `commit-crafter` skill that consumes DEVLOG output.

## [3.1.0] — 2026-04-17

### Fixed
- `CLAUDE.md` is now documented as auto-loaded by Claude Code, with an explicit warning about context-window cost and a ~150-line soft limit.
- Frontmatter `description` rewritten to include all relevant trigger keywords (`DEVLOG`, `DECISIONS`, `ADR`, `architecture decision record`, `project journal`, `cross-session memory`). Prior versions under-triggered because the description only covered literal command phrasings.
- Merged duplicated lists in "Soft triggers" and "Logging threshold" into a single "What counts as meaningful" section used for both purposes.
- Archiving threshold raised from 200 lines / 4 weeks to 500 lines / 8 weeks. The old threshold triggered on healthy projects within a month.
- `/dev-tracker diff` now has an explicit no-git fallback (prior versions left this undefined).

### Added
- Language policy: tracker prose follows the user's working language; code identifiers and status keys stay in English.
- Timezone convention: timestamps include a TZ abbreviation when the project may be worked on across machines.
- "Correcting past entries" section: append corrections instead of silently rewriting; use `superseded by D-###` for decisions.
- Monorepo / multi-project-in-one-repo guidance.
- Concurrent sessions safety: commit tracker files as isolated commits, check `git status` before appending.
- Archive fatigue rule: stop suggesting archival after three declines, record the preference.
- `examples/` directory with three populated example files (`DEVLOG.example.md`, `DECISIONS.example.md`, `CLAUDE.example.md`).
- `evals/evals.json` with eight realistic test prompts for triggering and behavior evaluation.
- `CHANGELOG.md` (this file).

### Changed
- Session start workflow now includes writing the in-progress marker as an explicit step (previously described but not listed in the numbered workflow).
- TODO handling explicitly forbids translating user content and warns against item-by-item interrogation.
- Writing quality rules now suggest linking to file/line or commit hash instead of embedding code.

## [3.0.0] — prior release (v3 in zip: `dev-tracker-v3-production`)

### Added
- Hard triggers vs soft triggers — explicit separation of "always act" and "suggest only" behaviors.
- No-git fallback section with explicit anti-hallucination rules.
- In-progress marker for detecting abruptly ended sessions.
- Archiving workflow (threshold originally 200 lines / 4 weeks).
- Stricter `[done]` rule for TODO: either explicit user confirm, or clearly completed in the current session with a save/end signal.
- New session-end step: log data operations (DB migrations, backfills, manual fixes).
- LICENSE file (MIT).

### Fixed
- Session end git commands are now conditional on git availability (prior version assumed git was always present).
- Frontmatter `version` bumped to 1.1.0 after being stale in v2.

## [2.0.0] — prior release (v2 in zip: `dev-tracker-v2`)

### Added
- In-progress marker section.
- Archiving section (200-line threshold).
- Multi-project workflow expanded with `taskList` field and explicit `cd` step.
- Audit triggers elaborated in `references/audit-heuristics.md`.
- `.gitignore`, LICENSE.

### Known issues (fixed later in 3.0.0)
- Frontmatter `version` was not bumped (remained `1.0.0`).
- README was not updated to reflect new features.
- Session-end workflow listed git commands unconditionally, encouraging hallucination when git was unavailable.

## [1.0.0] — initial release (v1 in zip: `dev-tracker-skill-production`)

Initial skill with:
- Four-file taxonomy: `TODO.md`, `DEVLOG.md`, `DECISIONS.md`, `CLAUDE.md`.
- Source-of-truth priority.
- Session start / session end workflows.
- Decision entry format based on ADR-lite.
- Multi-project registry support.
- Six `/dev-tracker` commands.
- Safety rule for destructive operations.
