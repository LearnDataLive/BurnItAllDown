# Eval suite

Five cases, each a realistic plan with fixed constraints, testing whether the `burn-it-all-down` skill/agent surfaces the leverage points a strong adversarial reviewer should find — and avoids the guardrail failures (invented constraints, generic advice, pure negativity, proposing fixes that violate a stated constraint).

Each case is a directory: `prompt.md` (the invocation) plus `graders/*.md` (LLM-judged pass criteria), matching the native `claude plugin eval` schema.

## Status: unverified against the real tool

As of 2026-09-03 (Claude Code CLI v2.1.259), `claude plugin eval` is gated behind an early-access rollout (`plugin eval is currently in early access` when run). The schema used here — `prompt.md` frontmatter (`name`, `tags`, `plugins`, `runs`, `max_turns`, `timeout_seconds`, `allowed_tools`, `model`, `append_system_prompt`, `env`), optional `case.yaml` (`schema_version: "1.1"`, `name`, `context.scaffold_script`, `context.history_file`, `context.add_dirs`), and `graders/*.md` (`type: regex|tool_used|tool_order|file_exists|llm|baseline` plus type-specific fields, `withOnly`/`scored` for ablation) — came from Anthropic's own `claude-code-guide` agent, which has this material embedded in its instructions because there is no public docs page for it yet. It was cross-checked via two independent queries that returned matching field names, so it's a reasonable-confidence reconstruction, **not a confirmed-working example** — no public repo ships eval cases in this exact format yet, and it has not been run end-to-end here because the CLI rejects it as early-access-gated in this environment.

**Before relying on this suite:** once `claude plugin eval` is available (`claude update` and a fresh session for most accounts; a gated-client env var via your Anthropic contact for Bedrock/Vertex/Foundry-style setups), run `claude plugin eval` from the repo root and fix whatever the real schema turns out to disagree with.

## Running (once available)

```
claude plugin eval
```

from the repo root. Add `--case <glob>` to run a single case, `--verbose` for per-message trace, `--json` for machine-readable output.

## Prior format

An earlier, informal `evals.json` (a single JSON file with `expected_leverage_points` / `must_include` / `must_avoid` per case) was used for a manual Claude-A/Claude-B grading pass before this native format was known to exist. It has been superseded by the `graders/*.md` files above, which cover the same content per case, and was removed rather than kept alongside to avoid two sources of truth.
