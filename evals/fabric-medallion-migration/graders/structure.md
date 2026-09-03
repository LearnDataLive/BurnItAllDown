---
type: llm
criteria: |
  - The response contains all five stages of the review, clearly labeled and in order: enumerate constraints, classify each constraint, propose asymmetric shortcuts, red-team the plan, and a close (ranked leverage points + steelman + anti-sycophancy check).
  - No stage is silently skipped; if a stage genuinely has nothing to add, that is stated explicitly rather than the stage being omitted.
  - Every constraint from the plan (deadline, RLS-per-layer, one-shot source access, no-low-code, 11-year history gate, QA-parity, team size) is enumerated in Stage 1.
  - At least one constraint is classified as TRULY FIXED, or is explicitly defended as non-negotiable with a stated reason -- the review does not treat every constraint as equally negotiable.
  - Every leverage point or red-team finding cites the specific constraint number or concrete detail (e.g. "16 weeks", "2 data engineers", "11 years") it is attacking, rather than speaking generically.
focus: structural completeness and specificity, not whether you personally agree with each classification
---
Grades whether the review follows the mandated five-stage structure faithfully and grounds its claims in the actual plan rather than producing generic template filler.
