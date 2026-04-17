# Contributing to dev-tracker

Thanks for considering a contribution. This skill has been shaped by real production use and careful iteration, so the bar for changes is "does it make the skill measurably better in a concrete way that we can demonstrate with an eval?"

## Philosophy

Before proposing a change, read the skill's operating principles in [`SKILL.md`](plugins/dev-tracker/SKILL.md). A few that shape every decision:

1. **Under-logging is recoverable, over-logging is not.** Rules favor caution in writing to DECISIONS and CLAUDE.md.
2. **No hallucination.** If git isn't available, the skill says so rather than fabricating.
3. **User-owned means user-owned.** `TODO.md` never gets rewritten automatically, period.
4. **Clear boundaries.** This skill reads git but does not write git. It does not generate commits, push, or branch. If you want that, it's a different skill.

If your proposal conflicts with any of these, either reframe it or accept that it won't be merged.

## What kinds of changes are welcome

### Always welcome

- Bug fixes in `SKILL.md` instructions that cause wrong behavior in practice
- New eval prompts that cover behaviors not currently tested
- Documentation improvements in `README.md`, `CHANGELOG.md`, or reference files
- New realistic examples in `examples/`
- Translation of examples into other languages (keep English keys for `D-###`, `Status: active`, etc.)

### Welcome with discussion first

- New slash commands (open an issue first — commands add cognitive load)
- Changes to the four-file taxonomy (`TODO`/`DEVLOG`/`DECISIONS`/`CLAUDE`)
- Changes to session boundary rules or trigger policy

### Likely out of scope

- Git write operations (commits, pushes). Candidate for a separate skill.
- Issue tracker integrations (GitHub, Jira). Candidate for a separate skill.
- Non-markdown output formats. The skill's value is that its output is human-readable in any editor.

## How to propose a change

1. **Open an issue** describing the problem or proposal. For small fixes (typos, clarifications), you can go straight to a PR.
2. **Propose an eval prompt** in `plugins/dev-tracker/evals/evals.json` that demonstrates the problem the change solves. If no eval fits, that's a signal the change may be speculative.
3. **Submit a PR** that:
   - edits `SKILL.md` with the new/changed rule
   - adds or modifies the eval in `evals.json`
   - bumps the version in both `plugin.json` and the `SKILL.md` frontmatter
   - adds a CHANGELOG entry

## Running the evals

The skill is designed to be iterable through [Anthropic's skill-creator eval loop](https://github.com/anthropics/skills/tree/main/skills/skill-creator).

```bash
# From the skill-creator directory
python -m scripts.run_loop \
  --eval-set /path/to/skill-dev-tracker/plugins/dev-tracker/evals/evals.json \
  --skill-path /path/to/skill-dev-tracker/plugins/dev-tracker \
  --max-iterations 5 \
  --verbose
```

This splits the eval set into train/held-out, scores trigger accuracy and behavior quality, and iterates on the skill description.

## Style

- Instructions in `SKILL.md` should be imperative ("Write the entry", not "The entry should be written")
- Explain *why* a rule exists where the reason isn't obvious. Rules with visible rationale survive future edits better.
- Avoid `MUST` / `SHOULD` ceremonial language — plain imperatives are clearer
- Keep `SKILL.md` under 550 lines. Overflow goes into `references/`.

## Versioning

We follow [Semantic Versioning](https://semver.org/):

- **Patch** (`3.2.0` → `3.2.1`): typo fixes, clarifications, new examples, new evals
- **Minor** (`3.2.0` → `3.3.0`): new rules, new commands, new workflows — backward-compatible
- **Major** (`3.x.y` → `4.0.0`): breaking changes to the file taxonomy, frontmatter schema, or command set

Version bumps happen in three places simultaneously:
1. `plugins/dev-tracker/SKILL.md` frontmatter `version:`
2. `plugins/dev-tracker/.claude-plugin/plugin.json` `version`
3. `.claude-plugin/marketplace.json` plugin entry `version`

A CI check validates these stay in sync.

## Reviewer guidance

PR reviewers should ask:

1. Does this change have a matching eval prompt? (If not, how will we regression-test it?)
2. Does it fit the "Under-logging is recoverable" principle? Or does it encourage more writing?
3. Does it add a new slash command or file type? (Higher scrutiny for these.)
4. Is the rationale visible in the change itself, or only in the PR description? (Prefer the former.)

## Acknowledgements

The iteration discipline here (v1 → v2 → v3 → v3.1 → v3.2, each with CHANGELOG, each with honest post-mortems of regressions) was shaped by the process described in [Anthropic's skill-creator documentation](https://github.com/anthropics/skills/tree/main/skills/skill-creator). If you're contributing, reading that doc will make your changes more likely to land cleanly.
