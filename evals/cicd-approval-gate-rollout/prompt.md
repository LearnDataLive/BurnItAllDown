---
name: cicd-approval-gate-rollout
tags: [constraints, process-rollout, contradiction-detection]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: Roll Out Mandatory Manual Approval Gate for All Merges to Main

Goal: Reduce production incidents caused by unreviewed changes.

Fixed constraints:
1. Every merge to main requires sign-off from 3 named senior engineers, no exceptions.
2. The gate applies to all repositories, including internal tooling and documentation-only repos.
3. Rollout must be complete across 40 repositories within 4 weeks.
4. No CI pipeline changes are allowed during rollout -- only branch protection rules.
5. The 3 senior engineers are also expected to keep their current project workload unchanged.
