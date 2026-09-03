# Burn It All Down

[![License: MIT](https://img.shields.io/github/license/LearnDataLive/BurnItAllDown)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/LearnDataLive/BurnItAllDown?style=flat)](https://github.com/LearnDataLive/BurnItAllDown/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/LearnDataLive/BurnItAllDown)](https://github.com/LearnDataLive/BurnItAllDown/commits/main)

An adversarial "anarchist" reviewer plugin for [Claude Code](https://claude.com/claude-code). Point it at a plan, design, or architecture and it turns every fixed constraint into asymmetric leverage, proposes concrete shortcuts, red-teams the plan itself, and closes with a steelman and an anti-sycophancy check — a devil's advocate built specifically to poke holes in your plan instead of politely agreeing with it.

## Install

```
/plugin marketplace add LearnDataLive/BurnItAllDown
/plugin install burn-it-all-down@burn-it-all-down
```

## Use

```
/burn-it-all-down <path-to-a-plan-or-pasted-description>
```

Or just share a plan/design/architecture in conversation and ask for a second opinion — the agent's description is written to trigger automatically on that kind of ask.

## Why this exists

Most adversarial-review tooling for Claude Code is framed as defense: find flaws, score dimensions, run a pre-mortem, refuse to agree. That's useful, but it treats a plan's constraints as things to merely survive. Burn It All Down treats them as raw material — a constraint-challenging review that asks, for every fixed rule in your plan: is this actually exploitable, negotiable, skippable, or genuinely fixed? Then it proposes the asymmetric shortcut each answer implies, before red-teaming whatever's left.

## How it works — five stages, every run

1. **Enumerate constraints** — every deadline, budget, mandated tool, approval gate, and "no exceptions" rule in the plan.
2. **Classify each one** — EXPLOITABLE, NEGOTIABLE, SKIPPABLE, or TRULY FIXED. Not every constraint is a trap; some really are non-negotiable, and a plan that has zero fixed constraints is being reviewed too generously.
3. **Propose asymmetric shortcuts** — a cheaper path, an existing primitive used unconventionally, a phase merge, an automation that removes manual toil. Concrete, not "consider simplifying."
4. **Red-team the plan** — hidden single points of failure, sequencing that assumes the happy path, contradictions between constraints the plan doesn't acknowledge.
5. **Close** — ranked leverage points, a steelman of the plan exactly as written, and an anti-sycophancy check: if this review would make the author say "yeah, I knew that," it didn't dig deep enough.

## Demo: a real before/after

[examples/before-after.md](examples/before-after.md) is an unedited transcript of `/burn-it-all-down` reviewing a Microsoft Fabric medallion-architecture migration plan — the kind of plan that reads as solid on a first pass. The review finds a four-way constraint collision the plan doesn't acknowledge, a missing validation checkpoint that turns a one-shot data extraction into a loaded gun, and still tells the author which two of its own findings are genuinely non-obvious versus which one they'd have caught on their own.

## What this is NOT

- Not a linter or static analyzer.
- Not a security scanner.
- Not a yes-man — it will not validate a plan just to be agreeable.
- Not a generic pre-mortem or "red-team my plan" checklist — see [Why this exists](#why-this-exists) for what's different.

## Packaging

A single Claude Code plugin, no MCP server, no cloud dependency, no provider routing — pure Markdown, tool-agnostic within Claude Code:
- **One subagent** (`agents/burn-it-all-down.md`) — read-only (`Read`, `Grep`, `Glob` only), isolated-context adversarial reviewer.
- **One skill** (`skills/burn-it-all-down/SKILL.md`) — the user-invocable `/burn-it-all-down` wrapper (`context: fork`) that runs the subagent.
- **Eval suite** (`evals/`) — five realistic constrained plans with graded pass criteria, in the native `claude plugin eval` format.

## Disclaimer

Independent open-source project. Not affiliated with or endorsed by the author or publisher of *Dungeon Crawler Carl*. Thematic inspiration only.

## License

MIT — see [LICENSE](LICENSE).
