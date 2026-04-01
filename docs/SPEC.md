# peer-review Specification

Canonical reference for modes, configuration, and CLI support. README and CONTRIBUTING.md reference this file — keep it as the single source of truth.

## Modes

| Invocation                             | Behavior                                                      | Default Rounds |
| -------------------------------------- | ------------------------------------------------------------- | -------------- |
| `/peer-review <plan>`                  | Structured peer review (default mode)                         | ROUNDS (2)     |
| `/peer-review idea <topic>`            | Multi-perspective brainstorm                                  | ROUNDS (2)     |
| `/peer-review redteam <plan>`          | Find flaws, failures, exploits                                | ROUNDS (2)     |
| `/peer-review debate <question>`       | Pro/con argument with judge synthesis                         | ROUNDS (2)     |
| `/peer-review premortem <plan>`        | "It failed in 6 months — why?"                                | ROUNDS (2)     |
| `/peer-review advocate <plan>`         | Good cop / bad cop: one defends, one attacks                  | ROUNDS (2)     |
| `/peer-review quick <prompt>`          | Fast second opinion, no synthesis                             | 1 (always)     |
| `/peer-review gpt <prompt>`            | Single-target: GPT only                                       | 1 (always)     |
| `/peer-review gemini <prompt>`         | Single-target: Gemini only                                    | 1 (always)     |
| `/peer-review help`                    | Show all modes, options, and examples                         | N/A            |
| `/peer-review history`                 | Show recent peer reviews from this session                    | N/A            |
| `/peer-review diff`                    | Review staged git changes                                     | ROUNDS (2)     |
| `/peer-review refactor <code-or-plan>` | Review refactoring decisions: patterns, SOLID, dependencies   | ROUNDS (2)     |
| `/peer-review deploy <rollout-plan>`   | Review deployment/rollout plans                               | ROUNDS (2)     |
| `/peer-review api <api-design>`        | Review API designs: consistency, evolution, client experience | ROUNDS (2)     |
| `/peer-review perf <code-or-plan>`     | Performance review: bottlenecks, scaling, capacity            | ROUNDS (2)     |
| `/peer-review gate`                    | Review gate: review Claude's own output before showing it     | 1 (always)     |
| `/peer-review delegate <task>`         | Delegate implementation to GPT/Gemini with write permissions  | 1 (always)     |
| `/peer-review status`                  | Show active and recent background peer review jobs            | N/A            |
| `/peer-review result [job-id]`         | Retrieve completed background review results                  | N/A            |

**Notes:**

- Quick, gate, and delegate are capped at 1 round — quick prioritizes speed, gate checks a binary ALLOW/BLOCK verdict, and delegate generates patches.
- Any multi-round mode accepts `--rounds N` to override the default.
- If no subcommand is given, defaults to `review` mode.

### Adding a New Mode

Follow the **3-touch pattern** in `plugins/peer-review/commands/peer-review.md`:

1. **Modes table** (Step 0 area) — add a row with invocation, behavior, and default rounds
2. **Step 1 prompts** — add role-differentiated prompts for GPT (tactical/implementation) and Gemini (strategic/architectural)
3. **Step 5 template** — add a custom summary section if needed, otherwise the default Decision Packet template applies

If the mode has unique output (like gate's ALLOW/BLOCK verdict), use the **4-touch pattern**: add a dedicated Step (e.g., Step 10-13).

Update the mode list in the skill's `description` frontmatter so Claude Code triggers the skill for the new mode.

## Configuration

These values live at the top of the skill file (`plugins/peer-review/commands/peer-review.md`):

| Key                      | Default                            | Description                                            |
| ------------------------ | ---------------------------------- | ------------------------------------------------------ |
| `GPT_MODEL`              | `gpt-5.4`                          | Pinned GPT model                                       |
| `GEMINI_MODEL`           | `gemini-3.1-pro-preview`           | Primary Gemini model                                   |
| `GEMINI_FALLBACK`        | `gemini-2.5-pro`                   | Fallback on transient errors (429, 503, timeout, etc.) |
| `GPT_CLI`                | `codex`                            | `"codex"` (primary) or `"copilot"` (fallback)          |
| `ROUNDS`                 | `2`                                | Cross-examination rounds (1-4)                         |
| `TIMEOUT_HARD`           | `180`                              | Seconds — Bash tool timeout                            |
| `MAX_CROSSEXAM_CHARS`    | `12000`                            | Truncate peer output in cross-exam                     |
| `MAX_TOTAL_PROMPT_CHARS` | `40000`                            | Hard ceiling per dispatch                              |
| `DEFAULT_EFFORT`         | `""` (model default)               | Reasoning effort: low, medium, high, xhigh             |
| `JOB_DIR`                | `${TMPDIR:-/tmp}/peer-review-jobs` | Persistent directory for `--background` jobs           |

### Model Aliases

Shorthand for `--gpt-model` and `--gemini-model` flags:

| Alias   | Resolves To           | Best For              |
| ------- | --------------------- | --------------------- |
| `spark` | `gpt-5.3-codex-spark` | Quick mode, cheap     |
| `mini`  | `gpt-5.4-mini`        | Balanced cost/quality |
| `flash` | `gemini-2.5-flash`    | Quick mode, fast      |
| `pro`   | `gemini-2.5-pro`      | Previous gen pro      |

### Options

| Flag                     | Description                                                   |
| ------------------------ | ------------------------------------------------------------- |
| `--rounds N`             | Override cross-examination rounds (1-4)                       |
| `--verbose`              | Show exact prompts sent and raw model outputs                 |
| `--quiet`                | Skip model sections, show only Decision Packet                |
| `--gpt-model <model>`    | Override GPT model for this run                               |
| `--gemini-model <model>` | Override Gemini model for this run                            |
| `--branch [name]`        | For diff mode: compare against a branch (default: main)       |
| `--steelman`             | Steelman cross-exam: strengthen before critiquing             |
| `--iterate [N]`          | Autoresearch loop: review, auto-fix, re-review (default: 3)   |
| `--json`                 | Emit machine-readable JSON export (Decision Packet v2 schema) |
| `--json-redacted`        | Like `--json`, but auto-redacts detected secrets              |
| `--modes <m1,m2,...>`    | Run multiple modes in parallel (cap: 4)                       |
| `--allow-sensitive`      | Override block-by-default privacy gate for diff mode          |
| `--effort <level>`       | Control reasoning effort: low, medium, high, xhigh            |
| `--background`           | Dispatch review async, return job ID immediately              |
| `--resume [job-id]`      | Resume a prior review session                                 |

### Multi-Mode Presets

| Preset            | Modes                  |
| ----------------- | ---------------------- |
| `preset:release`  | redteam, deploy, perf  |
| `preset:security` | redteam, api           |
| `preset:quality`  | review, refactor, perf |

## CLI Support Matrix

| Feature                  | Codex CLI (primary)              | Copilot CLI (fallback)               | Gemini CLI                       |
| ------------------------ | -------------------------------- | ------------------------------------ | -------------------------------- |
| **Install**              | `npm i -g @openai/codex`         | `brew install github/gh/copilot-cli` | `npm i -g @google/gemini-cli`    |
| **Auth**                 | `codex login` / `OPENAI_API_KEY` | `gh auth login`                      | `gemini auth` / `GEMINI_API_KEY` |
| **Prompt delivery**      | stdin pipe                       | stdin pipe                           | stdin pipe (`-p ""`)             |
| **Sandbox**              | `--sandbox read-only`            | None (see SECURITY-REVIEW.md F3)     | `--approval-mode plan`           |
| **Non-interactive flag** | `--ask-for-approval never`       | `--no-ask-user`                      | `--approval-mode plan`           |
| **Effort control**       | `--reasoning-effort <level>`     | Not supported (silently skipped)     | `--thinking-budget-tokens <N>`   |
| **Model override**       | `-m <model>`                     | `--model <model>`                    | `--model <model>`                |

### Rounds Behavior

| Rounds | What Happens                                       | Use When                        |
| ------ | -------------------------------------------------- | ------------------------------- |
| 1      | Two independent reviews, no debate                 | Quick feedback                  |
| 2      | Reviews + one round of cross-examination (default) | Most reviews                    |
| 3      | + rebuttal round                                   | Complex architectural decisions |
| 4      | + final position statements                        | High-stakes decisions           |
