---
id: minion-dependency-inventory
name: Minion Dependency Inventory
type: skill
inputs:
  - selected repository key
  - project root records under selected repository scanRoot
  - root evidence map with rule ids and file paths
  - project facts
  - existing project-local CLAUDE.md files
  - repository memory readiness record
outputs:
  - per-project primary dependency manifest path
  - per-project lockfile path or absence record
  - per-project package manager or build tool command family
  - per-project dependency update governing docs list
  - per-project update applicability decision
tools:
  - builtin:filesystem
failure_behavior:
  - If a project has no provable primary manifest, mark it `blocked` for update and continue to the next project.
  - If dependency instructions are ambiguous, mark the project `ready-with-caution` only when the manifest is still directly observable.
---

## Purpose
Determine exactly which file and toolchain controls dependency updates for each project in the selected repository.

## Method
For each identified project, inspect project-root manifests, lockfiles, README files, and project-local `CLAUDE.md`.
Use concrete files to identify one primary dependency manifest in the project root.
Prefer the manifest that directly controls dependency declarations for the detected build system.

Map the primary manifest and command family using deterministic evidence:
- Node.js: `package.json`
- Python: `pyproject.toml`, `Pipfile`, `requirements.txt`, `setup.py`, or `setup.cfg` based on the proven build-system fact
- Java: `pom.xml`, `build.gradle`, or `build.gradle.kts`
- .NET: `Directory.Packages.props`, `*.csproj`, `*.fsproj`, or `*.vbproj` based on central-package-management evidence
- Go: `go.mod`
- Rust: `Cargo.toml`
- Ruby: `Gemfile`
- PHP: `composer.json`
- Dart / Flutter: `pubspec.yaml`

Identify lockfiles if present, but do not treat them as editable targets.

Read existing `README*` and project-local `CLAUDE.md` files for dependency-management rules before choosing commands.
If project rules forbid a common command family, record the approved alternative and use that alternative in downstream update steps.

Emit `update applicability` per project:
- `ready` when the primary manifest and governing docs are directly observable.
- `ready-with-caution` when the primary manifest is observable but governing docs are incomplete or the project remains provisional.
- `blocked` when the manifest or governing rules cannot be proven.

## Output Guarantees
Each project receives at most one primary dependency manifest.
No package-manager command family is selected without file-backed evidence.
Lockfiles are recorded only as regeneration targets, never as manual-edit targets.