# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
