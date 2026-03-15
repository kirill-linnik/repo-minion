# repo-minion

`repo-minion` is a foundation exploration and maintenance agent for multi-repository workspaces.

It performs evidence-based discovery in a configured repository scope, generates project-local `CLAUDE.md` policy files when needed, writes fact-only memory for human review, and can optionally update dependencies project by project after analysis readiness is established.

## What It Does

1. Validates repository scan configuration.
2. Resolves a target repository by key or alias.
3. Scans only the configured `scanRoot` for that repository.
4. Identifies projects using filesystem evidence.
5. Extracts facts per project: language, build system, and tests presence (`present` or `absent`).
6. Builds an evidence-backed project understanding layer: domain/goals, architecture, dependencies, and main flows.
7. Runs validation and quality gates (completeness, coverage, contradictions, confidence).
8. Validates existing `CLAUDE.md` files against facts and understanding evidence.
9. Generates or updates `CLAUDE.md` with a model-agnostic, agent-optimized structure.
10. Writes repository-scoped memory to `.repo-minion/memory/<repository-key>.md` with verification health and provisional markers.
11. For `update <repository>` requests, checks whether analysis memory is ready and runs analysis first when needed.
12. Inventories project dependency manifests, lockfiles, and governing docs.
13. Uses GitHub MCP as the primary source for release notes and migration guidance, with maintainer or registry docs as fallback.
14. Updates dependencies project by project with precise versions and minimal compatibility changes.
15. Refreshes affected project `CLAUDE.md` files and `.repo-minion/memory/<repository-key>.md` when update work reveals new or corrected knowledge.
16. Returns a structured update report including version changes, code changes, documentation updates, knowledge refreshes, and follow-up lockfile-regeneration commands.

## What It Does Not Do

- No automatic repository selection.
- No manual lockfile editing.
- No test execution or simulated test execution.
- No ungrounded architecture or intent inference.
- No hypotheses in memory.

## Quality Controls

- Evidence-first: every emitted fact must be traceable to concrete files.
- Anti-speculation: unresolved or low-confidence items go to `Open Questions`.
- Phase gates: understanding facts are emitted only after inventory, extraction, cross-validation, contradiction, and emission gates pass.
- Confidence model: `high`, `medium`, `low`, `unknown` used deterministically.
- Coverage diagnostics: scan and readability metrics are captured for review.
- Provisional mode: projects with failed completeness or coverage gates are marked `provisional`.

## Configuration

Config file: `repo-minion.config.json`

Schema file: `repo-minion.config.schema.json`

The config supports shared excludes and a repository map:

```json
{
	"$schema": "./repo-minion.config.schema.json",
	"excludePaths": ["node_modules", "dist", "build", "out", ".git"],
	"repositories": {
		"repo-key": {
			"aliases": ["alias-a", "alias-b"],
			"scanRoot": "path/to/repo-or-subtree"
		}
	}
}
```

## GitHub MCP Contract

GitHub MCP is a runtime capability, not something repo-minion installs from repository files.

- This repository's workspace MCP configuration is [`.vscode/mcp.json`](.vscode/mcp.json).
- The configured VS Code MCP server name is `github`.
- The `minion-release-notes-research` skill reads the workspace MCP configuration from `.vscode/mcp.json` and uses the workspace `github` server as the primary source for release notes and latest-version research.
- When GitHub MCP is unavailable or insufficient for a dependency, the skill falls back to package-maintainer or registry documentation and records the fallback source in the update report.

## Path Validation

`scanRoot` accepts Windows and macOS/Linux styles:

- Windows absolute: `C:\code\repo` or `C:/code/repo`
- macOS/Linux absolute: `/Users/me/repo`
- UNC: `\\server\share\repo`
- Relative: `repos/app`, `repos\\app`, `./repos/app`

If `scanRoot` is invalid, inaccessible, or does not exist, the agent returns actionable path-debug diagnostics and stops.

## Repository Trust Boundary

For each run, one repository is selected by key or alias.
Selection must be explicit in user input.
The agent must never auto-select a repository based on defaults, prior context, or single-entry configuration.

- The selected repository `scanRoot` is the single-grant trust boundary.
- The agent requests access once to that `scanRoot`.
- The agent may read recursively inside that boundary.
- The agent does not read outside that boundary.
- For analysis-only tasks, the resolved repository still requires explicit confirmation before traversal.
- For direct `update <repository>` tasks, the task phrase itself authorizes work after unique repository resolution.

## Output Structure

- Agent definition: `.github/agents/repo-minion.agent.md`
- Skills: `.github/agents/skills/*.skill.md`
- Agent-owned state root: `.repo-minion/`
- Memory per repository: `.repo-minion/memory/<repository-key>.md`

Key skill flow:

Shared entry:
1. `minion-exploration-scope`

Analysis mode:
1. `minion-exploration-scope`
2. `minion-structural-mapping`
3. `minion-project-identification`
4. `minion-project-fact-extraction`
5. `minion-project-understanding`
6. `minion-claude-md-validation`
7. `minion-claude-md-generation`
8. `minion-memory-assembly`

Update mode:
1. `minion-exploration-scope`
2. `minion-update-routing`
3. If readiness is missing or stale, run the full analysis mode sequence first.
4. `minion-dependency-inventory`
5. `minion-release-notes-research`
6. `minion-dependency-application`
7. If new or corrected knowledge was learned, rerun `minion-claude-md-validation`, `minion-claude-md-generation`, and `minion-memory-assembly`.
8. `minion-update-summary`

`.repo-minion/` is safe to delete and regenerate.

## Memory Policy

Memory is facts only.

- Only statements directly supported by files are stored.
- Unknowns go to `Open Questions`.
- `CLAUDE.md` is authoritative per project root.
- Memory contains explicit analysis-readiness markers for update mode.
- Humans review memory before any further automation for analysis-only tasks.
- Verified facts include evidence references and rule ids.
- Projects can be marked `verified` or `provisional` based on completeness/coverage gates.
- Update mode refreshes `CLAUDE.md` and repository memory when verified knowledge changes.

## Update Workflow

Use `update <repository-key-or-alias>` to trigger dependency maintenance.

Update mode works as follows:

1. Resolve the repository from configuration.
2. Check `.repo-minion/memory/<repository-key>.md` for explicit readiness markers.
3. If analysis is incomplete or missing, run the analysis workflow first.
4. Inspect each project's manifest, lockfile, `README.md`, and `CLAUDE.md` to determine allowed dependency-management commands.
5. Research upgrades using GitHub MCP as the primary source, then fall back to package-maintainer or registry documentation when needed.
6. Update project manifests with precise versions, apply minimal compatibility changes, and record lockfile-regeneration commands.
7. Refresh affected `CLAUDE.md` files and repository memory if dependency work revealed new or corrected knowledge.
8. Return a structured report.

## Human Review Checklist

- Confirm `repo-minion.config.json` matches schema and intended scope.
- Confirm user explicitly selected a repository key or alias (no implicit default selection).
- Confirm analysis-only runs explicitly confirmed the resolved repository key before traversal.
- Confirm direct `update <repository>` runs used an explicit repository key or alias and did not bypass unique resolution.
- Confirm selected repository key or alias resolves to exactly one repository.
- Confirm all recorded project roots are within selected `scanRoot`.
- Confirm each project has project-local `CLAUDE.md`.
- Confirm recorded language/build/test facts are evidence-backed.
- Confirm understanding outputs (domain, architecture, flows, dependencies) include evidence traces.
- Confirm completeness and coverage reports were generated.
- Confirm projects with failed gates are marked `provisional`.
- Confirm CLAUDE validation findings and required updates are reflected.
- Confirm memory file path is `.repo-minion/memory/<repository-key>.md`.
- Confirm memory includes `Analysis Status` and `Dependency Update Readiness` markers.
- Confirm `Open Questions` has unresolved items and no assumptions.
