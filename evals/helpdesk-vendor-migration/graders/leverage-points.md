---
type: llm
criteria: |
  - Constraint 4 (40 signed Freshdesk seats, vendor says not renegotiable) is classified as TRULY FIXED or NEGOTIABLE with a concrete angle if NEGOTIABLE is chosen (e.g. checking the signed contract for a seat-count amendment clause or true-up window) rather than being accepted or dismissed with no comment.
  - The ADR's frozen SAML configuration (decision 2) is treated as a forcing function rather than a blocker: the review proposes integrating Freshdesk SSO purely as a service provider against the existing, unchanged IdP metadata, without requesting any IdP-side change.
  - The 2.3-million-ticket historical migration gate (constraint 2) is flagged as negotiable or skippable, with a concrete phased proposal (e.g. cut over live/open tickets and recent history first, backfill the older archive as a read-only export after go-live) rather than treating full historical migration as a hard blocker to cutover.
  - The red-team stage explicitly names the collision between the ADR's 100-requests/minute API gateway limit for chat events and the plan's 2-hour cumulative downtime budget for live chat -- e.g. that peak chat volume during a cutover window could exceed 100 requests/minute and get throttled, counting against the downtime budget in a way neither document accounts for.
  - The red-team stage names the single point of failure created by combining the 20-hours/week contractor being the only integration engineer with the frozen SAML configuration: if the SSO integration misbehaves, there is no fast path to get IT Security to change anything before the Q1 audit review.
focus: does the review reach the substance of these findings, especially the ones that require combining a detail from the plan with a detail from the ADR
---
Grades whether the review surfaces the concrete, plan-and-ADR-specific leverage points a strong reviewer should find in this scenario.
