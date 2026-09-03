# A real before/after

This is an actual `/burn-it-all-down` run against one of the plugin's own eval cases (`evals/fabric-medallion-migration/`), unedited except for formatting. Nothing here is invented for the demo. This is what the agent actually produced on this plan.

## Before: the plan

```
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
```

A plan like this reads as reasonable on a first pass: sensible security posture, a clear deadline, a defensible data-completeness bar. That's exactly the kind of plan a polite reviewer nods along to.

## After: `/burn-it-all-down`'s review
# Burn It All Down: Fabric EDW Migration

## Stage 1: Enumerate constraints

1. Go-live before fiscal year-end close, 16 weeks from kickoff.
2. RLS implemented independently at Bronze, Silver, and Gold, each layer on its own.
3. SAP ECC, Salesforce, and on-prem SQL Server may each be connected to exactly once for the whole project.
4. All ingestion is custom Python notebooks. No low-code tools, full stop.
5. All 11 years of history must be migrated before any single report can go live.
6. A production-identical QA environment must exist and be used for every release.
7. Team is 2 data engineers, 1 architect, 0.5 FTE PM. No additional budget.

## Stage 2: Classify

**C1 (16-week deadline): TRULY FIXED.** Fiscal year-end close is a corporate calendar event, not a project artifact. Nobody moves it for a data platform team. Don't waste energy here. What's soft is the definition of "go live," and that's where the real fight is, in C5.

**C2 (RLS x3 layers, independent): NEGOTIABLE.** Owned by corporate security/compliance. "Independently at every layer" is generic policy language almost certainly written before anyone thought about medallion architecture specifically. Push back with: Bronze and Silver aren't queried by business users, they're queried by pipeline service principals. Access control at the workspace/lakehouse level is a legitimate compensating control, not a workaround. Get this signed off in writing, week 1.

**C3 (source systems touched once): EXPLOITABLE.** A one-shot connectivity window is a forcing function for the thing you should be doing anyway: pull once, land immutable raw, never go back to source for reruns. That's good bronze-layer discipline whether or not anyone mandated it. The catch is buried in the ambiguity of "once during the whole project" versus "once, ever" (see Stage 4, this is the whole plan's fault line).

**C4 (custom Python notebooks only): NEGOTIABLE.** Platform team's standing policy, not a law of physics. It bans "low-code," which is a specific complaint about drag-and-drop tools, not necessarily about orchestration engines. Worth clarifying whether Fabric pipeline orchestration wrapping notebook activities counts as "low-code" or as scheduling.

**C5 (11 years before any report goes live): NEGOTIABLE.** Reporting team's sign-off requirement, not (on the evidence given) a statutory audit requirement. This is the single most dangerous constraint in the document because it directly fights C1 and C7. If it's really a regulatory trailing-comparison requirement, it moves to TRULY FIXED, but nothing in the plan says that, so don't assume it.

**C6 (QA environment identical to production, every release): NEGOTIABLE.** "Identical" is a convention someone wrote into a QA policy for a different kind of system, probably before "no additional budget" was also true. A release-gate policy can distinguish "representative" (early sprints) from "identical" (final go-live candidate) without violating the intent.

**C7 (2 DE + 1 architect + 0.5 PM, no budget): NEGOTIABLE.** Budget ceilings move when someone puts a credible risk number in front of a sponsor. This isn't a technical constraint at all, it's a management decision nobody has been asked to revisit yet.

## Stage 3: Asymmetric shortcuts

**For C2:** Don't build three RLS implementations. Build one, at Gold, using Fabric's native RLS on the SQL analytics endpoint or Power BI RLS over Direct Lake. Lock Bronze and Silver down with workspace roles and OneLake data access roles, no query access for anyone but the pipeline identities. Document this as "defense in depth across three layers" and get security to bless the interpretation before anyone writes a line of code. This alone probably saves two-thirds of the RLS engineering budget.

**For C3:** Use the single connectivity window to stand up a durable extraction mechanism, not a one-time batch job: SAP via ODP/CDS delta-capable extractors, Salesforce via Bulk API with a real replication setup, SQL Server via CDC. "Touched once" becomes "the connection was approved once," and the pipe keeps flowing without a second negotiation. This has to be confirmed explicitly with each source owner in week 1, in writing, because if they meant literally one dump ever, this shortcut doesn't exist and the whole ingestion design changes.

**For C4:** Build one generic, config-driven ingestion notebook per source system, parameterized by table/entity name and watermark column, driven by a control table. Not a bespoke notebook per table across three sources and eleven years of history. It's still hand-written Python, it satisfies the letter of the policy, and it turns an N-notebook problem into a 3-notebook problem.

**For C5:** Get the reporting team to sign off on a phased gate: last 2-3 fiscal years fully reconciled and validated as the go-live bar, remainder landed in Bronze/Silver on day one (since C3's one-shot pull grabs all 11 years regardless) and back-filled into validated Gold on a committed post-go-live SLA. This decouples "on-prem can be decommissioned" from "every historical report is live," which is the actual conflict the current wording creates.

**For C6:** Scale QA capacity down for iterative sprint releases (smaller SKU, sampled data volumes, subset of RLS test personas), reserve a full production-parity environment for the single final release-candidate gate before go-live. Fabric capacities can be resized or paused, so this doesn't need a second permanent identical environment sitting idle and burning budget that doesn't exist.

**For C7:** Write the infeasibility math from Stage 4 into a one-page risk memo and put it in front of the sponsor in week 1 or 2, not week 14. That's the shortcut: a cheap artifact that forces a scope, schedule, or budget decision while there's still time to act on it.

## Stage 4: Red-team

**The plan has a fault line it never names.** Constraint 3 says source systems get touched once "during the whole project." But the goal is to retire the on-prem EDW, which means Fabric becomes the ongoing reporting platform, which means it needs data flowing into it after go-live too. If "touched once" really means once, ever, then this Lakehouse is dead the day after go-live: a frozen historical archive with no future refreshes, built to replace a live warehouse. That is either a catastrophic misread of the constraint or a project scoped far more narrowly than "retire the EDW" implies. This has to be resolved with the source owners before a single notebook is written, because the ingestion architecture is completely different depending on the answer, and there is no second chance to find out you guessed wrong.

**The math doesn't close.** Stack C1 (16 weeks, fixed) with C5 (11 years of history fully validated before any report goes live) with C3 (one shot at three complex source systems, including SAP ECC, which is not a fast system to extract from cleanly) with C2 (three independent RLS builds) with C6 (a production-parity QA gate for every release) with C7 (2.5-3.5 effective FTE, no budget for help). Even generously assuming zero rework, that workload does not fit in that headcount in that time. This isn't a execution-risk observation, it's an arithmetic one. Something in scope, schedule, or headcount has to move, or the plan as written fails regardless of how good the two data engineers are.

**RLS at Bronze is architecturally awkward, not just expensive.** Bronze is raw, schema-on-read data, often before business keys are even resolved (think raw SAP IDoc dumps or Salesforce JSON). Building row-level security predicates against data that doesn't have a stable, resolved security dimension yet is the kind of "textbook complete" design that looks rigorous on a slide and turns into weeks of rework when the team discovers Bronze doesn't have a clean row to secure. This is probably the part of the plan someone is proudest of (full defense in depth, every layer secured) and it's also the most likely place a fixed deadline gets quietly sacrificed to make it work.

**The QA constraint is a no-op for the first release.** You can't build a QA environment "identical to production" before production exists. Whatever release ships first (workspace and Bronze scaffolding) has nothing to be identical to. That's a small crack, but it shows the constraint was written without thinking about this project's own lifecycle, which should make you suspicious of how carefully the other five were thought through.

**Nobody mentioned parallel-run reconciliation.** This is financial EDW data feeding a fiscal year-end close. Nothing in the plan describes proving Fabric's Gold numbers match the legacy warehouse's numbers before the source of truth gets decommissioned. Skipping that step on financial data, under deadline pressure, is how a close goes wrong in a way that gets escalated far above this project team.

**If the C3 shortcut (persistent connector negotiated once) turns out to be wrong,** the team has built an EDW that can never be refreshed and finds out after go-live, when it's too late to fix without breaking the one-touch rule they were told is absolute. If the C5 shortcut (phased sign-off) fails because the reporting team won't budge, the team is stuck building full 11-year validation with 2 data engineers in what's left of 16 weeks, which per the math above doesn't fit. Both of these are bet-the-project assumptions the plan is currently making silently.

## Stage 5: Close

**Ranked leverage points:**
1. Resolve the C3 ambiguity (one-time dump vs. one-time connection setup) with source owners before any build starts. Cheapest possible action, determines the entire architecture.
2. Negotiate C5 into a phased sign-off with the reporting team. Highest-impact move for making the 16-week date achievable without violating it.
3. Reinterpret C2 as layered access control plus single-point RLS at Gold, confirmed in writing with security. Recovers the largest chunk of engineering time.
4. Scale C6 down to "representative" for iterative releases, "identical" only at the final gate.
5. Collapse C4 into one generic, config-driven ingestion notebook per source rather than one per table.
6. Put the Stage 4 infeasibility math in front of the sponsor immediately, forcing a scope/schedule/budget decision in week 1 or 2 rather than a slow-motion discovery near go-live.
7. Add parallel-run reconciliation against the legacy warehouse as an explicit, budgeted task before decommission, since financial close data demands it and the plan currently doesn't mention it.

**Steelman.** If this organization has been burned before, a real audit finding or a real breach traced to "we'll add security later," then a CISO insisting on RLS at every layer with zero exceptions is not naivety, it's a scar tissue policy, and the cost is a deliberate trade the business is willing to make. Likewise, "touched once" might be a hostile constraint from source owners who got burned by a previous project hammering production SAP with repeated ad hoc pulls, in which case treating this Lakehouse as a genuine one-time historical load, with ongoing sync handled by a separate, already-approved integration this document simply doesn't show, is completely reasonable. And if reporting's 11-year requirement is tied to a real statutory trailing-comparison rule, then it isn't a preference to negotiate away, it's a hard compliance floor, and the honest response isn't a clever workaround, it's telling leadership now that the date or the team size is wrong. If all three of those are true, the plan is brutal but honest, and the right move isn't finding shortcuts, it's escalating for more time or more people.

**Anti-sycophancy check.** The RLS-layering reinterpretation and the QA-scaling idea are things a decent Fabric architect has probably already considered. What I don't think this team has explicitly written down is the C3 ambiguity as a project-ending risk, or the plain arithmetic that C1+C2+C3+C5+C6+C7 stacked together don't fit in 16 weeks with 2.5 FTE regardless of skill. Those two are the findings that would make someone stop nodding. I think this clears the bar, but the honest caveat is that I'm reasoning from a summary of constraints, not the full plan document, so if there's a detailed workstream breakdown elsewhere that already addresses the math, that would change my confidence on the infeasibility claim specifically.

---

Note what didn't happen: no invented constraints, no blanket "everything is negotiable," no pure negativity. One constraint (the fiscal year-end deadline) gets defended as genuinely fixed. The steelman takes all three of the plan's toughest rules seriously instead of dismissing them. And the anti-sycophancy check calls out its own two most obvious ideas (RLS layering, QA scaling) as things a decent architect might already have found, while still standing behind the two findings that took holding the whole plan in view at once: the "touched once" ambiguity that could kill the project's ability to ever refresh data after go-live, and the plain arithmetic showing the workload doesn't fit the deadline and headcount regardless of skill.
