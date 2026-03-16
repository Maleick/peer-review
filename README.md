# peer-review

A Claude Code skill that orchestrates multi-LLM peer review. Send any plan, idea, or question to multiple AI models for structured feedback with cross-examination, then cherry-pick the best insights.

## What It Does

Instead of getting one AI's opinion, peer-review dispatches your prompt to multiple external LLMs (currently GPT and Gemini via the GitHub Copilot CLI), has them critique each other's responses across configurable rounds of deliberation, and synthesizes everything into a structured Decision Packet with numbered action items you can accept, cherry-pick, or discard.

Each model gets a role-differentiated prompt designed to produce genuinely different perspectives — not three versions of the same answer.

## Requirements

- **Claude Code** — the orchestration environment that runs the skill ([claude.ai/download](https://claude.ai/download))
- **GitHub Copilot CLI** — provides access to both GPT and Gemini models. Requires a GitHub Copilot subscription.
- **macOS** with zsh (not tested on Windows or Linux — contributions welcome)

The Copilot CLI must be installed and authenticated before using the skill. Auth is handled via `gh auth login` or `copilot login` (no API keys needed).

## Installation

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/peer-review
# From the cloned repo directory:
cp .claude/skills/peer-review/SKILL.md ~/.claude/skills/peer-review/
```

Or clone this repo and symlink:

```bash
git clone https://github.com/<your-username>/peer-review.git
ln -s "$(pwd)/peer-review/.claude/skills/peer-review" ~/.claude/skills/peer-review
```

## Usage

```
/peer-review <your plan or question>           # structured review (default)
/peer-review idea <topic>                      # multi-perspective brainstorm
/peer-review debate <question>                 # pro/con with judge's verdict
/peer-review advocate <plan>                   # good cop / bad cop analysis
/peer-review redteam <plan>                    # adversarial failure analysis
/peer-review premortem <plan>                  # "it failed in 6 months — why?"
/peer-review refactor <code description>       # refactoring strategy & SOLID analysis
/peer-review deploy <deployment plan>          # deployment readiness & Go/No-Go
/peer-review api <api design>                  # API design review & scorecard
/peer-review perf <performance issue>          # performance analysis & bottleneck ID
/peer-review diff                              # review staged/unstaged git changes
/peer-review quick <prompt>                    # fast single-round opinion
/peer-review gpt <prompt>                    # single-model: GPT only
/peer-review gemini <prompt>                   # single-model: Gemini only
/peer-review help                              # list all modes and options
/peer-review history                           # show previous reviews in session
```

Legacy alias: `/brainstorm` maps to the same modes.

### Options

```
--rounds N              # override cross-examination rounds (1-4)
--verbose               # show exact prompts sent and raw outputs
--quiet                 # skip model sections, show only Decision Packet
--gpt-model <model>     # override GPT model (e.g., gpt-5.4)
--gemini-model <model>  # override Gemini model
```

## Modes

| Mode | What It Does | Good For |
|------|-------------|----------|
| **review** (default) | Implementation + strategic review with different lenses | Plans, PRs, architecture docs |
| **idea** | Builder + strategist brainstorm from different angles | Exploring new approaches |
| **debate** | One model argues FOR, one argues AGAINST, with judge's verdict | Contested decisions |
| **advocate** | One model defends the plan (good cop), one attacks it (bad cop) | Balanced strength/weakness analysis |
| **redteam** | Adversarial analysis: attack vectors, failure modes, silent failures | Security, reliability, edge cases |
| **premortem** | "It's 6 months later and this failed badly — why?" | Risk assessment, planning gaps |
| **refactor** | SOLID violations, coupling hotspots, migration strategy | Code refactoring decisions |
| **deploy** | Rollback plans, blast radius, monitoring gaps, Go/No-Go checklist | Deployment readiness |
| **api** | Consistency, versioning, pagination, error handling, design scorecard | API design review |
| **perf** | Query optimization, caching, scaling bottlenecks, performance assessment | Performance issues |
| **diff** | Reviews staged/unstaged git changes with full review treatment | Code review of changes |
| **quick** | Raw second opinion from both models, no cross-examination | Fast sanity checks |
| **help** | Lists all modes, options, and example invocations | Quick reference |
| **history** | Shows previous reviews from the current session | Tracking review history |

## Configuration

Edit the Configuration block in `SKILL.md` to customize:

```
GPT_MODEL: auto            # "auto" = discover latest; or pin e.g. "gpt-5.4"
GEMINI_MODEL: auto         # "auto" = discover latest; or pin e.g. "gemini-3-pro-preview"
ROUNDS: 2                  # cross-examination rounds (1-4)
```

### Rounds

The `ROUNDS` setting controls how many times the models exchange critiques:

| Rounds | Behavior | Best For |
|--------|----------|----------|
| 1 | No cross-examination (role prompts only) | Quick feedback |
| 2 | One critique round (default) | Most reviews |
| 3 | Critique + rebuttal | Complex decisions |
| 4 | Full deliberation with final positions | High-stakes architecture |

Higher rounds cost proportionally more API calls but produce deeper analysis. 2 rounds is the sweet spot for most use cases.

## How It Works

1. **Round 1 — Generate:** Both models receive the prompt with different role personas and respond independently
2. **Round 2 — Critique:** Each model reads the other's response and identifies agreements, disagreements, and blind spots
3. **Round 3+ — Deliberate:** (if ROUNDS >= 3) Models rebut critiques and refine their positions
4. **Synthesis:** Claude reads all rounds and produces a Decision Packet with numbered action items
5. **Cherry-pick:** Accept all, cherry-pick by number, ask a follow-up ("Refine"), or discard

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `PREFLIGHT_FAIL: copilot CLI not installed` | Copilot CLI not in PATH | Install: `brew install github/gh/copilot-cli` then authenticate with `gh auth login` or `copilot login` |
| `PREFLIGHT_FAIL: gemini CLI not installed` | Gemini CLI not in PATH | Install: `npm install -g @anthropic-ai/gemini` then authenticate with `gemini auth` |
| `GPT_FAILED: exit code 1` | Usually auth expiry or rate limit | Run `gh auth login` or `copilot login` to re-authenticate. Check `copilot --version` for compatibility |
| `GEMINI_FAILED: exit code 1` | Auth expiry, rate limit, or agent warnings | Run `gemini auth` to re-authenticate. Warnings about unrecognized keys are harmless |
| Empty output from a model | Model returned nothing | Retry with `/peer-review quick` for simpler dispatch, or use single-target mode |
| Timeout / no response | CLI hanging on large prompt | Set Bash timeout to 180000ms. Consider splitting prompt into smaller pieces |

**Tip:** Use `/peer-review gpt <prompt>` or `/peer-review gemini <prompt>` to test each CLI independently when debugging.

## Example Output Structure

```
## Peer Review: review — "API rate limiting design"

### Claude's Take
> [Brief analysis leveraging codebase context the external models don't have]

### GPT (Implementation Lens)
[Concrete risks, edge cases, severity ratings, fixes]

### Gemini (Strategic Lens)
[Systemic risks, alternatives, long-term implications]

### Cross-Examination Highlights
[Where they challenged each other, where they converged]

### Decision Packet
Recommended path: ...
Top 3 risks to mitigate: ...
Actionable items:
1. [HIGH CONFIDENCE] ... (source: consensus)
2. [MEDIUM] ... (source: GPT)
3. [LOW] ... (source: Gemini, challenged by GPT)

### Priority Matrix
| | High Impact | Low Impact |
|---|------------|-----------|
| Low Effort | Items 1, 3 | Item 5 |
| High Effort | Item 2 | Item 4 |

---
What would you like to do with this feedback?
- Accept all / Cherry-pick / Refine / Discard
```

### Mode-Specific Output Sections

- **deploy**: Deployment Readiness Checklist with Go/No-Go verdict
- **api**: API Design Scorecard (consistency, evolvability, client experience, 1-5)
- **perf**: Performance Assessment (primary bottleneck, quick wins, load testing items)
- **debate**: Judge's Verdict with strongest case and recommended compromise
- **advocate**: Advocate vs. Critic Summary with net assessment

## License

MIT
