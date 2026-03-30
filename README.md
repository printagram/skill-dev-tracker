# dev-tracker

A Claude Code skill that maintains a structured development journal across sessions, preventing context loss between conversations.

## Problem

Each Claude Code session starts with a blank slate. Without persistent tracking:
- Work done in previous sessions gets forgotten or misrecorded
- Architectural decisions lose their reasoning over time
- Next steps get lost between conversations
- The same questions get re-asked

## Solution

`dev-tracker` maintains three files in your project root:

| File | Purpose |
|------|---------|
| **DEVLOG.md** | Chronological journal — what happened when, what's next |
| **DECISIONS.md** | Permanent decision register — what was agreed, why, when to revisit |
| **CLAUDE.md** | Living reference — architecture, conventions, standards (updated, not duplicated) |

## Features

- **Session start**: shows last session's summary + next steps + blockers
- **During work**: reminds to save progress after major blocks
- **Session end**: writes full entry (Done/Discussions/Decisions/Blockers/Next)
- **Decisions register**: numbered entries (D-001, D-002...) with context, rationale, revisit conditions
- **In-progress marker**: protects against abrupt session endings
- **Audit**: scans all project files for undocumented decisions and conventions
- **Archiving**: auto-suggests archiving when DEVLOG exceeds 200 lines
- **Language-aware**: matches your project's language
- **Environment-aware**: works in Claude Code (file access) and Claude.ai (text output)

## Installation

### Option 1: Global (all projects)

```bash
mkdir -p ~/.claude/skills/dev-tracker
cp SKILL.md ~/.claude/skills/dev-tracker/
```

### Option 2: Per-project

```bash
mkdir -p .claude/skills/dev-tracker
cp SKILL.md .claude/skills/dev-tracker/
```

## Commands

| Command | What it does |
|---------|-------------|
| `/dev-tracker` | Update today's log entry |
| `/dev-tracker status` | Show last entry + next steps + blockers |
| `/dev-tracker init` | Create DEVLOG.md + DECISIONS.md (reconstructs history if possible) |
| `/dev-tracker decisions` | Show all active decisions grouped by status |
| `/dev-tracker diff` | Summarize changes since last DEVLOG entry |
| `/dev-tracker audit` | Scan all sources for undocumented decisions — show gap table |
| `"session start"` | Show last session summary, ask for today's plan |
| `"session end"` | Write full session entry, update all files |

## DEVLOG.md Format

```markdown
# Dev Log

## 2026-03-30 — PDF reports + Email integration
### Done
- 7 PDF reports: Invoice, Quote, Creditnote, Delivery, Receipt, Print List
- Email sending via Resend (6 modes)

### Discussions
- [Email service]: compared Resend vs Gmail API vs SMTP
  → Outcome: chose Resend — simpler setup, webhooks built-in

### Decisions
- Chose Resend for email (see D-006)

### Blockers
- Print Production Sheet blocked — no Access report structure exported yet

### Next
1. User Roles (Operator/Manager/Director)
2. Export XLS
3. Deploy to Hostinger
```

## DECISIONS.md Format

```markdown
# Decisions

## [D-001] Short decision title
Date: YYYY-MM-DD | Status: active | Ref: DEVLOG YYYY-MM-DD
Context: what situation or problem triggered this decision
Alternatives: what else was considered
Decision: what was decided
Rationale: why this option over the others
Conventions: specific rules that follow from this decision
Revisit when: condition under which to reconsider
```

**Statuses:** `active` | `superseded by D-###` | `deferred` | `rejected`

## Audit

`/dev-tracker audit` scans memory files, CLAUDE.md, DECISIONS.md, and DEVLOG.md to find gaps:

```
| Item                  | Found in              | Should go to    | Status  |
|-----------------------|-----------------------|-----------------|---------|
| No nested components  | feedback_no_nested.md | DECISIONS.md    | MISSING |
| Money fields 130px    | CLAUDE.md:110         | DECISIONS.md    | COVERED |
| Exo 2 font            | CLAUDE.md:11          | CLAUDE.md only  | OK      |
```

Does not auto-update — shows the table and asks which items to add.

## License

MIT
