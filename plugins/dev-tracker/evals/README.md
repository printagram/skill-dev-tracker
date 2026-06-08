# Evals

Test prompts for evaluating the `dev-tracker` skill.

## Running

Use the `skill-creator` eval loop (from `/mnt/skills/examples/skill-creator/`):

```bash
python -m scripts.run_loop \
  --eval-set evals/evals.json \
  --skill-path . \
  --max-iterations 5 \
  --verbose
```

This runs each prompt with and without the skill, grades the results, and iterates on the skill description to improve trigger accuracy.

## The prompts

The 13 prompts in `evals.json` cover trigger recognition, workflow execution, anti-hallucination, multi-project handling, the correction workflow, and audit triggering:

1. Explicit `session start` command (trigger accuracy)
2. Natural-language resume ("what did we do last time?")
3. Session end with git available
4. Session end in a non-git directory (anti-hallucination check)
5. Ambiguous mid-work save request (soft trigger recognition)
6. Multi-project alias switching
7. Correction-not-silent-rewrite workflow
8. Audit trigger from user confusion
9. Context restore after a break (read-only `/dev-tracker context`)
10. `save progress` is a hard action (no offer, no confirmation)
11. Anti-fragmentation: same-day save appends rather than splitting
12. Recurring-blocker detection at session end
13. Anti-overlogging guard for `DECISIONS.md`

## Assertions

Each prompt in `evals.json` now carries objective assertions (`regex`, `contains_all`) plus an `llm_judge` check for behavior that cannot be matched literally — e.g. "invents no git stats", "writes no `[in-progress]` marker", "waits for confirmation before writing". Tighten or extend these before treating the pass rate as a benchmark.

Keep purely subjective qualities (tone, helpfulness, "does this feel right") out of the assertions — review them via the eval viewer rather than forcing them into checks.
