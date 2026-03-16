# peer-review Project Guidelines

## Serena MCP

Always initialize Serena at the start of a session when working on this project:

```
mcp__plugin_serena_serena__activate_project with project: "/opt/peer-review"
mcp__plugin_serena_serena__check_onboarding_performed
```

If onboarding hasn't been performed, run `mcp__plugin_serena_serena__onboarding` to set it up.

## Project Structure

- `.claude/skills/peer-review/SKILL.md` — The entire skill runtime (single file). All modes, prompts, templates, and configuration live here.
- `peer-review-workspace/evals/` — Eval definitions (`evals.json`) and grading scripts (`grade_all.py`)
- `peer-review-workspace/evals/iteration-1/` — Eval run snapshots with `eval_metadata.json` and `result.md` per run

## Key Patterns

- **3-touch pattern** for adding modes: (1) modes table, (2) Step 1 prompts, (3) Step 5 template
- **Security invariants** that must be preserved: heredoc randomization, Python one-liner temp file writes, DATA START/DATA END markers, Copilot CLI `--no-ask-user` flags, trap cleanup
- **Role differentiation**: GPT = tactical/implementation lens, Gemini = strategic/architectural lens
- **CLI dependency**: All external model calls go through `copilot` CLI (GitHub Copilot CLI), authenticated via `gh` CLI or `copilot login`

## Testing

Run evals with: `cd peer-review-workspace/evals && python3 grade_all.py`
