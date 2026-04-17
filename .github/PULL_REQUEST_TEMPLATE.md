<!--
Thanks for the contribution. Please fill out all sections. PRs without a linked eval are unlikely to be merged — see CONTRIBUTING.md.
-->

## What does this change?

<!-- One sentence. -->

## Why?

<!-- What problem is this solving? Link to an issue if one exists. -->

## Type of change

- [ ] Bug fix (behavior was wrong)
- [ ] New rule in SKILL.md
- [ ] New slash command
- [ ] Documentation / examples
- [ ] Eval additions
- [ ] Other (describe)

## Principle alignment

<!-- Check the principles this change respects. Check none if the change contradicts one and you think it's justified — then explain below. -->

- [ ] Under-logging is recoverable, over-logging is not
- [ ] No hallucination (no invented git output, no fake state)
- [ ] User-owned TODO (no automatic rewrites)
- [ ] Clear boundaries (reads but does not write git, no issue tracker integration)

<!-- If you unchecked any, explain why that's acceptable here. -->

## Eval coverage

<!-- Required for behavior changes. Point to the eval in plugins/dev-tracker/evals/evals.json that tests this change. If you added a new eval, reference its `name`. -->

- Eval: `eval-name-here`

## Versioning

<!-- If this is a behavior change, version needs to bump. Check one: -->

- [ ] Patch (typo, clarification, new example — no behavior change)
- [ ] Minor (new rule or command, backward-compatible)
- [ ] Major (breaking change to file taxonomy, frontmatter, or commands)
- [ ] No version bump needed

Versions updated in all three places:
- [ ] `plugins/dev-tracker/SKILL.md` frontmatter
- [ ] `plugins/dev-tracker/.claude-plugin/plugin.json`
- [ ] `.claude-plugin/marketplace.json`
- [ ] `plugins/dev-tracker/CHANGELOG.md` entry added

## Checklist

- [ ] I read [CONTRIBUTING.md](../blob/main/CONTRIBUTING.md)
- [ ] CI passes (`validate.yml` workflow)
- [ ] I manually tested the new behavior in Claude Code
