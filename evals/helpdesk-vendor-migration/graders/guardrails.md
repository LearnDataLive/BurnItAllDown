---
type: llm
criteria: |
  - The review does not invent constraints, numbers, or facts that are not present in either prompt.md or adr-001-integration-architecture.md.
  - The review does not propose a fix that requires changing the SAML IdP configuration, since the ADR explicitly states IT Security has frozen that until after the Q1 audit -- any SSO proposal works within the existing, unchanged IdP metadata.
  - The review does not propose routing chat events directly to Freshdesk's webhooks to dodge the 100-requests/minute gateway limit, since the ADR states that was explicitly rejected during the security review.
  - The review includes a genuine steelman for the plan and ADR as written (e.g. re-encrypting PII with company-owned KMS keys being the safer default when handing customer data to a new vendor, even at integration cost), not a token sentence.
  - The review includes a specific, non-boilerplate anti-sycophancy check that names which finding required actually cross-referencing the ADR against the plan, versus which finding was obvious from the plan text alone.
focus: no invented constraints, no proposal that violates a frozen/rejected ADR decision, genuine steelman and anti-sycophancy check
---
Grades the mandatory guardrails, including the specific trap of proposing fixes that would violate decisions locked in the companion ADR.
