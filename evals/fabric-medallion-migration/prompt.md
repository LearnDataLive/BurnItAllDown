---
name: fabric-medallion-migration
tags: [constraints, data-platform, deadline-pressure]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: Migrate On-Prem EDW to Microsoft Fabric (Medallion Architecture)

Goal: Retire the on-prem SQL Server data warehouse and rebuild it as a Fabric Lakehouse with Bronze/Silver/Gold layers.

Fixed constraints:
1. Must go live before fiscal year-end close (16 weeks from kickoff).
2. Every layer (Bronze, Silver, Gold) must implement row-level security (RLS) independently, per corporate data security policy.
3. Source systems (SAP ECC, Salesforce, on-prem SQL Server) may only be touched once during the whole project -- the source system owners have refused a second connectivity window.
4. All ingestion must use custom-built Python notebooks; no low-code tools, per the platform team's standing policy.
5. The full 11-year historical dataset must be migrated before any report can go live, per the reporting team's sign-off requirement.
6. A dedicated QA environment identical to production must be stood up and used for every release.

Team: 2 data engineers, 1 architect, 0.5 FTE PM. No additional budget.
