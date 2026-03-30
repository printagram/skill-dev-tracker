---
name: dev-tracker
version: 1.2.0
description: Track development progress across sessions with DEVLOG.md — a structured project journal. Use this skill whenever the user starts a work session and wants to review what was done before, when they say "save progress", "update log", "what did we do", "session start", "session end", /dev-tracker, or when a significant block of work is completed and progress should be recorded. Also use proactively — after completing a major feature, fixing a complex bug, or making an architectural decision, remind the user to update the dev log. This skill is essential for any multi-session project to prevent context loss between conversations. NOTE: In Claude.ai (non-Code) environments, file system access is unavailable — present log entries as formatted text for the user to copy manually.
---

# Dev Tracker

A universal development journal that keeps track of what was done, what's planned, and what decisions were made — across sessions, across projects, across stacks.

## Why this matters

Each Claude Code session starts fresh. Without a persistent log, work done in previous sessions can be forgotten or misrecorded. This skill maintains a structured DEVLOG.md that serves as the single source of truth for project progress.

## Environment awareness

This skill behaves differently depending on the environment:

- **Claude Code** — full access to the file system. Read and write DEVLOG.md directly.
- **Claude.ai (chat)** — no file system access. Present the formatted log entry as a code block and ask the user to paste it into their file manually. Never pretend to write a file that doesn't exist in this context.

Always detect the environment at session start. If uncertain, ask: "Are you working in Claude Code, or should I give you text to copy into your DEVLOG.md?"

## The DEVLOG.md file

Location: project root (next to CLAUDE.md if it exists).

Format — newest entries first:

```markdown
# Dev Log

## YYYY-MM-DD — Brief session title
### Done
- what was actually completed (facts, not intentions)
- include file names for new files, feature names for features

### Discussions
- [Topic]: brief summary of what was discussed, alternatives considered
  → Outcome: what was agreed, what was deferred, what was rejected and why
- (omit this section if no significant discussions happened)

### Decisions
- key architectural or design decisions with brief reasoning
- conventions agreed upon; reference D-### from DECISIONS.md if logged there
- (omit this section if no decisions were made)

### Blockers
- what didn't work, what is blocked, what needs investigation
- include error names or symptom description, not full traces
- (omit this section if nothing is blocked)

### Next
1. concrete next step (highest priority first)
2. second next step
3. ...
```

**Writing style — good vs bad:**

| ❌ Too vague | ✅ Specific |
|---|---|
| "Worked on the frontend" | "Orders list: added status filter + row colors by priority" |
| "Fixed a bug" | "Fixed: KeyError in sync_access.py when order has no client_id" |
| "Chose Resend" | "Chose Resend over Gmail API — simpler setup, webhooks built-in" |
| "Continue working on email" | "1. Implement Ready Notification template 2. Add batch sending" |

## How to use this skill

### Session start

When a user begins working on a project (or explicitly asks to start a session):

1. Look for DEVLOG.md in the project root
2. If it exists, read the most recent entry
3. Check for any `[in-progress]` marker — if present, note that the previous session may have ended abruptly and the entry might be incomplete
4. Present a brief summary:
   ```
   Last session (YYYY-MM-DD): [1-2 line summary of what was done]
   Next steps were: [list from ### Next]
   Blockers: [list from ### Blockers, if any]
   What are we working on today?
   ```
5. If DEVLOG.md doesn't exist, create it with a header and note that this is the first tracked session

### During work — when to update

Update DEVLOG.md after:
- A major feature is completed (new page, new API, new component)
- A complex bug is fixed
- An important architectural decision is made
- A significant refactoring is done
- Multiple related changes that form a logical unit

Don't update after:
- Minor fixes (typos, small CSS tweaks)
- Exploratory work that didn't lead anywhere
- Individual file edits that are part of a larger task in progress

When updating mid-session, append to the current day's ### Done section. Don't create a new date entry.

### Handling abrupt session endings

At the start of a session that has significant work in progress, write a temporary marker to DEVLOG.md:

```markdown
## 2026-03-30 — [in-progress]
### Done
- ...work so far...
```

Replace `[in-progress]` with the real session title at session end. This way, if the session ends unexpectedly, the next session can see that the entry is incomplete.

### Reminding the user

After completing a significant block of work, suggest updating the log:
```
[summary of what was just completed]. Want me to save this to the dev log?
```

Keep reminders brief and non-intrusive. If the user declines or ignores, don't repeat for the same block of work.

### Session end

When the user says they're done, switching topics, or the conversation is clearly wrapping up:

1. Write or update the day's entry in DEVLOG.md with everything done in this session
2. Remove the `[in-progress]` marker if present; replace with a real session title
3. Log important discussions in `### Discussions` — alternatives considered, outcome reached
4. Include any decisions made (with reasoning) in `### Decisions`
5. If a decision is significant — add or update an entry in DECISIONS.md
6. Record any blockers — things that didn't work, open questions, known issues
7. Write concrete Next steps based on what was discussed
8. If architectural decisions were made, check if CLAUDE.md needs updating — add only new sections that help future sessions understand the project structure. Don't duplicate DEVLOG content. If new non-obvious files were created, add them to the `## Structure` section in CLAUDE.md.

### Writing style

- **Language**: match the project's language. Check CLAUDE.md for language preferences. If no preference is set, match the language the user is communicating in.
- **Brevity**: bullet points, not paragraphs. Each bullet should be one line.
- **Specificity**: name files, features, components — not "worked on the frontend" but "Orders list: added status filter + row colors"
- **Decisions need reasoning**: not just "chose Resend" but "chose Resend over Gmail API — simpler setup, webhooks built-in"
- **Next steps are concrete and numbered**: not "continue working on email" but "1. Implement Ready Notification template 2. Add batch sending from orders list"
- **Blockers are actionable**: describe the symptom and what was tried, not just "it didn't work"

### What NOT to put in DEVLOG

- Code snippets (that's what git is for)
- Debugging logs or error traces (summarize the problem, not the output)
- Detailed technical specs (those go in CLAUDE.md)
- Meeting notes or discussion transcripts
- Anything that can be derived from `git log`

### DEVLOG vs CLAUDE.md vs DECISIONS.md vs Memory

These serve different purposes:
- **DEVLOG.md** — chronological journal: what happened when, what's next
- **DECISIONS.md** — permanent decision register: what was agreed, why, and when to revisit
- **CLAUDE.md** — living reference: architecture, conventions, standards (things that are true NOW); should include a `## Structure` section listing only non-obvious files with their purpose — not a full file tree, just the ones whose role isn't clear from the name alone
- **Memory files** — cross-session context: user preferences, project metadata, feedback patterns

When a decision is made:
1. Record the discussion and outcome in DEVLOG.md `### Discussions` (historical record with date)
2. If the decision is significant or will affect future work — add a numbered entry to DECISIONS.md
3. If it affects how future code should be written, also add it to CLAUDE.md (reference)
4. If it's about user preferences or workflow, save to memory (personal context)

## The DECISIONS.md file

Location: project root (next to DEVLOG.md).

Purpose: permanent, searchable register of important decisions, agreements, and established conventions — independent of any specific session date. Unlike DEVLOG, entries here never get archived.

Format:

```markdown
# Decisions

## [D-001] Short decision title
Date: YYYY-MM-DD | Status: active | Ref: DEVLOG YYYY-MM-DD
Context: what situation or problem triggered this decision
Alternatives: what else was considered (briefly)
Decision: what was decided
Rationale: why this option over the others
Conventions: any specific rules or patterns that follow from this decision
Revisit when: concrete condition under which this should be reconsidered
```

**Status values:**
- `active` — in effect
- `superseded by D-###` — replaced by a newer decision
- `deferred` — discussed but not decided yet
- `rejected` — considered and explicitly ruled out

**When to add to DECISIONS.md** (not every decision needs to be here):
- Architectural choices that will affect many future files or sessions
- Explicit rejections of common alternatives (so they don't get re-proposed)
- Conventions and agreements that apply to a specific module, integration, or workflow
- Anything you'd need to explain to a new team member joining the project

**When NOT to add** (keep it in DEVLOG ### Decisions only):
- One-off implementation choices with no future implications
- Minor style or naming decisions
- Temporary workarounds (mark those as `deferred` at most)

**Example entries:**

```markdown
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

### `/dev-tracker decisions` command

Show all active entries from DECISIONS.md, grouped by status. Useful at session start when working on a module with existing decisions.

## Creating DEVLOG.md for the first time

If no DEVLOG.md exists and the project already has work done:

1. Check git history (last 10–15 commits only), existing files, and CLAUDE.md to reconstruct recent activity
2. Do not attempt to reconstruct history older than ~2 weeks — too much risk of inaccuracy
3. Create DEVLOG.md with entries for the work that can be identified
4. Mark ALL reconstructed entries with `[unverified — reconstructed from git/files]`
5. Ask the user to verify and correct before treating as reliable

## Handling multiple sessions per day

If there are multiple sessions on the same date, append to the existing day's entry rather than creating duplicates. Use time markers if needed:

```markdown
## 2026-03-30 — PDF reports + Email integration
### Done
- [morning] 7 PDF reports: Invoice, Quote, Creditnote, Delivery, Receipt, Print List
- [afternoon] Email sending via Resend (6 modes working)
- Webhook tracking for email status
```

## Archiving old entries

When DEVLOG.md exceeds ~200 lines or spans more than 4 weeks, archive older entries by ISO week:

1. Create `DEVLOG-YYYY-W##.md` (e.g. `DEVLOG-2026-W12.md`) with the entries for that week
2. Keep the current and previous week's entries in the main DEVLOG.md
3. Add a note at the bottom of DEVLOG.md: `_Older entries archived in DEVLOG-2026-W12.md, DEVLOG-2026-W11.md, ..._`

ISO week number: use `date +%G-W%V` on Linux/macOS to get the current week (e.g. `2026-W13`).

Don't archive automatically — suggest it to the user when the threshold is reached.

## Explicit commands

The user can invoke specific actions:

- `/dev-tracker` or "update log" — write/update today's DEVLOG entry with current session's work
- `/dev-tracker status` or "what did we do" — show the last entry and current next steps (including any blockers)
- `/dev-tracker decisions` — show all active entries from DECISIONS.md, grouped by status
- `/dev-tracker init` — create DEVLOG.md and DECISIONS.md (with limited reconstruction if project has history)
- `/dev-tracker diff` — summarize what changed since the last DEVLOG entry (based on git diff or modified files); useful when resuming after a break
- `/dev-tracker audit` — full structural audit: scan all sources, find gaps, suggest updates (see Audit section below)
- "session start" — show last session summary + blockers + relevant active decisions + ask for today's plan
- "session end" or "save progress" — write full session entry + update DECISIONS.md if needed + update Next steps + clear in-progress marker

## Audit (`/dev-tracker audit`)

A comprehensive scan that finds decisions, conventions, and specs that exist in project files but are not yet captured in DEVLOG.md or DECISIONS.md.

### What it scans

1. **Memory files** — all `.md` files in the project's memory directory (feedback, project, user, reference types)
2. **CLAUDE.md files** — workspace root + all sub-project CLAUDE.md files
3. **DECISIONS.md** — current state of the decision register
4. **DEVLOG.md** — current state of the dev log

### What it looks for

For each source file, extract:
- **Decisions/conventions** — patterns, rules, standards that affect future work
- **Architectural choices** — technology picks with alternatives considered
- **UI/UX patterns** — styling rules, layout standards, component behaviors
- **Data standards** — types, naming, validation rules
- **Process conventions** — workflows, procedures, communication rules

### How it reports

Present findings as a structured table:

```
| Item | Found in | Should go to | Status |
|------|----------|-------------|--------|
| No nested components | feedback_no_nested.md | DECISIONS.md | MISSING |
| Money fields 130px | CLAUDE.md:110 | DECISIONS.md D-007 | COVERED |
| Exo 2 font | CLAUDE.md:11 | CLAUDE.md only | OK (implementation detail) |
```

Status values:
- **MISSING** — not in DECISIONS.md or DEVLOG.md, needs to be added
- **COVERED** — already captured with a specific D-### reference
- **PARTIAL** — captured but missing important details
- **OK** — belongs where it is, doesn't need to be in DECISIONS.md (implementation details, specs that live in CLAUDE.md)

### What it does NOT do

- Does not auto-update files without user confirmation
- Does not add trivial implementation details to DECISIONS.md
- Does not duplicate CLAUDE.md content — only flags decisions and conventions that lack a DECISIONS.md entry

### Filtering: what belongs in DECISIONS.md vs CLAUDE.md only

Not everything is a "decision". Use this filter:

**Add to DECISIONS.md** if:
- It was a deliberate choice between alternatives
- It's a convention that affects multiple files or sessions
- Future you (or a new team member) would need to know WHY this was done
- It's been explicitly agreed upon and should not be changed without discussion

**Keep in CLAUDE.md only** if:
- It's an implementation detail (file paths, storage paths, font CDN URLs)
- It's a specification (exact PDF layout, exact column order)
- It can be derived by reading the code
- It has no meaningful alternatives — it's just how it works

### After the audit

1. Show the gap table to the user
2. Ask which MISSING items to add to DECISIONS.md
3. Add confirmed items with proper D-### numbering, context, rationale, and revisit condition
4. Update DEVLOG.md with an audit entry noting what was added
