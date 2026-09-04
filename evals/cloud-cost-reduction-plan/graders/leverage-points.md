---
type: llm
criteria: |
  - The prod-db-01 exclusion is classified as TRULY FIXED for the cluster itself, but the review also probes the boundary of that exclusion -- e.g. proposing to look at read replicas, caching layers, adjacent non-prod copies, or network/traffic patterns connected to prod-db-01 that are not literally the excluded cluster and so are not covered by the Reliability team's ownership.
  - The 30-day visibility preference and the hallway "20%" figure are each explicitly flagged as not real constraints, and the review proposes treating them as informal success signals to report against rather than as deadlines or targets to design around.
  - At least one concrete, high-leverage shortcut is proposed that is grounded in the plan's own context numbers rather than generic cost-cutting advice -- for example, a cleanup pass on the sandbox account (explicitly called out as never having had one) as a fast, low-risk win, or evaluating additional savings-plan/reserved-instance purchases ahead of the 5-month renewal window while pricing is known.
  - The red-team stage identifies the "whoever wants to pick this up, part-time" staffing model as a real risk of the effort stalling or never being prioritized against regular workload, independent of any stated constraint.
  - The red-team stage identifies the absence of a named owner or a defined success metric as a risk in its own right, not merely restated as a missing constraint.
focus: does the review find genuine, plan-specific substance on an open canvas rather than treating "few constraints" as "nothing to say"
---
Grades whether the review surfaces concrete, plan-specific findings despite -- and because of -- how open-ended this plan is.
