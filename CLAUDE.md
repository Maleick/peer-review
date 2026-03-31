# peer-review Project Guidelines

## Serena MCP

Always initialize Serena at the start of a session when working on this project:

```
mcp__plugin_serena_serena__activate_project with project: "/Users/maleick/Projects/peer-review"
mcp__plugin_serena_serena__check_onboarding_performed
```

If onboarding hasn't been performed, run `mcp__plugin_serena_serena__onboarding` to set it up.

## Project Structure

- `plugins/peer-review/commands/peer-review.md` — The skill runtime (marketplace plugin format). All modes, prompts, templates, and configuration live here.
- `.claude-plugin/marketplace.json` — Marketplace manifest for plugin installation via `claude plugins install`
- `schemas/decision-packet.schema.json` — Formal JSON Schema for Decision Packet v2 output (validates `--json` export)
- `peer-review-workspace/evals/` — Eval definitions (`evals.json`) and grading scripts (`grade_all.py`)
- `peer-review-workspace/evals/iteration-1/` — Eval run snapshots with `eval_metadata.json` and `result.md` per run
- `soul.md` — Persona definition (loaded at session start)
- `docs/` — Extended documentation
- `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY-REVIEW.md` — Project meta-docs

## Key Patterns

- **3-touch pattern** for adding modes: (1) modes table, (2) Step 1 prompts, (3) Step 5 template
- **4-touch pattern** for new modes with unique output: (1) modes table, (2) Step 1 prompts, (3) Step 5 template, (4) dedicated Step (e.g., Step 10-13 for gate/delegate/background/resume)
- **Security invariants** that must be preserved: heredoc randomization, Python one-liner temp file writes, DATA START/DATA END markers, Codex CLI `exec -s read-only` / Copilot CLI `--no-ask-user` flags, trap cleanup
- **Role differentiation**: GPT = tactical/implementation lens, Gemini = strategic/architectural lens
- **CLI dependency**: GPT calls go through `codex` CLI (OpenAI Codex CLI, preferred) or `copilot` CLI (GitHub Copilot CLI, fallback). Gemini calls go through `gemini` CLI (Gemini CLI). Auth: `codex login` / `OPENAI_API_KEY` for Codex, `gh auth login` / `copilot login` for Copilot, `gemini auth` / `GEMINI_API_KEY` for Gemini
- **Model aliases**: `spark`→gpt-5.3-codex-spark, `mini`→gpt-5.4-mini, `flash`→gemini-2.5-flash, `pro`→gemini-2.5-pro

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
/peer-review gate                       # review Claude's own output
/peer-review delegate <task>            # delegate coding to GPT/Gemini
/peer-review --effort high <plan>       # control reasoning effort
/peer-review --gpt-model spark <plan>   # use model alias
/peer-review --background <plan>        # async review
/peer-review status                     # check background jobs
/peer-review result                     # get completed results
```
