---
id: minion-memory-assembly
name: Minion Memory Assembly
type: skill
inputs:
  - selected repository key
  - scan scope facts
  - project list with names and roots
  - root evidence map with rule ids and file paths
  - project facts
  - per-project domain and goals summary
  - per-project architecture summary
  - CLAUDE.md paths
outputs:
  - .repo-minion/memory/<repository-key>.md
  - repository memory assembly report
  - analysis readiness record
  - per-project dependency update readiness map
tools:
  - builtin:filesystem
failure_behavior:
  - If domain or goals summary is unavailable for a project, record the project name and root only and note the missing summary in Open Questions.
  - If CLAUDE.md paths are unavailable, record path as missing and continue.
---

## Purpose
Produce a lightweight, human-readable portfolio overview of a scanned repository: what projects it contains, what they collectively do, how projects within the repository relate to each other, and links to CLAUDE.md files for depth.
This file is the overview layer of a two-layer memory model. CLAUDE.md files are the detail layer.
The memory file is also the readiness contract for dependency update mode.
This skill may be rerun after dependency updates to reconcile newly learned or corrected repository facts.

## Method
Write `.repo-minion/memory/<repository-key>.md` inside the **repo-minion agent workspace root** (not inside the scanned repository's `scanRoot`). Resolve this path relative to the agent's own workspace root, never relative to the target repository.

> **CRITICAL — path disambiguation:**
> The agent workspace root is the directory that contains `repo-minion.config.json`.
> The scanned repository `scanRoot` is the directory configured in that config file.
> These are two separate directories. The memory file MUST be written under the agent workspace root,
> NOT under `scanRoot`.
>
> Correct example (agent workspace root = `E:\drive\repo-minion`, repository key = `swimplify`):
>   `E:\drive\repo-minion\.repo-minion\memory\swimplify.md`   ← CORRECT
>
> Wrong example (do NOT write here):
>   `E:\drive\swimplify\.repo-minion\memory\swimplify.md`     ← WRONG — this is inside scanRoot

When update mode changes dependencies or learns new stable facts, refresh the existing memory file instead of leaving stale facts in place.
Preserve the same structure, but replace outdated repository or project facts with the newly verified values.

Produce the file with the following structure:

```
# Repository: <repository-key>
Scan date: <date>
Scan root: <scanRoot>

## Analysis Status
Status: completed
Ready for dependency update: yes | no
Reason: <machine-checkable reason>

## Common Goal / Domain
One-paragraph synthesis of what this repository is collectively "about" based on the domain and goals summaries of its projects.
If projects span distinct domains with no visible connection, state that explicitly.

## Projects
For each project: name, one-line purpose, root path (relative to scanRoot), primary manifest path, lockfile path if any, detected build system, link to CLAUDE.md file, status: `verified` (CLAUDE.md exists and project completeness gates passed) or `provisional` (CLAUDE.md missing, incomplete, or gated).

## Cross-Project Connections
Active: scan configuration files (env templates, docker-compose, nginx config, CI/CD pipelines), dependency manifests, and import statements for references from one project within this repository to another project within the same repository.
Record each signal as:
  signal_type | source_project | found_in | target_project

Signal types to detect:
- `import_path` — an import or require in one project's source that resolves into another project's root
- `package_ref` — a dependency in one project's manifest that matches another project's package name
- `service_call` — a URL, service name, or host in one project's config or code that matches another project's known service name or port
- `shared_config` — a config file (env, docker-compose, CI pipeline) that declares or wires two or more projects together

If no signals are found after scanning, record: "No cross-project connections detected."

## CLAUDE.md Index
| Project | CLAUDE.md path | Status |

## Dependency Update Readiness
For each project record:
  project | primary_manifest | lockfile | package_manager_or_tooling | readiness (`ready`, `ready-with-caution`, `blocked`) | reason

Readiness rules:
- `ready` when the project root, primary manifest, and governing `CLAUDE.md` or equivalent project instructions are present.
- `ready-with-caution` when the project is `provisional` but the manifest and dependency-management rules are still directly observable.
- `blocked` when the manifest, project root, or governing dependency instructions cannot be proven.

## Open Questions
Low-confidence items only: things that could not be confirmed from available evidence.
Prioritize by impact: high, medium, low.
```

## Output Guarantees
Overview contains only directly observed facts.
CLAUDE.md files are the authoritative source of project-level detail.
The `Analysis Status` and `Dependency Update Readiness` sections must be explicit enough that update mode can decide whether to proceed without heuristic guessing.
If update mode revealed new or corrected facts, the memory file is rewritten so it remains the current repository snapshot.
Human review is required before any further automation acts on this memory for analysis-only runs.
