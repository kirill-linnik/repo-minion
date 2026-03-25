---
id: minion-update-routing
name: Minion Update Routing
type: skill
inputs:
  - selected repository key
  - selected repository scanRoot boundary record
  - .repo-minion/memory/<repository-key>.md
  - explicit user task
outputs:
  - update intent classification
  - analysis readiness decision
  - update routing decision
tools:
  - builtin:filesystem
failure_behavior:
  - If repository memory is missing for an update request, route to analysis first.
  - If repository memory exists but lacks explicit analysis-readiness markers, route to analysis first.
  - If repository memory contradicts the selected repository key or scanRoot, stop and request human review.
---

## Purpose
Decide whether repo-minion should stop after analysis, run analysis before update mode, or proceed directly into dependency updates.
This skill runs immediately after repository resolution for update tasks.

## Method
Inspect the explicit user task and classify it as one of:
- `analysis`
- `update`
- `unsupported`

For `analysis`, stop after the analysis workflow completes.

For `update`, load `.repo-minion/memory/<repository-key>.md` from the repo-minion workspace root.

> **CRITICAL — path disambiguation:**
> The repo-minion workspace root is the directory that contains `repo-minion.config.json`.
> This is NOT the scanned repository's `scanRoot`.
> These are two separate directories. Always resolve the memory file path relative to the agent workspace root,
> NOT relative to `scanRoot`.
>
> Correct example (agent workspace root = `E:\drive\repo-minion`, repository key = `swimplify`):
>   `E:\drive\repo-minion\.repo-minion\memory\swimplify.md`   ← CORRECT
>
> Wrong example (do NOT look here):
>   `E:\drive\swimplify\.repo-minion\memory\swimplify.md`     ← WRONG — this is inside scanRoot

Treat the memory file as the only readiness contract.
Look for explicit markers in the `Analysis Status` and `Dependency Update Readiness` sections.

Route the update request according to these rules:
- If the memory file is missing, route to `analysis required first`.
- If `Analysis Status` is absent, incomplete, or not `completed`, route to `analysis required first`.
- If the memory file lists zero projects, stop and report that no projects were identified for update.
- If at least one project has readiness `ready` or `ready-with-caution`, route to `update may proceed` after analysis prerequisites are available.
- If all projects are `blocked`, stop and report the blocking reasons.

Never infer readiness from prose alone when the explicit markers are missing.
Never bypass repository memory for update mode.

## Output Guarantees
The routing decision is deterministic and based only on explicit task text plus repository memory markers.
No dependency update begins without either a completed memory contract or a preceding analysis run.