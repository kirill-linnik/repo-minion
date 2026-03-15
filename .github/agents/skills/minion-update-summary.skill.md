---
id: minion-update-summary
name: Minion Update Summary
type: skill
inputs:
  - per-project dependency upgrade candidate list
  - updated project manifests with precise versions
  - required code changes for dependency compatibility
  - required documentation updates for dependency compatibility
  - refreshed project-local CLAUDE.md files when knowledge changed
  - refreshed repository memory file when knowledge changed
  - knowledge reconciliation notes
  - lockfile regeneration command list
outputs:
  - structured final update report
tools:
  - builtin:filesystem
failure_behavior:
  - If no dependency changes were applied, still emit the structured report and state that no safe updates were made.
---

## Purpose
Produce the final dependency-update report in a deterministic structure that summarizes stack detection, version changes, compatibility work, documentation updates, and follow-up actions.

## Method
The final answer for update mode must use this exact structure:

1. `Detected Stack`
Describe the project stack and how it was identified from repository files.

2. `Dependency Updates`
Provide a table with columns:
- package or library name
- old version
- new version
- notes

3. `Code Changes`
Show required code modifications in unified diff format.

4. `Documentation Updates`
Show updated sections of existing documentation files only, including `CLAUDE.md` when project knowledge changed.

5. `Summary of Impact`
Explain what changed, why it changed, how it affects the project, which knowledge artifacts were refreshed, and any follow-up actions such as lockfile-regeneration commands.

6. `Project Instructions`
State which project-local instruction files governed dependency-management choices.

7. `Command Safety Check`
State that package-manager commands were selected only after verifying they do not violate project-local rules.

If analysis had to run first, mention that at the start of the report.
If some projects were skipped or blocked, include them in the table or summary rather than omitting them.
If project-local `CLAUDE.md` files or `.repo-minion/memory/<repository-key>.md` were refreshed, state what was updated and why.

## Output Guarantees
Update mode reports use one consistent structure.
Skipped, blocked, and unchanged dependencies remain visible in the final report.