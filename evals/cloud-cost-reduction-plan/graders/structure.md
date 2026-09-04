---
type: llm
criteria: |
  - The response contains all five stages of the review, clearly labeled and in order: enumerate constraints, classify each constraint, propose asymmetric shortcuts, red-team the plan, and a close (ranked leverage points + steelman + anti-sycophancy check).
  - Stage 1 lists only what the plan actually states as fixed or as a stated preference -- the prod-db-01 exclusion and the 30-day visibility preference -- and does not invent additional numbered "fixed constraints" (a specific budget cap, a hard deadline, a mandated team size, or a hard reduction percentage) that the plan does not state.
  - The hallway "20% would be nice" figure and the 30-day visibility preference are explicitly identified as non-binding (a rumor and a preference, respectively), not silently promoted to hard requirements in Stage 1 or Stage 2.
  - Stage 4 (red-team) surfaces at least two concrete risks that are not tied to any stated constraint at all -- for example, no named owner or success metric for the effort, or the part-time/"whoever wants to pick this up" staffing model risking the project stalling indefinitely -- proving the stage still does real work on an open canvas rather than being padded with generic filler because there was little to attack in Stages 1-2.
  - No stage is silently skipped or reduced to a single filler sentence just because the plan has few fixed constraints; if a stage has genuinely little to add, that is stated explicitly.
focus: whether the review stays honest about how few constraints actually exist, rather than inventing structure to fill the five stages
---
Grades whether the review follows the mandated five-stage structure without padding it with invented constraints on a plan that has almost none.
