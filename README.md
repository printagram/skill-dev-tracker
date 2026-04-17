# dev-tracker

> Cross-session memory for Claude Code — a structured development journal that doesn't hallucinate, doesn't spam, and doesn't rewrite your notes.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-3.2.0-blue.svg)](plugins/dev-tracker/CHANGELOG.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-orange.svg)](https://code.claude.com)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-purple.svg)](https://agentskills.io)

## What it does

Claude Code loses context between sessions. You come back to a project after a week and Claude doesn't know what was decided, what's blocked, or what's next. You end up re-explaining the same things every time.

`dev-tracker` fixes that with four files, each with a clear owner:

| File | Purpose | Who writes it |
|------|---------|---------------|
| `TODO.md` | Raw notes, rough ideas | **You** — Claude never rewrites it automatically |
| `DEVLOG.md` | Timestamped session log with Done / Decisions / Blockers / Next | Claude |
| `DECISIONS.md` | Architecture decisions in ADR-lite format (`D-001`, `D-002`…) | Claude |
| `CLAUDE.md` | Durable conventions that Claude Code auto-loads every session | Claude (with strict size discipline) |

## Why another journal skill?

There are a few out there. This one is different because it:

- **Refuses to hallucinate git output.** No invented commits, no fake file-change counts. If git isn't available, it says so and uses conversation context only.
- **Has explicit anti-fragmentation.** One DEVLOG entry per session, with session boundaries defined by hard triggers — not by "how long was the pause".
- **Has explicit anti-overlogging for decisions.** Under-logging is recoverable via `/dev-tracker audit`. Over-logging silently pollutes `DECISIONS.md` and degrades it forever. So the default is: when in doubt, *don't* write to DECISIONS.
- **Treats `TODO.md` as user-owned.** Never rewrites, translates, or clears it automatically.
- **Treats `CLAUDE.md` as context tax.** Has a size warning because every byte there is paid on every future session.
- **Handles corrections properly.** Wrong past entries get appended corrections, not silent rewrites. The audit trail stays intact.
- **Speaks your language.** Tracker prose follows your working language; keys like `D-001`, `Status: active` stay in English.

Four rounds of iteration went into this (v1 → v2 → v3 → v3.1 → v3.2), each driven by real use on production projects. See [CHANGELOG](plugins/dev-tracker/CHANGELOG.md) for the full story including the v2 release bug that got fixed in v3.

## Install

### Option 1 — Plugin marketplace (recommended)

In Claude Code:

```
/plugin marketplace add printagram/skill-dev-tracker
/plugin install dev-tracker@printagram-skills
```

### Option 2 — Direct install

```bash
# User-level (available in all projects)
git clone https://github.com/printagram/skill-dev-tracker.git
mkdir -p ~/.claude/skills
cp -r skill-dev-tracker/plugins/dev-tracker ~/.claude/skills/

# Or project-local (scoped to one project)
cp -r skill-dev-tracker/plugins/dev-tracker .claude/skills/
```

Restart Claude Code after either option.

## Usage

### Natural language (preferred)

```
You: session start
You: session start db_assistant       # with a project alias
You: save progress
You: session end
You: what did we do last time?
```

### Slash commands

```
/dev-tracker              Write/update current session entry
/dev-tracker status       Show last entry, blockers, next steps
/dev-tracker context      Full context restore after a break (read-only)
/dev-tracker init         Create DEVLOG.md, DECISIONS.md, TODO.md
/dev-tracker decisions    Show active decisions grouped by status
/dev-tracker diff         Summarize changes since last entry
/dev-tracker audit        Find undocumented decisions
```

### Multi-project

Create a `.dev-tracker-projects.json` in a parent directory listing your projects. See [project-registry.example.json](plugins/dev-tracker/references/project-registry.example.json). Then:

```
You: session start WebUI
You: session start API
```

Each alias gets its own DEVLOG / DECISIONS / TODO. Supports monorepos (one repo, multiple tracked sub-projects) and non-git directories.

## Example output

A session-end entry looks like this:

```markdown
## 2026-03-18 14:40 CET — Access ODBC field name mismatch

### Done
- discovered sync failures were caused by code using `id_Order`
  while Access exposes the field as `idx_Order`
- replaced all 14 occurrences across sync_access_to_supabase.py,
  sync_schema.py, mod_sync.bas
- re-ran sync, 523/523 orders synced clean
- updated CLAUDE.md with the correct field name

### Decisions
- canonical field name in all sync code is `idx_Order` (see D-003)

### Next
1. write remediation script for the `201\` corrupted folders
2. document the sync lifecycle in CLAUDE.md
```

See [`plugins/dev-tracker/examples/`](plugins/dev-tracker/examples/) for full examples of all four files.

## How it's different from…

| Tool | Scope | What it does well | Where dev-tracker fits |
|------|-------|-------------------|-----------------------|
| Claude's built-in `CLAUDE.md` | Auto-loaded conventions | Fast context | Too broad — mixes decisions, conventions, and narrative. dev-tracker splits these. |
| Generic "memory" plugins | Arbitrary state | Flexible | No discipline on what to record; drifts into noise. |
| Git commit messages | History of changes | Precise, timestamped | Describes *what* changed, not *why* or *what's next*. |
| Issue trackers | Work to be done | Team coordination | Too heavy for a solo developer's cross-session notes. |

`dev-tracker` fills the gap between "Claude remembers nothing between sessions" and "full-blown project management". It's a journal, not a tracker, not a wiki, not a commit history.

## What's new in 3.2

- `save progress` is now a hard action — writes immediately, no "should I save?"
- `/dev-tracker context` for returning to a project after a break (read-only)
- Anti-fragmentation formalized (one entry per session, explicit boundary rules)
- Anti-overlogging rule for DECISIONS
- Session title guidance with anti-patterns
- Recurring blockers detection

Full history in [CHANGELOG](plugins/dev-tracker/CHANGELOG.md).

## Contributing

Bug reports, feature requests, and PRs are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

The skill is iterated with evals — see [`plugins/dev-tracker/evals/`](plugins/dev-tracker/evals/). If you add behavior, add a matching eval prompt.

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, adapt it. If you improve it, consider upstreaming.

## Acknowledgements

The ADR-lite decision format is adapted from [Michael Nygard's original ADR pattern](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions). Progressive-disclosure skill structure follows [Anthropic's Agent Skills spec](https://agentskills.io).

Built and iterated on real production work at [Printagram](https://printagram.com.mt) — a printing, embroidery, and advertising company in Malta migrating from Microsoft Access to a modern cloud stack. If the rules feel opinionated, it's because they come from breakage.
