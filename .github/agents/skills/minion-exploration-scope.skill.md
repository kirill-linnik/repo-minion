---
id: minion-exploration-scope
name: Minion Exploration Scope
type: skill
inputs:
  - repo-minion.config.json
outputs:
  - selected repository key
  - selected repository scanRoot boundary record
  - excludePaths record
  - explicit user selection confirmation record
tools:
  - builtin:filesystem
failure_behavior:
  - If config is missing, unreadable, or invalid, stop and request human correction.
  - If user input does not include an explicit repository key/alias, stop and request explicit selection.
  - If resolved repository is not explicitly confirmed by the user, stop and request confirmation.
  - If repository key/alias cannot be resolved uniquely, stop and request human correction.
  - If selected scanRoot path is invalid, inaccessible, or does not exist, stop and return a path-debug report.
---

## Purpose
Read multi-repository scan configuration, validate settings, resolve the exact repository the user means, and establish a single-grant trust boundary rooted at the selected repository `scanRoot`.

## Method
Load `repo-minion.config.json` from repository root.
Validate config structure against `repo-minion.config.schema.json`.
Validate that `excludePaths` is present and is an array of relative path hints.
Validate that `repositories` is present and is an object map.
For each repository key, validate `scanRoot` is present and follows allowed Windows/macOS/Linux path formats.
For each repository key, validate `aliases` is present and is an array of strings.
Require an explicit repository reference from user input (repository key or alias) before resolution.
Do not infer repository intent from context, prior runs, default ordering, or a single configured repository.
Resolve the user-referenced repository by exact key match or alias match.
If resolution is not unique or not explicit, ask the user which repository they mean.
If the user did not provide a repository reference, do not infer intent; ask which repository key or alias to use.
If exactly one repository exists in config, still require explicit selection; single-entry config is not authorization.
If no repositories match, stop and ask the user for a valid key or alias from config.
If multiple repositories match the same alias, treat configuration as invalid and notify user about that.
If multiple repositories match, stop and ask the user to disambiguate using the exact repository key.
After resolving a unique repository, echo the resolved key and require explicit user confirmation before scanning.
Never guess `scanRoot`, never synthesize missing config values, and never continue with partial resolution.
Validate the resolved `scanRoot` exists and is readable.
If the path is invalid or missing, return a path-debug report with the repository key, configured path, resolved path basis, and failure reason.
Do not request additional repository access outside the resolved `scanRoot`; treat that boundary as the only readable subtree.
Record the selected repository key, normalized `scanRoot`, and `excludePaths` values for memory assembly.
Record the explicit selection and confirmation tokens used to authorize the repository boundary.

## Trust Boundary Initialization
After the user confirms the resolved repository, perform a single directory listing of the `scanRoot` itself as the first filesystem operation.
This single listing establishes the access grant for the entire `scanRoot` subtree and is the only access prompt the user should see during the entire scan session.
All subsequent filesystem reads and directory listings under `scanRoot` must reuse this grant without requesting further approvals.
Do not list or read any subdirectory before issuing this root-level listing.
If the root listing fails (access denied, path not found), stop and return the path-debug report; do not retry individual subdirectories.

## Output Guarantees
Outputs only configuration facts present in `repo-minion.config.json`.
Path-debug output contains only observable validation failures.
No files outside the selected repository `scanRoot` are read.
No additional subfolder access prompts are requested under the selected repository `scanRoot`.
No assumptions, inferred intent, or fabricated values are emitted.
Single-repository configuration never bypasses explicit selection and confirmation requirements.
