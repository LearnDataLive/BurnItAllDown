---
type: llm
criteria: |
  - The response contains all five stages of the review, clearly labeled and in order: enumerate constraints, classify each constraint, propose asymmetric shortcuts, red-team the plan, and a close (ranked leverage points + steelman + anti-sycophancy check).
  - All five stated constraints (3-named-approver rule, applies to all repos including docs-only, 4-week/40-repo timeline, no CI pipeline changes, unchanged workload for the 3 engineers) are enumerated in Stage 1.
  - At least one constraint is classified as TRULY FIXED or explicitly defended as non-negotiable.
  - Every finding cites a specific constraint or number from the plan (3 engineers, 40 repos, 4 weeks) rather than speaking generically.
focus: structural completeness and specificity
---
Grades whether the review follows the mandated five-stage structure and grounds claims in the specific plan.
