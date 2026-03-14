---
id: minion-project-fact-extraction
name: Minion Project Fact Extraction
type: skill
inputs:
  - selected repository key
  - project root records (path and ecosystem candidates)
  - root evidence map with rule ids and file paths
outputs:
  - per-project language fact
  - per-project build system fact
  - per-project tests presence fact
  - per-fact evidence trace map
tools:
  - builtin:filesystem
failure_behavior:
  - If a fact cannot be proven from files, do not emit it as a fact and add it to Open Questions.
---

## Purpose
Extract only provable project facts for each identified project root.

## Method
For each project root record, treat ecosystem candidates from identification as the authoritative upstream classification input.
Use the root evidence map as the required provenance source for upstream decisions.
Do not perform independent ecosystem discovery in this step.
Validate candidate ecosystems against root-local artifacts before emitting facts.
Derive language from confirmed ecosystem candidate mappings.
Derive build system from confirmed root-local build descriptors.
Use deterministic precedence when multiple indicators exist.
Determine tests presence only from explicit test evidence.
Treat unknown as the default outcome unless proof obligations are met.
Record only facts directly supported by files in that project.

### Evidence Scope Guardrails

Use root-local manifest/build artifacts as the primary validation source.
Use extension-based evidence only as secondary confirmation when needed.
Allow extension-based confirmation only from project-owned paths such as root, `src/`, `app/`, and `lib/`.
Ignore generated/dependency paths: `node_modules`, `.venv`, `vendor`, `dist`, `build`, `out`, `.terraform`, `.git`.
Do not emit language/build facts from file names alone when no matching manifest/build descriptor exists.

### Ecosystem Confirmation and Language Derivation

Confirm each ecosystem candidate from root-local artifacts and map to language:

| Ecosystem candidate | Confirmation artifacts | Derived language |
|---|---|---|
| Node.js / TypeScript / JavaScript | `package.json` | TypeScript or JavaScript (TypeScript if `tsconfig.json` or `*.ts`/`*.tsx` exists, otherwise JavaScript) |
| Python | `pyproject.toml`, `setup.py`, `setup.cfg`, `Pipfile`, `requirements.txt` | Python |
| Java | `pom.xml`, `build.gradle`, `build.gradle.kts` | Java |
| .NET | `*.sln`, `*.csproj`, `*.fsproj`, `*.vbproj` | C# / F# / VB.NET based on project files present |
| Go | `go.mod` | Go |
| Rust | `Cargo.toml` | Rust |
| Ruby | `Gemfile` | Ruby |
| PHP | `composer.json` | PHP |
| Elixir | `mix.exs` | Elixir |
| Dart / Flutter | `pubspec.yaml` | Dart |
| C / C++ (CMake) | `CMakeLists.txt` | C/C++ |
| Swift | `Package.swift` | Swift |
| Terraform | `*.tf` | Terraform |
| Helm | `Chart.yaml` | Helm |

If no ecosystem candidate can be confirmed from root-local artifacts, do not emit language fact and add to `Open Questions`.

### Build System Evidence Rules

Map build system from concrete build descriptors:

| Build system | Evidence artifacts |
|---|---|
| npm | `package.json` + `package-lock.json` |
| pnpm | `package.json` + `pnpm-lock.yaml` |
| yarn | `package.json` + `yarn.lock` |
| Maven | `pom.xml` |
| Gradle | `build.gradle` or `build.gradle.kts` |
| dotnet | `*.sln`, `*.csproj`, `*.fsproj`, `*.vbproj` |
| pip/pyproject | `pyproject.toml`, `requirements.txt`, `setup.py`, `setup.cfg` |
| pipenv | `Pipfile` |
| poetry | `pyproject.toml` + `poetry.lock` |
| Go modules | `go.mod` |
| Cargo | `Cargo.toml` |
| Bundler | `Gemfile` |
| Composer | `composer.json` |
| CMake | `CMakeLists.txt` |
| SwiftPM | `Package.swift` |
| Terraform CLI | `*.tf` |

### Test Presence Rules

Set tests presence to `present` if any explicit test artifact exists:

- Test directories: `test/`, `tests/`, `__tests__/`, `spec/`.
- Test config files: `pytest.ini`, `tox.ini`, `phpunit.xml`, `jest.config.*`, `vitest.config.*`, `karma.conf.*`.
- Build/test declarations: `package.json` contains `scripts.test`; `pom.xml` contains surefire/failsafe plugins; Gradle test task configuration; `.csproj` references common test SDK packages.
- Language-specific test files: patterns like `*_test.go`, `test_*.py`, `*.spec.ts`, `*.test.ts`, `*.spec.js`, `*.test.js`, `*Tests.java`, `*Test.java`.

Set tests presence to `absent` only if no explicit test artifacts are found.

### Conflict and Precedence Rules

If Node.js candidate is confirmed and TypeScript signals exist, set language to TypeScript; otherwise set to JavaScript.
If multiple JS package managers are indicated by lockfiles, select the lockfile present in root using precedence `pnpm-lock.yaml` > `yarn.lock` > `package-lock.json`.
If both Maven and Gradle descriptors exist in root, set build system to `multi-build` and add detail in `Open Questions` only if conflicting wrapper/tool files are present.
If Python tooling indicators conflict, use precedence `poetry` > `pipenv` > `pip/pyproject` based on root files.
If build system cannot be proven, do not emit build-system fact and add an item to `Open Questions`.
If language cannot be proven, do not emit language fact and add an item to `Open Questions`.

### Evidence Trace Requirement

For each emitted fact, include evidence trace fields:
- `fact_key`: `language`, `build_system`, or `tests_presence`
- `rule_id`: deterministic rule name used for the decision
- `evidence_files`: exact relative file paths used as proof
- `decision`: `emitted` or `open_question`

## Output Guarantees
Outputs only the fields: language, build system, tests presence.
No inferred architecture, intent, or roadmap data is produced.
Each emitted fact is auditable via explicit evidence file paths.
