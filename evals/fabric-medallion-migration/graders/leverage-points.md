---
type: llm
criteria: |
  - Constraint 2 (RLS implemented independently at every layer) is identified as EXPLOITABLE or NEGOTIABLE (either label is acceptable), with the concrete fix being a single parameterized/reusable RLS module invoked at each layer instead of three separate hand-built implementations.
  - Constraint 3 (source systems touched only once) is used as a forcing function: the review proposes a maximal/defensive one-shot extraction (e.g. pulling full history, schema metadata, and change-tracking data in that single window) rather than treating the constraint as purely a limitation.
  - Constraint 4 (custom Python only, no low-code) is flagged as a negotiable team/platform convention rather than a hard technical requirement, with a proposal to use low-code tooling for simple ingestion plumbing while keeping custom code for genuine transformation logic.
  - Constraint 5 (full 11-year history required before any report goes live) is flagged as skippable or negotiable, with a concrete proposal to phase Gold-layer/report exposure (e.g. ship recent years first, backfill older history after go-live) instead of gating all reporting on complete historical migration.
  - The red-team stage explicitly names a single point of failure or unacknowledged risk arising from the combination of the 16-week deadline, the one-shot source access, and the 2-person engineering team (e.g. no recovery path if a schema issue surfaces after the extraction window closes).
focus: does the review reach the substance of these specific findings, not whether the wording matches exactly
---
Grades whether the review surfaces the concrete, plan-specific leverage points a strong reviewer should find in this scenario.
