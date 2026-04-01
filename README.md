# peer-review

[![GitHub Release](https://img.shields.io/github/v/release/Maleick/peer-review?style=flat-square&label=release)](https://github.com/Maleick/peer-review/releases)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Language](https://img.shields.io/badge/language-Shell%20%2F%20Markdown-brightgreen?style=flat-square)](plugins/peer-review/commands/peer-review.md)
[![GitHub Stars](https://img.shields.io/github/stars/Maleick/peer-review?style=flat-square)](https://github.com/Maleick/peer-review/stargazers)
[![Status](https://img.shields.io/badge/status-Active-green?style=flat-square)](CHANGELOG.md)
[![Claude Code](https://img.shields.io/badge/compatible-Claude%20Code-blueviolet?style=flat-square)](https://claude.ai/download)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)

Get a second opinion from other AI models without leaving Claude Code. This skill sends your prompt to GPT (via OpenAI Codex CLI) and Gemini (via Gemini CLI), has them debate each other, and brings back a structured summary you can act on.

## Quick Start

**Prerequisites:**

1. [Claude Code](https://claude.ai/download) installed
2. [OpenAI Codex CLI](https://github.com/openai/codex) installed (`npm install -g @openai/codex` or `brew install --cask codex`)
3. [Gemini CLI](https://github.com/google/gemini-cli) installed (`npm install -g @google/gemini-cli`)
4. Authenticated: `codex login` (or set `OPENAI_API_KEY`) and `gemini auth` (or set `GEMINI_API_KEY`)

**Optional fallback:** [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/copilot-cli) — used for GPT if Codex CLI is unavailable (`brew install github/gh/copilot-cli`, then `gh auth login`)

**Install the skill:**

Via plugin marketplace (recommended — auto-updates when you push changes):

```bash
/plugin marketplace add maleick/peer-review
/plugin install peer-review@peer-review
```

Manual install (no auto-updates):

```bash
# Clone this repo
git clone https://github.com/Maleick/peer-review.git ~/Projects/peer-review

# Copy command to Claude Code
mkdir -p ~/.claude/commands
cp ~/Projects/peer-review/plugins/peer-review/commands/peer-review.md ~/.claude/commands/
```

**Use it:**

```
/peer-review Should we use Redis or Memcached for our session cache?
```

That's it. Claude dispatches your question to GPT-5.4 (via Codex CLI) and Gemini (via Gemini CLI), they each review it from different angles, debate each other's responses, and you get back a numbered list of action items you can accept, cherry-pick, or discard.

## What Happens When You Run It

```
You type:  /peer-review We plan to add WebSocket support to the API

Claude does this:
  1. Sends your prompt to GPT via Codex CLI (as an "implementation reviewer" — finds edge cases, concrete risks)
  2. Sends your prompt to Gemini via Gemini CLI (as a "strategic reviewer" — finds architectural issues)
  3. Shows each model the other's response and asks them to critique it
  4. Reads everything and produces a Decision Packet with numbered action items
  5. Asks you: Accept all? Cherry-pick? Refine? Discard?
```

The key insight: each model gets a **different reviewer persona** tuned to its strengths. GPT focuses on tactical implementation risks. Gemini focuses on strategic architecture concerns. Then they challenge each other, which filters out weak arguments and surfaces genuine consensus.

## Modes

20 modes covering review, brainstorming, adversarial analysis, deployment, API design, performance, and more. Highlights:

| Command                          | What It Does                                    |
| -------------------------------- | ----------------------------------------------- |
| `/peer-review <plan>`            | Implementation + architecture review (default)  |
| `/peer-review redteam <plan>`    | Find attack vectors, failure modes, blind spots |
| `/peer-review debate <question>` | Pro/con argument with judge synthesis           |
| `/peer-review gate`              | Review Claude's own output before proceeding    |
| `/peer-review delegate <task>`   | Hand off coding to GPT/Gemini with write access |
| `/peer-review diff`              | Review staged/unstaged git changes              |
| `/peer-review --modes m1,m2`     | Run multiple modes in parallel (cap: 4)         |

Full modes table, options, model aliases, multi-mode presets, and configuration reference: **[docs/SPEC.md](docs/SPEC.md)**

Legacy alias: `/brainstorm` maps to the same modes.

### Steelman Mode (`--steelman`)

Instead of adversarial cross-examination (find weaknesses), steelman mode asks each model to first make the **strongest possible version** of the other's argument before critiquing it. This produces deeper analysis — models can't dismiss arguments they've just strengthened. No extra cost (same number of CLI calls).

### Iteration Mode (`--iterate`)

Turns peer-review into a convergence loop. After each review, Claude auto-accepts HIGH CONFIDENCE items, applies fixes to the file, and re-reviews. Repeats until no new HIGH items appear or max iterations reached.

```
/peer-review refactor src/auth.ts --iterate 3

Iteration 1: 9 items (3 HIGH) → auto-apply 3 fixes
Iteration 2: 5 items (1 HIGH, 4 RESOLVED) → auto-apply 1 fix
Iteration 3: 2 items (0 HIGH) → convergence achieved
```

Safety rails prevent runaway iteration: a **validation gate** syntax-checks each fix before applying (Python, JS/TS, Shell, JSON), **scope control** blocks deletions/renames/multi-file/schema changes without approval, and a **diff size guard** pauses on fixes exceeding 50 lines. Type `stop` at any iteration to halt, or `override` to switch to manual cherry-pick. Regressions (more items than before) trigger an automatic pause.

## Example Output

```markdown
## Peer Review: review — "API rate limiting design"

### Claude's Take

> [Analysis using codebase context that external models don't have]

### GPT (Implementation Reviewer)

[Concrete risks, edge cases, severity ratings, specific fixes]

### Gemini (Strategic Reviewer)

[Systemic risks, alternative approaches, long-term implications]

### Cross-Examination Highlights

- GPT challenged Gemini's caching suggestion as premature optimization
- Both converged on the need for per-tenant rate limiting

### Decision Packet

Summary: 8 items — 1 critical, 3 high, 3 medium, 1 low
Recommended path: ...
Actionable items:

1. [HIGH CONFIDENCE] Add per-tenant limits _(consensus)_
2. [MEDIUM] Consider token bucket over sliding window _(GPT)_
3. [LOW] Evaluate distributed rate limiting _(Gemini, challenged by GPT)_

### Priority Matrix

|                 | Low Effort | High Effort |
| --------------- | ---------- | ----------- |
| **High Impact** | Items 1, 3 | Item 2      |
| **Low Impact**  | Item 5     | Item 4      |

---

What would you like to do with this feedback?

- **Accept all** / **Cherry-pick** (e.g., "keep 1, 3") / **Refine** / **Discard**
```

## How It Works Under the Hood

1. Claude verifies the Codex CLI and Gemini CLI are installed and authenticated (falls back to Copilot CLI for GPT if needed)
2. Scans your prompt for sensitive data (API keys, JWT tokens, AWS keys, GitHub/Slack tokens, PEM keys, high-entropy strings, credentials) — blocks dispatch if found
3. Builds role-differentiated prompts for each model
4. Dispatches to both models **in parallel** via their respective CLIs
5. Sanitizes model output before reuse (cross-exam, TODOs, JSON export, iteration fixes)
6. Each cross-examination round includes the original task, the model's own prior response, and the peer's response — so models maintain context across rounds
7. Synthesizes all rounds into a Decision Packet with confidence levels based on cross-exam convergence
8. Presents the cherry-pick menu

**Security:** Prompts are written to temp files with restricted permissions, piped via stdin (never command-line args), and cleaned up after each call. Heredoc delimiters use cryptographically random suffixes to prevent injection. Cross-examination uses randomized DATA boundary markers. Diff mode uses block-by-default privacy — sensitive diffs require `--allow-sensitive`. JSON exports are scanned for leaked secrets post-write.

## Troubleshooting

| Problem                                                | Fix                                                                         |
| ------------------------------------------------------ | --------------------------------------------------------------------------- |
| `PREFLIGHT_FAIL: No GPT CLI found`                     | Install Codex CLI: `npm install -g @openai/codex`, then `codex login`       |
| `PREFLIGHT_FAIL: Gemini CLI not found`                 | Install Gemini CLI: `npm install -g @google/gemini-cli`, then `gemini auth` |
| `PREFLIGHT_WARN: Codex CLI auth may not be configured` | Run `codex login` or set `OPENAI_API_KEY`                                   |
| `GPT_FAILED` with exit code                            | Re-authenticate: `codex login`. If using Copilot fallback: `gh auth login`  |
| `GEMINI_FAILED` with exit code                         | Re-authenticate: `gemini auth` or check `GEMINI_API_KEY`                    |
| Rate limited                                           | Wait a few minutes, or use `/peer-review quick` for a lighter call          |
| Timeout (no response after 3 min)                      | Prompt may be too large. Split into smaller reviews.                        |
| Empty/partial output                                   | Model returned a stub. Retry, or use single-target mode to isolate.         |

**Debug tip:** Test each model independently with `/peer-review gpt <prompt>` or `/peer-review gemini <prompt>`.

## License

MIT
