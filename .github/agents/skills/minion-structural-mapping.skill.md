---
id: minion-structural-mapping
name: Minion Structural Mapping
type: skill
inputs:
  - selected repository key
  - selected repository scanRoot boundary
  - excludePaths
outputs:
  - observable directory and file map under selected repository scanRoot
tools:
  - builtin:filesystem
failure_behavior:
  - If traversal cannot complete, record only successfully observed paths and stop further steps.
---

## Purpose
Walk the selected repository subtree and record observable structure using filesystem evidence only.

## Method
Recursively enumerate directories and files under the selected repository `scanRoot`.
Apply `excludePaths` as traversal optimization hints.
Record only paths that are directly observed.
Do not infer project boundaries or roles during this step.

## Output Guarantees
Produces a path map derived only from observed filesystem entries.
Includes no interpretation beyond observed structure.
