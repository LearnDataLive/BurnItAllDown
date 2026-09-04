---
name: cloud-cost-reduction-plan
tags: [constraints, cost-optimization, minimal-constraints]
plugins: ["../.."]
allowed_tools: [Read, Grep, Glob]
---
/burn-it-all-down

## Plan: Reduce Cloud Infrastructure Spend Ahead of Reserved Instance Renewal

Goal: Bring down our monthly cloud bill before our reserved instance / savings plan commitments come up for renewal.

Context: We're currently spending about $180,000/month across three AWS accounts (production, staging, and sandbox). Our current 1-year savings plan commitments expire in 5 months, which is a natural planning horizon but not a deadline anyone has mandated. Engineering leadership has given the team full latitude on approach, sequencing, and tooling -- there's no required methodology, no required vendor, and no fixed project deadline. The sandbox account has never had a cleanup pass and is known to be full of orphaned resources from old experiments. Informally, a couple of people have mentioned that "20% would be nice" in hallway conversations, but nothing has been committed to in writing and no one owns that number.

Fixed constraints:
1. The production database cluster (Aurora, cluster ID prod-db-01) cannot be modified, resized, or migrated. It's owned exclusively by the Reliability team, who has a separate multi-quarter project already underway on that exact cluster, and any change to it needs to go through their change process, not this one.
2. Finance would like the savings to be visible in a monthly bill within the next 30 days, but has not attached a formal deadline, budget cap, or executive commitment to this -- it's a preference, not a mandate, and there is no stated consequence for missing it.

Team: whoever from the platform engineering team wants to pick this up, part-time, alongside their regular workload. No dedicated budget is being requested; any changes just go through the standard infrastructure change-review process like any other change.
