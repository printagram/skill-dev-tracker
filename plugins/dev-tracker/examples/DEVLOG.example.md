# Dev Log

_This is an example DEVLOG showing well-formed entries across multiple sessions on a real-style project. Use as a reference for tone, specificity, and format._

_Older entries archived in `DEVLOG-2026-W10.md`._

## 2026-03-18 09:15 CET — Folder migration script: date validation bug

### Done
- fixed `migrate_folders.py` rejecting valid `2024-12` folders because regex required 3-digit months
- replaced `r'(\d{3})'` with `r'(0[1-9]|1[0-2])'` in `_parse_year_month()`
- added unit test `test_parse_year_month_boundary` covering 01, 09, 10, 12
- verified migration completed for 47 folders in `U:\ORDERS\2024`

### Blockers
- four orders placed in corrupted `201\` folder (missing month digit) still need manual remediation

### Next
1. write remediation script for the `201\` folders
2. verify `idx_Order` field mapping in Access ODBC matches what Supabase expects

## 2026-03-18 14:40 CET — Access ODBC field name mismatch

### Done
- discovered sync failures were caused by code using `id_Order` while Access exposes the field as `idx_Order`
- grepped the project for `id_Order` — found 14 occurrences across `sync_access_to_supabase.py`, `sync_schema.py`, and `mod_sync.bas`
- replaced all occurrences, re-ran sync, 523/523 orders synced clean
- updated `CLAUDE.md` with the correct field name so this does not recur

### Decisions
- canonical field name in all sync code is `idx_Order` (see D-003)

### Next
1. write remediation script for the `201\` folders (carried over)
2. document the sync lifecycle in `CLAUDE.md`

## 2026-03-19 10:00 CET — [in-progress]

### Done
- started writing the `201\` folder remediation script

_This entry will be completed at session end. If you see it at session start, the previous session ended abruptly — check for uncommitted work before continuing._
