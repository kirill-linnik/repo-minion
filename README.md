# repo-minion

`repo-minion` is a foundation exploration agent for multi-repository workspaces.

It performs read-only, evidence-based discovery in a configured repository scope, generates project-local `CLAUDE.md` policy files when needed, and writes fact-only memory for human review.

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
11. Stops and requires human review.

## What It Does Not Do

- No dependency upgrades.
- No command execution.
- No network access.
- No MCP usage.
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

## Path Validation

`scanRoot` accepts Windows and macOS/Linux styles:

- Windows absolute: `C:\code\repo` or `C:/code/repo`
- macOS/Linux absolute: `/Users/me/repo`
- UNC: `\\server\share\repo`
- Relative: `repos/app`, `repos\\app`, `./repos/app`

If `scanRoot` is invalid, inaccessible, or does not exist, the agent returns actionable path-debug diagnostics and stops.

## Repository Trust Boundary

For each run, one repository is selected by key or alias.
Selection must be explicit in user input and explicitly confirmed after resolution.
The agent must never auto-select a repository based on defaults, prior context, or single-entry configuration.

- The selected repository `scanRoot` is the single-grant trust boundary.
- The agent requests access once to that `scanRoot`.
- The agent may read recursively inside that boundary.
- The agent does not read outside that boundary.

## Output Structure

- Agent definition: `.github/agents/repo-minion.agent.md`
- Skills: `.github/agents/skills/*.skill.md`
- Agent-owned state root: `.repo-minion/`
- Memory per repository: `.repo-minion/memory/<repository-key>.md`

Key skill flow:
1. `minion-exploration-scope`
2. `minion-structural-mapping`
3. `minion-project-identification`
4. `minion-project-fact-extraction`
5. `minion-project-understanding`
6. `minion-claude-md-validation`
7. `minion-claude-md-generation`
8. `minion-memory-assembly`

`.repo-minion/` is safe to delete and regenerate.

## Memory Policy

Memory is facts only.

- Only statements directly supported by files are stored.
- Unknowns go to `Open Questions`.
- `CLAUDE.md` is authoritative per project root.
- Humans review memory before any further automation.
- Verified facts include evidence references and rule ids.
- Projects can be marked `verified` or `provisional` based on completeness/coverage gates.

## Human Review Checklist

- Confirm `repo-minion.config.json` matches schema and intended scope.
- Confirm user explicitly selected a repository key or alias (no implicit default selection).
- Confirm user explicitly confirmed the resolved repository key before traversal.
- Confirm selected repository key or alias resolves to exactly one repository.
- Confirm all recorded project roots are within selected `scanRoot`.
- Confirm each project has project-local `CLAUDE.md`.
- Confirm recorded language/build/test facts are evidence-backed.
- Confirm understanding outputs (domain, architecture, flows, dependencies) include evidence traces.
- Confirm completeness and coverage reports were generated.
- Confirm projects with failed gates are marked `provisional`.
- Confirm CLAUDE validation findings and required updates are reflected.
- Confirm memory file path is `.repo-minion/memory/<repository-key>.md`.
- Confirm `Open Questions` has unresolved items and no assumptions.
