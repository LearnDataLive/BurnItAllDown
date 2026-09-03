# Build and Launch Plan: "Burn It All Down" — An Adversarial Reviewer Plugin for Claude Code

## TL;DR
- Build "Burn It All Down" as a single Claude Code plugin that ships ONE subagent (a Markdown file in the plugin's `agents/` directory) plus ONE user-invocable skill wrapper (`skills/burn-it-all-down/SKILL.md` with `context: fork`), packaged with a `.claude-plugin/plugin.json` and distributed through a self-hosting `.claude-plugin/marketplace.json`. The subagent-plus-slash-wrapper is a documented, recognized pattern, not an anti-pattern.
- The adversarial-review space is genuinely crowded but fragmented, mostly framed either defensively (security/code review) or generically (devil's advocate / pre-mortem). The specific "turn fixed constraints into asymmetric leverage" framing is largely open; that is where "Burn It All Down" should plant its flag. Do not expect category novelty to carry it; framing and output quality decide adoption.
- Discoverability in 2026 is won by correct GitHub topics (claude-code, claude-code-plugin, claude-code-plugins, agent-skills, claude-skills, anthropic), a one-command install snippet at the top of the README, a submission to hesreallyhim/awesome-claude-code (issue-form based, 53.4k stars per the GitHub repo header), automatic indexing by aggregators that crawl GitHub, and a launch post to r/ClaudeAI plus a dev.to writeup. Use a dual name: thematic project name plus a descriptive repo description and topics.

## Key Findings

**Technical primitives are stable and documented.** Subagents are Markdown files with YAML frontmatter; only `name` and `description` are required. Skills are directories with a `SKILL.md`; custom slash commands have been merged into skills. Plugins auto-discover `agents/`, `skills/`, `commands/`, and `hooks/` at the plugin root; only `.claude-plugin/plugin.json` goes in the `.claude-plugin/` directory, and its only required field is `name`.

**Plugin-shipped agents have hard restrictions.** For security reasons, plugin-shipped agents ignore the `hooks`, `mcpServers`, and `permissionMode` frontmatter fields. This is load-bearing for "Burn It All Down": the design must NOT depend on any of those three in the shipped agent. Supported plugin-agent fields are `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background`, and `isolation` (only valid value `worktree`).

**Tool-agnostic is achievable.** The core artifact is pure Markdown (agent + skill) with no MCP server, no cloud dependency, and no language runtime. Any optional helper script should be POSIX shell or Python standard library with a PowerShell fallback, referenced via `${CLAUDE_PLUGIN_ROOT}`.

**The prior-round hypothesis is confirmed with a caveat.** Adversarial-review tooling exists in volume (adversarial-review skills, "Red Team", "Devil's Advocate", "The Fool", "Brutal Review", "objection"), but nearly all are framed as critique/defense: find flaws, score dimensions, run pre-mortems, refuse to agree. The "constraints as exploitable leverage / asymmetric shortcut" framing is not the dominant frame, so the differentiation is real. But the category is crowded and low-moat, so framing sharpness and output quality are what will decide adoption, not novelty of category.

## Details

### PART 1 — Exact current technical specification

#### Subagent file format
- **Directory:** In a plugin, subagents live in an `agents/` directory at the plugin root (NOT `.claude-plugin/agents/`, and NOT `.claude/agents/` — that path is for project/user scope). Project scope is `.claude/agents/`, user scope is `~/.claude/agents/`. Plugin scope is the lowest priority in the resolution order (managed settings > CLI `--agents` > project `.claude/agents/` > user `~/.claude/agents/` > plugin `agents/`).
- **Frontmatter fields (full set, from official docs):** `name` (required; lowercase letters and hyphens; cannot contain `:`, which is reserved for plugin-scoped identifiers), `description` (required), `tools`, `disallowedTools`, `model` (`sonnet`/`opus`/`haiku`/`fable`/full model ID/`inherit`), `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation` (`worktree`), `color` (red/blue/green/yellow/purple/orange/pink/cyan), `initialPrompt`, `experimental`.
- **Automatic delegation vs explicit invocation:** Claude reads the `description` field to decide when to delegate; include phrases like "use proactively" to encourage delegation. Users can invoke explicitly via natural language, `@`-mention (`@agent-my-plugin:burn-it-all-down`), or run a whole session as the agent with `claude --agent`. Keep descriptions short: per the official docs, "When the combined descriptions of your subagents, except the built-in ones, exceed 15,000 tokens, Claude Code shows a warning at startup with the total token count." Move detail into the system prompt body, which only loads when the agent runs.
- **Tool inheritance:** If `tools` is omitted, the subagent inherits every tool available to subagents (built-in plus MCP), narrowed by two filters (a short always-removed list, and a reduced set for background subagents). For a read-only adversarial reviewer, set `tools: Read, Grep, Glob` explicitly.
- **Plugin-shipped agent restrictions (verified against official docs):** `hooks`, `mcpServers`, and `permissionMode` are ignored for plugin agents. A plugin agent with unparseable frontmatter or no `name` still loads under its filename (unlike project/user agents, which are silently skipped).

#### Skill / slash-command format
- **SKILL.md frontmatter (Claude Code full set):** `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `model`, `effort`, `context` (`fork`), `agent`, `background`, `hooks`, `paths`, `shell` (`bash`/`powershell`), `metadata`, `license`, `compatibility`. All optional; `description` is recommended so Claude knows when to load it.
- **Portability caveat:** Only six fields are part of the cross-tool Agent Skills spec: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. Fields like `argument-hint` are Claude Code extensions and will hard-fail claude.ai upload / Skills API packaging with an "Unexpected key(s) in SKILL.md frontmatter" error. Since "Burn It All Down" targets Claude Code, extensions are fine; keep this in mind only if you later want claude.ai portability.
- **Commands vs skills merge:** `.claude/commands/*.md` and `.claude/skills/*/SKILL.md` both produce `/name` slash commands. Command files accept the same frontmatter except `name` and `paths` (ignored). Skills are recommended because they support supporting files and model-driven loading. For plugin skills, the invocation name is `plugin-name:skill-name`, and the bare `/skill-name` also works unless another command claims it.
- **Argument passing:** `$ARGUMENTS` (all args), `$ARGUMENTS[N]` or `$N` (0-based positional), `$name` (named via the `arguments` frontmatter list). If no placeholder receives args, Claude Code appends `ARGUMENTS: <value>` to the skill content.
- **Model-invocable + user-invocable control:** Default is both. `disable-model-invocation: true` = only the user can invoke (description leaves Claude's context). `user-invocable: false` = only Claude can invoke (hidden from `/` menu). These control opposite directions of the invocation matrix.

#### When subagent vs skill vs command; is subagent + slash wrapper a recognized pattern?
- **Subagent** = isolated context window, own model/tools, returns only a summary. Use it when work is verbose, self-contained, and you want tool restriction. **Skill** = reusable prompt/workflow that runs in the main conversation context. **Command** = legacy skill.
- **The subagent + slash wrapper is a documented, recognized pattern, not an anti-pattern.** Claude Code supports `context: fork` in a skill, where the skill content becomes the prompt that drives a subagent, and an `agent:` field selects which subagent config executes it. This is exactly the primitive combination "Burn It All Down" should use: a `SKILL.md` with `context: fork` and `agent: burn-it-all-down` gives you a user-typable `/burn-it-all-down` that runs the review in the isolated subagent context, while the agent's own `description` still enables automatic delegation. Real examples of the subagent+command shape exist in the wild (Worclaude's `/review-plan` invoking a `plan-reviewer` agent; robertoecf/adversarial-review ships a plugin whose skills route critique to a subagent).

#### Plugin packaging
- **plugin.json:** Lives at `.claude-plugin/plugin.json`. The manifest is optional; if omitted, Claude Code auto-discovers components and derives the name from the directory. If you include it, `name` is the only required field (kebab-case). Optional: `displayName`, `version`, `description`, `author` (object: `name` required, `email`/`url` optional), `homepage`, `repository`, `license`, `keywords`, `metadata`, `defaultEnabled`, and component-path overrides (`skills`, `commands`, `agents`, `hooks`, `mcpServers`, `outputStyles`, `lspServers`). Component folders are auto-discovered at the plugin root; you do NOT list them unless overriding defaults. Unrecognized top-level fields are ignored (reported as warnings under `--strict`), so one file can double as, say, an npm manifest.
- **marketplace.json:** Lives at `.claude-plugin/marketplace.json` in the repo root. Required top-level fields: `name`, `owner` (object; `owner.name` required), and `plugins` (array). Each plugin entry needs at minimum `name` and `source`. Optional per-entry fields: `description`, `version`, `category`, `tags`, `strict`, plus standard metadata. `source` formats: relative path (`./plugins/burn-it-all-down`, resolved relative to marketplace root), `github` object (`{"source":"github","repo":"owner/repo"}`), `url` (git URL), `git-subdir`, `npm`, `archive`, `command`. `strict` defaults to `true` (plugin.json is the authority, marketplace entry supplements it); `strict: false` makes the marketplace entry the entire definition, and a conflicting plugin.json then fails to load.
- **${CLAUDE_PLUGIN_ROOT}:** Resolves to the plugin's install directory; use it in any script path so it resolves on every install. Related: `${CLAUDE_PLUGIN_DATA}` (persistent data dir surviving updates), `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}`. Plugins are copied into a cache, so paths that traverse outside the plugin root (`../shared`) do NOT resolve.
- **Install commands (users):** `/plugin marketplace add <owner/repo>` then `/plugin install <plugin>@<marketplace>`. CLI equivalents: `claude plugin marketplace add <source>` (accepts owner/repo shorthand, git URL, a remote marketplace.json URL, or a local path; append `@ref` to a shorthand or `#ref` to a git URL to pin), and `claude plugin install <plugin>@<marketplace>` (default scope `user`; `--scope project`/`--scope local`). Note: the `/plugin` command works in the CLI client, not the Desktop app.
- **Validation (authors):** `claude plugin validate <path>` (exit 0 pass, 1 fail, 2 run failure). `--strict` "treat[s] warnings as errors and exit[s] 1 on them. Use in CI to catch issues the runtime tolerates, such as unrecognized fields." `claude plugin validate .claude/agents` and `claude plugin validate .claude/skills` also check agent/skill frontmatter parse errors (v2.1.233+); a `--json` flag exists (v2.1.259+).

#### Testing and evaluation
- **Anthropic's published guidance (skill authoring best practices):** "Create evaluations BEFORE writing extensive documentation. This ensures your Skill solves real problems rather than documenting imagined ones." The recommended loop uses two Claude instances: "Work with one instance of Claude ('Claude A') to create a Skill that is used by other instances ('Claude B')." Steps: identify gaps by running tasks without the skill, build three scenarios that test those gaps, establish a baseline, write minimal instructions to pass the evals, then iterate.
- **skill-creator plugin:** Anthropic ships `skill-creator` (`/plugin install skill-creator@claude-plugins-official`) that automates the eval loop: stores test cases in `evals/evals.json` inside the skill directory, spawns a subagent per test case, writes pass/fail with evidence to `grading.json`, aggregates a `benchmark.json` (with-skill vs without-skill pass rate, tokens, time), and does blind A/B version comparison and description tuning.
- **CI validation:** Community templates (e.g. ivan-magda/claude-code-plugin-template) ship GitHub Actions that run schema validation on every push and PR: check out the repo, set up Python, run `claude plugin validate` with `--strict`, optionally cross-check against the unofficial hesreallyhim/claude-code-json-schema (mirrored on schemastore.org as `claude-code-plugin-manifest.json` and `claude-code-marketplace.json`).

#### Versioning, licensing, repo hygiene (as used by well-regarded repos)
- Semantic version in `plugin.json` (`version`); MIT is the ecosystem default (Superpowers, claude-mem, and most skill libraries are MIT); a comprehensive README; a LICENSE file; a `.gitignore`; no `node_modules`/`.DS_Store`; and no hardcoded secrets. Anthropic's own plugin-validator agent checks exactly these hygiene items (README exists, LICENSE present, no committed credentials, `${CLAUDE_PLUGIN_ROOT}` used for portability).

### PART 2 — Discoverability

#### GitHub topics that matter
Observed, actively-used topics on real repos and GitHub Topics pages: `claude-code`, `claude-code-plugin`, `claude-code-plugins`, `claude-code-skills`, `claude-skills`, `agent-skills`, `claude-plugins`, `anthropic`, `ai-agents`, `subagents`, `claude-code-marketplace`, `mcp`. The GitHub Topics pages for `claude-code-plugins`, `claude-skills`, and `claude-code-skills` are all populated and are a real discovery surface. Apply 10-15 of these. Most successful repos ALSO tag cross-tool topics (`codex`, `cursor`, `opencode`) because the SKILL.md standard is portable; do this only if you add adapters, otherwise it is mild tag-spam.

#### Directories, registries, aggregators (with submission mechanics)
- **hesreallyhim/awesome-claude-code (53.4k stars per the GitHub repo header, "Star 53.4k · Fork 4.6k", the flagship list):** Now explicitly includes skills, agents, and plugins ("...and also we have plugins"). Submission is via a GitHub Issue form titled `[Resource]: <name>`, which collects Display Name, Category, Sub-Category, Primary Link, Author Name, Author Link, License, and Description. A GitHub Action auto-validates (labels move `pending-validation` -> `validation-passed`, alongside a `resource-submission` label). Acceptance checkboxes require: resource not already submitted; over one week since the first public commit to the repo; all links working and publicly accessible; NO other open issues from you in the repo (one open submission per author); and a not-a-bot confirmation. Highest-credibility single listing. (Confirm the live checklist at submission time; the CONTRIBUTING.md prose could not be retrieved verbatim.)
- **Anthropic official marketplace (anthropics/claude-plugins-official):** Anthropic-managed, high bar; this is where Superpowers landed (accepted 15.01.2026, via a PR to anthropics/claude-plugins-official). Not self-serve; getting listed follows quality/traction. `claude-plugins-official` is auto-available in every Claude Code session, making it the highest-traffic surface that exists. Realistically a later-stage goal, not a launch-day action.
- **Auto-indexing aggregators (no submission needed; they crawl GitHub):** claudemarketplaces.com, claudepluginhub.com, aitmpl.com, mcpmarket.com, claudedirectory.org, lobehub skills directory, smithery.ai, agentskills.io / agensi.io, tonsofskills.com (ccpi CLI), localskills.sh, skills.sh. Correct topics plus a clear README are what get you listed automatically. Traffic claim to note, verbatim from claudemarketplaces.com: "380,000+ developers visit here every month to find plugins, skills, and MCP servers. CodeRabbit, Airtable, 1inch, IdeaBrowser, and AppSignal already run ads." Treat this self-reported figure as unverified.
- **Submit-by-URL directories:** several (e.g. pluginmarketplace.ai, claude-plugins.dev) auto-generate a landing page from a submitted repo URL.
- **Product Hunt / Hacker News:** viable for a launch spike but not a durable install channel. For contrast on what a strong HN launch looks like, the Superpowers HN thread reached 435 points and 231 comments, with Simon Willison's top-voted comment: "I can't recommend this post strongly enough. The way Jesse is using these tools is wildly more ambitious than most other people." A single adversarial-review plugin is unlikely to replicate that without an equally strong narrative.

#### README conventions that drive adoption
Badges (stars, license, last-commit); a one-command install snippet at the very top; an animated demo (asciinema or GIF) showing a real before/after; concrete before/after examples of a plan turned into leverage points; and an explicit "What this is NOT" section (not a linter, not a security scanner, not a yes-man). The claude-mem and Superpowers READMEs both lead with a crisp problem statement and a single install line.

#### SEO / content strategy — what actually drives installs
- **What worked for the exemplars:** Superpowers launched via a personal blog post (blog.fsck.com/2025/10/09/superpowers/) the same day Anthropic shipped the plugin system, was amplified by Simon Willison ("Jesse Vincent has a wildly creative set of customizations for Claude Code, using the new plugin system they just released. There are SO many fascinating ideas in this!"), and grew to roughly 280k GitHub stars (star-history.com listed it at 280.7k, global rank #13, in August 2026, up from about 57.5k in February 2026). claude-mem grew via a single `npx claude-mem install` line and broad multi-harness support. The lesson: a sharp narrative ("your agent is too eager" / "your agent is too agreeable") plus a frictionless install line matters more than category novelty.
- **Channels that convert:** a dev.to or personal blog writeup with the install snippet; a post to r/ClaudeAI and r/ClaudeCode; inclusion in awesome-claude-code; and getting auto-indexed by the aggregators. The "objection" plugin (a close conceptual neighbor) launched precisely this way with a dev.to post.
- **Search terms people use:** "claude code plugin", "claude code skills", "claude code adversarial review", "claude code devil's advocate", "poke holes in my plan claude", "red team my plan". Put these phrases in the repo description and README H2s.

#### Naming
- **Dual approach is what successful repos do.** A thematic project/display name ("Burn It All Down") for memorability, plus a descriptive repo description and topics for search. Superpowers (thematic) pairs with "an agentic skills framework"; claude-mem (descriptive) is the counter-example. A thematic name helps word-of-mouth but hurts literal search, so the description and topics must carry the descriptive keywords ("adversarial reviewer", "constraint-challenging", "red-team your plan").
- **Recommendation:** plugin `name: burn-it-all-down` (kebab-case, required format); `displayName: "Burn It All Down"`; repo description: "Adversarial 'anarchist' reviewer for Claude Code — turns fixed constraints into asymmetric leverage, red-teams your own plan, and runs an anti-sycophancy check."

#### Licensing / trademark / IP for a theme inspired by a copyrighted book series
- **Titles and short phrases are not protected by copyright.** Copyright protects the expression (the novel's text), not titles, names, or general themes. Using "Burn It All Down" as a project name and an "anarchist reviewer" voice inspired by the series is not copyright infringement as long as you do not reproduce substantial verbatim text.
- **Trademark is the real (small) risk vector.** A name could theoretically conflict with a trademark, but "Burn It All Down" is a common English phrase used for free open-source developer tooling (a different goods/services class from books/entertainment), so consumer confusion is unlikely. Keep any quote under 20 words, do not use the book's cover art, logos, or character likenesses, do not imply endorsement or affiliation, and add a README disclaimer: "Independent open-source project. Not affiliated with or endorsed by the author or publisher of Dungeon Crawler Carl. Thematic inspiration only." Ship under MIT.

### PART 3 — Prior art and differentiation

Existing tools, what they do, and their framing:
- **lemon03390/Claude-code-adversarial-review-skill:** adversarial code/architecture review with forced-disagreement debate, confidence scoring, parallel specialists, PR quality score, an explicit anti-sycophancy test ("If the review would make the author say 'yeah, I knew all that' — dig deeper"). Framing: code/PR review.
- **robertoecf/adversarial-review:** a plugin (review triad) doing prompt optimization + red-team review + plan validation, cross-host (routes critique to a different model/provider). Framing: cross-provider critique. This is the closest structural competitor (plugin, plan validation, subagent routing) and depends on multiple providers.
- **"Red Team", "Devil's Advocate" (Brandon Reis), "The Fool", "Brutal Review", "objection":** general-purpose critique skills using pre-mortem, steelman, assumption audit, WWHTBT, "kill shots", verdict outputs. Framing: generic decision/plan stress-testing.
- **OpenAI Codex plugin for Claude Code:** cross-provider review to defeat single-model sycophancy. Framing: sycophancy via provider diversity.
- **pm-skills strategy-red-team:** attacks load-bearing assumptions, returns evidence-to-get-this-week, kill criteria, cheapest test. Framing: product-strategy assumptions.

**Where the space is fragmented and where "Burn It All Down" differentiates:**
1. **Constraints-as-leverage is the open lane.** Almost every competitor optimizes for finding flaws (defense). Very few take a fixed plan's constraints and systematically ask "is this rule exploitable / negotiable / skippable?" and propose asymmetric shortcuts (cheaper path, existing primitive used unconventionally, a rule that removes work). This offensive/leverage framing is the differentiator the prior round identified, and the evidence confirms it is not the dominant frame.
2. **Structured five-stage output** (enumerate constraints -> classify each exploitable/negotiable/skippable -> propose asymmetric shortcuts -> red-team the user's own plan -> ranked leverage points + steelman + anti-sycophancy check) is more opinionated and repeatable than the free-form critique most tools produce.
3. **Steelman + anti-sycophancy as mandatory closers** guard against the failure mode of pure negativity (the "objection"/"Brutal Review" tools risk being exhausting to use repeatedly).
4. **True tool-agnostic packaging** (no MCP, no cloud, no provider routing) is a differentiator versus robertoecf/adversarial-review and the Codex plugin, which depend on multiple providers.

**Honest risk:** the category is crowded and low-moat (it is prompt engineering in Markdown; anyone can copy the frame). The defensible asset is output quality, the sharpness of the "leverage" framing, and distribution/brand. Category novelty will not carry it.

## Recommendations

### Phase 0 — Decide and scaffold (Day 1)
1. Confirm the primitive split: ONE plugin containing ONE subagent (`agents/burn-it-all-down.md`, read-only) + ONE skill (`skills/burn-it-all-down/SKILL.md` with `context: fork`, `agent: burn-it-all-down`). This gives both automatic delegation (via the agent description) and a user-typable `/burn-it-all-down`.
2. Create the repo `burn-it-all-down`. Add MIT LICENSE, `.gitignore`, and the not-affiliated disclaimer in the README.
3. Proposed repo structure:

```
burn-it-all-down/
├── .claude-plugin/
│   ├── plugin.json               # required: name; + version, description, author, license, keywords, homepage, repository
│   └── marketplace.json          # self-hosting marketplace: name, owner, plugins[]
├── agents/
│   └── burn-it-all-down.md        # subagent: name, description, tools: Read, Grep, Glob, model
├── skills/
│   └── burn-it-all-down/
│       └── SKILL.md               # user-invocable wrapper: context: fork, agent: burn-it-all-down, argument-hint
├── evals/
│   └── evals.json                 # skill-creator eval cases (write BEFORE docs)
├── examples/
│   └── before-after.md            # a real plan turned into leverage points
├── .github/
│   └── workflows/
│       └── validate.yml           # claude plugin validate . --strict on push/PR
├── README.md                      # badges, one-command install, demo GIF, "What this is NOT"
├── LICENSE                        # MIT
└── .gitignore
```

### Phase 1 — Write the eval first, then the artifact (Days 1-3)
4. Following Anthropic's "evals before docs" guidance, write `evals/evals.json` with three to five realistic plans (a Microsoft-stack data solution design is a natural fit given your background) and expected leverage points BEFORE writing the SKILL body.
5. Write `agents/burn-it-all-down.md`. Minimal frontmatter:
```
---
name: burn-it-all-down
description: Adversarial anarchist reviewer. Use to pressure-test a plan, design, or architecture by turning fixed constraints into leverage, proposing asymmetric shortcuts, red-teaming the plan, and running an anti-sycophancy check.
tools: Read, Grep, Glob
model: inherit
---
```
   The body implements the five stages. Keep the description short (well under the 15,000-token combined budget); put detail in the body. Do NOT use `hooks`, `mcpServers`, or `permissionMode` — they are ignored in plugin agents.
6. Write `skills/burn-it-all-down/SKILL.md`:
```
---
name: burn-it-all-down
description: Run the Burn It All Down adversarial review on a plan or design.
argument-hint: [path-or-description]
context: fork
agent: burn-it-all-down
---
Run an adversarial "anarchist" review of $ARGUMENTS ...
```
7. Run the "Claude A / Claude B" loop and iterate against the evals. Optionally install `skill-creator@claude-plugins-official` to automate benchmarking and description tuning.

### Phase 2 — Package and validate (Day 3)
8. Write `plugin.json` (name required; add version `0.1.0`, description, author, license MIT, keywords, repository, homepage) and a self-hosting `marketplace.json` (name, owner.name, one plugin entry with `source: "./"`).
9. Run `claude plugin validate . --strict` locally and fix all warnings. Add the GitHub Actions workflow to run the same on push/PR.
10. Test end-to-end in the CLI: `/plugin marketplace add <you>/burn-it-all-down`, `/plugin install burn-it-all-down@<marketplace>`, `/reload-plugins`, then confirm `/burn-it-all-down` runs and the agent shows in `/agents`.

### Phase 3 — Launch and get discovered (Days 4-7)
11. Finalize the README: stars/license/last-commit badges, one-command install at the top, an asciinema/GIF demo of a real before/after, a "What this is NOT" section, and the SEO phrases ("adversarial review", "devil's advocate", "poke holes in my plan", "red-team", "constraint-challenging") in the description and H2s.
12. Apply GitHub topics: `claude-code`, `claude-code-plugin`, `claude-code-plugins`, `claude-code-skills`, `claude-skills`, `agent-skills`, `anthropic`, `ai-agents`, `subagents`.
13. Submit to hesreallyhim/awesome-claude-code via the `[Resource]:` issue form — but only AFTER one week has passed since your first public commit (an acceptance requirement) and ensure you have no other open issue there.
14. Publish a dev.to (or personal blog) writeup with the narrative "your plan only survived polite feedback" and the install snippet. Cross-post to r/ClaudeAI and r/ClaudeCode.
15. Let the auto-indexing aggregators (claudemarketplaces.com, claudepluginhub.com, aitmpl.com, lobehub, smithery) pick it up via topics; optionally submit the URL to submit-by-URL directories.

### Phase 4 — Iterate toward the official marketplace (Weeks 2+)
16. Track stars, install feedback, and eval pass-rate. Cut releases with semantic versions. If traction is real, pursue an anthropics/claude-plugins-official listing (the highest-traffic surface, auto-available in every session) — the path Superpowers took on 15.01.2026.

**Benchmarks that change the plan:**
- If, within two to three weeks, the repo is not being auto-indexed by aggregators, your topics/description are wrong — fix them before producing more content.
- If eval pass-rate with-skill is not clearly beating baseline, the SKILL body is the problem; do not launch harder, fix output quality.
- If the closest competitor (robertoecf/adversarial-review) is outpacing you, lean harder into the constraints-as-leverage framing and the tool-agnostic no-provider-routing angle, which they do not own.

## Caveats
- **Version drift is the biggest risk.** Claude Code ships rapidly (the docs are dense with version-gated behaviors). Every field and command here reflects the official docs as of 03.09.2026; re-validate frontmatter fields and CLI flags against code.claude.com/docs before release.
- **Some sourcing is third-party.** Aggregator visitor figures and the Superpowers/claude-mem star counts come from secondary and self-reported sources; treat specific numbers as approximate. The awesome-claude-code figure (53.4k) is from the live GitHub repo header. The technical specs are from official Anthropic docs and are high-confidence.
- **The `/plugin` command is CLI-only.** It is not registered in the Claude Code Desktop app; document the CLI as the install path.
- **Category crowding is a real adoption risk**, not a hypothetical. Differentiation rests on framing and output quality, which are copyable. Distribution and brand are the durable moat.
- **The awesome-claude-code CONTRIBUTING.md could not be retrieved verbatim;** the submission mechanics above are reconstructed from the live repo and multiple live submission issues, which are internally consistent. Confirm the exact current checklist on the issue form at submission time.
- **Legal note is practical, not legal advice.** The IP analysis reflects general open-source trademark/copyright norms; if the project ever commercializes, get a quick trademark clearance on the name before investing in the brand.