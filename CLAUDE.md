# peer-review Project Guidelines

## Serena MCP

Initialize Serena at session start:

```
mcp__plugin_serena_serena__activate_project with project: "/Users/maleick/Projects/peer-review"
mcp__plugin_serena_serena__check_onboarding_performed
```

If onboarding hasn't been performed, run `mcp__plugin_serena_serena__onboarding`.

## Project Structure

```
plugins/peer-review/
  commands/peer-review.md   ← The skill runtime. All modes, prompts, templates, config.
  .claude-plugin/plugin.json
.claude-plugin/marketplace.json  ← Marketplace manifest (claude plugins install)
schemas/decision-packet.schema.json  ← JSON Schema for --json export
docs/
  SPEC.md                   ← Canonical reference: modes, config, CLI matrix, options
peer-review-workspace/evals/
  evals.json, grade_all.py  ← Eval framework
  iteration-1/              ← 17 named eval dirs (eval-0 through eval-16)
soul.md                     ← Persona definition (loaded at session start)
```

## Key Patterns

- **Adding modes** — 3-touch and 4-touch patterns in [docs/SPEC.md](docs/SPEC.md#adding-a-new-mode)
- **Security invariants** — heredoc randomization, Python one-liner temp file writes, DATA START/DATA END markers, Codex CLI `exec -s read-only` / Copilot CLI `--no-ask-user` flags, stdin redirection for all CLIs, trap cleanup. See [SECURITY-REVIEW.md](SECURITY-REVIEW.md) for full threat model.
- **Role differentiation** — GPT = tactical/implementation lens, Gemini = strategic/architectural lens
- **Model aliases** — `spark`→gpt-5.3-codex-spark, `mini`→gpt-5.4-mini, `flash`→gemini-2.5-flash, `pro`→gemini-2.5-pro

## Environment Prerequisites

Required CLIs (at least one GPT CLI + Gemini CLI):

- `codex` (preferred GPT): `codex login` or set `OPENAI_API_KEY`
- `copilot` (fallback GPT): `gh auth login` then `copilot login`
- `gemini`: `gemini auth` or set `GEMINI_API_KEY`

## Commands

```bash
# Evals
cd peer-review-workspace/evals && python3 grade_all.py

# Validate JSON schema
python3 -c "import json, jsonschema; jsonschema.validate(json.load(open('output.json')), json.load(open('schemas/decision-packet.schema.json')))"
```

## Quick Start

```bash
/peer-review <plan>                          # default structured review
/peer-review redteam <plan>                  # adversarial analysis
/peer-review --modes redteam,deploy,perf <plan>  # parallel multi-mode
/peer-review gate                            # review Claude's own output
/peer-review delegate <task>                 # delegate coding to GPT/Gemini
/peer-review --effort high <plan>            # control reasoning effort
/peer-review --gpt-model spark <plan>        # use model alias
/peer-review --background <plan>             # async review
/peer-review status                          # check background jobs
/peer-review result                          # get completed results
```

## Gotchas

- **Description string sync** — `marketplace.json` and `plugin.json` have duplicate description fields. Update both when changing.
- **Eval count in CONTRIBUTING.md** — The eval count in CONTRIBUTING.md is manually maintained. Update when adding evals.
- **`peer-review-workspace/` is gitignored** — Eval workspace is local-only. Eval definitions (`evals.json`) and grading scripts are tracked; run outputs are not.
- **Gemini `-p ""` is structural** — The `-p ""` in Gemini dispatch triggers headless mode (not configurable). The actual prompt arrives via stdin redirection. Don't move `-p ""` into GEMINI_FLAGS.
- **Copilot CLI has no sandbox** — When Codex CLI is unavailable and Copilot fallback activates, GPT dispatch runs with ambient permissions. See SECURITY-REVIEW.md F3.
