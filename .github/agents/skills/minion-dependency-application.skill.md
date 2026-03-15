---
id: minion-dependency-application
name: Minion Dependency Application
type: skill
inputs:
  - selected repository key
  - selected repository scanRoot boundary record
  - per-project dependency upgrade candidate list
  - per-dependency migration action list
  - per-project dependency update governing docs list
  - existing project-local CLAUDE.md files
  - .repo-minion/memory/<repository-key>.md
outputs:
  - updated project manifests with precise versions
  - required code changes for dependency compatibility
  - required documentation updates for dependency compatibility
  - refreshed project-local CLAUDE.md files when knowledge changed
  - refreshed repository memory file when knowledge changed
  - knowledge reconciliation notes
  - lockfile regeneration command list
tools:
  - builtin:filesystem
  - builtin:command
failure_behavior:
  - If a project is marked `blocked`, skip it and record the reason.
  - If release-note research does not justify a safe upgrade path for a dependency, leave that dependency unchanged and record why.
---

## Purpose
Apply dependency upgrades project by project while respecting repository instructions, project-local rules, and the migration guidance gathered earlier.
Keep project-level and repository-level knowledge artifacts current when update work reveals new facts.

## Method
Process projects one by one within the selected repository.
Before changing a project, read its governing docs list, including `README*` and `CLAUDE.md`, and follow any documented package-manager restrictions.

For each project marked `ready` or `ready-with-caution`:
- Update only the primary manifest with precise dependency versions.
- Do not manually edit lockfiles.
- Record the exact command needed to regenerate each lockfile after manifest changes.
- Apply only the minimal code changes required by documented breaking changes and recommended migrations.
- Update existing documentation files only when dependency versions, commands, APIs, or instructions became outdated.
- If dependency research or compatibility work reveals new or corrected facts about the project, refresh the corresponding existing `CLAUDE.md` so it reflects the latest verified knowledge.
- Do not create new files.
- Do not run tests.

If a dependency upgrade requires code changes:
- Replace deprecated APIs with the documented supported alternative.
- Keep refactors local and compatibility-driven.
- Do not restructure the project or rewrite large sections.

After processing all projects:
- Reconcile repository-level knowledge into `.repo-minion/memory/<repository-key>.md` when dependency changes or update work revealed new or corrected repository facts.
- Reuse the established memory structure and keep changes evidence-backed.
- Leave `CLAUDE.md` and repository memory unchanged when no new stable facts were learned.

## Output Guarantees
Every manifest change uses a precise version.
Lockfiles are never manually edited.
Code and documentation changes remain scoped to documented compatibility work.
Project-local `CLAUDE.md` files and repository memory are refreshed when update mode learns something new that would otherwise leave them stale.