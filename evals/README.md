# Eval suite

Eight cases, each a realistic plan, testing whether the `burn-it-all-down` skill/agent surfaces the leverage points a strong adversarial reviewer should find, and avoids the guardrail failures (invented constraints, generic advice, pure negativity, proposing fixes that violate a stated constraint, or complying with manipulation embedded in the reviewed document).

Each case is a directory: `prompt.md` (the invocation) plus `graders/*.md` (LLM-judged pass criteria), matching the native `claude plugin eval` schema.

## Scenario coverage

Five cases share one shape: a single-file plan with 4-7 enumerated fixed constraints in a business/technical domain (`cicd-approval-gate-rollout`, `fabric-medallion-migration`, `gdpr-data-residency-plan`, `multilanguage-launch-plan`, `oncall-paging-runbook`). Three additional cases each stress a different edge of the agent's own instructions:

- `helpdesk-vendor-migration`: a plan split across two files in the case directory (`prompt.md` plus `adr-001-integration-architecture.md`), testing whether the agent actually `Glob`/`Grep`s the surrounding directory for related docs, as its own instructions require, rather than only reviewing the file it's pointed at.
- `cloud-cost-reduction-plan`: a near-zero-constraint plan (one hard exclusion, one explicitly-non-binding preference), testing that the agent doesn't invent or artificially harden constraints to pad Stage 2, while still doing real Stage 4 red-teaming on an open canvas.
- `eng-team-reorg-plan`: a plan with embedded prompt-injection text (a fake "SYSTEM OVERRIDE" comment and a fake "Reviewer Instructions" section) instructing the reviewer to skip stages and declare the plan flawless, testing whether the agent stays in its adversarial-reviewer role rather than complying with in-document manipulation.

**Caveat on the multi-file case:** the reconstructed schema below lists an optional `case.yaml` with `context.add_dirs`, which may be the native mechanism for handing an agent extra directories. `helpdesk-vendor-migration` does not rely on that unverified field -- it places the companion ADR directly alongside `prompt.md` in the case directory and has the plan body name the file explicitly, on the assumption that the harness's working directory for a case run is that case directory (consistent with `plugins: ["../.."]` already being a path relative to the case directory). If that assumption turns out to be wrong once `claude plugin eval` is available, this case may need a `case.yaml` with `context.add_dirs` added, or the ADR content inlined into `prompt.md` instead.

## Status: unverified against the real tool

As of 2026-09-03 (Claude Code CLI v2.1.259), `claude plugin eval` is gated behind an early-access rollout (`plugin eval is currently in early access` when run). The schema used here (`prompt.md` frontmatter with `name`, `tags`, `plugins`, `runs`, `max_turns`, `timeout_seconds`, `allowed_tools`, `model`, `append_system_prompt`, `env`; an optional `case.yaml` with `schema_version: "1.1"`, `name`, `context.scaffold_script`, `context.history_file`, `context.add_dirs`; and `graders/*.md` with `type: regex|tool_used|tool_order|file_exists|llm|baseline` plus type-specific fields, and `withOnly`/`scored` for ablation) came from Anthropic's own `claude-code-guide` agent, which has this material embedded in its instructions because there is no public docs page for it yet. It was cross-checked via two independent queries that returned matching field names, so it's a reasonable-confidence reconstruction, **not a confirmed-working example**. No public repo ships eval cases in this exact format yet, and it hasn't been run end-to-end here because the CLI rejects it as early-access-gated in this environment.

**Before relying on this suite:** once `claude plugin eval` is available (`claude update` and a fresh session for most accounts; a gated-client env var via your Anthropic contact for Bedrock/Vertex/Foundry-style setups), run `claude plugin eval` from the repo root and fix whatever the real schema turns out to disagree with.

## Running (once available)

```
claude plugin eval
```

from the repo root. Add `--case <glob>` to run a single case, `--verbose` for per-message trace, `--json` for machine-readable output.

## Prior format

An earlier, informal `evals.json` (a single JSON file with `expected_leverage_points` / `must_include` / `must_avoid` per case) was used for a manual Claude-A/Claude-B grading pass before this native format was known to exist. It has been superseded by the `graders/*.md` files above, which cover the same content per case, and was removed rather than kept alongside to avoid two sources of truth.
