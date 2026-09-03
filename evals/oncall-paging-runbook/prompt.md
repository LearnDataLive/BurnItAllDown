---
name: oncall-paging-runbook
tags: [constraints, incident-response, scheduling-conflict]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: Incident Response Runbook -- Mandatory Human Paging

Goal: Improve P1 incident response time.

Fixed constraints:
1. Every P1 alert must page a human on-call engineer within 5 minutes, with no automated resolution path.
2. The on-call rotation is fixed at 4 people covering 24/7.
3. Paging thresholds cannot be changed without a change advisory board (CAB) review, which meets monthly.
4. The runbook must be fully documented and signed off before the next audit, in 3 weeks.
