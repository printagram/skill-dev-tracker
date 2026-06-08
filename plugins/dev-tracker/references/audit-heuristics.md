# Audit Heuristics

Detailed heuristics for the **undocumented-decisions** half of `/dev-tracker audit`.

The full audit procedure — step order, status-drift detection, and the external-memory check — lives in `SKILL.md` under "Audit workflow". The status-drift verdicts (`SHIPPED` / `PARTIAL` / `NOT-FOUND`) are defined there. This file covers only how to recognize a hidden decision and how to report decision-coverage findings. Keep the two in sync; if they ever disagree, `SKILL.md` wins.

## Likely a decision

Treat an item as a likely decision if it:
- defines a convention repeated across multiple files
- explicitly chooses one approach over alternatives
- sets a policy for integrations, deployment, data flow, or architecture
- creates a stable module-level or repo-level rule
- rejects a common alternative with reasoning
- locks down a contract (API shape, DB schema, file layout) that other parts of the system depend on

## Likely not a decision

Do not elevate an item if it is:
- a one-off implementation detail
- a temporary workaround without future impact
- a tiny naming change
- a raw task or brainstorm note
- a fact already obvious from code alone

## Detection patterns

When scanning `DEVLOG.md` and `CLAUDE.md`, look for these surface signals of hidden decisions:

- "we chose X over Y" / "decided to use X instead of Y" → probably belongs in `DECISIONS.md`
- "always use X" / "never use X" → probably a convention for `CLAUDE.md`
- "the field is called X, not Y" → operational fact for `CLAUDE.md`
- repeated mentions of the same library, pattern, or flag across multiple sessions → candidate convention
- rejections ("X didn't work because...") → candidate decision with `rejected` status

## Check the external memory store before flagging MISSING

If the project uses Claude Code's auto-memory (`memory/` + `MEMORY.md`) or any store referenced by the registry `taskList` field, search it before deciding a convention is undocumented. A decision already recorded there is `COVERED (external)`, not `MISSING` — re-adding it to `DECISIONS.md` creates a duplicate and dilutes both files. See "Coexistence with Claude Code auto-memory" in `SKILL.md`.

## Audit output shape (undocumented decisions)

Return a table like this, then wait for user confirmation before writing anything:

| Item | Found in | Should go to | Status |
|------|----------|--------------|--------|
| Poll every 5 min | DEVLOG 2026-04-10 | DECISIONS.md | MISSING |
| Currency formatting rule | CLAUDE.md | CLAUDE.md | OK |
| Redis cache TTL = 300s (mentioned 4 times) | DEVLOG (multiple) | DECISIONS.md | MISSING |
| Telegram-over-email channel choice | memory/project_notifications.md | — (already recorded) | COVERED (external) |
| `idx_Order` field name convention | DEVLOG 2026-03-12 | CLAUDE.md | PARTIAL |
| Money-format rule sitting in the log | DEVLOG 2026-03-12 | CLAUDE.md | WRONG-FILE |

Status values (decision coverage):
- `MISSING` — not documented anywhere, including the external memory store
- `PARTIAL` — mentioned but not formalized (e.g., noted in DEVLOG but belongs in DECISIONS or CLAUDE)
- `WRONG-FILE` — documented, but in the wrong file (e.g., a durable convention living in DEVLOG instead of CLAUDE/DECISIONS)
- `COVERED (external)` — already recorded in the project's auto-memory / external store; do not duplicate
- `OK` — documented correctly

Do not reuse the bare word `COVERED` for the wrong-file case — it collides with `COVERED (external)`. Use `WRONG-FILE`.

`PARTIAL` here means "mentioned but not formalized" in the **Undocumented Decisions** table. It is distinct from the status-drift `PARTIAL` (a partially-shipped task) defined in `SKILL.md` — the two never share a table (per the SKILL.md step that returns Status Drift and Undocumented Decisions as separate tables), so the same token stays unambiguous in context.

Do not batch-promote everything. Confirm each elevation individually, or in small groups the user can approve together. False promotions pollute `DECISIONS.md` and reduce its long-term value.
