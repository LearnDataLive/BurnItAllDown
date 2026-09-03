---
type: llm
criteria: |
  - Constraint 2 (uniform replication of every table regardless of content) is flagged as skippable or negotiable, with a proposal to classify tables by sensitivity/PII content first and replicate only the subset that actually needs residency compliance, reducing volume against the tiny budget.
  - Constraint 4 (existing pipelines cannot be modified, only new jobs added on top) is treated as a real constraint to design within, with a concrete proposal such as a metadata-driven downstream replication job reading off existing landing zones/shortcuts rather than proposing to modify the ingestion pipelines.
  - Constraint 1 (24-hour replication window) is flagged as worth verifying against the actual regulatory text rather than accepted at face value and over-engineered on a 2-contractor-week budget.
  - The red-team stage explicitly names the timeline risk created by constraint 3: scope frozen at week 6 while the audit is at week 8 leaves only a short buffer to catch and fix any gap the audit finds.
  - At least one leverage point or shortcut is tied directly to the 2-contractor-week budget constraint.
focus: does the review reach the substance of these findings
---
Grades whether the review surfaces the concrete, plan-specific leverage points, especially the budget and timeline pressure points.
