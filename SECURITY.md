# Security Policy

## Scope

`dev-tracker` is a skill that reads and writes markdown files in the working directory. It has no network access and no authentication. The security surface is small, but not zero:

- The skill reads `TODO.md`, which may contain sensitive notes
- The skill can write to `DEVLOG.md`, `DECISIONS.md`, and `CLAUDE.md`
- The skill references git if available (`git log --oneline`, `git diff --stat`) — read-only

The skill does **not**:
- Make network requests
- Write to files outside the project's tracker files
- Write to git (no commits, no pushes)
- Execute arbitrary shell commands beyond the git read operations

## Reporting a vulnerability

If you find a way for the skill's instructions to cause Claude to:
- Read or leak sensitive files outside the tracker scope
- Execute unexpected shell commands
- Exfiltrate data via a crafted prompt in `TODO.md` or existing log files
- Persistently modify system state

…please report privately via email to **eduard@printagram.com.mt** with the subject `[security] skill-dev-tracker: <short description>`.

Please do not open a public issue for security reports.

## Response expectations

- Acknowledgment within 5 business days
- Initial assessment within 10 business days
- Fix (if confirmed) within 30 days for high-impact issues
- Public disclosure coordinated with the reporter after a fix is released

## Known considerations

**Prompt injection via tracker files.** Because the skill reads `TODO.md`, `DEVLOG.md`, and other tracker files into Claude's context, an attacker who can write to those files can influence Claude's behavior in future sessions. This is an inherent property of skills that read files — not a bug specific to dev-tracker. Treat your tracker files as trusted input, same as your source code.

**Context-cost amplification.** `CLAUDE.md` is auto-loaded by Claude Code on every session. A large or hostile `CLAUDE.md` can consume significant context. The skill warns against bloating `CLAUDE.md` and suggests a ~150-line soft limit.
