# peer-review

Get a second opinion from other AI models without leaving Claude Code. This skill sends your prompt to GPT and a second Claude instance (via the GitHub Copilot CLI), has them debate each other, and brings back a structured summary you can act on.

## Quick Start

**Prerequisites:**

1. [Claude Code](https://claude.ai/download) installed
2. [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/copilot-cli) installed (`brew install github/gh/copilot-cli`)
3. GitHub authenticated (`gh auth login`) with a Copilot subscription

**Install the skill:**

```bash
# Clone this repo into your projects directory
git clone https://github.com/<your-username>/peer-review.git ~/Projects/peer-review

# The skill is automatically available when you open Claude Code in this directory.
# To make it available globally, symlink it:
mkdir -p ~/.claude/skills
ln -s ~/Projects/peer-review/.claude/skills/peer-review ~/.claude/skills/peer-review
```

**Use it:**

```
/peer-review Should we use Redis or Memcached for our session cache?
```

That's it. Claude dispatches your question to GPT-5.4 and Claude Sonnet, they each review it from different angles, debate each other's responses, and you get back a numbered list of action items you can accept, cherry-pick, or discard.

## What Happens When You Run It

```
You type:  /peer-review We plan to add WebSocket support to the API

Claude does this:
  1. Sends your prompt to GPT (as an "implementation reviewer" — finds edge cases, concrete risks)
  2. Sends your prompt to Claude Sonnet (as a "strategic reviewer" — finds architectural issues)
  3. Shows each model the other's response and asks them to critique it
  4. Reads everything and produces a Decision Packet with numbered action items
  5. Asks you: Accept all? Cherry-pick? Refine? Discard?
```

The key insight: each model gets a **different reviewer persona** tuned to its strengths. GPT focuses on tactical implementation risks. Claude Sonnet focuses on strategic architecture concerns. Then they challenge each other, which filters out weak arguments and surfaces genuine consensus.

## Modes

Every mode gives the two models different roles. Pick the one that matches what you need:

| Command                          | What It Does                                          | Example                                                    |
| -------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| `/peer-review <plan>`            | Implementation + architecture review                  | `/peer-review We plan to add caching with Redis`           |
| `/peer-review idea <topic>`      | Brainstorm from builder + strategist angles           | `/peer-review idea How should we handle auth?`             |
| `/peer-review debate <question>` | One model argues FOR, one AGAINST, judge decides      | `/peer-review debate Should we adopt GraphQL?`             |
| `/peer-review redteam <plan>`    | Find attack vectors, failure modes, blind spots       | `/peer-review redteam Our rate limiter uses fixed windows` |
| `/peer-review premortem <plan>`  | "It's 6 months later and this failed — why?"          | `/peer-review premortem Our migration plan is to...`       |
| `/peer-review advocate <plan>`   | One defends, one attacks (good cop / bad cop)         | `/peer-review advocate Our caching strategy uses...`       |
| `/peer-review refactor <code>`   | SOLID analysis, coupling hotspots, migration strategy | `/peer-review refactor We're extracting a service from...` |
| `/peer-review deploy <plan>`     | Rollback, blast radius, monitoring gaps, Go/No-Go     | `/peer-review deploy Rolling deploy of v2.0 with...`       |
| `/peer-review api <design>`      | Consistency, versioning, client experience, scorecard | `/peer-review api POST /users returns 201 with...`         |
| `/peer-review perf <issue>`      | Bottlenecks, caching, scaling, capacity planning      | `/peer-review perf Our search does a full table scan`      |
| `/peer-review diff`              | Review your staged/unstaged git changes               | `/peer-review diff` or `/peer-review diff --branch main`   |
| `/peer-review quick <prompt>`    | Fast one-round opinion, no debate                     | `/peer-review quick Is this regex safe?`                   |
| `/peer-review gpt <prompt>`      | GPT only (single model)                               | `/peer-review gpt Explain this error message`              |
| `/peer-review claude <prompt>`   | Claude only (single model)                            | `/peer-review claude Review this SQL query`                |
| `/peer-review help`              | Show all modes and options                            |                                                            |
| `/peer-review history`           | Show previous reviews this session                    |                                                            |

Legacy alias: `/brainstorm` maps to the same modes.

## Options

```
--rounds N                # 1-4 cross-examination rounds (default: 2)
--verbose                 # show exact prompts sent and raw model outputs
--quiet                   # skip model sections, show only the Decision Packet
--gpt-model <model>       # override GPT model for this run
--claude-model <model>    # override Claude model for this run
--branch [name]           # for diff mode: compare against a branch (default: main)
--steelman                # steelman cross-exam: models strengthen each other's arguments before critiquing
--iterate [N]             # autoresearch loop: review → auto-fix → re-review → converge (default: 3 iterations)
```

### Steelman Mode (`--steelman`)

Instead of adversarial cross-examination (find weaknesses), steelman mode asks each model to first make the **strongest possible version** of the other's argument before critiquing it. This produces deeper analysis — models can't dismiss arguments they've just strengthened. No extra cost (same number of CLI calls).

### Iteration Mode (`--iterate`)

Turns peer-review into a convergence loop. After each review, Claude Opus auto-accepts HIGH CONFIDENCE items, applies fixes to the file, and re-reviews. Repeats until no new HIGH items appear or max iterations reached.

```
/peer-review refactor src/auth.ts --iterate 3

Iteration 1: 9 items (3 HIGH) → auto-apply 3 fixes
Iteration 2: 5 items (1 HIGH, 4 RESOLVED) → auto-apply 1 fix
Iteration 3: 2 items (0 HIGH) → convergence achieved
```

You can type `stop` at any iteration to halt, or `override` to switch to manual cherry-pick. Regressions (more items than before) trigger an automatic pause.

## Configuration

Edit the config block at the top of `.claude/skills/peer-review/SKILL.md`:

```
GPT_MODEL: gpt-5.4                # update when new models ship
CLAUDE_MODEL: claude-sonnet-4.6    # must differ from the orchestrating Claude instance
ROUNDS: 2                          # cross-examination rounds (1-4)
MAX_TOTAL_PROMPT_CHARS: 40000      # hard ceiling per dispatch
MAX_CROSSEXAM_CHARS: 12000         # truncate peer output in cross-exam rounds
```

**To see available models**, run: `copilot -s --no-ask-user -p "List all available models"`

### Rounds

| Rounds | What Happens                                       | Use When                        |
| ------ | -------------------------------------------------- | ------------------------------- |
| 1      | Two independent reviews, no debate                 | Quick feedback                  |
| 2      | Reviews + one round of cross-examination (default) | Most reviews                    |
| 3      | + rebuttal round                                   | Complex architectural decisions |
| 4      | + final position statements                        | High-stakes decisions           |

## Example Output

```markdown
## Peer Review: review — "API rate limiting design"

### Claude's Take

> [Analysis using codebase context that external models don't have]

### GPT (Implementation Reviewer)

[Concrete risks, edge cases, severity ratings, specific fixes]

### Claude Sonnet (Strategic Reviewer)

[Systemic risks, alternative approaches, long-term implications]

### Cross-Examination Highlights

- GPT challenged Claude's caching suggestion as premature optimization
- Both converged on the need for per-tenant rate limiting

### Decision Packet

Summary: 8 items — 1 critical, 3 high, 3 medium, 1 low
Recommended path: ...
Actionable items:

1. [HIGH CONFIDENCE] Add per-tenant limits _(consensus)_
2. [MEDIUM] Consider token bucket over sliding window _(GPT)_
3. [LOW] Evaluate distributed rate limiting _(Claude, challenged by GPT)_

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

1. Claude verifies the Copilot CLI is installed and authenticated
2. Scans your prompt for sensitive data (API keys, credentials) and warns before sending
3. Builds role-differentiated prompts for each model
4. Dispatches to both models **in parallel** via the Copilot CLI
5. Each cross-examination round includes the original task, the model's own prior response, and the peer's response — so models maintain context across rounds
6. Synthesizes all rounds into a Decision Packet with confidence levels based on cross-exam convergence
7. Presents the cherry-pick menu

**Security:** Prompts are written to temp files with restricted permissions, piped via stdin (never command-line args), and cleaned up after each call. Heredoc delimiters use cryptographically random suffixes to prevent injection. Cross-examination uses randomized DATA boundary markers.

## Troubleshooting

| Problem                                        | Fix                                                                          |
| ---------------------------------------------- | ---------------------------------------------------------------------------- |
| `PREFLIGHT_FAIL: copilot CLI not installed`    | `brew install github/gh/copilot-cli` then `gh auth login`                    |
| `PREFLIGHT_FAIL: GitHub auth not configured`   | `gh auth login` or `copilot login`                                           |
| `GPT_FAILED` or `CLAUDE_FAILED` with exit code | Re-authenticate: `gh auth login`. Check your Copilot subscription is active. |
| Rate limited                                   | Wait a few minutes, or use `/peer-review quick` for a lighter call           |
| Timeout (no response after 3 min)              | Prompt may be too large. Split into smaller reviews.                         |
| Empty/partial output                           | Model returned a stub. Retry, or use single-target mode to isolate.          |

**Debug tip:** Test each model independently with `/peer-review gpt <prompt>` or `/peer-review claude <prompt>`.

## License

MIT
