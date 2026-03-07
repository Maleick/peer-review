# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
