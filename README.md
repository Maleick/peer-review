# peer-review

A Claude Code skill that orchestrates multi-LLM peer review. Send any plan, idea, or question to multiple AI models for structured feedback with cross-examination, then cherry-pick the best insights.

## What It Does

Instead of getting one AI's opinion, peer-review dispatches your prompt to multiple external LLMs (currently Codex/GPT and Gemini), has them critique each other's responses across configurable rounds of deliberation, and synthesizes everything into a structured Decision Packet with numbered action items you can accept, cherry-pick, or discard.

Each model gets a role-differentiated prompt designed to produce genuinely different perspectives — not three versions of the same answer.

## Requirements

- **Claude Code** — the orchestration environment that runs the skill ([claude.ai/download](https://claude.ai/download))
- **Codex CLI** — OpenAI's CLI tool, authenticated via OAuth
- **Gemini CLI** — Google's CLI tool, authenticated via OAuth
- **macOS** with zsh (not tested on Windows or Linux — contributions welcome)

Both Codex and Gemini CLIs must be installed and authenticated before using the skill. The skill uses OAuth-based authentication (no API keys needed).

## Installation

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/peer-review
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
/peer-review quick <prompt>                    # fast single-round opinion
/peer-review codex <prompt>                    # single-model: Codex only
/peer-review gemini <prompt>                   # single-model: Gemini only
```

Legacy alias: `/brainstorm` maps to the same modes.

## Modes

| Mode | What It Does | Good For |
|------|-------------|----------|
| **review** (default) | Implementation + strategic review with different lenses | Plans, PRs, architecture docs |
| **idea** | Builder + strategist brainstorm from different angles | Exploring new approaches |
| **debate** | One model argues FOR, one argues AGAINST, with judge's verdict | Contested decisions |
| **advocate** | One model defends the plan (good cop), one attacks it (bad cop) | Balanced strength/weakness analysis |
| **redteam** | Adversarial analysis: attack vectors, failure modes, silent failures | Security, reliability, edge cases |
| **premortem** | "It's 6 months later and this failed badly — why?" | Risk assessment, planning gaps |
| **quick** | Raw second opinion from both models, no cross-examination | Fast sanity checks |

## Configuration

Edit the Configuration block in `SKILL.md` to customize:

```
CODEX_MODEL: gpt-5.4      # update when new models ship
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

## Example Output Structure

```
## Peer Review: review — "API rate limiting design"

### Claude's Take
> [Brief analysis leveraging codebase context the external models don't have]

### Codex (Implementation Lens)
[Concrete risks, edge cases, severity ratings, fixes]

### Gemini (Strategic Lens)
[Systemic risks, alternatives, long-term implications]

### Cross-Examination Highlights
[Where they challenged each other, where they converged]

### Decision Packet
Recommended path: ...
Top 3 risks to mitigate: ...
Actionable items:
1. ...
2. ...

---
What would you like to do with this feedback?
- Accept all / Cherry-pick / Refine / Discard
```

## License

MIT
