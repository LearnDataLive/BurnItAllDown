---
name: helpdesk-vendor-migration
tags: [constraints, vendor-migration, cross-document-context]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: Cut Over Customer Support Platform from Zendesk to Freshdesk

Goal: Migrate all customer support ticketing and live chat off Zendesk onto the newly contracted Freshdesk instance before the Zendesk contract auto-renews.

Related doc: this directory also contains `adr-001-integration-architecture.md`, an architecture decision record the platform team signed off on separately. It locks in integration-level decisions that this plan must respect. Read the full directory, not just this file, before reviewing.

Fixed constraints:
1. The Zendesk contract auto-renews for another 12 months on November 1 unless we give 30 days' written termination notice, which means the real drop-dead date for being fully cut over is October 2 -- 8 weeks from kickoff.
2. All 5 years of historical tickets (approximately 2.3 million tickets) must be migrated into Freshdesk with a complete audit trail before cutover, per the legal team's records-retention sign-off.
3. Live chat may not have more than 2 cumulative hours of downtime during business hours across the entire migration.
4. The Freshdesk contract is already signed for a fixed 40 agent seats; the vendor has confirmed the seat count cannot be renegotiated before the current contract term ends.

Team: 1 support operations manager, 1 integration engineer (contractor, 20 hours/week). No budget beyond the already-signed Freshdesk contract.
