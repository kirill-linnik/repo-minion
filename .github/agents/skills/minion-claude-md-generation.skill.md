---
id: minion-claude-md-generation
name: Minion CLAUDE Generation
type: skill
inputs:
  - selected repository key
  - project facts
  - project understanding summary
  - per-project domain and goals summary (evidence-backed)
  - per-project runtime topology hints
  - project understanding evidence trace map
  - project understanding completeness report
  - project research coverage report
  - project open questions and confidence levels
  - project evidence traces
  - CLAUDE.md validation status
  - CLAUDE.md validation findings map
  - per-project required CLAUDE.md updates
outputs:
  - generated or replaced project-local CLAUDE.md files
  - per-project CLAUDE.md generation rationale map
  - per-project unresolved instruction gaps
tools:
  - builtin:filesystem
failure_behavior:
  - If required facts are missing, do not generate and add the gap to Open Questions.
  - If understanding completeness is insufficient, generate only high-confidence sections and mark the rest as unresolved.
---

## Purpose
Create or replace project-root `CLAUDE.md` files when missing or invalid.

## Method
For each project root with missing or invalid `CLAUDE.md`, write a new file in that same project root.
Populate content strictly from extracted facts, validation findings, and evidence-backed project understanding outputs.
Use an agent-optimized structure that minimizes lookup time and ambiguity for any LLM runtime.
Prioritize high-value operational guidance first (entrypoints, workflows, boundaries, and constraints).
Include only high/medium-confidence understanding items as facts.
Place low-confidence or unresolved items in a dedicated `Open Questions` section.
Embed concise evidence references for critical sections so agents can verify quickly.
Add an explicit `agent-generated, review required` disclaimer.
Do not write `CLAUDE.md` outside identified project roots.

### Required CLAUDE.md Structure (Model-Agnostic)

Generate sections in this order:
1. `Project Snapshot` (language, build system, test presence, project root)
2. `Domain and Goals` (what problem the project solves, for whom, and any core invariants or business constraints; populated from domain and goals summary; omit section if no high/medium-confidence domain signals exist and add to Open Questions instead)
3. `Fast Start for Agents` (entrypoints, key commands if known from files, main code paths)
4. `Architecture and Boundaries` (components, ownership boundaries, interaction edges, runtime topology: single service / multi-service / worker+API / monolith modules)
5. `Main Flows` (request/event/job flow chains)
6. `Dependencies and Integrations` (direct deps, external systems, data boundaries)
7. `Repository Rules and Constraints` (do-not-touch zones, trust boundaries, safety constraints)
8. `Evidence Notes` (short references to source files/rules)
9. `Open Questions` (unresolved or low-confidence items)
10. `Disclaimer` (`agent-generated, review required`)

### Optimization Rules For Faster Agent Execution

- Use short, deterministic bullets.
- Prefer absolute clarity over prose.
- Put critical navigation hints at top (`where to start`, `where flows begin`, `where configs live`).
- Avoid repeating the same fact across sections.
- Never include speculative statements in fact sections.
- Keep unresolved content isolated in `Open Questions`.

## Output Guarantees
Each generated file is project-local and fact-only.
No cross-project policy file is produced.
Generated files are optimized for fast agent orientation while remaining evidence-bound.
