# A real before/after

This is an actual `/burn-it-all-down` run against one of the plugin's own eval cases (`evals/fabric-medallion-migration/`), unedited except for formatting. Nothing here is invented for the demo — this is what the agent actually produced on this plan.

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

# BURN IT ALL DOWN: On-Prem EDW → Fabric Medallion Migration

## Stage 1 — Enumerate Constraints

1. **Deadline**: Must go live before fiscal year-end close — 16 weeks from kickoff.
2. **Security architecture**: Every layer (Bronze, Silver, Gold) must implement RLS *independently*, per corporate data security policy.
3. **Source access**: SAP ECC, Salesforce, and on-prem SQL Server may only be touched **once** for the entire project — no second connectivity window, full stop, source owners already said no.
4. **Tooling mandate**: All ingestion must be custom-built Python notebooks. No low-code tools. Platform team standing policy.
5. **Historical completeness gate**: The full 11-year historical dataset must be migrated before *any* report can go live. Reporting team sign-off requirement.
6. **QA parity**: A dedicated QA environment identical to production must be stood up and used for every release.
7. **Resourcing**: 2 data engineers, 1 architect, 0.5 FTE PM. No additional budget.

## Stage 2 — Classify

**1. 16-week deadline** — TRULY FIXED. Fiscal year-end close is a finance/board calendar event, not something this team owns or can renegotiate. What's *not* fixed is what "go live" has to mean by that date — that's where the leverage lives (see #5).

**2. RLS independently at every layer** — EXPLOITABLE. The policy says every layer needs its own RLS. It does not say every layer needs its own *bespoke, hand-rolled* RLS implementation. That's an assumption the plan bakes in, not a requirement of the policy text.

**3. Touch source only once** — TRULY FIXED as stated (source owners already refused, this isn't a negotiation, it's a closed door), but EXPLOITABLE in how the single window is used. The plan doesn't say *what* gets pulled in that window — that's a design choice, not a constraint.

**4. Custom Python only, no low-code** — NEGOTIABLE. Owner: platform team. It's described as a "standing policy," which is the tell — standing policies are usually written for a general case (governance over transformation logic, reviewability of business rules) and get applied too literally to connector/orchestration plumbing that isn't the thing the policy actually cares about. Worth a direct conversation, not silent compliance.

**5. Full 11-year history before any report goes live** — SKIPPABLE. Owner: reporting team. The plan's real goal is "retire on-prem EDW and have Gold reporting live for fiscal year-end close." Nothing about that goal requires that *every* historical report be live on day one — it requires the extraction to be complete (because of constraint 3) and the *current* reporting need to be met. Sign-off criteria conflating "extracted" with "exposed in Gold" is a self-imposed trap, not a technical necessity.

**6. QA identical to production, used for every release** — NEGOTIABLE. Owner: whoever wrote the release/QA governance policy (unnamed in the plan, which is itself a flag). "Identical" is doing a lot of unexamined work here — identical schema and security logic is achievable for free; identical data volume and compute capacity is not achievable at all under "no additional budget." Someone can push back on what "identical" means.

**7. Team size / no additional budget** — TRULY FIXED. Given as a hard resourcing envelope with no negotiation language attached. Take it as bedrock and design around it — this is the constraint that makes constraints 4 and 6 dangerous in combination (see Stage 4).

## Stage 3 — Asymmetric Shortcuts

**On #2 (RLS independently per layer):** Don't write RLS three times. Build one metadata-driven security module — a single entitlement/security-dimension table plus one reusable predicate function/library — and invoke it at Bronze, Silver, and Gold. Each layer still "implements RLS independently" in the sense that matters to the policy (each layer enforces its own check, a bug in one doesn't bypass another), but you write the logic once and parameterize it three times instead of building three separate systems. This alone probably saves multiple engineer-weeks that this team does not have.

**On #3 (one-shot source access):** Treat the single connectivity window as a maximal, defensive extraction, not a minimal one. Pull full schema metadata, all CDC/change columns, deleted/audit records, and the entire 11-year history into Bronze as immutable Delta/Parquet in that one window — over-extract everything you can plausibly need, because there is no second chance. This turns a hard limitation into forced good medallion hygiene: Bronze becomes the permanent raw archive, and every downstream rebuild (Silver, Gold, RLS fixes, schema changes) replays off Bronze without ever going back to SAP ECC, Salesforce, or SQL Server again.

**On #4 (no low-code):** Go back to the platform team with a scoped ask: keep custom Python for all transformation/business logic (Silver, Gold — the auditable, reviewable stuff the policy is actually protecting), and use Fabric's native Copy/pipeline orchestration for the raw connector layer into Bronze. Frame it as "the policy governs business logic, not plumbing." If they say no, you've lost nothing; if they say yes, you've freed real engineering hours on a team of two.

**On #5 (11 years before any report):** Split extraction completeness from reporting exposure. Bronze/Silver get the full 11 years in the single-touch window (forced anyway by #3). Gold — and therefore reports — go live at fiscal year-close with the current fiscal year plus 2–3 prior years, enough for close activities and audit trail. The remaining ~8 years of history get built into Gold marts on a background schedule *after* go-live, since the hard part (getting the data out of source) is already done. Take this specific split back to the reporting team as the sign-off ask, instead of accepting "all 11 years live" as a precondition.

**On #6 (QA identical to prod):** Negotiate "identical" down to identical schema, RLS logic, and pipeline code, running on a right-sized Fabric capacity SKU with a representative data subset (current FY + statistically sampled prior years) rather than a full duplicate of production data and capacity. Reserve full-scale volume/performance testing for a short, scheduled dry run against production capacity during an off-hours window, rather than paying to keep a full duplicate environment running permanently. This is the only way "no additional budget" and "QA environment" coexist at all.

## Stage 4 — Red-Team

**Bus factor.** Two data engineers are being asked to build SAP ECC extraction, Salesforce extraction, SQL Server extraction, Bronze/Silver/Gold transformation logic, and (per the plan as written) three separate RLS implementations, in 16 weeks, with a 0.5 FTE PM to coordinate it and an architect whose hands-on coding contribution is unstated. If either engineer is out for two weeks, there is no bench. The plan does not mention this risk anywhere, and nothing in the constraints buys slack for it.

**The single point of failure the plan doesn't name.** Constraint 3 is the whole project's fuse: one extraction window, no second chance, from source systems that famously have extraction-format nightmares (SAP ECC pool/cluster tables, RFC/IDoc quirks; an 11-year-old on-prem SQL Server schema that has almost certainly drifted across a decade of undocumented changes). The plan assumes the extraction goes right the first time. There is no reconciliation/validation checkpoint described *during* the connectivity window to confirm the pull is complete and correct before the window closes. If a bug or missing table surfaces in week 10 during Silver modeling, there is no way back — and the plan has no contingency for that scenario at all. This is the single biggest unacknowledged risk in the document.

**The part the author is probably proudest of is the expensive part.** RLS enforced independently at every layer reads like a serious, mature security design — defense in depth. But under a 16-week clock with 2 engineers, "3 separate implementations of the same control" is exactly the kind of architectural purity that quietly eats the schedule, and it's the piece most likely to get cut for time under pressure, ironically compromising the exact security posture it was meant to protect.

**Named contradiction the plan doesn't own.** Constraints 1, 3, and 5 collide: a 16-week deadline (#1), combined with "touch source exactly once, ever" (#3), combined with "11 years must be fully migrated and reporting-signed-off before any report goes live" (#5), with only 2 engineers and no budget (#7). That is: an irreversible, one-shot, must-be-complete-and-correct extraction of over a decade of data from three heterogeneous enterprise systems, fully validated, fully signed off, all inside 16 weeks, by two people, using only hand-written Python (#4) because low-code is banned. Any one of these constraints alone is manageable. All four stacked together, with no reconciliation gate and no bench, is a plan that is more likely to either miss the deadline or get "signed off" on data nobody actually validated.

**Unmentioned dependencies.** Fabric capacity procurement and licensing lead time; on-prem data gateway setup for SQL Server/SAP ECC connectivity (itself needing security/network approval, potentially its own "window" risk analogous to #3); Entra ID / group membership data that RLS at every layer depends on — where does that entitlement data come from, and is pulling it itself constrained by #3 if it lives in SAP or AD? None of this is addressed, and any one of them can quietly consume weeks of the 16 before a single notebook gets written.

**If a Stage 3 shortcut fails.** If the "over-extract everything defensively in the one window" move (the fix for #3) misses something — a table, a CDC config, a schema edge case — the team is now worse off than if they'd gone in with low confidence, because they will believe they were thorough. The shortcut only works if it includes an explicit reconciliation/row-count/checksum validation step run *before* the connectivity window closes, and the plan as written has no such step at all.

## Stage 5 — Close

**Ranked leverage points (highest impact / lowest risk first):**

1. Add a reconciliation/validation checkpoint inside the single extraction window (Stage 4) — without this, constraint 3 is a loaded gun. Non-negotiable fix, costs almost nothing, prevents catastrophe.
2. Decouple "extraction complete" from "report live" on the 11-year requirement (#5 shortcut) — single biggest schedule relief, directly resolves the #1/#3/#5 contradiction, and only requires a conversation with the reporting team about sign-off phrasing.
3. Centralize RLS as one parameterized module instead of three builds (#2 shortcut) — big time savings, essentially zero policy risk since the control still runs independently at each layer.
4. Right-size the QA environment (#6 shortcut) — not optional, since "identical to prod" and "no additional budget" are mathematically incompatible as written; this has to be resolved one way or another regardless of anything else in this review.
5. Push back on "no low-code" scope to exclude ingestion plumbing (#4) — moderate impact, some organizational friction with the platform team, but frees the scarcest resource (engineer hours) on the highest-risk part of the timeline.
6. Surface the unmentioned dependencies (gateway, capacity procurement, RLS entitlement source) to the architect/PM in week 1, not week 10.

**Steelman.** Two of these constraints hold up fully as written, and the plan deserves credit for not softening them. RLS independently at every layer (#2) is a legitimate defense-in-depth control if the actual policy intent is that a bug in one layer's security can't cascade into blanket exposure elsewhere — that's a real, sane requirement for a system handling SAP and CRM data, not bureaucratic box-checking, and the shortcut proposed above changes *implementation*, not the control itself. And treating "touch source only once" (#3) as genuinely fixed, then over-extracting defensively in that single window, is exactly what a competent team should do even without an adversarial review pointing it out — real ERP/CRM systems often can't tolerate a second full pull, and building Bronze as the permanent raw archive from one careful extraction is sound medallion architecture, not a trick. If the team had already planned it that way, this review would be confirming good instincts, not correcting bad ones.

**Anti-sycophancy check.** This clears the bar on the load-bearing finding: the #1/#3/#5 contradiction — one-shot irreversible extraction, gated by full 11-year completeness and reporting sign-off, inside a fixed 16-week window with 2 engineers — is not something visible from reading any single constraint in isolation. It only shows up when you hold four separate lines of the document together, and the plan as written has zero mention of a reconciliation gate or a phased Gold exposure to relieve it. Same for the #6/#7 collision: "QA environment identical to production" flatly cannot coexist with "no additional budget," and the plan doesn't flag that tension anywhere. Where I'd expect a shrug of recognition is the RLS-parameterization suggestion (#2) — that's a fairly standard "don't repeat yourself" call any competent engineer would eventually reach on their own. But the extraction/history/deadline contradiction and the missing validation checkpoint are structural findings the author would have had to be actively looking for cross-constraint conflict to catch, and nothing in the plan suggests they were.

---

Note what didn't happen: no invented constraints, no blanket "everything is negotiable," no pure negativity. Two constraints get defended as genuinely fixed. The steelman is a real argument, not a hedge. And the anti-sycophancy check calls out its own most obvious finding (#2) as something a competent engineer would've found anyway — while standing behind the two findings (the four-way contradiction, the missing reconciliation gate) that actually required holding the whole plan in view at once.
