# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.1.0] — 2026-03-31

### Fixed

- **Doc drift:** Fixed 21 stale `SKILL.md` references across CONTRIBUTING.md, SECURITY-REVIEW.md, schemas, and Serena memories — now point to `plugins/peer-review/commands/peer-review.md`
- **CONTRIBUTING.md:** Fixed stale file paths, clarified Codex CLI is primary (Copilot is fallback), added marketplace plugin format note
- **Skill file:** Removed dead code — `TIMEOUT_SOFT` config (unenforced), `cherry_pick_state` from session schema, re-review #N promise from history mode
- **Skill file:** Broadened Gemini failover from 429-only to all transient errors (429, 503, timeout)
- **Skill file:** Added minimum output threshold check (20 chars quick, 50 chars multi-round)
- **Skill file:** Added git availability check for diff/iterate modes and branch existence validation for --branch
- **Skill file:** Added warning when --iterate round produces zero applied fixes
- **Skill file:** Documented why quick/gate/delegate cap at 1 round (NOTE after modes table)
- **Skill file:** Added concrete confidence scoring example in Step 5.1
- **Skill file:** Documented truncation threshold relationships (8K file + 12K peer + 40K total)
- **Skill file:** Applied model anonymization consistently in tie-breaker AND multi-mode collision

### Changed

- **README:** Added badge block (release, license, language, last commit, stars, repo size, status, Claude Code, platform, PRs welcome)
- **README:** Added missing modes (gate, delegate, status, result), flags (--effort, --json-redacted, --background, --resume), config (DEFAULT_EFFORT, JOB_DIR), and model aliases table
- **SECURITY-REVIEW.md:** Added v2.0.0 addendum covering BUG-1 through BUG-7 audit findings
- **Skill file:** Consolidated cross-exam prompts into parametrized template (~150 lines saved)
- **Skill file:** Extracted tier assignment rules into single definition referenced by Steps 5, 7, and 8
- **Skill file:** Consolidated privacy gate patterns into single definition (Step 0.2 + Step 0.5)
- **Skill file:** Moved all config to Configuration section at top
- **Skill file:** Added job cleanup rule — jobs older than 7 days auto-deleted on next /peer-review status

### Removed

- Deleted obsolete `docs/superpowers/plans/2026-03-16-copilot-cli-migration.md` (contradicts current Codex CLI architecture)
- Removed `TIMEOUT_SOFT` config (never enforced on macOS)
- Removed `cherry_pick_state` from session JSON schema (unused field)

### Meta

- Version bumped to 2.1.0 across marketplace.json and plugin.json
- Target: skill file reduced from 1402 to <1350 lines via deduplication

## [2.0.0] — 2026-03-31

### Fixed

- **BUG-1:** Notes section incorrectly documented Gemini prompt delivery as `$(cat "$PROMPT_FILE")` — now correctly describes sed-based `$ESCAPED_PROMPT` approach
- **BUG-2:** Added `<EFFORT_FLAG_GEMINI>` placeholder to Gemini bash template — `--effort` flag now applies to both GPT and Gemini
- **BUG-3:** Verified `Bash(git diff:*)` in allowed-tools covers `git diff --numstat` (no change needed)
- **BUG-4:** Fixed `ROUNDS` → `RESOLVED_ROUNDS` inconsistency throughout Steps 4, 4.5, 5, and Notes section
- **BUG-5:** Removed phantom `/peer-review cancel` subcommand reference from status output
- **BUG-6:** Documented that Copilot CLI does not support effort control — effort flag silently skipped on fallback
- **BUG-7:** Clarified COPILOT_FLAGS `-s` is for standalone copilot binary only (not gh copilot extension)

### Changed

- Removed old `claude-plugin/` directory (superseded by `plugins/peer-review/` marketplace format)
- Removed orphaned `peer-review-workspace/skill-snapshot/SKILL.md` (v0.2 relic)
- Updated README manual install path to `plugins/peer-review/commands/peer-review.md`
- Cleaned up stale remote branch `fix/project-health-audit-v0.9.1`
- Bumped plugin version to 2.0.0

## [1.1.0] — 2026-03-31

### Changed

- Bumped plugin version from 1.0.0 to 1.1.0 for marketplace auto-update
- Updated model alias resolution example in skill file from `--gpt-model spark` to `--gemini-model flash`

## [1.0.0] — 2026-03-30

### Breaking

- **Converted from manual skill to marketplace plugin format** — peer-review is now installable via `claude plugins install peer-review@peer-review` instead of manual `.claude/skills/` setup

### Added

- `.claude-plugin/marketplace.json` manifest for marketplace distribution
- `plugins/peer-review/.claude-plugin/plugin.json` plugin descriptor
- Plugin auto-update on repo push

### Changed

- Restructured from `claude-plugin/` to `plugins/peer-review/` directory layout
- Skill file moved from `claude-plugin/skills/peer-review/SKILL.md` to `plugins/peer-review/commands/peer-review.md`

## [0.10.0] — 2026-03-30

### Added

- **Review Gate mode** (`/peer-review gate`) — proactive quality check that reviews Claude's own output via GPT and Gemini before proceeding. Dispatches Claude's last response to both models with gate-specific prompts (correctness checker + architecture checker). Returns ALLOW/FLAGGED/BLOCKED verdict. Inspired by OpenAI Codex plugin stop-gate pattern, adapted for skill architecture (Step 10)
- **Task Delegation mode** (`/peer-review delegate`) — hands off implementation tasks to GPT/Gemini with write-capable permissions. External models generate patches in unified diff format. User reviews and chooses between GPT's or Gemini's implementation before applying. Safety constraints prevent auto-application (Step 11)
- **Background Execution** (`--background` flag) — dispatch reviews asynchronously with job management. New commands: `/peer-review status` (list jobs), `/peer-review result [job-id]` (retrieve completed results). Jobs persist in `JOB_DIR` with manifest tracking (Step 12)
- **Session Resumability** (`--resume [job-id]` flag) — continue prior review sessions across conversation turns. Saves session state (model outputs, decision packet, cherry-pick state) to persistent files. Supports resuming cherry-pick, adding cross-exam rounds, or re-reviewing (Step 13)
- **Effort Control** (`--effort <level>` flag) — control reasoning effort for dispatched models. Values: low, medium, high, xhigh. Maps to `--reasoning-effort` for Codex CLI and `--thinking-budget-tokens` for Gemini CLI
- **Model Aliases** — shorthand aliases for common models: `spark` (gpt-5.3-codex-spark), `mini` (gpt-5.4-mini), `flash` (gemini-2.5-flash), `pro` (gemini-2.5-pro). Use with `--gpt-model` and `--gemini-model` flags
- **Formal Output Schema** (`schemas/decision-packet.schema.json`) — JSON Schema for Decision Packet v2 output. Validates `--json` export format. Adds per-finding `confidence_score` (0.0-1.0 numeric) alongside existing HIGH/MEDIUM/LOW labels. Optional `file`, `line_start`, `line_end`, and `recommendation` fields for code-location-specific findings

### Changed

- JSON export (`--json`) now includes `confidence_score` (numeric 0.0-1.0), `effort` level, and optional file/line location fields per item
- Help mode updated with new modes (gate, delegate), flags (--effort, --background, --resume), model aliases, and job management commands
- Skill description frontmatter expanded to cover all new capabilities

## [0.9.0] — 2026-03-30

### Added

- **Gemini auto-failover** — primary `gemini-3.1-pro-preview`, automatic fallback to `gemini-2.5-pro` on 429/capacity errors. New `GEMINI_FALLBACK` config value.
- **Decision Packet v2** — tiered output (Ship Blocker / Before Next Sprint / Backlog), dependency arrows, effort estimates, and conflict flags
- **Tie-breaker model** (`gpt-5.4-mini`) for resolving HIGH CONFIDENCE deadlocks between GPT and Gemini
- **`--json-redacted` flag** — redacted JSON export that auto-strips detected secrets from output
- **Convergence identity tracking** — persistent tracking of convergence patterns across review rounds
- **Structured collision detection** — detects conflicting recommendations across modes
- **History prompt storage** — review prompts stored for `/peer-review history` retrieval
- **Multi-mode dispatch** (`--modes redteam,deploy,perf`) — run multiple review modes in parallel with cross-mode collision detection

## [0.8.0] — 2026-03-30

### Breaking

- **Copilot CLI replaced by Codex CLI + Gemini CLI as primary providers** — GPT dispatch now uses OpenAI Codex CLI (`codex exec`) instead of GitHub Copilot CLI. Gemini dispatch continues using Gemini CLI directly. Copilot CLI remains as an optional fallback for GPT.
- Install Codex CLI: `npm install -g @openai/codex` or `brew install --cask codex`
- Install Gemini CLI: `npm install -g @google/gemini-cli`
- Auth: `codex login` (or `OPENAI_API_KEY`) for GPT, `gemini auth` (or `GEMINI_API_KEY`) for Gemini

### Added

- **Auto-install offers** — if Codex CLI or Gemini CLI are missing during pre-flight, the skill offers to install them automatically
- **GPT provider fallback** — if Codex CLI is unavailable, automatically falls back to Copilot CLI for GPT dispatch
- **Codex sandbox mode restored** — `--sandbox read-only --ask-for-approval never` flags provide file system isolation during review (was lost in Copilot CLI migration)
- `GPT_CLI` config option to explicitly choose `codex` (default) or `copilot` as GPT provider

### Changed

- Pre-flight checks now detect Codex CLI first, then Copilot CLI as fallback, then Gemini CLI
- GPT dispatch template uses `cat "$PROMPT_FILE" | codex exec -s read-only -m <model> -`
- Copilot CLI dispatch template preserved as fallback (used when `GPT_CLI=copilot`)
- Privacy notice updated: prompts go directly to OpenAI (via Codex) instead of through GitHub infrastructure

## [0.7.0] — 2026-03-17

### Changed

- **Prompt dispatch uses stdin** instead of command-line arguments (`copilot < "$PROMPT_FILE"` replaces `copilot -p "$(cat "$PROMPT_FILE")"`). Eliminates prompt exposure in `ps` output and removes the ARG_MAX size limit on prompts
- **Stderr captured to temp file** instead of discarded with `2>/dev/null`. Failure diagnostics now include stderr content alongside exit codes
- **DATA markers randomized per-dispatch** — cross-examination and file context markers now use `DATA_<8_RANDOM_HEX>_START` / `DATA_<8_RANDOM_HEX>_END` (same randomization pattern as heredoc delimiters). Prevents model output from containing the exact marker string to escape the data boundary
- **Diff mode uses intelligent chunking** — large diffs are split by file/hunk with priority ordering (most-changed files first, source over config/test), and excluded files are listed explicitly. Replaces naive 8000-char truncation
- **Pre-flight checks verify auth status** — `gh auth status` is checked in addition to `command -v copilot`, catching expired or missing auth before dispatch instead of mid-run

### Fixed

- Fixed stray `n()` call at end of `grade_all.py` that would crash the grading script

## [0.6.0] — 2026-03-16

### Breaking

- **Codex + Gemini CLIs replaced by GitHub Copilot CLI** — single CLI handles both GPT and Gemini model access. Install via `brew install github/gh/copilot-cli`. Auth via `gh auth login` or `copilot login`. Requires a GitHub Copilot subscription.
- `--codex-model` flag renamed to `--gpt-model`
- `/peer-review codex` single-model mode renamed to `/peer-review gpt`

### Changed

- Single CLI dependency (Copilot CLI) replaces two separate CLIs (Codex CLI + Gemini CLI)
- Simplified auth: GitHub OAuth via `gh` CLI or `copilot login` instead of separate auth flows
- All model dispatch uses `copilot -p ... -s --no-ask-user --model <model>`
- Copilot CLI flags (`--no-ask-user`, `-s`) replace Codex sandbox flags (`-a never --sandbox read-only --ephemeral`). Note: Copilot CLI has no sandbox equivalent — see SECURITY-REVIEW.md

## [0.5.0] — 2025-03-09

### Added

- Confidence indicators on Decision Packet items: [HIGH CONFIDENCE], [MEDIUM], [LOW] based on cross-examination convergence
- Priority Matrix (Impact x Effort quadrant table) after Decision Packet
- Follow-up tracking: add TODOs in code, draft GitHub issues, or summarize as checklist
- Adversarial injection eval (eval-7) testing heredoc escapes, DATA END injection, instruction overrides
- Meta self-review eval (eval-8) testing whether the skill can critically review its own design
- CLAUDE.md with Serena MCP initialization instructions

## [0.4.0] — 2025-03-09

### Added

- Four new review modes: `refactor`, `deploy`, `api`, `perf` — each with role-differentiated prompts
- `diff` mode: reviews staged/unstaged git changes with full review treatment
- `help` command: inline reference table of all modes and options
- `history` command: shows previous reviews from the current session
- Context injection (Step 0.5): auto-reads referenced file paths (up to 3 files, 8000 char limit)
- `--verbose` flag: shows exact prompts sent and raw outputs
- `--quiet` flag: skips model sections, shows only Decision Packet + cherry-pick menu
- `--codex-model` / `--gemini-model` flags: per-invocation model overrides with injection-safe validation
- Deployment Readiness Checklist with Go/No-Go verdict (deploy mode)
- API Design Scorecard with consistency/evolvability/client experience scores (api mode)
- Performance Assessment with bottleneck ID, quick wins, load testing items (perf mode)
- Four new mode evals (eval-3 through eval-6) with 27 new assertions in grade_all.py
- Graceful handling of missing eval directories in grade_all.py

## [0.3.0] — 2025-03-07

### Added

- Codex sandbox mode (`-a never --sandbox read-only --ephemeral`) to prevent reviewer models from modifying the workspace
- Randomized heredoc delimiters (`PEER_REVIEW_EOF_<8_RANDOM_HEX>`) to block delimiter injection attacks
- Python one-liner for temp file writing — avoids all shell escaping issues with single quotes, backticks, and `$()`
- `chmod 600` on temp files to restrict read access to the current user
- DATA START/DATA END markers in cross-examination prompts for injection resistance
- `stderr` suppression (`2>/dev/null`) to hide Codex MCP startup noise and Gemini agent warnings
- Context growth control via `MAX_CROSSEXAM_CHARS` (default 12000) to prevent token explosion in rounds 3-4
- Source attribution tags (Codex/Gemini/consensus) on Decision Packet action items
- Troubleshooting table in README with common failure modes and fixes

## [0.2.0] — 2025-03-07

### Added

- Seven distinct modes: review (default), idea, redteam, debate, premortem, advocate, quick
- Role-differentiated prompts — Codex gets implementation-focused personas, Gemini gets strategic/architectural personas
- Multi-round cross-examination with configurable `ROUNDS` (1-4)
- Per-invocation `--rounds N` override
- Decision Packet with numbered, cherry-pickable action items
- Accept all / cherry-pick / refine / discard workflow
- Single-target modes (`codex`, `gemini`) for debugging individual CLIs
- Judge's Verdict section for debate mode
- Advocate vs. Critic Summary section for advocate mode

### Changed

- Renamed from "brainstorm" to "peer-review" (`/brainstorm` kept as legacy alias)

### Fixed

- Eval pass rate improved from 50% to 100%

## [0.1.0] — 2025-03-07

### Added

- Initial "brainstorm" skill with basic dispatch to Codex and Gemini CLIs
- Simple prompt forwarding to both models
- Basic result presentation
