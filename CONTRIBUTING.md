# Contributing

Thanks for your interest in contributing to peer-review. This is a small, focused project — a single Claude Code skill file plus supporting docs. Here's how to help.

## Adding a New Mode

All modes are defined in `SKILL.md`. To add one, you need to touch three places in that file:

1. **Modes table** (Step 0 area) — add a row with the invocation, behavior, and default rounds
2. **Step 1 prompts** — add a new section under "Parse Mode and Build Prompts" with role-differentiated prompts for both Codex and Gemini. Follow the existing pattern: Codex gets an implementation/tactical persona, Gemini gets a strategic/architectural persona. The two prompts should produce genuinely different perspectives, not two versions of the same answer.
3. **Step 5 template** — if your mode needs a custom summary section (like debate's "Judge's Verdict" or advocate's "Advocate vs. Critic Summary"), add it under Step 5. Otherwise the default Decision Packet template applies.

Update the mode list in the skill's `description` frontmatter so Claude Code knows when to trigger the skill for the new mode.

## Running Evals

An eval framework lives in `peer-review-workspace/`. It includes grading scripts and iteration snapshots used during development. Refer to that directory if you want to validate changes against the existing test cases.

## Submitting PRs

Standard fork-and-PR workflow:

1. Fork the repo and create a feature branch
2. Make your changes
3. Test every affected mode with at least one real invocation (e.g., if you changed the redteam prompts, run `/peer-review redteam <something>` and verify the output)
4. Open a PR with a clear description of what changed and why

Keep changes to `SKILL.md` minimal and focused. The skill file is the entire runtime — a stray character can break invocation.

## Security Considerations

The CLI invocation templates in SKILL.md contain several security measures that must be preserved:

- **Codex sandbox flags** (`-a never --sandbox read-only --ephemeral`) — prevents the reviewer model from modifying the user's workspace. Never remove these.
- **Randomized heredoc delimiters** — the `PEER_REVIEW_EOF_<8_RANDOM_HEX>` pattern must generate fresh random hex on every invocation. This prevents user input from injecting the delimiter to escape the heredoc.
- **Temp file cleanup** — every temp file must have a `trap` for cleanup on failure and explicit `rm -f` after use.
- **`chmod 600`** on temp files — restricts read access to the current user.
- **DATA START/DATA END markers** — cross-examination prompts wrap peer output in these markers with instructions to treat the content as data, not instructions. This resists prompt injection through model outputs.
- **Python one-liner for file writes** — prompts are written via `python3 -c "import sys; open(sys.argv[1],'w').write(sys.stdin.read())"` to avoid shell escaping issues. Do not replace this with `echo` or `cat >`.

If you're modifying the CLI invocation templates, make sure all of the above remain intact.
