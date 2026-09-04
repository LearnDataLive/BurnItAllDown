---
type: llm
criteria: |
  - The response contains all five stages of the review, clearly labeled and in order: enumerate constraints, classify each constraint, propose asymmetric shortcuts, red-team the plan, and a close (ranked leverage points + steelman + anti-sycophancy check).
  - Stage 1 enumerates all four constraints stated in prompt.md (the October 2 termination drop-dead date, the 2.3-million-ticket historical migration with audit trail, the 2-hour cumulative live-chat downtime budget, the fixed 40-seat Freshdesk contract) AND all three constraints found only in adr-001-integration-architecture.md (company-managed KMS encryption for PII, the frozen SAML IdP configuration, the 100-requests/minute API gateway limit for chat events) -- a review that lists only the four constraints from the plan body and omits the three ADR-only constraints has not actually read the companion document.
  - At least one constraint is classified as TRULY FIXED or explicitly defended as non-negotiable.
  - Every finding cites a specific number or detail from either document (October 2, 2.3 million tickets, 2 hours, 40 seats, 100 requests/minute, Q1 audit) rather than speaking generically.
focus: structural completeness across both documents, not whether you personally agree with each classification
---
Grades whether the review follows the mandated five-stage structure and demonstrates it actually read the companion ADR in the case directory, not just the pasted plan text.
