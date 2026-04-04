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
| `/dev-tracker init` | Create DEVLOG.md + DECISIONS.md + TODO.md |
| `/dev-tracker decisions` | Show all active decisions grouped by status |
| `/dev-tracker diff` | Summarize changes since last DEVLOG entry |
| `/dev-tracker audit` | Scan all sources for undocumented decisions |
| `"session start"` | Show last session summary, ask for today's plan |
| `"session end"` | Write full session entry, update all files |

## Why this matters

Each Claude Code session starts fresh. Without a persistent log, work done in previous sessions can be forgotten or misrecorded. This skill maintains a structured DEVLOG.md that serves as the single source of truth for project progress.

## Environment awareness

- **Claude Code** — full file system access. Read and write files directly.
- **Claude.ai (chat)** — no file system access. Present formatted entries as code blocks for the user to paste manually. Never pretend to write a file that doesn't exist.

Always detect the environment at session start. If uncertain, ask: "Are you working in Claude Code, or should I give you text to copy into your files?"

## The TODO.md file

Location: project root. Purpose: a free-form personal buffer where the user captures ideas, tasks, and notes without worrying about structure or format.

**This is the user's file — Claude never edits it without explicit confirmation.**

### What the user writes

Anything, in any format, any language:
```
сделать экспорт в excel срочно
идея - мобилка для курьеров, подумать
роли пользователей - важно, блокирует клиентский портал
может интегрировать malta post?
fix the invoice PDF alignment
```

No rules, no sections, no formatting required.

### What Claude does at session start

1. Read TODO.md if it exists
2. Classify each item silently:
   - **Urgent/blocked** — something blocking current work
   - **Next step** — concrete task for near future
   - **Idea** — worth capturing but not yet decided
   - **Unclear** — needs a question before acting
3. Ask clarifying questions only for unclear items — one question at a time, not a list
4. Propose where each item belongs:
   - Concrete task → `### Next` in DEVLOG.md
   - Architectural idea → DECISIONS.md (as `deferred`)
   - Vague idea worth keeping → CLAUDE.md or DECISIONS.md
5. After user confirms, transfer items to their proper places
6. Mark processed items in TODO.md as `[done]` — do NOT delete automatically
7. When all items are marked `[done]`, ask: "TODO.md is fully processed — want me to clear it?"

### Rules

- Claude reads TODO.md but never restructures or reformats it
- Claude never deletes items without user confirmation
- If TODO.md doesn't exist, don't create it automatically — only on `/dev-tracker init` or explicit request
- If TODO.md is empty or has only `[done]` items, skip it silently at session start

### Example processing

User's TODO.md:
```
роли пользователей срочно
идея мобилка для курьеров
fix invoice pdf
```

Claude at session start:
```
Found 3 items in TODO.md:
- "роли пользователей срочно" → add to ### Next (top priority)?
- "fix invoice pdf" → add to ### Next?
- "идея мобилка для курьеров" → save as deferred idea in DECISIONS.md?

One question: for "мобилка для курьеров" — is this something to plan now,
or just capture as a future idea?
```

After confirmation, TODO.md becomes:
```
[done] роли пользователей срочно
[done] идея мобилка для курьеров
[done] fix invoice pdf
```

## The DEVLOG.md file

Location: project root. Newest entries first.
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
- Anything that can be derived from `git log`

## Session workflow

### Session start

1. Read TODO.md if it exists — process items (see TODO.md section above)
2. If TODO.md has more than 5 `[done]` items, suggest: "TODO has N completed items — archive them?"
3. Read DEVLOG.md — find the most recent entry
4. Check for `[in-progress]` marker — note if previous session ended abruptly
5. Present summary:
```
   Last session (YYYY-MM-DD): [1-2 line summary]
   Next steps: [list from ### Next]
   Blockers: [if any]
   TODO items to discuss: [if any unprocessed]
   What are we working on today?
```
6. If DEVLOG.md doesn't exist, create it and note this is the first tracked session

### During work — when to update

**Update after:** major feature completed, complex bug fixed, architectural decision made, multiple related changes forming a logical unit.

**Don't update after:** minor fixes, exploratory work that led nowhere, individual edits part of a larger task.

When updating mid-session, append to the current day's `### Done`. Don't create a new date entry.

### Handling abrupt session endings

Write a temporary marker at session start:
```
## 2026-03-30 — [in-progress]
### Done
- ...work so far...
```
Replace with real title at session end.

### Reminding the user

After a significant block of work:
```
[summary of what was completed]. Want me to save this to the dev log?
```
If the user declines, don't repeat for the same block.

### Switching projects mid-session

When the user moves to a different project directory during a session:

1. **Do NOT call session end** for the previous project automatically
2. Keep track of which projects were touched during the session
3. When work begins in the new project, read its DEVLOG.md/TODO.md as if starting a mini-session
4. At actual session end (user says "session end"), update DEVLOG.md for **each project** that was worked on
5. Each project's DEVLOG gets only the entries relevant to that project — don't mix

Example: if the user works on `pdfx_app` then switches to `tex-tools`, session end writes to both DEVLOGs independently.

### Session end

1. Write or update the day's entry **for each project touched during the session**
2. Remove `[in-progress]`; replace with real session title
3. Log discussions in `### Discussions`
4. Log decisions with reasoning in `### Decisions`
5. If decision is significant — add to DECISIONS.md
6. Record blockers
7. Write concrete numbered Next steps
8. Check if CLAUDE.md needs updating — add only new sections, don't duplicate DEVLOG content

### Multiple sessions per day

Each session gets its own `## heading` with a time marker. Don't merge sessions — it makes the log harder to read and creates conflicts when updating.

```
## 2026-03-30 14:00 — Email integration
### Done
- Email sending via Resend (6 modes working)
...

---

## 2026-03-30 09:00 — PDF reports
### Done
- 7 PDF reports: Invoice, Quote, Creditnote, Delivery, Receipt, Print List
...
```

Use `---` separator between same-day sessions for clarity.

## The DECISIONS.md file

Location: project root. Permanent register of important decisions — entries never get archived.
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

**Add when:** architectural choices affecting many future files, explicit rejections of common alternatives, conventions applying to a specific module, anything a new team member would need explained.

**Don't add when:** one-off implementation choices, minor naming decisions, temporary workarounds.

### DECISIONS.md vs CLAUDE.md

**DECISIONS.md** — deliberate choice between alternatives; future work depends on knowing WHY.

**CLAUDE.md only** — implementation detail, spec, or something derivable from reading the code.

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
```

## File roles summary

| File | Who writes | Purpose |
|------|-----------|---------|
| TODO.md | User | Free-form buffer: ideas, tasks, notes |
| DEVLOG.md | Claude | Chronological journal: what happened, what's next |
| DECISIONS.md | Claude | Permanent decision register: what was agreed and why |
| CLAUDE.md | Claude | Living reference: architecture, conventions, standards |

## Safety: destructive operations

Before performing any irreversible action during a tracked session (deleting files, removing credentials, migrating data, force-pushing), **verify the replacement or backup is working first**. Never remove the old thing until the new thing is confirmed.

Example of what NOT to do: remove API keys from config files before verifying that .env loading works. Always: create .env → test that scripts work via .env → only then clear configs.

This applies to all destructive operations, not just credentials. If in doubt, ask the user before proceeding.

## Creating files for the first time

`/dev-tracker init` creates DEVLOG.md + DECISIONS.md + TODO.md with headers.

If project already has work done:
1. Check git history (last 10–15 commits only) — **only if the project is tracked by git**
2. If the project is untracked or has no meaningful git history, start with a clean slate — don't attempt to reconstruct
3. Don't reconstruct history older than ~2 weeks
4. Mark reconstructed entries with `[unverified — reconstructed from git/files]`
5. Ask user to verify before treating as reliable

## Archiving old entries

When DEVLOG.md exceeds ~200 lines or spans more than 4 weeks:

1. Create `DEVLOG-YYYY-W##.md` with older entries
2. Keep current and previous week in main DEVLOG.md
3. Add note: `_Older entries archived in DEVLOG-2026-W12.md_`

Get ISO week: `date +%G-W%V`

Suggest archiving — never do it automatically.

## Audit (`/dev-tracker audit`)

Scans all project sources to find decisions and conventions not yet captured.

**Scans:** memory files, CLAUDE.md files, DECISIONS.md, DEVLOG.md, TODO.md.

**Reports:**
```
| Item                  | Found in        | Should go to    | Status  |
|-----------------------|-----------------|-----------------|---------|
| No nested components  | feedback.md     | DECISIONS.md    | MISSING |
| Money fields 130px    | CLAUDE.md:110   | DECISIONS.md    | COVERED |
| Exo 2 font            | CLAUDE.md:11    | CLAUDE.md only  | OK      |
```

**Status values:** `MISSING` | `COVERED` | `PARTIAL` | `OK`

After audit: show table, ask which MISSING items to add, add confirmed items with D-### numbering, update DEVLOG with audit entry.

Audit never auto-updates files without user confirmation.
