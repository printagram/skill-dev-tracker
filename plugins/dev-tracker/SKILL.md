---
name: dev-tracker
description: Maintain a structured development journal for software projects that preserves context across Claude Code sessions. Use this skill whenever the user mentions session start, session end, save progress, DEVLOG, DECISIONS, ADR, architecture decision record, project journal, development log, cross-session memory, what was done previously, or asks to resume work on a tracked project — even if they do not explicitly name the skill. Also use when the user wants to initialize tracking files, audit undocumented conventions, or maintain continuity across multiple related work sessions on the same codebase.
argument-hint: "[status | context | init | decisions | diff | audit]"
version: 3.3.0
---

# Dev Tracker

A production-ready development journal for Claude Code that preserves project context across sessions without adding unnecessary process overhead.

Use this skill for tracked software work where losing session context is costly. The skill keeps a clear boundary between:
- raw user notes
- chronological progress logs
- permanent architectural decisions
- living implementation conventions

## What this skill manages

| File | Purpose | Ownership |
|------|---------|-----------|
| `TODO.md` | Raw notes, ideas, rough tasks, reminders | User-owned |
| `DEVLOG.md` | Chronological session log: what happened, blockers, next steps | Claude-managed |
| `DECISIONS.md` | Important decisions with rationale and revisit conditions | Claude-managed |
| `CLAUDE.md` | Conventions, architecture notes, standards, recurring implementation facts | Skill-managed if skill-created; append-only if team-authored — see warning below |

When a project uses Claude Code's native `memory/` store, the "Claude-managed" rows above defer to it — see "Coexistence with Claude Code auto-memory" below before treating `DEVLOG.md` / `DECISIONS.md` as the system of record.

### ⚠️ CLAUDE.md warning

In Claude Code, `CLAUDE.md` is auto-loaded into the context of every session for that project. This means every byte written to `CLAUDE.md` is a byte of context tax paid on every future run.

Keep `CLAUDE.md` lean. Only write durable, high-signal content:
- architecture invariants that will not change for months
- naming and layout conventions a new contributor must know
- stable operational facts (e.g. "database uses field `idx_Order`, not `id_Order`")

Do NOT write to `CLAUDE.md`:
- session narratives (those go in `DEVLOG.md`)
- one-off implementation details
- anything that will become obsolete within a sprint
- decisions with rationale (those go in `DECISIONS.md`, not here)

If a skill-managed `CLAUDE.md` grows past ~150 lines, prompt the user to review and trim it. Never fire this trim prompt at a team-authored `CLAUDE.md` — see ownership below.

### CLAUDE.md ownership: never clobber human-authored instructions

`CLAUDE.md` is frequently a hand-written, version-controlled project brief — not a file this skill created. Before editing it, decide which kind it is:

- **Skill-managed** — scaffolded by this skill itself (e.g. from the `CLAUDE.md` template in `references/templates.md`), holding only the Architecture / Conventions / Operational Notes scaffold. Safe to append durable conventions to, and safe to suggest trimming.
- **Team-authored** — checked into the repo, containing project instructions, onboarding, or rules clearly written by a human. Treat it as read-mostly: append a durable convention only on explicit user request, never reorder or rewrite it, and never apply the ~150-line trim prompt to it.

When unsure, assume team-authored. Adding a convention is reversible; silently restructuring someone's project brief is not.

## Coexistence with Claude Code auto-memory

Claude Code has its own persistent memory: a `memory/` directory of topic files plus a `MEMORY.md` index that is auto-loaded every session. When a project already uses it, that store — not this skill's files — is the system of record. Do not run a second, competing journal beside it; parallel journals produce duplicate decisions and conflicting histories.

Detect it: look for `memory/MEMORY.md` (often under `.claude/.../memory/`) or a `taskList` path in the project registry.

When auto-memory is present:
- **Decisions of record live in the memory store, not `DECISIONS.md`.** Before writing any `D-###`, search the memory store. If the choice is already captured there, reference it — do not copy it into `DECISIONS.md`. Only stand up a local `DECISIONS.md` if the user explicitly wants ADR-style entries alongside memory.
- **Chronology may already live in a memory file** (e.g. a `project_chronology.md` or task-list file). Append session history there, in the project's established format, instead of starting a parallel `DEVLOG.md` — unless the user wants both.
- **The memory index is itself context tax.** `MEMORY.md` is auto-loaded every session, exactly like `CLAUDE.md`, so the same lean discipline applies: one-line index entries, detail pushed into topic files. If `MEMORY.md` nears its size limit, flag it and offer to trim index lines — overflow there silently truncates what loads.
- **Audit reflects this.** During `/dev-tracker audit`, a convention already recorded in the memory store is `COVERED (external)`, never `MISSING` (see the Audit workflow).

When no auto-memory store exists, this skill's four files are the system of record and everything below applies normally.

## Coexistence with Claude Code native tasks

Claude Code also has an in-session task list — `TaskCreate` / `TaskUpdate` / `TaskList`, status `pending → in_progress → completed`. It is ephemeral working memory for the *current* session: it disappears when the session ends and is never the system of record. It does not replace this skill's durable, cross-session task tracking (`TODO.md`, the last DEVLOG entry's `### Next`, or the registry `taskList` file).

Treat them as two layers, not two competitors:
- **Durable layer** (this skill's files / the memory store) — the backlog and history that survive across sessions. Authoritative; it sits in the source-of-truth priority. The ephemeral list does not.
- **Live layer** (native tasks) — the checklist for executing *today's* work. Fast, visible, harness-integrated.

How they hand off:
- **At session start**, optionally seed the native list from the durable layer: the last DEVLOG `### Next`, plus the registry `taskList` items you actually plan to touch today. Do not import the whole backlog — only today's slice, or the live list becomes noise.
- **During work**, drive progress through the native list (`in_progress` / `completed`). Do not mirror every native task back into the files mid-session — that is exactly the churn the "Anti-fragmentation" rule prevents.
- **At session end**, fold the live list back into the durable record: completed native tasks become `### Done` lines and get marked done / archived in the registry `taskList` file (or `TODO.md`); still-pending ones flow into `### Next` and the backlog. Then let the native list go — it would vanish anyway.

Never treat the native task list as the place a decision, blocker, or backlog item is *recorded*. It is where work is *run*, not where it is *remembered*.

## Operating rules

### Source of truth priority

When sources disagree, use this priority:

1. User's current instruction
2. Actual repository state and command output
3. `DECISIONS.md`
4. `DEVLOG.md`
5. `TODO.md`

Do not let older notes override current user intent or current code.

### Environment awareness

Prefer direct file updates when running inside Claude Code with filesystem access.

If direct file access is unavailable, produce complete ready-to-paste content blocks instead of claiming files were updated.

### Language

Write tracker entries in the user's working language. If the conversation is in Russian, German, or any language other than English, all prose in `DEVLOG.md`, `DECISIONS.md`, `CLAUDE.md`, and the session start summary must be in that language. Keep code identifiers, file paths, git commands, commit hashes, and technical keys (like `D-001`, `Status: active`, field names) in English even then — they are universal references.

Never translate the user's `TODO.md` contents; that file is user-owned.

### Date and timezone

Use ISO format `YYYY-MM-DD HH:MM` for all timestamps. Use the user's local time by default, and include a timezone abbreviation (`CET`, `UTC`, `EEST`) once per session entry in the session title line when the project may be worked on from multiple machines or timezones.

Example session title:
```markdown
## 2026-04-17 14:30 CET — LED controller firmware update
```

If uncertain which timezone the user is in, ask once, then record the answer in `CLAUDE.md` under Operational Notes.

### Trigger policy

#### Hard triggers — always act
Run the relevant dev-tracker workflow immediately when:
- the user says `session start`
- the user says `session end`
- the user says `save progress`
- the user asks what was done previously
- the user asks to update the development log
- the user uses `/dev-tracker`
- the user uses `/dev-tracker status`
- the user uses `/dev-tracker context`
- the user uses `/dev-tracker init`
- the user uses `/dev-tracker decisions`
- the user uses `/dev-tracker diff`
- the user uses `/dev-tracker audit`

#### Soft triggers — suggest, do not force
Offer to log when a meaningful unit of work completes (see "What counts as meaningful" below).

Do not interrupt rapid iterative debugging or micro-edit loops unless the user explicitly wants active checkpointing.

### What counts as meaningful

Update the log after:
- a feature is completed
- a bug is fixed
- a refactor lands
- a significant decision is made
- a group of related changes forms one coherent milestone

Do not create noisy entries for:
- tiny edits
- exploratory dead ends
- single-line changes that are part of a larger unfinished block

This is one list used for two purposes: it decides when to **suggest** logging (soft trigger) and what to actually **write down** when the user agrees.

## Session start workflow

When the user says `session start`, starts work, or asks what happened previously:

1. Detect the project context
2. Read `TODO.md` if it exists
3. Read the latest entry in `DEVLOG.md` if it exists
4. Read `DECISIONS.md` if it exists and only surface relevant active decisions
5. Resolve the requested project — accept either a registry alias OR a folder name (e.g. `session start WebUI` → the `supabase-webui` dir). Match case-insensitively against registry aliases first, then against directory names under the workspace.
6. If git is available, inspect recent commits for context
7. **Drift sniff (lightweight only).** If git is available, compare the date of the latest `DEVLOG.md` entry against `git log`. If newer commits exist, add one line to the summary — "⚠ DEVLOG behind git by ~N days / M commits — run `/dev-tracker audit` to reconcile" — and stop there. Do NOT verify each task's status inline; full status-drift reconciliation is the `audit` command's job, not session start's.
8. Present a concise session-start summary
9. Write an in-progress marker (see "In-progress marker" section)
10. Ask what today's focus is only if it is not already obvious from the user's request
11. Optionally seed the in-session task list (`TaskCreate`) from the last `### Next` and today's slice of the registry task list — see "Coexistence with Claude Code native tasks"

### TODO handling

Treat `TODO.md` as a raw inbox owned by the user.

Read it, classify items silently into:
- urgent
- next
- idea
- unclear

Then (if the project uses Claude Code auto-memory, target the memory store instead of `DECISIONS.md` / `DEVLOG.md` — see "Coexistence with Claude Code auto-memory"):
- propose moving concrete near-term work into `DEVLOG.md` under `### Next`
- propose saving architectural ideas into `DECISIONS.md` as `deferred` when appropriate
- propose moving stable conventions into `CLAUDE.md`
- ask at most one clarification question at a time for unclear items
- do not interrogate the user item by item — batch clarifications and only ask when the ambiguity actually blocks progress

Do not rewrite, reorganize, translate, or delete `TODO.md` automatically.

Never mark an item as `[done]` unless one of these is true:
- the user explicitly confirms it should be marked done
- the item was clearly completed in the current session AND the user asked to save or end the session

If all items are marked `[done]`, ask whether the user wants the file cleared. Never clear it automatically.

### Session start output format

Use this shape:

```markdown
## Session Start — YYYY-MM-DD

Last session:
- one or two lines only

Active decisions:
- only if directly relevant

Recent commits:
- short list only if useful

Suggested next steps:
1. ...
2. ...

Open blockers:
- only if real blockers exist
```

If this is the first tracked session, state that clearly and initialize tracking files if the user asked for setup.

## During work

### Hard save: `save progress`

If the user says `save progress` (or any clear equivalent like "write down what we did", "log this"), act immediately — do not offer, do not ask for confirmation. Append to the current session entry if one is open, otherwise create one.

### Soft save: suggesting a checkpoint

After a meaningful block completes and the user pauses, briefly summarize progress and suggest logging it.

Preferred phrasing:
- "I can save this to the dev log."
- "This looks like a good checkpoint for the log."

Suggest once per block. If the user ignores or declines, do not repeat for the same block.

### Anti-fragmentation: one entry per session

A single work session produces exactly one DEVLOG entry. Multiple saves during the same session append to that entry — they do not create new entries.

A session boundary is defined only by these explicit triggers:
- user says `session start` or `session end`
- more than ~12 hours since the last activity (clear break — morning after, Monday after a weekend)
- user switches to a different project alias

Do not treat a long gap within the same workday (lunch, a meeting, a two-hour debugging pause) as a new session. The goal is a clean daily log, not per-commit journaling.

If you find yourself about to write a second entry on the same day for the same project without one of the triggers above, stop and append instead.

## Session end workflow

When the user says `session end`, asks to save progress, or requests a structured summary:

> If the project uses Claude Code auto-memory (see "Coexistence with Claude Code auto-memory"), steps 5–8 route to the memory store: append the session chronology to the project's memory chronology file and reference decisions already recorded there, instead of writing a parallel `DEVLOG.md` / `DECISIONS.md`.
>
> If you drove this session through Claude Code native tasks (`TaskList`), reconcile them before writing (see "Coexistence with Claude Code native tasks"): completed tasks become `### Done` lines, still-pending ones flow into `### Next`.

1. If git is available, run `git log --oneline` since session start to collect commits
2. If git is available, run `git diff --stat` to count files and lines changed
3. If git is available, cross-reference commits with the durable task list (registry `taskList` / `TODO.md`, not the ephemeral native list) to identify completed tasks
4. Generate a structured summary before writing the entry
5. Update or create the current `DEVLOG.md` session entry (replace the `[in-progress]` marker if present)
6. If a durable task-list file (registry `taskList`) exists, move completed tasks to archive
7. Add any important architectural decisions to `DECISIONS.md`
8. Update `CLAUDE.md` only with durable conventions or reference material (respect the size warning)
9. Keep next steps concrete and ordered by priority
10. Log any important data operations performed, such as DB migrations, backfills, or manual data fixes

### No-git fallback

If git is unavailable or the project is not a git repository:
- rely on current conversation context, repository files, and existing logs only
- do not attempt fake reconstruction
- do not invent commits, diffs, or file-change counts
- still write a normal `DEVLOG.md` entry based on confirmed work
- for `/dev-tracker diff`, state clearly that git is unavailable, then summarize changes by reading the existing `DEVLOG.md` tail and comparing to the current conversation — do not fabricate file-level diffs

### Session end entry format

Use this format in `DEVLOG.md`:

```markdown
## YYYY-MM-DD HH:MM TZ — Session title

### Done
- concrete completed work
- include feature names, modules, commands, or files when useful

### Discussions
- topic: summary
  -> outcome: what was decided, deferred, or rejected

### Decisions
- brief decision summary
- reference `D-###` if added to `DECISIONS.md`

### Blockers
- short blocker descriptions only
- include symptom or failing area, not raw logs

### Next
1. top priority next step
2. second next step
```

Omit empty sections.

If multiple sessions occur on the same date, create separate time-stamped entries rather than merging them.

### Writing the session title

The session title is the first thing a future reader scans to find past work. Make it concrete.

The title should name one of:
- the main feature that was completed ("LED controller firmware update")
- the main problem that was solved ("ODBC field name mismatch in sync")
- the main decision that was made ("chose polling over webhooks for Access sync")

Avoid generic titles:
- ❌ "Work session"
- ❌ "Updates"
- ❌ "Progress"
- ❌ "Daily log"

If the session genuinely covered several unrelated things, pick the most significant one and list the others in `### Done`. Do not stuff the title with multiple topics.

### Recurring blockers

At session end, before writing the new entry, scan the last ~5 DEVLOG entries. If a blocker in the current session also appeared in one of them:
- mark it explicitly as recurring: `- [recurring since YYYY-MM-DD] <blocker description>`
- if it has recurred three times or more, suggest opening a dedicated `D-###` entry to capture the root cause or an explicit decision to accept the cost

This turns DEVLOG from a journal into a diagnostic signal — repeated friction is the loudest flag that something needs a structural fix.

### Correcting past entries

If a past `DEVLOG.md` or `DECISIONS.md` entry is discovered to be wrong, do not silently rewrite it. Preserve the original record and append a correction:

```markdown
### Correction (YYYY-MM-DD)
The earlier claim that X was completed is incorrect. Actual state: Y.
```

For `DECISIONS.md`, use the `superseded by D-###` status instead of editing the old decision text — add a new `D-###` entry that explains the reversal.

This rule exists because silent rewrites destroy the audit trail that makes the journal useful in the first place.

### Git-based session summary requirements

If git is available, include as many of these as are grounded in real output:
- number of commits during the session
- short high-level change summary
- files touched, preferably grouped by area
- completed tasks inferred from commits or task list
- work still in progress

If git is not available, skip git-specific claims entirely.

## Writing quality rules

Good entries are:
- factual
- specific
- brief
- useful to a future session

Prefer:
- "Added order status filter to `orders_table.tsx`"
- "Fixed null handling in `sync_access.py` when `client_id` is missing"
- "Chose polling every 5 minutes over webhooks for initial sync"

Avoid:
- "Worked on frontend"
- "Fixed a bug"
- "Made improvements"

Do not put code snippets, stack traces, or long technical specs in `DEVLOG.md`. If a code reference is essential, link to the file and line or the commit hash instead.

## Decision rules

Add an entry to `DECISIONS.md` only if the choice:
- affects multiple future files or modules
- compares real alternatives
- creates a reusable convention
- would be important for a new contributor to understand

Do not add one-off implementation details, tiny naming choices, or temporary experiments.

### When in doubt, do not add

If none of the four criteria above is clearly satisfied, do not write to `DECISIONS.md`. Instead, mention the choice in the current DEVLOG entry under `### Decisions` and offer to formalize it:

> "This looks like it might be a decision worth recording. Should I add it to DECISIONS.md as D-###?"

Wait for the user to confirm. This asymmetry is intentional: a missing decision can always be added later via `/dev-tracker audit`, but a polluted `DECISIONS.md` silently loses its value — every false entry dilutes the file and trains future readers to skim past it.

The cost of under-logging is recoverable. The cost of over-logging is not.

### Decision entry format

Use sequential IDs like `D-001`, `D-002`, and so on.

```markdown
## [D-001] Short decision title
Date: YYYY-MM-DD | Status: active | Ref: DEVLOG YYYY-MM-DD HH:MM

Context: what situation triggered the decision
Alternatives: what else was considered
Decision: what was chosen
Rationale: why this option won
Conventions: follow-on implementation rules, if any
Revisit when: clear condition for reconsideration
```

Allowed statuses:
- `active`
- `deferred`
- `rejected`
- `superseded by D-###`

## Multi-project workflow

If a `.dev-tracker-projects.json` file exists, use it to resolve aliases. See `references/project-registry.example.json`.

The registry supports an optional `taskList` field pointing to a memory-based task list file. When present, the session start summary includes a full task table from that file.

For `session start [ProjectName | folder]`:
1. resolve the argument as a registry alias first; if no alias matches, treat it as a folder name and match (case-insensitive) against directory names under the workspace
2. switch into the project directory
3. read that project's log files, task list, and `CLAUDE.md`
4. if git is available, run `git log --oneline -10` for recent context, and apply the lightweight drift sniff (step 7 of "Session start workflow") — flag if `DEVLOG.md` is behind git and point to `/dev-tracker audit`
5. present the session start summary scoped to that project, including task table when present
6. summarize only that project

Without an argument, detect the project from the current working directory.

### Monorepo / multiple projects in one repository

The rule "never mix work from different repositories into one devlog entry" does not help inside a monorepo. Inside one repository with multiple logical projects (e.g. `/apps/api`, `/apps/web`, `/packages/shared`), each project should have its own `DEVLOG.md`, `DECISIONS.md`, `TODO.md` in its own directory. Use the registry `dir` field to scope each alias.

If work in one session genuinely spans multiple monorepo projects, write a brief entry in each project's `DEVLOG.md` and cross-reference them:

```markdown
### Done
- implemented shared auth helper (see also DEVLOG in /apps/api, same timestamp)
```

Never write one session spanning multiple projects into a single consolidated log.

## Concurrent sessions safety

If two Claude Code instances may touch the same tracker files simultaneously (common when the user runs work in parallel Terminal windows or Parallels VMs):

- commit tracker files (`DEVLOG.md`, `DECISIONS.md`, `CLAUDE.md`) to git as separate, isolated commits rather than mixing them with code changes
- use commit messages like `chore(devlog): session 2026-04-17 CET` so merges remain clean
- at session start, quickly check `git status` on tracker files — if another instance modified them since last session, read the latest version before appending
- never force-push or rewrite history on tracker files

## In-progress marker

At session start, write a temporary marker in `DEVLOG.md`:

```markdown
## YYYY-MM-DD HH:MM TZ — [in-progress]
### Done
- ...work so far...
```

Replace `[in-progress]` with a real session title at session end. If the next session finds an `[in-progress]` marker, note that the previous session ended abruptly and carry over any incomplete work before writing the new marker.

## Archiving

When `DEVLOG.md` exceeds approximately 500 lines or spans more than 8 weeks:

1. Suggest archiving older entries to `DEVLOG-YYYY-W##.md`
2. Keep current and previous week in the main file
3. Add a note: `_Older entries archived in DEVLOG-2026-W12.md_`
4. Never archive automatically — always ask the user first

Get ISO week: `date +%G-W%V`

If the user consistently declines archiving across three consecutive suggestions, stop suggesting and note in `CLAUDE.md` that the user prefers a long flat log.

## Audit workflow

Run or strongly suggest `/dev-tracker audit` when:
- the user explicitly requests an audit
- the user seems confused about past decisions
- the discussion is about architecture and prior choices matter
- the project has accumulated several sessions and hidden conventions are likely
- a new contributor is onboarding to a tracked project
- a major refactor is planned that may invalidate past decisions

`audit` is the heavyweight reconciliation command — it does the full cross-check that session start deliberately skips. Run it on demand, not on every session.

During audit:
1. scan `DEVLOG.md`, `DECISIONS.md`, `CLAUDE.md`, `TODO.md`, and any task-list / external memory store referenced by the registry (`taskList` field) or present at a known path (e.g. `.claude/.../memory/`)
2. **Status-drift detection (do this first — it is the highest-value check).** For every task or item whose notes claim a status (done / pending / planned / blocked / not-started), verify it against actual repo state: `git log` for the feature's commits + a `grep`/file check for the artifact. Report each mismatch — most commonly "marked pending but already shipped" (and its inverse, "marked done but no trace"). For mass scans, fan out parallel read-only search agents grouped by feature. Verdicts: `SHIPPED` / `PARTIAL` / `NOT-FOUND`, each with a commit hash or file path as evidence.
3. **Git-gap.** If `DEVLOG.md`'s latest entry predates recent commits, report the gap and offer to backfill compressed daily entries reconstructed from `git log` + memory.
4. identify repeated conventions, explicit choices, or rejected alternatives (undocumented-decisions scan)
5. compare them against existing decision entries — **and before marking any item `MISSING`, check the external memory store too.** If the decision/convention is already recorded there, status is `COVERED (external)`, not `MISSING`. Skipping this produces false-positive promotions that pollute `DECISIONS.md`. If a project keeps its decisions of record in an external store, say so plainly rather than re-adding duplicates.
6. return findings first, as separate tables: **Status Drift** (from steps 2-3) and **Undocumented Decisions** (from steps 4-5)
7. ask which items to fix / formalize
8. only then update files — and when fixing drift, correct the stale source (task list, index, note) to match verified reality

Never auto-write audit findings without user confirmation.

See `references/audit-heuristics.md` for detailed heuristics and the audit report table format.

## Safety rule

Before destructive operations:
- verify the replacement is working first
- do not delete the old path, config, credentials, or data until the new setup is confirmed

If the action is irreversible and the replacement has not been validated, stop and ask.

### Never write secrets or personal data into tracker files

`DEVLOG.md`, `DECISIONS.md`, and `CLAUDE.md` are often committed to git and re-read every session — the wrong home for anything sensitive. When logging work, and especially data operations like migrations, backfills, or manual fixes, record *what* happened, never the sensitive payload:

- no credentials, API keys, tokens, connection strings, or `.env` values
- no customer or employee PII (names tied to contact details, emails, phone numbers, addresses, government or tax IDs)
- no financial record contents (account numbers, card data, invoice lines tied to a person)

Prefer counts and references over contents: "backfilled 1659 products with empty `price_item`", not the rows; "fixed the payment on order #25647", not the customer's bank details. If a sensitive value is essential to reproduce the work later, point to where it lives (a secret manager, a row id) instead of pasting it in.

## Commands — `$ARGUMENTS` dispatch

This skill *is* the `/dev-tracker` command (in Claude Code a skill and a slash command are the same mechanism). When the user types `/dev-tracker`, the skill runs, and whatever follows the command name arrives as `$ARGUMENTS` — e.g. `/dev-tracker audit` → `$ARGUMENTS` = `audit`. Dispatch on it deterministically; do not wait to infer the subcommand from surrounding prose.

| `$ARGUMENTS` | Action | Where the detail lives |
|--------------|--------|------------------------|
| _(empty)_ | Write or update the current session entry | "During work" → Hard save |
| `status` | Show last entry, blockers, next steps (read-only) | below |
| `context` | Full read-only context restore — does NOT write an `[in-progress]` marker | below |
| `init` | Create missing tracking files | `references/templates.md` |
| `decisions` | Show active decisions grouped by status | `DECISIONS.md` |
| `diff` | Summarize changes since the last entry | "No-git fallback" when git is absent |
| `audit` | Heavyweight reconciliation | "Audit workflow" |

If `$ARGUMENTS` matches none of the above, treat the whole string as free-text intent (e.g. `/dev-tracker resume the webui work`) and pick the closest workflow rather than erroring. The natural-language hard/soft triggers under "Trigger policy" still apply when the skill is invoked without a slash command.

Each subcommand in detail:

### `/dev-tracker`
Write or update the current session entry.

### `/dev-tracker status`
Show the last relevant entry, blockers, and next steps.

### `/dev-tracker context`
Full context restore for returning to a project after a break. Outputs, in order:
1. Last session summary (title, Done, Next) — one block
2. Currently active decisions (status `active` only, grouped by area if many)
3. Open blockers from the last 3 sessions, with recurring ones flagged
4. Any `[in-progress]` marker left from an abrupt session
5. A short "where you left off" paragraph synthesizing 1–4 into plain prose

Use this when the user says things like "I'm back after two weeks, bring me up to speed" or "remind me where we are on this project". Difference from `session start`: `context` is read-only and does not write an `[in-progress]` marker — it is for orientation, not for starting work.

### `/dev-tracker init`
Create initial tracking files if missing:
- `DEVLOG.md`
- `DECISIONS.md`
- `TODO.md`

### `/dev-tracker decisions`
Show active decisions grouped by status.

### `/dev-tracker diff`
Summarize meaningful changes since the last devlog entry. Use git if available; otherwise fall back to comparing the last DEVLOG entry against the current conversation state (see "No-git fallback").

### `/dev-tracker audit`
Heavyweight reconciliation. Does three things, in order: (1) **status drift** — verify every claimed task status against git + code, surface "marked pending but shipped" and its inverse; (2) **git-gap** — flag if DEVLOG trails recent commits, offer backfill; (3) **undocumented decisions** — scan for conventions/choices missing from `DECISIONS.md`, but mark `COVERED (external)` rather than `MISSING` when they already live in an external memory store. Report findings first as separate tables. Never auto-write without user confirmation.

## Implementation notes

When initializing files, use the templates from `references/templates.md`.

When a project registry is needed, follow `references/project-registry.example.json`.

When auditing for undocumented decisions, use the heuristics in `references/audit-heuristics.md`.

For concrete examples of well-formed tracker files, see the `examples/` directory.

Keep behavior lightweight. The goal is reliable continuity, not bureaucratic overhead.
