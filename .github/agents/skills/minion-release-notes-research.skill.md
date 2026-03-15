---
id: minion-release-notes-research
name: Minion Release Notes Research
type: skill
inputs:
  - workspace MCP configuration at .vscode/mcp.json
  - workspace GitHub MCP server name `github`
  - per-project primary dependency manifest path
  - per-project lockfile path or absence record
  - per-project package manager or build tool command family
  - per-project dependency update governing docs list
outputs:
  - per-project dependency upgrade candidate list
  - per-dependency current version and target version
  - per-dependency breaking-change summary
  - per-dependency migration action list
  - per-dependency research provenance
tools:
  - builtin:filesystem
  - github:mcp
failure_behavior:
  - If `.vscode/mcp.json` does not define a `github` server, record the workspace MCP contract failure and fall back to package-maintainer or registry documentation.
  - If GitHub MCP is unavailable for a dependency, use package-maintainer or registry documentation as fallback and record the fallback source.
  - If neither GitHub MCP nor maintainer documentation can establish a safe target version, leave the dependency unchanged and record it as unresolved.
  - If a GitHub tag or release version is not confirmed present in the canonical registry for that ecosystem, treat it as unpublished and fall back to the registry-confirmed latest version. Record the discrepancy as `github-tag-unpublished; registry-latest used`.
---

## Purpose
Research available dependency upgrades and the migration work required before any manifest or code change is made.

## Method
Load `.vscode/mcp.json` and confirm the workspace defines a `github` MCP server.

For each updatable project, enumerate direct dependencies from the primary manifest.
Prefer exact currently declared versions from the manifest over inferred lockfile versions.

For each dependency candidate:
1. Confirm the configured `primarySource` is `github-mcp` and use the workspace `github` MCP server first:
   a. Call `get_latest_release` on the upstream GitHub repository.
   b. If `get_latest_release` returns **404** (the repository does not publish formal releases), call `list_tags` on the same repository and derive the latest version from the most recent semver tag. Record the fallback as `github-mcp:tags`.
   c. If `get_latest_release` or `list_tags` returns **403** (SAML enforcement or insufficient token scope), treat GitHub MCP as unavailable for this dependency and proceed immediately to the configured fallback sources. Do not retry GitHub MCP for that dependency. Record the error code and organization in the provenance note.
   d. Any other GitHub MCP error (network failure, unexpected 5xx) is treated the same as 403 — fall through to configured fallback sources and record the failure.
2. If GitHub MCP did not provide sufficient data, use only the configured fallback source classes in order.
3. Determine the latest available version, including major releases.
4. **Registry cross-check (required for all ecosystem packages):** After determining a candidate version from GitHub MCP, verify that version exists in the dependency's canonical package registry before accepting it as the target:
   - Python → PyPI: fetch `https://pypi.org/pypi/<package>/<version>/json` and confirm a 200 response.
   - Node.js → npm registry: fetch `https://registry.npmjs.org/<package>/<version>` and confirm a 200 response.
   - Java (Maven Central) → use `maven-tools` MCP `check_version_exists` for the exact groupId:artifactId:version.
   - CocoaPods → verify the podspec for the version exists in the CocoaPods Specs trunk or the pod's source repository.
   If the registry does not have the candidate version, the tag was never published and must be rejected as a target. Use the highest version the registry confirms as the authoritative latest instead. Record the discrepancy in the provenance note as `github-tag-unpublished; registry-latest used`. Never emit a target version derived solely from a GitHub release tag or git tag without confirming it is present in the corresponding registry.
5. Summarize breaking changes, required migrations, deprecated API replacements, and recommended compatibility changes.
6. Record the provenance source used for the decision and whether it came from the primary source or a configured fallback.

Select target versions conservatively but precisely:
- Use a precise target version number, not an open-ended range.
- Prefer the latest stable version when release notes and migration guidance are sufficient.
- Skip pre-release versions unless the current project already tracks pre-releases and project-local docs explicitly allow them.

## Output Guarantees
Every proposed target version has recorded provenance.
Every breaking-change note is tied to release-note or maintainer documentation evidence.
Dependencies without adequate migration evidence are left unchanged rather than guessed.
The emitted provenance identifies the workspace MCP server as `github`.