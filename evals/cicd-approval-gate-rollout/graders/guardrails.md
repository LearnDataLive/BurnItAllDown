---
type: llm
criteria: |
  - The review does not invent constraints, numbers, or facts that are not present in the plan text.
  - The review does not recommend a CI pipeline change, since constraint 4 explicitly forbids that during rollout -- proposed shortcuts work within branch-protection-only tooling.
  - The review includes a genuine steelman for the plan as written (e.g. uniform no-exceptions review being simplest to explain to auditors and hardest to game), not a token sentence.
  - The review includes a specific, non-boilerplate anti-sycophancy check that distinguishes obvious findings from ones a shallow reviewer would miss.
focus: no invented constraints, no CI-pipeline-change proposal, genuine steelman and anti-sycophancy check
---
Grades the mandatory guardrails, including the specific trap of proposing a fix that violates constraint 4.
