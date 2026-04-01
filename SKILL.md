---
name: dev-tracker
description: "Use this skill ANY TIME the user starts or ends a work session, says 'save progress', 'session start/end', 'what did we do', 'update log', or uses /dev-tracker. Also use proactively after completing major features, fixes, or architectural decisions. Essential for multi-session projects — prevents context loss between conversations. Works in Claude Code (file access) and Claude.ai (text output mode)."
---

# Dev Tracker

A universal development journal that keeps track of what was done, what's planned, and what decisions were made — across sessions, across projects, across stacks.

## Quick reference

| Command | What it does |
|---------|-------------|
| `/dev-tracker` | Write/update today's DEVLOG entry |
| `/dev-tracker status` | Show last entry + next steps + blockers |
| `/dev-tracker init` | Create DEVLOG.md + DECISIONS.md |
| `/dev-tracker decisions` | Show all active decisions grouped by status |
| `/dev-tracker diff` | Summarize changes since last DEVLOG entry |
| `/dev-tracker audit` | Scan all sources for undocumented decisions |
| `"session start"` | Show last session summary, ask for today's plan |
| `"session end"` | Write full session entry, update all files |

## Why this matters

Each Claude Code session starts fresh. Without a persistent log, work done in previous sessions can be forgotten or misrecorded. This skill maintains a structured DEVLOG.md that serves as the single source of truth for project progress.

## Environment awareness

This skill behaves differently depending on the environment:

- **Claude Code** — full file system access. Read and write DEVLOG.md directly.
- **Claude.ai (chat)** — no file system access. Present the formatted log entry as a code block and ask the user to paste it manually. Never pretend to write a file that doesn't exist.

Always detect the environment at session start. If uncertain, ask: "Are you working in Claude Code, or should I give you text to copy into your DEVLOG.md?"

## The DEVLOG.md file

Location: project root (next to CLAUDE.md if it exists). Newest entries first.
```
# Dev Log

## YYYY-MM-DD — Brief session title
### Done
- what was actually completed (facts, not intentions)
- include file names for new files, feature names for features

### Discussions
- [Topic]: brief summary of what was discussed, alternatives considered
  → Outcome: what was agreed, what was deferred, what was rejected and why
- (omit if no significant discussions happened)

### Decisions
- key architectural or design decisions with brief reasoning
- reference D-### from DECISIONS.md if logged there
- (omit if no decisions were made)

### Blockers
- what didn't work, what is blocked, what needs investigation
- include error names or symptom description, not full traces
- (omit if nothing is blocked)

### Next
1. concrete next step (highest priority first)
2. second next step
```

### Writing style — good vs bad

| ❌ Too vague | ✅ Specific |
|-------------|-----------|
| "Worked on the frontend" | "Orders list: added status filter + row colors by priority" |
| "Fixed a bug" | "Fixed: KeyError in sync_access.py when order has no client_id" |
| "Chose Resend" | "Chose Resend over Gmail API — simpler setup, webhooks built-in" |
| "Continue working on email" | "1. Implement Ready Notification template 2. Add batch sending" |

### What NOT to put in DEVLOG

- Code snippets (that's what git is for)
- Debugging logs or error traces (summarize the problem, not the output)
- Detailed technical specs (those go in CLAUDE.md)
- Meeting notes or discussion transcripts
- Anything that can be derived from `git log`

## Session workflow

### Session start

1. Look for DEVLOG.md in the project root
2. If it exists, read the most recent entry
3. Check for `[in-progress]` marker — if present, note that the previous session may have ended abruptly
4. Present a brief summary:
```
   Last session (YYYY-MM-DD): [1-2 line summary]
   Next steps: [list from ### Next]
   Blockers: [list from ### Blockers, if any]
   What are we working on today?
```
5. If DEVLOG.md doesn't exist, create it and note this is the first tracked session

### During work — when to update

**Update after:**
- A major feature is completed
- A complex bug is fixed
- An important architectural decision is made
- Multiple related changes that form a logical unit

**Don't update after:**
- Minor fixes (typos, small CSS tweaks)
- Exploratory work that didn't lead anywhere
- Individual file edits that are part of a larger task in progress

When updating mid-session, append to the current day's `### Done`. Don't create a new date entry.

### Handling abrupt session endings

At the start of significant work, write a temporary marker:
```
## 2026-03-30 — [in-progress]
### Done
- ...work so far...
```

Replace `[in-progress]` with the real session title at session end. If the session ends unexpectedly, the next session can see the entry is incomplete.

### Reminding the user

After completing a significant block of work:
```
[summary of what was just completed]. Want me to save this to the dev log?
```

Keep reminders brief. If the user declines, don't repeat for the same block.

### Session end

1. Write or update the day's entry with everything done
2. Remove `[in-progress]` marker; replace with real session title
3. Log important discussions in `### Discussions`
4. Include decisions with reasoning in `### Decisions`
5. If a decision is significant — add to DECISIONS.md
6. Record blockers with symptom description
7. Write concrete numbered Next steps
8. If architectural decisions were made, check if CLAUDE.md needs updating — add only new sections, don't duplicate DEVLOG content

### Multiple sessions per day

Append to the existing day's entry, don't create duplicates:
```
## 2026-03-30 — PDF reports + Email integration
### Done
- [morning] 7 PDF reports: Invoice, Quote, Creditnote, Delivery, Receipt, Print List
- [afternoon] Email sending via Resend (6 modes working)
```

## The DECISIONS.md file

Location: project root. Purpose: permanent, searchable register of important decisions — unlike DEVLOG, entries here never get archived.
```
# Decisions

## [D-001] Short decision title
Date: YYYY-MM-DD | Status: active | Ref: DEVLOG YYYY-MM-DD
Context: what situation or problem triggered this decision
Alternatives: what else was considered (briefly)
Decision: what was decided
Rationale: why this option over the others
Conventions: any specific rules or patterns that follow
Revisit when: concrete condition under which to reconsider
```

**Status values:** `active` | `superseded by D-###` | `deferred` | `rejected`

### When to add to DECISIONS.md

**Add when:**
- Architectural choices that affect many future files or sessions
- Explicit rejections of common alternatives (so they don't get re-proposed)
- Conventions that apply to a specific module, integration, or workflow
- Anything you'd need to explain to a new team member

**Don't add when:**
- One-off implementation choices with no future implications
- Minor style or naming decisions
- Temporary workarounds

### DECISIONS.md vs CLAUDE.md filter

**Add to DECISIONS.md** if it was a deliberate choice between alternatives, affects multiple files, or future work depends on knowing WHY it was done.

**Keep in CLAUDE.md only** if it's an implementation detail, a spec, or can be derived by reading the code.

### Example entries
```
## [D-001] Sync: polling every 5 min instead of webhooks
Date: 2026-03-30 | Status: active | Ref: DEVLOG 2026-03-30
Context: needed Access → Supabase sync without complex infrastructure
Alternatives: webhooks (complex to debug), event queue (overkill at current scale)
Decision: polling every 5 min via cron job
Rationale: simpler to debug, sufficient for <50 clients, no extra infra
Conventions: sync script always logs timestamp + row count to sync_log table
Revisit when: client count >50 or sync delay >10 min becomes a complaint

## [D-002] proc_expense / proc_email: no Pydantic migration
Date: 2026-03-15 | Status: active | Ref: DEVLOG 2026-03-15
Context: proposed Pydantic for input validation in proc_expense and proc_email
Alternatives: Pydantic v2 models with to_legacy_dict()
Decision: rejected
Rationale: extra dependency, current dict structure sufficient, migration cost high
Conventions: keep flat dict pattern; validate manually at entry points only
Revisit when: a second validation failure caused by missing field in prod
```

## DEVLOG vs CLAUDE.md vs DECISIONS.md

| File | Purpose |
|------|---------|
| DEVLOG.md | Chronological journal: what happened when, what's next |
| DECISIONS.md | Permanent decision register: what was agreed, why, when to revisit |
| CLAUDE.md | Living reference: architecture, conventions, standards (things true NOW) |

When a decision is made:
1. Record discussion + outcome in DEVLOG.md `### Discussions`
2. If significant — add numbered entry to DECISIONS.md
3. If it affects how future code should be written — add to CLAUDE.md
4. If it's about user preferences — save to memory

## Creating DEVLOG.md for the first time

If no DEVLOG.md exists and the project already has work done:

1. Check git history (last 10–15 commits only) and existing files
2. Do not reconstruct history older than ~2 weeks — too much risk of inaccuracy
3. Mark ALL reconstructed entries with `[unverified — reconstructed from git/files]`
4. Ask the user to verify before treating as reliable

## Archiving old entries

When DEVLOG.md exceeds ~200 lines or spans more than 4 weeks:

1. Create `DEVLOG-YYYY-W##.md` (e.g. `DEVLOG-2026-W12.md`) with older entries
2. Keep current and previous week in main DEVLOG.md
3. Add note at bottom: `_Older entries archived in DEVLOG-2026-W12.md_`

Get current ISO week: `date +%G-W%V`

Don't archive automatically — suggest it when the threshold is reached.

## Audit (`/dev-tracker audit`)

Scans all project sources to find decisions and conventions not yet captured in DEVLOG.md or DECISIONS.md.

### What it scans
- Memory files (all `.md` files in project memory directory)
- CLAUDE.md files (workspace root + sub-projects)
- DECISIONS.md and DEVLOG.md current state

### How it reports
```
| Item                  | Found in        | Should go to    | Status  |
|-----------------------|-----------------|-----------------|---------|
| No nested components  | feedback.md     | DECISIONS.md    | MISSING |
| Money fields 130px    | CLAUDE.md:110   | DECISIONS.md    | COVERED |
| Exo 2 font            | CLAUDE.md:11    | CLAUDE.md only  | OK      |
```

**Status values:** `MISSING` | `COVERED` | `PARTIAL` | `OK`

### After the audit
1. Show the gap table to the user
2. Ask which MISSING items to add to DECISIONS.md
3. Add confirmed items with D-### numbering, context, rationale, revisit condition
4. Update DEVLOG.md with an audit entry noting what was added

Audit does NOT auto-update files without user confirmation.