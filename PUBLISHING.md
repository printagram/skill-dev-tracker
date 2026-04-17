# Publishing guide

Step-by-step walkthrough for publishing this repository to `github.com/printagram/skill-dev-tracker` for the first time. Keep this file in the repo for future reference (or move it to `docs/` after the first publish).

## Prerequisites

- A GitHub account with access to the `printagram` organization (or a personal account — replace `printagram` with your username throughout)
- `git` installed locally (obviously)
- `gh` CLI installed and authenticated (optional but recommended — makes repo creation one command)

## 1. Create the repository on GitHub

**Option A — via `gh` CLI (fastest):**

```bash
gh repo create printagram/skill-dev-tracker \
  --public \
  --description "Cross-session memory for Claude Code. A structured development journal that doesn't hallucinate, doesn't spam, and doesn't rewrite your notes." \
  --homepage "https://github.com/printagram/skill-dev-tracker" \
  --no-clone
```

**Option B — via web UI:**

1. Go to <https://github.com/organizations/printagram/repositories/new> (or your personal account's new-repo page)
2. Name: `skill-dev-tracker`
3. Description: *Cross-session memory for Claude Code. A structured development journal that doesn't hallucinate, doesn't spam, and doesn't rewrite your notes.*
4. Visibility: **Public**
5. Do **not** check "Add a README", "Add .gitignore", or "Choose a license" — we already have all three
6. Create repository

## 2. Initialize the local repository

From inside the unzipped `skill-dev-tracker/` folder:

```bash
cd skill-dev-tracker
git init -b main
git add .
git commit -m "Initial release: dev-tracker v3.2.0"
```

Verify what will be committed:

```bash
git status
# Should show: nothing to commit, working tree clean
git log --oneline
# Should show your single initial commit
```

## 3. Connect and push

```bash
git remote add origin https://github.com/printagram/skill-dev-tracker.git
git push -u origin main
```

## 4. Configure the repository on GitHub

After the first push, go to the repo settings and:

### 4a. Topics (for discoverability)

In the repo's **About** sidebar → Settings gear → Topics. Add:

```
claude-code  claude-skills  agent-skills  devlog  adr
architecture-decision-records  session-memory  developer-tools
```

These are the topics people actually search for in the Claude Code skills ecosystem in 2026. Including `claude-code` and `claude-skills` is the most important — that's what the awesome lists and marketplaces crawl.

### 4b. Enable Discussions

Settings → Features → **Discussions** → enable. The issue template config (`config.yml`) routes "question or discussion" links there.

### 4c. Branch protection (optional but recommended)

Settings → Branches → Add rule for `main`:
- Require pull request before merging
- Require status checks: `validate-structure`, `validate-json`, `validate-versions-in-sync`, `validate-skill-frontmatter`

This prevents future you from accidentally pushing a broken `plugin.json` directly to main.

### 4d. Tag the first release

```bash
git tag -a v3.2.0 -m "dev-tracker v3.2.0 — initial public release"
git push origin v3.2.0
```

Then on GitHub: Releases → Draft a new release → pick the tag → Generate release notes → copy/adapt the v3.2.0 section from `plugins/dev-tracker/CHANGELOG.md` into the release body → Publish.

## 5. Verify it works as a marketplace

On any machine with Claude Code installed:

```
/plugin marketplace add printagram/skill-dev-tracker
```

Claude Code should respond that the marketplace was added. Then:

```
/plugin install dev-tracker@printagram-skills
```

If both commands succeed, the publish is complete and working.

## 6. Announce (optional)

Places to post if you want visibility:

- **awesome-claude-skills** — <https://github.com/travisvn/awesome-claude-skills> — open a PR adding your skill under an appropriate category (probably "Development & Technical" or a new "Memory & Context" section)
- **awesome-agent-skills** (VoltAgent) — <https://github.com/VoltAgent/awesome-agent-skills>
- **awesome-claude-code** — <https://github.com/hesreallyhim/awesome-claude-code>
- **r/ClaudeAI** or **r/ClaudeCode** on Reddit
- **Hacker News** — only if you have a strong first comment ready; otherwise it vanishes

The PR to awesome-claude-skills is the highest-leverage move. Keep your entry short: one sentence describing what it does, one sentence on what makes it different.

## Future releases

For subsequent versions (3.3, 4.0, etc.):

1. Bump version in **three places** (CI will fail if they drift):
   - `.claude-plugin/marketplace.json` → `plugins[0].version`
   - `plugins/dev-tracker/.claude-plugin/plugin.json` → `version`
   - `plugins/dev-tracker/SKILL.md` → frontmatter `version:`
2. Add a `CHANGELOG.md` entry at the top, dated, grouped into Added / Changed / Fixed / Removed
3. Commit: `git commit -m "Release v3.3.0"`
4. Tag: `git tag -a v3.3.0 -m "..."`
5. Push: `git push && git push --tags`
6. Draft a GitHub release from the tag with the changelog excerpt

Users with the marketplace already added will see the update via `/plugin update`.

## Troubleshooting

**`/plugin marketplace add printagram/skill-dev-tracker` says "marketplace not found":**
- Verify the repo is public
- Verify `.claude-plugin/marketplace.json` is committed at the repo root (not inside a subdirectory)
- Try the full git URL form: `/plugin marketplace add https://github.com/printagram/skill-dev-tracker.git`

**CI fails on the first push:**
- Check the Actions tab for which workflow step failed
- Most likely cause: version mismatch between the three manifests, or invalid JSON
- Fix locally, commit, push again

**Someone reports the skill doesn't trigger for them:**
- That's an eval issue. Reproduce their exact prompt locally, add it to `plugins/dev-tracker/evals/evals.json`, and iterate on the `description` field in `SKILL.md` frontmatter until it triggers reliably
