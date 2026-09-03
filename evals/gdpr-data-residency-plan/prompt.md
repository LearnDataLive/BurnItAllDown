---
name: gdpr-data-residency-plan
tags: [constraints, compliance, budget-pressure]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: EU Data Residency Compliance for Customer Data Platform

Goal: Ensure all EU customer data satisfies data residency requirements ahead of an upcoming audit.

Fixed constraints:
1. All customer data must be replicated to a secondary EU region within 24 hours of ingestion.
2. Replication applies uniformly to every table in the customer data lake, regardless of content.
3. The audit is in 8 weeks; no scope can be added after week 6.
4. Existing ingestion pipelines cannot be modified -- only new replication jobs may be added on top.
5. Budget covers 2 contractor-weeks total.
