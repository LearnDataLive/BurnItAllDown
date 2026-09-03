# Burn It All Down

An adversarial "anarchist" reviewer plugin for [Claude Code](https://claude.com/claude-code). Turns a plan's fixed constraints into asymmetric leverage, red-teams your own plan, and closes with a steelman + anti-sycophancy check.

> **Status:** early scaffold — not yet functional. Structure and packaging are being built out in phases; see `PLAN.md`-equivalent build notes for the roadmap.

## What this is

A single Claude Code plugin shipping:
- **One subagent** (`agents/burn-it-all-down.md`) — read-only, isolated-context adversarial reviewer.
- **One skill** (`skills/burn-it-all-down/SKILL.md`) — a user-invocable `/burn-it-all-down` wrapper (`context: fork`) that runs the subagent.

No MCP server, no cloud dependency, no provider routing — pure Markdown, tool-agnostic within Claude Code.

## What this is NOT

- Not a linter or static analyzer.
- Not a security scanner.
- Not a yes-man — it will not validate a plan just to be agreeable.

## Disclaimer

Independent open-source project. Not affiliated with or endorsed by the author or publisher of *Dungeon Crawler Carl*. Thematic inspiration only.

## License

MIT — see [LICENSE](LICENSE).
