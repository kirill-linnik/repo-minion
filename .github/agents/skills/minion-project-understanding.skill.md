---
id: minion-project-understanding
name: Minion Project Understanding
type: skill
inputs:
  - selected repository key
  - structural path map
  - project root records (path and ecosystem candidates)
  - root evidence map with rule ids and file paths
  - project facts
outputs:
  - per-project domain and goals summary (evidence-backed)
  - per-project architecture summary (components and boundaries)
  - per-project dependency summary (internal and external)
  - per-project main flows and entrypoints summary
  - per-project understanding evidence trace map
  - per-project open questions and confidence levels
  - per-project understanding completeness report
  - per-project research coverage report
tools:
  - builtin:filesystem
failure_behavior:
  - If understanding signals are insufficient, emit only directly supported structural summaries and add unresolved items to Open Questions.
  - If phase gates fail or evidence thresholds are not met, do not emit understanding facts.
---

## Purpose
Produce an evidence-backed project understanding layer after fact extraction, without introducing ungrounded claims.

## Method
For each identified project, analyze project-local source and configuration files to extract explicit signals about purpose, architecture, dependencies, and runtime flows.
Use only observable artifacts and code-level references.
Prefer declarative sources first: README files, manifest descriptions, service configuration, API/route declarations, and dependency descriptors.
Then refine from implementation structure: module boundaries, package imports, entrypoint files, and wiring/bootstrap code.
For each statement, attach evidence trace with exact relative file paths and rule ids.
If intent or flow cannot be proven, do not emit it as fact; add it to Open Questions with confidence `low`.

### Mandatory Phase Gates

Execute in order and stop fact emission if any gate fails:

1. `inventory_gate`: enumerate candidate files under project root and record coverage counters.
2. `extraction_gate`: produce raw claim candidates as tuples (`claim_type`, `statement`, `evidence_files`).
3. `cross_validation_gate`: validate each non-trivial claim with independent evidence sources.
4. `contradiction_gate`: detect conflicting evidence and move unresolved items to `Open Questions`.
5. `emission_gate`: emit only claims that satisfy thresholds and trace requirements.

### Evidence Thresholds

- Domain/goals: at least 2 evidence items from at least 2 source classes.
- Architecture: at least 2 components plus at least 1 interaction proof.
- Dependencies: direct manifest evidence required; integration claims require config or import evidence.
- Main flows: observable chain required (`entrypoint -> router/consumer -> handler/service`).

If thresholds are not met, do not emit the claim as fact.

### Source Classes

Classify evidence into source classes:
- `docs`
- `manifests`
- `config`
- `code_structure`
- `runtime_flow`

Single-source-class claims must be downgraded or moved to `Open Questions`.

### Evidence Sources
- Project documentation: `README*`, `docs/**`, architecture notes.
- Build and dependency descriptors: manifests and lockfiles already identified in prior steps.
- Runtime/config files: environment templates, application settings, framework config, deployment descriptors.
- Entrypoints and wiring: `main*`, `app*`, server bootstrap files, dependency injection or startup files.
- API/workflow definitions: route/controller declarations, handlers, message consumers, jobs, pipeline definitions.
- Data boundary signals: ORM configs, schema/migration files, repository/data-access layers.

### Output Rules
- Domain and goals: only from explicit textual or code-level business signals.
- Architecture: component boundaries and interactions only when directly visible in code/config.
- Dependencies: summarize direct dependencies from manifests plus notable runtime integrations proven by imports/config.
- Main flows: summarize request/event/job paths only when entrypoint-to-handler chain is observable.
- Confidence levels: `high`, `medium`, `low` per item, based on evidence completeness.

### Deterministic Confidence Rules

- `high`: at least 3 evidence files, at least 2 source classes, and no contradictions.
- `medium`: 2 evidence files and at least 2 source classes.
- `low`: partial support or single-source-class support.
- `unknown`: insufficient or contradictory evidence; do not emit as fact.

### Speculation Firewall

Do not place speculative phrasing in facts.
If wording requires terms such as `likely`, `probably`, `seems`, `appears`, `maybe`, or `assume`, move the item to `Open Questions`.

### Contradiction Policy

When conflicting signals exist, record both observations with evidence and emit no resolved fact.
Add one explicit `Open Questions` item describing what evidence is missing to resolve the conflict.

### Evidence Trace Schema

For each emitted understanding item, include:
- `claim_id`
- `claim_type`
- `statement`
- `rule_id`
- `evidence_files`
- `evidence_snippets`
- `source_classes`
- `confidence`
- `status` (`emitted` or `open_question`)

### Coverage and Completeness Reports

Output a research coverage report with:
- scanned file count
- scanned directory count
- excluded file count
- unreadable file list
- evidence-bearing file count

Output a completeness report with pass/fail for:
- domain/goals
- architecture
- dependencies
- main flows
- contradiction resolution

### Null-Result Rule

If strong evidence is not available, emit only structural summaries and unresolved questions.
Do not force narrative understanding from weak or partial evidence.

### Recommended Additional Outputs
- External integrations map (databases, queues, third-party APIs, cloud services).
- Runtime topology hints (single service, multi-service, worker + API, monolith modules).
- Cross-project link candidates (shared libs, referenced sibling projects).
- Risk and ambiguity notes for human review.

## Output Guarantees
No output item is emitted without evidence trace.
Unproven interpretations are excluded from facts and moved to Open Questions.
The skill remains read-only and evidence-bound.
Coverage and completeness diagnostics are always produced, even when no understanding facts are emitted.
