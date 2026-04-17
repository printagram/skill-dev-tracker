# Project Conventions

_Auto-loaded by Claude Code on every session. Keep it lean — every line is context tax._

## Architecture

- Order data source of truth: Access DB at `Z:\DB\PRINTAGRAM_DB.accdb`
- Cloud mirror: Supabase project `printagram-orders` in eu-west-1
- Sync: one-way, Access → Supabase, 5-minute polling (see D-001, D-002)
- Object storage: Cloudflare R2 bucket `sdb-printagram` mounted via rclone

## Conventions

- Primary key field is `idx_Order` everywhere (see D-003)
- All sync code lives under `scripts/sync/`
- New Supabase tables require a migration file in `supabase/migrations/`
- Python formatting: `ruff format` with default settings
- No direct `pyodbc` connections outside `scripts/sync/` — wrap in the `AccessClient` helper

## Operational Notes

- Timezone: CET/CEST (Malta). All DEVLOG timestamps use local time with `CET` or `CEST` suffix.
- Windows dev machine is Parallels VM; network drives `Z:` (DB) and `U:` (orders) are shared from macOS host
- rclone R2 mount runs via LaunchAgent `com.rclone.r2mount.plist` on macOS — do not unmount manually during work sessions
- When Access shows "file-level locking" errors, close all forms before sync runs
