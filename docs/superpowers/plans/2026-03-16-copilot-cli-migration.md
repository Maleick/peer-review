# Copilot CLI Migration — GSD Pass-Off Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete the migration from separate Codex + Gemini CLIs to a unified GitHub Copilot CLI for the peer-review skill, including all docs, evals, and testing.

**Architecture:** The skill now dispatches to GPT and Gemini models via `copilot -p "prompt" -s --no-ask-user --model <model>` instead of separate `codex exec` and `gemini` commands. One CLI, one auth (GitHub OAuth via `gh`), two models. The core SKILL.md changes are already done — this plan covers the remaining docs, evals, and validation work.

**Tech Stack:** GitHub Copilot CLI 1.0.5, `gh` CLI auth, Python 3 (evals grading), Bash (skill dispatch)

**What's already done:**
- SKILL.md fully migrated (Codex → GPT, gemini CLI → copilot CLI, all templates/labels/config updated)
- CLAUDE.md updated
- Copilot CLI installed and verified working with both `gpt-5.4` and `gemini-3-pro-preview` models
- Quick mode live-tested end-to-end (both models responded)

**What's left:**
1. Update eval grading assertions (grade_all.py checks for "Codex" keyword — will fail)
2. Update eval metadata and evals.json descriptions
3. Update project docs (README, CHANGELOG, SECURITY-REVIEW, CONTRIBUTING)
4. Investigate GPT exit code 1 issue (output arrives but exit code is non-zero)
5. Run multi-mode end-to-end tests
6. Commit

---

## Chunk 1: Eval Infrastructure Updates

### Task 1: Fix grade_all.py assertion for role differentiation

The `has-role-differentiation` assertion on line 108-109 checks for the literal string "Codex" in results. Since output now says "GPT", this will fail on all new eval runs.

**Files:**
- Modify: `peer-review-workspace/evals/grade_all.py:108-109`

- [ ] **Step 1: Update the has-role-differentiation assertion**

Change:
```python
"has-role-differentiation": combined(
    keywords_present(["Codex"], 1),
    keywords_present(["Gemini"], 1),
),
```

To:
```python
"has-role-differentiation": combined(
    keywords_present(["GPT"], 1),
    keywords_present(["Gemini"], 1),
),
```

- [ ] **Step 2: Update identifies-cli-dependency assertion (line 219-221)**

Change:
```python
"identifies-cli-dependency": keywords_present(
    ["CLI", "dependency", "codex", "gemini", "coupling", "version"], 2
),
```

To:
```python
"identifies-cli-dependency": keywords_present(
    ["CLI", "dependency", "copilot", "gemini", "coupling", "version"], 2
),
```

- [ ] **Step 3: Run grading script to verify no assertion compile errors**

Run: `cd /opt/peer-review/peer-review-workspace/evals && python3 grade_all.py --list`
Expected: Lists all checks without errors

- [ ] **Step 4: Commit**

```bash
git add peer-review-workspace/evals/grade_all.py
git commit -m "fix(evals): update assertions from Codex to GPT/copilot"
```

### Task 2: Update evals.json expected_output descriptions

The `expected_output` fields in evals.json reference "Codex" throughout. These are documentation/descriptions (not graded), but should be accurate for anyone reading them.

**Files:**
- Modify: `peer-review-workspace/evals/evals.json`

- [ ] **Step 1: Replace all "Codex" references with "GPT" in expected_output fields**

Specific replacements across all eval entries:
- "implementation-focused Codex feedback" → "implementation-focused GPT feedback"
- "Codex should identify" → "GPT should identify"
- "adversarial analysis from Codex" → "adversarial analysis from GPT"
- "strong PRO-microservices argument from Codex" → "strong PRO-microservices argument from GPT"
- "tactical Codex feedback" → "tactical GPT feedback"
- "deployment engineering feedback from Codex" → "deployment engineering feedback from GPT"
- "API implementation feedback from Codex" → "API implementation feedback from GPT"
- "performance engineering feedback from Codex" → "performance engineering feedback from GPT"
- "Codex (ways to game" → "GPT (ways to game"
- "strong defense from Codex" → "strong defense from GPT"
- "technical post-mortem from Codex" → "technical post-mortem from GPT"

Use global find-replace of "Codex" → "GPT" within the file. The word only appears in expected_output descriptions.

- [ ] **Step 2: Verify JSON is still valid**

Run: `python3 -c "import json; json.load(open('peer-review-workspace/evals/evals.json'))"`
Expected: No error

- [ ] **Step 3: Commit**

```bash
git add peer-review-workspace/evals/evals.json
git commit -m "docs(evals): update eval descriptions from Codex to GPT"
```

### Task 3: Update eval_metadata.json files

Each eval directory under `peer-review-workspace/evals/iteration-1/` has an `eval_metadata.json` that may reference "Codex".

**Files:**
- Modify: All `peer-review-workspace/evals/iteration-1/eval-*/eval_metadata.json` files that contain "Codex"

- [ ] **Step 1: Find and list affected files**

Run: `grep -rl "Codex" peer-review-workspace/evals/iteration-1/`
Expected: List of eval_metadata.json files

- [ ] **Step 2: Replace "Codex" with "GPT" in all affected metadata files**

For each file, replace "Codex" → "GPT" in description/expected_output fields. These are non-graded documentation fields — the assertions in grade_all.py are what matter for correctness.

- [ ] **Step 3: Verify all JSON files are valid**

Run: `for f in peer-review-workspace/evals/iteration-1/eval-*/eval_metadata.json; do python3 -c "import json; json.load(open('$f'))" && echo "OK: $f" || echo "FAIL: $f"; done`
Expected: All OK

- [ ] **Step 4: Commit**

```bash
git add peer-review-workspace/evals/iteration-1/
git commit -m "docs(evals): update eval metadata from Codex to GPT"
```

---

## Chunk 2: Project Documentation Updates

### Task 4: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Read current README**

Read the file to understand the scope of changes needed.

- [ ] **Step 2: Update all Codex → GPT references**

Key changes:
- Installation: remove `codex` and `gemini` CLI install instructions, replace with `copilot` CLI install (`brew install github/gh/copilot-cli`)
- Auth: replace "OpenAI API key" and "Google OAuth" instructions with `gh auth login` or `copilot login`
- Usage examples: update any `--codex-model` flags to `--gpt-model`
- Single-target mode: `/peer-review codex` → `/peer-review gpt`
- Model names: mention `gpt-5.4` and `gemini-3-pro-preview` as defaults
- Prerequisites: GitHub Copilot subscription (Pro, Pro+, Business, or Enterprise)

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: update README for Copilot CLI migration"
```

### Task 5: Update CHANGELOG.md

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Read current CHANGELOG**

Read to understand the versioning scheme and format.

- [ ] **Step 2: Add a new version entry at the top**

Add an entry for this migration (suggest v0.6.0 since this is a significant change). Key points:
- **Breaking:** `codex` and `gemini` CLIs replaced by `copilot` CLI — users must install Copilot CLI and have a GitHub Copilot subscription
- **Breaking:** `--codex-model` flag renamed to `--gpt-model`
- **Breaking:** `/peer-review codex` mode renamed to `/peer-review gpt`
- **Changed:** Single CLI dependency instead of two
- **Changed:** Auth simplified to GitHub OAuth (no separate OpenAI/Google auth needed)
- **Changed:** Both GPT and Gemini models now explicitly configurable via `GPT_MODEL` and `GEMINI_MODEL` config

- [ ] **Step 3: Commit**

```bash
git add CHANGELOG.md
git commit -m "docs: add v0.6.0 changelog for Copilot CLI migration"
```

### Task 6: Update SECURITY-REVIEW.md

**Files:**
- Modify: `SECURITY-REVIEW.md`

- [ ] **Step 1: Read current SECURITY-REVIEW**

Understand what security properties are documented.

- [ ] **Step 2: Update security review for new CLI**

Key changes:
- Replace "Codex sandbox (`-a never --sandbox read-only --ephemeral`)" references with Copilot CLI flags (`--no-ask-user`, `-s`)
- Note: Copilot CLI does NOT have an equivalent sandbox/read-only mode — this is a security regression worth documenting
- Update the "Gemini sandbox limitation" note — both models now go through the same CLI with the same limitations
- Update provider references: "OpenAI for Codex" → "OpenAI for GPT (via GitHub Copilot)"
- Note: prompts now route through GitHub's infrastructure as an intermediary before reaching OpenAI/Google

- [ ] **Step 3: Commit**

```bash
git add SECURITY-REVIEW.md
git commit -m "docs: update security review for Copilot CLI migration"
```

### Task 7: Update CONTRIBUTING.md

**Files:**
- Modify: `CONTRIBUTING.md`

- [ ] **Step 1: Read and update Codex references**

Replace "Codex" → "GPT" and update any CLI-specific instructions.

- [ ] **Step 2: Commit**

```bash
git add CONTRIBUTING.md
git commit -m "docs: update CONTRIBUTING for Copilot CLI migration"
```

---

## Chunk 3: Testing & Validation

### Task 8: Investigate GPT exit code 1

During the live test, `copilot -p ... --model gpt-5.4` returned exit code 1 but still produced valid output. This needs investigation.

**Files:**
- None (investigation only, may lead to SKILL.md adjustment)

- [ ] **Step 1: Test GPT with stderr visible**

Run: `copilot -p "Say hello" -s --no-ask-user --model gpt-5.4`
Check: What exit code is returned? Does stderr contain anything useful?

- [ ] **Step 2: Test GPT without -s flag to see full output**

Run: `copilot -p "Say hello" --no-ask-user --model gpt-5.4`
Check: Is there stats output that reveals why exit code is 1?

- [ ] **Step 3: Test with a simpler prompt**

Run: `copilot -p "Reply with exactly: OK" -s --no-ask-user --model gpt-5.4; echo "EXIT=$?"`
Check: Is exit code 1 consistent or intermittent?

- [ ] **Step 4: Document findings**

If exit code 1 is consistent with GPT model but output is valid:
- Update SKILL.md Step 3 (Handle Failures) to note that GPT via Copilot may return exit code 1 while still producing valid output
- Adjust the failure check in the bash template: instead of checking `$EXIT_CODE -ne 0`, check for empty output as the failure condition
- OR: remove exit code checking entirely and rely solely on empty-output detection

If exit code 0 is normal:
- The initial exit code 1 was likely because GPT tried to use tools (it said "I pulled the official CLI docs first") — the `--no-ask-user` flag may cause tool-using attempts to fail with code 1 while the final text response still comes through. This is likely a non-issue for production prompts that don't trigger tool use.

- [ ] **Step 5: If template change needed, update SKILL.md and commit**

### Task 9: End-to-end mode tests

Run at least 3 different modes to validate the Copilot CLI integration works across the skill.

**Files:**
- None (testing only)

- [ ] **Step 1: Test review mode (2 rounds)**

Run: `/peer-review quick What are the trade-offs of using a single CLI tool for multiple AI model providers?`
Expected: Both GPT and Gemini respond, output follows quick mode format

- [ ] **Step 2: Test single-target GPT mode**

Run: `/peer-review gpt What is 2+2?`
Expected: Only GPT responds, single-target format

- [ ] **Step 3: Test single-target Gemini mode**

Run: `/peer-review gemini What is 2+2?`
Expected: Only Gemini responds, single-target format

- [ ] **Step 4: Test model override flag**

Run: `/peer-review quick --gpt-model gpt-5.2 Is this working?`
Expected: GPT uses gpt-5.2 instead of gpt-5.4, both respond

- [ ] **Step 5: Test help mode (no CLI dispatch)**

Run: `/peer-review help`
Expected: Inline help table shown with `gpt` (not `codex`) references, no CLI calls made

- [ ] **Step 6: Document test results**

Record pass/fail for each test. If any fail, debug and fix before committing.

---

## Chunk 4: Cleanup & Legacy Files

### Task 10: Handle legacy workspace files (optional, low priority)

The `peer-review-workspace/` directory contains old eval results, snapshots, and raw outputs that reference "Codex". These are historical artifacts.

**Files:**
- `peer-review-workspace/iteration-1/` (old eval results)
- `peer-review-workspace/skill-snapshot/SKILL.md` (old skill snapshot)
- `peer-review-workspace/grade_all.py` (duplicate grading script)

- [ ] **Step 1: Decide on approach**

Options:
- **Leave as-is:** These are historical records — "Codex" references are accurate for when they were created
- **Archive:** Move to `peer-review-workspace/archive/pre-copilot-migration/`
- **Delete:** Remove old snapshots and results if they're not needed

Recommendation: **Leave as-is.** Historical results should reflect the tool used at the time. Only the active evals under `peer-review-workspace/evals/` need updating (covered in Tasks 1-3).

- [ ] **Step 2: If archiving, move files and commit**

### Task 11: Update settings.local.json permissions (if needed)

**Files:**
- Modify: `.claude/settings.local.json`

- [ ] **Step 1: Check if codex-specific permissions exist**

Read `.claude/settings.local.json` and check if any allowlisted commands reference `codex`.

- [ ] **Step 2: Add copilot CLI to permissions if needed**

If the permissions allowlist includes bash commands for `codex`, replace with `copilot` equivalents. The skill dispatches via the Bash tool, so `copilot` needs to be in the allowlist.

- [ ] **Step 3: Commit if changed**

```bash
git add .claude/settings.local.json
git commit -m "chore: update permissions for copilot CLI"
```

---

## Chunk 5: Final Commit & Push

### Task 12: Final verification and commit

- [ ] **Step 1: Run full grep to verify no remaining "Codex" in active files**

Run: `grep -r "Codex" --include="*.md" --include="*.py" --include="*.json" /opt/peer-review/.claude/ /opt/peer-review/CLAUDE.md /opt/peer-review/README.md /opt/peer-review/CHANGELOG.md /opt/peer-review/SECURITY-REVIEW.md /opt/peer-review/CONTRIBUTING.md /opt/peer-review/peer-review-workspace/evals/`
Expected: No matches (or only in historical result files)

- [ ] **Step 2: Run grading script**

Run: `cd /opt/peer-review/peer-review-workspace/evals && python3 grade_all.py`
Expected: All skipped (no result files) — no assertion errors

- [ ] **Step 3: Review git diff for completeness**

Run: `git diff --stat`
Verify all expected files are modified.

- [ ] **Step 4: Create final commit (if not already committed in pieces)**

```bash
git add -A
git commit -m "feat: migrate from Codex+Gemini CLIs to unified GitHub Copilot CLI (v0.6.0)

Replace separate codex and gemini CLI dependencies with a single copilot CLI.
Both GPT and Gemini models are now called via copilot -p with --model flag.
Auth simplified to GitHub OAuth via gh CLI.

Breaking changes:
- --codex-model flag renamed to --gpt-model
- /peer-review codex mode renamed to /peer-review gpt
- Requires GitHub Copilot CLI (brew install github/gh/copilot-cli)
- Requires GitHub Copilot subscription (Pro, Pro+, Business, or Enterprise)"
```

- [ ] **Step 5: Push and verify**

Run: `git push origin main`

---

## Key Risks & Notes

1. **GPT exit code 1:** Needs investigation (Task 8). May be benign (tool-use attempts in non-interactive mode) or may need template adjustment. Test with pure-text prompts that won't trigger tool use.

2. **Security regression:** Codex CLI had `--sandbox read-only --ephemeral` flags that prevented the reviewer model from modifying files. Copilot CLI has `--no-ask-user` but no equivalent sandbox. Document this in SECURITY-REVIEW.md. For the peer-review use case this is low risk since prompts are review-only (no file modification instructions).

3. **Premium request quotas:** GitHub Copilot Pro+ gives 1500 premium requests/month. A 2-round peer review uses 4 CLI calls (2 models x 2 rounds). Heavy usage could exhaust the quota. Worth noting in README.

4. **Model availability:** The `--model` choices list may change as GitHub adds/removes models. The skill should gracefully handle "model not available" errors. Current failure handling (Task 8 / Step 3 in SKILL.md) already covers this.

5. **Legacy `codex` alias:** Some users might have muscle memory for `/peer-review codex`. Consider whether to add a backwards-compatibility alias `codex` → `gpt` in the mode parser. Recommendation: don't — clean break is clearer.
