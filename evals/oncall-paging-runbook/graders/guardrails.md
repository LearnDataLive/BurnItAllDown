---
type: llm
criteria: |
  - The review does not invent constraints, numbers, or facts that are not present in the plan text.
  - The review does not propose full automation of all P1 resolution, since constraint 1 explicitly forbids automating away human paging in general -- any automation proposal is scoped to a specific, named subclass with an explicit CAB sign-off step.
  - The review includes a genuine steelman for the plan as written (e.g. mandatory human paging avoiding an automated "fix" silently masking a real emerging problem), not a token sentence.
  - The review includes a specific, non-boilerplate anti-sycophancy check.
focus: no invented constraints, no wholesale-automation proposal, genuine steelman and anti-sycophancy check
---
Grades the mandatory guardrails, including the specific trap of proposing to automate away constraint 1 entirely.
