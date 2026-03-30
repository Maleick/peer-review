# peer-review Project Guidelines

## Serena MCP

Always initialize Serena at the start of a session when working on this project:

```
mcp__plugin_serena_serena__activate_project with project: "/Users/maleick/Projects/peer-review"
mcp__plugin_serena_serena__check_onboarding_performed
```

If onboarding hasn't been performed, run `mcp__plugin_serena_serena__onboarding` to set it up.

## Project Structure

- `.claude/skills/peer-review/SKILL.md` — The entire skill runtime (single file). All modes, prompts, templates, and configuration live here.
- `peer-review-workspace/evals/` — Eval definitions (`evals.json`) and grading scripts (`grade_all.py`)
- `peer-review-workspace/evals/iteration-1/` — Eval run snapshots with `eval_metadata.json` and `result.md` per run
- `soul.md` — Persona definition (loaded at session start)
- `docs/` — Extended documentation
- `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY-REVIEW.md` — Project meta-docs

## Key Patterns

- **3-touch pattern** for adding modes: (1) modes table, (2) Step 1 prompts, (3) Step 5 template
- **Security invariants** that must be preserved: heredoc randomization, Python one-liner temp file writes, DATA START/DATA END markers, Codex CLI `exec -s read-only` / Copilot CLI `--no-ask-user` flags, trap cleanup
- **Role differentiation**: GPT = tactical/implementation lens, Gemini = strategic/architectural lens
- **CLI dependency**: GPT calls go through `codex` CLI (OpenAI Codex CLI, preferred) or `copilot` CLI (GitHub Copilot CLI, fallback). Gemini calls go through `gemini` CLI (Gemini CLI). Auth: `codex login` / `OPENAI_API_KEY` for Codex, `gh auth login` / `copilot login` for Copilot, `gemini auth` / `GEMINI_API_KEY` for Gemini

## Environment Prerequisites

Required CLIs (at least one GPT CLI + Gemini CLI):

- `codex` (preferred GPT): `codex login` or set `OPENAI_API_KEY`
- `copilot` (fallback GPT): `gh auth login` then `copilot login`
- `gemini`: `gemini auth` or set `GEMINI_API_KEY`

## Testing

Run evals with: `cd peer-review-workspace/evals && python3 grade_all.py`

## Quick Start

```bash
# Use the skill in any Claude Code session:
/peer-review <your plan or code>        # default structured review
/peer-review redteam <plan>             # adversarial analysis
/peer-review --modes redteam,deploy,perf <plan>  # parallel multi-mode
```
