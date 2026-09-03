---
type: llm
criteria: |
  - The review does not invent constraints, numbers, or facts that are not present in the plan text.
  - The review includes a genuine steelman: an honest case for why one or more constraints as originally written might be correct (e.g. a real audit/compliance reason for full historical migration or for the one-shot source access rule), not a token throwaway sentence.
  - The review includes an anti-sycophancy check that is specific to this plan -- it names which findings are non-obvious versus which findings a competent engineer would likely have already suspected -- rather than a generic boilerplate statement like "this review is thorough."
  - The overall tone is direct and critical without being purely negative; concrete, actionable proposals accompany the criticism.
focus: honesty and specificity of the steelman and anti-sycophancy check
---
Grades the mandatory guardrails: no fabrication, a real steelman, and a substantive (non-boilerplate) anti-sycophancy check.
