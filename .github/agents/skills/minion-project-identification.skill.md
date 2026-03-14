---
id: minion-project-identification
name: Minion Project Identification
type: skill
inputs:
  - structural path map
outputs:
  - project root records under selected repository scanRoot (path plus ecosystem candidates)
  - root evidence map with rule ids and file paths
tools:
  - builtin:filesystem
failure_behavior:
  - If no qualifying manifest evidence is found, output an empty project list.
---

## Purpose
Identify project roots and preliminary ecosystem/technology candidates using concrete build or manifest files found in the selected repository structure.

## Method
Evaluate observed files against a deterministic project-root evidence matrix.
Treat a directory as a project root only when evidence satisfies one of the proof obligations:
- at least one strong root artifact exists in that same directory
- or at least two weak artifacts from the same ecosystem exist in that same directory
- or exactly one weak artifact from an ecosystem AND at least two source files bearing that ecosystem's canonical extensions exist anywhere within the directory subtree (excluding ignored paths); see Source-File Extension Supplement below
Use weak artifacts only to support confidence or pair with canonical source-file presence; never as a single-artifact root proof on their own.
If nested directories both contain strong root artifacts, record both roots.
If multiple manifests exist in one directory, treat it as one project root with multi-tool evidence.
Never infer roots from folder names alone.
Do not use file extensions alone as root proof; they may supplement exactly one weak artifact but cannot constitute a root on their own.
Ignore excluded/generated paths when evaluating root evidence: `node_modules`, `.venv`, `vendor`, `dist`, `build`, `out`, `.terraform`, `.git`.
For each emitted root, attach ecosystem candidates derived only from matched evidence rules.
Output normalized project root records.

### Project Root Evidence Matrix

| Ecosystem | Strong root artifacts | Weak supporting artifacts |
|---|---|---|
| Node.js / TypeScript / JavaScript | `package.json` | `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `tsconfig.json`, `vite.config.*`, `webpack.config.*`, `jest.config.*` |
| Python | `pyproject.toml`, `setup.py`, `setup.cfg`, `Pipfile` | `requirements.txt`, `poetry.lock`, `tox.ini`, `pytest.ini`, `manage.py`, `.python-version` |
| Java | `pom.xml`, `build.gradle`, `build.gradle.kts` | `settings.gradle`, `settings.gradle.kts`, `gradle.properties`, `mvnw`, `gradlew` |
| .NET | `*.sln`, `*.csproj`, `*.fsproj`, `*.vbproj` | `Directory.Build.props`, `Directory.Packages.props`, `global.json`, `NuGet.Config` |
| Go | `go.mod` | `go.sum`, `go.work` |
| Rust | `Cargo.toml` | `Cargo.lock`, `rust-toolchain.toml` |
| Ruby | `Gemfile` | `Gemfile.lock`, `.ruby-version`, `Rakefile` |
| PHP | `composer.json` | `composer.lock`, `phpunit.xml` |
| Elixir | `mix.exs` | `mix.lock` |
| Dart / Flutter | `pubspec.yaml` | `.metadata`, `analysis_options.yaml` |
| C / C++ (CMake) | `CMakeLists.txt` | `conanfile.*`, `vcpkg.json` |
| Swift | `Package.swift` | `Package.resolved` |
| Android | `local.properties` | `build.gradle`, `build.gradle.kts`, `settings.gradle`, `gradlew`, `gradle.properties`, `proguard-rules.pro` |
| iOS / macOS (Xcode) | `Podfile`, `*.xcodeproj` (directory), `*.xcworkspace` (directory) | `Podfile.lock`, `*.xcconfig`, `Info.plist` |
| Terraform | `*.tf` plus at least one of `.terraform.lock.hcl`, `providers.tf`, or a `terraform {}` block in any root `*.tf` | `*.tfvars`, `backend.tf`, `.terraform.lock.hcl` |
| Helm | `Chart.yaml` | `values.yaml` |

### Source-File Extension Supplement

When exactly one weak artifact is present and no strong artifact qualifies the directory, check for canonical source-file extensions within the directory subtree (excluding ignored paths). Two or more matching files promote the root with `evidence_strength: weak-extension`.

| Ecosystem | Canonical source-file extensions |
|---|---|
| Node.js / TypeScript / JavaScript | `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs` |
| Python | `.py` |
| Java | `.java`, `.kt`, `.groovy` |
| .NET | `.cs`, `.fs`, `.vb` |
| Go | `.go` |
| Rust | `.rs` |
| Ruby | `.rb` |
| PHP | `.php` |
| Elixir | `.ex`, `.exs` |
| Dart / Flutter | `.dart` |
| Android | `.java`, `.kt` (corroborates `local.properties` weak root only when no Java strong artifact is present) |
| iOS / macOS (Xcode) | `.swift`, `.m`, `.mm`, `.h` |
| Swift | `.swift` |
| C / C++ (CMake) | `.c`, `.cpp`, `.cc`, `.cxx`, `.h`, `.hpp` |

### Mobile Platform Notes

| Platform | Detection strategy | Key differentiator |
|---|---|---|
| Android | `local.properties` (strong) triggers an Android root; overlapping Java/Gradle strong artifacts (`build.gradle`) alone classify a root as Java — upgrade to Android only when `local.properties` is co-located | `local.properties` contains `sdk.dir`, which is Android-SDK-exclusive |
| iOS / macOS (Xcode) | `Podfile` or an `*.xcodeproj` / `*.xcworkspace` directory at root (strong) | CocoaPods and Xcode project bundles are Apple-platform-exclusive |
| Flutter / Dart | Covered by the Dart/Flutter row via `pubspec.yaml` (strong); co-presence of `android/`, `ios/`, `web/` sibling directories corroborates cross-platform Flutter; a `flutter:` key in `pubspec.yaml` confirms Flutter over plain Dart | Already in matrix |
| React Native | Covered by the Node.js row via `package.json` (strong); `metro.config.*` or `react-native.config.*` as weak corroboration upgrades the ecosystem label to React Native | React Native roots always have `package.json`; no separate matrix row needed |
| Kotlin Multiplatform Mobile (KMM) | Covered by the Java row via `build.gradle.kts` (strong); sibling directories `composeApp/` or `shared/` corroborate KMM | KMM roots always satisfy Java root rules; no separate matrix row needed |

### Ambiguity Handling

If a directory has only weak artifacts and fewer than two canonical source files for those ecosystems, do not mark it as a root.
If no strong artifacts are found anywhere, return an empty project list.
If root evidence conflicts or is incomplete, keep only directly provable roots and push unresolved cases to `Open Questions`.
If a directory has only one weak artifact but two or more canonical source files for that ecosystem (see Source-File Extension Supplement), mark it as a root with `evidence_strength: weak-extension`.
If a directory has only one weak artifact and fewer than two canonical source files for that ecosystem, do not mark it as a root.

### Evidence Trace Requirement

For each emitted root, include evidence trace fields:
- `rule_id`: deterministic rule name used for the decision
- `evidence_files`: exact relative file paths used as proof
- `evidence_strength`: `strong`, `weak-pair`, or `weak-extension`
- `ecosystem_candidates`: one or more ecosystem labels matched by evidence

## Output Guarantees
Every project root in output is tied to at least one observed manifest or build file.
No project root is emitted from naming conventions alone.
Every emitted root is auditable via explicit evidence file paths.
Ecosystem candidates are evidence-derived hints and not final language/build facts.
