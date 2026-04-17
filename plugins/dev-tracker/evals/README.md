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

The eight prompts in `evals.json` cover:

1. Explicit `session start` command (trigger accuracy)
2. Natural-language resume ("what did we do last time?")
3. Session end with git available
4. Session end in a non-git directory (anti-hallucination check)
5. Ambiguous mid-work save request (soft trigger recognition)
6. Multi-project alias switching
7. Correction-not-silent-rewrite workflow
8. Audit trigger from user confusion

## Adding assertions

The `assertions` array for each prompt is intentionally empty in the initial draft. Before running quantitative benchmarks, add objective checks. Example for prompt 3:

```json
"assertions": [
  { "text": "response contains a DEVLOG.md entry with YYYY-MM-DD HH:MM timestamp", "type": "regex", "pattern": "## \\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}" },
  { "text": "response includes Done / Next sections", "type": "contains_all", "values": ["### Done", "### Next"] },
  { "text": "response does NOT invent commit counts when git was not run", "type": "llm_judge", "criterion": "no fabricated git stats" }
]
```

Subjective checks (tone, helpfulness, "does this feel right") should stay qualitative — review them via the eval viewer rather than forcing assertions.
