# Audit Heuristics

Use these rules when evaluating whether a project contains undocumented decisions.

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

## When to run audit

Run or strongly suggest audit when:
- the user asks for it
- the team is unclear why a system works the way it does
- architectural discussions rely on earlier choices
- several sessions have accumulated and conventions may be undocumented
- a new contributor is onboarding to a tracked project
- before a major refactor that may invalidate past decisions

## Audit output shape

Return a table like this, then wait for user confirmation before writing anything:

| Item | Found in | Should go to | Status |
|------|----------|--------------|--------|
| Poll every 5 min | DEVLOG 2026-04-10 | DECISIONS.md | MISSING |
| Currency formatting rule | CLAUDE.md | CLAUDE.md | OK |
| Redis cache TTL = 300s (mentioned 4 times) | DEVLOG (multiple) | DECISIONS.md | MISSING |
| `idx_Order` field name convention | DEVLOG 2026-03-12 | CLAUDE.md | PARTIAL |

Allowed status values:
- `MISSING` — not documented anywhere
- `PARTIAL` — mentioned but not formalized (e.g., noted in DEVLOG but belongs in DECISIONS or CLAUDE)
- `COVERED` — documented but in wrong file
- `OK` — documented correctly

## Audit workflow

1. Scan `DEVLOG.md`, `DECISIONS.md`, `CLAUDE.md`, and `TODO.md`
2. Identify repeated conventions or explicit choices
3. Compare against existing decision entries
4. Report missing or partial coverage as a table
5. Ask the user which findings to formalize
6. Only then update files

Do not batch-promote everything. Confirm each elevation individually, or in small groups the user can approve together. False promotions pollute `DECISIONS.md` and reduce its long-term value.
