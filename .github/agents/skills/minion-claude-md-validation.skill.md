---
id: minion-claude-md-validation
name: Minion CLAUDE Validation
type: skill
inputs:
  - selected repository key
  - project facts
  - project understanding summary
  - project understanding evidence trace map
  - project understanding completeness report
  - project research coverage report
  - project open questions and confidence levels
  - project evidence traces
  - existing project-local CLAUDE.md files
outputs:
  - per-project CLAUDE.md validity status
  - per-project CLAUDE.md validation findings map
  - per-project required CLAUDE.md updates
tools:
  - builtin:filesystem
failure_behavior:
  - If validation evidence is incomplete, do NOT mark the existing CLAUDE.md as invalid. Record which claims could not be verified and leave them as-is.
  - If upstream completeness gates failed, treat their scope as unverifiable and exclude it from required updates; do not propagate gate failures into CLAUDE.md validity.
---

## Purpose
Identify specific, evidence-backed claims in an existing project-root `CLAUDE.md` that are demonstrably outdated or incorrect, and produce a minimal set of required updates.
The existing `CLAUDE.md` is trusted by default. Absence of confirming evidence is NOT grounds for an update — only strong contradicting evidence is.

## Method
Check for `CLAUDE.md` in each project root.
If present, treat its content as authoritative unless a specific claim is directly contradicted by an extracted fact with `confidence=high` or `status=emitted`.

For each verifiable claim in `CLAUDE.md` (language, build system, test presence, architecture, dependencies, main flows):
- Look for a **direct contradiction** in the extracted facts or understanding summary. A contradiction means the evidence positively asserts a different value, not merely that no confirming evidence was found.
- If a contradiction exists and the supporting evidence trace is present, record it as a required update with the specific field, current CLAUDE.md value, and evidence-backed correct value.
- If no contradiction is found — including when research coverage is insufficient or the topic falls under an open question — leave the claim unchanged.

Do NOT flag a claim as needing an update based on:
- Open questions or unresolved topics (uncertainty does not invalidate an existing statement)
- Low-confidence-only findings (a finding must have `confidence=high` to drive an update)
- Missing evidence (absence of verification is not contradiction)
- Insufficient research coverage (unverifiable scope is excluded, not invalidated)

Emit required updates only for fields where a high-confidence direct contradiction was found.
Classify the overall CLAUDE.md as needing updates only when at least one required update was produced.
Otherwise classify as valid.

## Output Guarantees
Required updates are each backed by a specific evidence trace reference.
No update is produced from absence of evidence, low-confidence findings, open questions, or coverage gaps.
Unverifiable sections are reported as out-of-scope for this validation run, not as invalid.
