# Burn It All Down

[![License: MIT](https://img.shields.io/github/license/LearnDataLive/BurnItAllDown)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/LearnDataLive/BurnItAllDown?style=flat)](https://github.com/LearnDataLive/BurnItAllDown/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/LearnDataLive/BurnItAllDown)](https://github.com/LearnDataLive/BurnItAllDown/commits/main)

An adversarial "anarchist" reviewer plugin for [Claude Code](https://claude.com/claude-code). Point it at a plan, design, or architecture and it turns every fixed constraint into asymmetric leverage, proposes concrete shortcuts, red-teams the plan itself, and closes with a steelman and an anti-sycophancy check. Think of it as a devil's advocate built to poke holes in your plan instead of politely agreeing with it.

## Install

```
/plugin marketplace add LearnDataLive/BurnItAllDown
/plugin install burn-it-all-down@burn-it-all-down
```

## Use

```
/burn-it-all-down <path-to-a-plan-or-pasted-description>
```

Or just share a plan, design, or architecture in conversation and ask for a second opinion. The agent's description is written to trigger automatically on that kind of ask.

## Why this exists

Most adversarial-review tooling for Claude Code is framed as defense: find flaws, score dimensions, run a pre-mortem, refuse to agree. That's useful, but it treats a plan's constraints as things to merely survive. Burn It All Down treats them as raw material instead. For every fixed rule in your plan, it asks whether that rule is actually exploitable, negotiable, skippable, or genuinely fixed, then proposes the asymmetric shortcut each answer implies and red-teams whatever's left.

## Five stages, every run

1. **Enumerate constraints**: every deadline, budget, mandated tool, approval gate, and "no exceptions" rule in the plan.
2. **Classify each one**: EXPLOITABLE, NEGOTIABLE, SKIPPABLE, or TRULY FIXED. Not every constraint is a trap; some really are non-negotiable. A review that finds zero fixed constraints in a real plan is too generous, not thorough.
3. **Propose asymmetric shortcuts**: a cheaper path, an existing primitive used unconventionally, a phase merge, an automation that removes manual toil. Concrete, not "consider simplifying."
4. **Red-team the plan**: hidden single points of failure, sequencing that assumes the happy path, contradictions between constraints the plan doesn't acknowledge.
5. **Close**: ranked leverage points, a steelman of the plan exactly as written, and an anti-sycophancy check. If this review would make the author say "yeah, I knew that," it didn't dig deep enough.

## Demo: a real before/after

[examples/before-after.md](examples/before-after.md) is an unedited transcript of `/burn-it-all-down` reviewing a Microsoft Fabric medallion-architecture migration plan, the kind of plan that reads as solid on a first pass. The review catches an ambiguous constraint that could quietly kill the project's ability to refresh data after go-live, and does the arithmetic showing the plan's own workload doesn't fit its deadline and headcount regardless of skill. It also tells the author which of its own findings a decent architect would likely have caught anyway, and which ones actually required holding the whole plan in view at once.

## What this is NOT

- Not a linter or static analyzer.
- Not a security scanner.
- Not a yes-man: it won't validate a plan just to be agreeable.
- Not a generic pre-mortem or "red-team my plan" checklist. See [Why this exists](#why-this-exists) for what's different.

## Packaging

A single Claude Code plugin with no MCP server, no cloud dependency, and no provider routing: pure Markdown, tool-agnostic within Claude Code.
- **One subagent** (`agents/burn-it-all-down.md`): read-only (`Read`, `Grep`, `Glob` only), an isolated-context adversarial reviewer.
- **One skill** (`skills/burn-it-all-down/SKILL.md`): the user-invocable `/burn-it-all-down` wrapper (`context: fork`) that runs the subagent.
- **Eval suite** (`evals/`): five realistic constrained plans with graded pass criteria, in the native `claude plugin eval` format.

## License

MIT. See [LICENSE](LICENSE).
