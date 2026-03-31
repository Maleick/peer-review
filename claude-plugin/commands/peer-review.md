---
name: peer-review
description: "Multi-LLM peer review — send plans, ideas, or code to GPT and Gemini for structured peer review with cross-examination. Supports review, idea, redteam, debate, premortem, advocate, refactor, deploy, api, perf, diff, quick, help, and history modes."
argument-hint: "[mode] [content or flags] [--rounds N] [--verbose] [--quiet] [--gpt-model <model>] [--gemini-model <model>] [--steelman] [--iterate] [--json] [--modes mode1,mode2]"
---

EXECUTE IMMEDIATELY.

## Argument Parsing

The first positional argument is the mode (review, idea, redteam, debate, premortem, advocate, refactor, deploy, api, perf, diff, quick, help, history). If omitted, defaults to `review`.

Remaining $ARGUMENTS are the content to review plus any flags.

## Execution

1. Read the skill protocol: `.claude/skills/peer-review/SKILL.md`
2. Parse mode and flags from arguments
3. Follow the skill protocol exactly — dispatch to GPT and Gemini, run cross-examination rounds, synthesize into Decision Packet

Stream all output live.
