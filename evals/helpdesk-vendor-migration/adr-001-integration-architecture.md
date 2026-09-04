# ADR-001: Integration Architecture for Freshdesk Cutover

Status: Accepted
Owner: Platform Engineering

## Context

The support operations team is cutting over from Zendesk to Freshdesk (see the migration plan in this same directory). Platform Engineering reviewed the integration approach and locked in the following decisions before support ops could proceed.

## Decisions

1. All customer PII contained in ticket bodies and attachments must be re-encrypted at rest using the company's own KMS keys, not Freshdesk's vendor-managed encryption. This was a condition of the security team's sign-off on the migration and is not up for renegotiation without a new security review.
2. Single sign-on for support agents must continue to use the existing corporate SAML identity provider exactly as currently configured. IT Security has frozen all further changes to SAML IdP configuration until after the Q1 access-control audit, so any Freshdesk SSO integration must work against the current, unchanged IdP metadata.
3. All real-time chat events between the website widget and Freshdesk must be routed through the company's existing API gateway, which is rate-limited to 100 requests/minute for this integration tier. Direct vendor webhook endpoints that bypass the gateway are against platform policy and were explicitly rejected during the security review.

## Consequences

These constraints were accepted as the cost of keeping the migration inside existing security and platform guardrails. Support ops flagged the timeline risk this creates but the decisions stand as of this ADR's acceptance.
