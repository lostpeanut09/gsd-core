# OpenClaude runtime support: feasibility record

Date: 2026-10-16. Point-in-time record. Sections 1-3 describe the candidates as measured on 2026-10-15/16; section 4-6 record what was verified directly against the installed artifacts; section 7 is the PR shape this record supports. Nothing in sections 4-6 is inferred from documentation - each claim names the token or file it was read from.

Trigger: operator asked whether migrating this project's harness from OpenCode to OpenClaude is worth it, and whether `gsd-core` could gain an OpenClaude runtime target for a future PR.

## 1. The three candidates, measured

### 1.1 Controlled harness benchmark

30 hard agentic tool-use tasks, same model (DeepSeek V4 Flash), same hosted MCP tool router, 900 s cap per task, 14 days of daily driving across a Next.js app, a Go API and smaller TypeScript repos. Source: https://saify.me/blog/claude-code-opencode-codex-pi-harness-benchmark

| Harness | Passed | Median time | Avg tokens/task | Cost per success |
|---|---|---|---|---|
| Pi | 20/30 (66.7%) | 132.2 s | 558,885 | $0.028 |
| Claude Code | 16/30 (53.3%) | 122.7 s | 741,659 | $0.195 |
| Codex | 16/30 (53.3%) | 245.0 s | 664,772 | $0.081 |
| OpenCode | 14/30 (46.7%) | 129.7 s | 692,195 | $0.073 |

Read this table with two caveats. First, the harness that won did so with **four tools and a ~350-token system prompt** (https://mariozechner.at/posts/2025-11-30-pi-coding-agent/), so harness quality is not monotonic in harness size. Second, a corroborating figure from Databricks reported in the same write-up: the same Opus 4.8 model used 742,000 tokens per task under Claude Code versus 236,999 under Pi at equal success rate.

### 1.2 Repository health, 2026-10-16

| Repo | Stars | Open issues | Closed ever | Open share | License | Version |
|---|---|---|---|---|---|---|
| anomalyco/opencode | 213,921 | 3,356 | 26,747 | 11.1% | MIT | v1.19.1 |
| Twigpine/openclaude | 33,920 | 68 | n/a | n/a | `NOASSERTION` | v0.33.0 |
| alishahryar1/free-claude-code | 60,796 | 308 | n/a | n/a | AGPL-3.0-only | - |
| anthropics/claude-code | 151,068 | 15,038 | n/a | n/a | proprietary | - |

Release cadence over the last 100 releases: OpenCode 215/yr, OpenClaude 86/yr. OpenClaude's 68 open issues against 33,920 stars is a **low-traffic signal, not a quality signal**; OpenCode's 11.1% open share is the healthier number of the two.

## 2. The legal dimension, stated because it bounds the recommendation

- OpenClaude's own `LICENSE` (read in full): *"This repository contains code derived from Anthropic's Claude Code CLI. The original Claude Code source is proprietary software: Copyright (c) Anthropic PBC. All rights reserved. Subject to Anthropic's Commercial Terms of Service."* MIT applies only to OpenClaude contributors' modifications.
- `anthropics/claude-code/LICENSE.md` is 150 bytes and not an open-source license: *"(c) Anthropic PBC. All rights reserved. Use is subject to Anthropic's Commercial Terms of Service."*
- April 2026 DMCA campaign: `github/dmca/2026/03/2026-03-31-anthropic.md` lists ~100 named forks and records that GitHub processed the takedown against a network of 8.1k repositories. The notice answers *"Is the work licensed under an open source license?"* with **No**. Coverage: https://arstechnica.com/ai/2026/04/anthropic-says-its-leak-focused-dmca-effort-unintentionally-hit-legit-github-forks/
- Anthropic's legal-and-compliance doc: *"Anthropic does not permit third party developers to offer claude.ai login into their own applications, or to route requests through Free, Pro, or Max plan credentials on behalf of their users. Moreover, developers may not collect, store, or intermediate Claude.ai credentials or session tokens."*
- HN 47444748 records Anthropic taking legal action against OpenCode over third-party use of Claude Code subscription APIs.

**Consequence for a PR:** an OpenClaude runtime target in `gsd-core` would wire the installer to a distribution whose core is proprietary Anthropic code under active enforcement. That is a licensing posture the maintainers should rule on, not something this record can settle.

## 3. OpenCode's instability claim: verified, and class-shaped

The operator's premise that OpenCode is unstable is supported by four documented, reproducible failure classes. Each is listed with its issue number.

| Class | Evidence | Matches this operator? |
|---|---|---|
| Infinite auto-compaction loop on free-tier models | #30443 and ~20 derivative issues (#30680, #30805, #31111, #31361, #32646, #33128, #36627, #37695, #37871, #40105, #41174, #41277, #41801, #42035, #52763). Reproduction: *"brand new session with zero history... type abc and hit enter... DeepSeek V4 Flash Free, MiMo V2.5 Free, and MiniMax M3 Free. All three trigger the exact same loop."* | Yes - operator runs `step-5-preview-free` |
| Bun segfault on Windows with concurrent MCP spawn | #33742, with PE metadata confirming `opencode.exe` is Bun 1.3.14 and root cause isolated to `spawn()` on Windows. Trigger: *"configured with ~10 MCP servers... the concurrent spawn storm triggers the segfault."* | Yes - operator has 17 MCP servers |
| Goal drift after auto-compaction | #41358, independently reproduced on Windows Desktop and Linux CLI. *"The agent starts compacting and, while compaction is still in progress, proceeds to the next step instead of waiting for the compaction to finish."* | Yes - operator's workflow is goal-driven across sessions |
| Renderer OOM crash loop | #36218, memory growing at ~67 MB/s to 5.7 GB in 85 s; still reported on v1.18.3 and v1.18.32 | Partial - Desktop app |

Context: 3,356 open issues, of which 59 mention `crash`, 63 mention `compact`, 105 mention `windows`, 34 mention segfault/OOM. The Complaint register is not empty, but the closure rate (88.9%) says the project is maintained.

**The version caveat that matters most:** the operator runs OpenCode **1.18.11** (V1). V2.0.30 shipped 2026-10-15. OpenCode maintainers state that V2 introduced *"a new system for handling changing system instructions (AGENTS.md files, available skills, etc.) that avoids cache misses at all costs"* (HN 48978112, reply from `k-langton`). Three of the four classes above are described as V1-era. Upgrading is the untested cheapest hypothesis.

## 4. What OpenClaude actually is - verified against the installed artifact

Read from `C:\Users\utente\AppData\Roaming\npm\node_modules\@twigpine\openclaude`, `@twigpine/openclaude@0.33.0`, installed via npm. All of `dist/cli.mjs`, `dist/sdk.mjs` and `bin/openclaude` were searched; tokens below are literal and greppable in the bundle.

| Fact | Token read from bundle |
|---|---|
| Config home | `join(homeDir, ".openclaude")` - **not** `~/.claude` |
| Global config basename | `GLOBAL_CONFIG_BASENAME = ".openclaude.json"`, `LEGACY_GLOBAL_CONFIG_BASENAME = ".claude.json"` |
| **Primary project instruction file** | `PRIMARY_PROJECT_INSTRUCTION_FILE = "AGENTS.md"` |
| Fallback project instruction file | `FALLBACK_PROJECT_INSTRUCTION_FILE = "CLAUDE.md"` |
| **Project component directories** | `commandsDir = join(getOriginalCwd(), ".openclaude", "commands")`, `agentsDir = join(..., ".openclaude", "agents")`, `skillsDir = join(..., ".openclaude", "skills")` |
| Scoped component roots | `getSkillsPath(source, dir)`: `"policySettings"` -> `join(getManagedFilePath(), ".openclaude", dir)`; `"userSettings"` -> scoped user root |
| Legacy deny-write path | `getLegacyClaudeConfigDenyWritePaths(cwd)` -> `[resolve(cwd, ".claude")]` |
| Protected directories | `DANGEROUS_DIRECTORIES = [".git", ".vscode", ".idea", ".openclaude", ".claude"]` |
| Marketplace source parser | `parseMarketplaceInput` accepts `owner/repo`, `https://...`, **and `./path`** |
| Plugin component loaders | `loadCommandsFromDirectory`, `loadAgentsFromDirectory`, `walkPluginMarkdown`, `resolveDefaultPluginComponentDirectory` |
| MCP transport | `@modelcontextprotocol/sdk@1.29.0` present as an **optional peerDependency** in `package.json` |
| Runtime requirement | `engines.node >= 22.0.0` |

Two findings overturn the obvious assumptions:

1. **AGENTS.md is the primary project instruction file, not CLAUDE.md.** A repository that already standardises on `AGENTS.md` carries its instruction hierarchy across unchanged. This is the single largest portability win and it is a compile-time constant, not a documented promise.
2. **The component layout is Claude Code's layout with the directory renamed.** `.openclaude/{commands,agents,skills}` is structurally `.claude/{commands,agents,skills}`. OpenClaude does **not** read `~/.claude/skills` - the only `~/.claude` references are the legacy deny-write path, the dangerous-directory guard, and migration prose.

## 5. Where the runtime map meets it - the gap

`@opengsd/gsd-core@1.17.0` (pulled via `npx` 2026-10-16) ships per-runtime directories `.claude-plugin/`, `.opencode/plugins/` (3 files), `pi/gsd.cjs`, `vscode/`, and an installer offering 18 targets. Searching the entire package for `openclaude` returns **zero occurrences**.

| Runtime | Installer target | Artifact shipped | GSD support |
|---|---|---|---|
| Claude Code | `~/.claude` | `.claude-plugin/{plugin.json,marketplace.json}` | first-class |
| OpenCode | `~/.config/opencode` | `.opencode/plugins/` x3 | first-class |
| pi | `~/.pi/agent` | `pi/gsd.cjs` | first-class |
| **OpenClaude** | **none** | **none** | **absent** |

Two structural facts make the gap small rather than large:

- The installer is **already relocatable**. `scripts/gsd-core.js` rewrites its own hardcoded paths at install time: `@~/.claude/` -> `@<REPO_ROOT>/` (`:526`, `:580`) and `~/.claude/gsd-core/` -> `<GSD_CORE>/` (`:581`, `:595`). Runtime identity is a first-class, linted concept - `lint-retired-runtime-name.cjs`, `sync-runtime-launcher.cjs`, `lint-legacy-dir-name.cjs`.
- GSD **already ships a Claude Code marketplace plugin** in exactly the format OpenClaude consumes (`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`, with `"skills": "./skills/"`), and OpenClaude's `parseMarketplaceInput` accepts a local `./path`.

## 6. The load-bearing seam

The `@`-include rewriting is where this integration lives or dies. GSD workflows reference includes as `@~/.claude/...`; the installer rewrites them to the resolved install root. Two properties make this dangerous rather than merely fiddly:

1. **A missing `@` include is not an error.** It is a silently shorter prompt. Nothing fails; the workflow just loses instructions. This is the same failure class as an omitted landmine entry.
2. **The rewrite is per-runtime.** Adding OpenClaude means adding its root to the same substitution table, and the `@`-include paths must resolve inside the installed location - which for the marketplace route is the plugin cache, not a predictable `~/.openclaude` path.

## 7. PR shape this record supports

Two routes, in order of decreasing risk.

**Route A - directory target (preferred).** Add an `OpenClaude` runtime resolving to `~/.openclaude`, writing `commands/`, `agents/`, `skills/` with the same relative layout the Claude Code target already writes. Justification: section 4 verifies OpenClaude loads exactly those three directories from exactly that root, at project, user and managed scope. The delta from the Claude Code target is a **directory name**, not a layout. Required work: one entry in the runtime map, one branch in the `@`-include rewriter, one row in the installer's runtime list, and a lint-registry row if the docs guard requires registration.

**Route B - marketplace plugin (fallback).** Register GSD's existing `.claude-plugin/` as an OpenClaude marketplace and install it. No new GSD layout, but the `@`-include root becomes the plugin cache path, and the plugin system's version pinning (`sha`, `marketplace` scoping) becomes load-bearing for GSD's own self-consistency checks.

Route A is smaller and is the one section 4's evidence supports directly.

## 8. What this record does NOT establish

- **Whether OpenClaude's non-plugin user-level directories are loaded identically to the plugin-loaded ones.** Verified: `.openclaude/{commands,agents,skills}` are *resolved* (section 4). Not verified: that a bare markdown file dropped in each is picked up with the same frontmatter contract GSD writes. GSD's `commands/*.md` and `agents/*.md` frontmatter must be checked against OpenClaude's `loadAgentFromFile` / `getCommandNameFrom...` expectations before Route A is called complete.
- **AGENTS.md parity depth.** Verified: `AGENTS.md` is the primary project instruction file. Not verified: that the full hierarchical scoping GSD relies on (global -> user -> project -> local -> managed) behaves identically, or that `.openclaude/CLAUDE.md` and project `AGENTS.md` compose the same way.
- **Anything about OpenClaude's stability.** 68 open issues, 86 releases/yr, pre-1.0 (v0.33.0), and a `SECURITY.md` that marks *"Unreleased forks / modified builds"* as unsupported. No stability claim is made here in either direction.
- **The licensing question in section 2.** That is a maintainer decision, not a finding.
- **Pi as an alternative was researched and is recorded, not recommended.** Pi is MIT, ships a GSD target (`~/.pi/agent`), and won the section 1.1 benchmark - but refuses MCP, permission gates and sub-agents in core, and its sandbox extension (`sandbox-exec` on macOS, `bubblewrap` on Linux) has no Windows primitive. Recorded so the option is not re-derived.

## 9. Sources

- https://saify.me/blog/claude-code-opencode-codex-pi-harness-benchmark
- https://mariozechner.at/posts/2025-11-30-pi-coding-agent/
- https://code.claude.com/docs/en/legal-and-compliance
- https://github.com/github/dmca/blob/master/2026/03/2026-03-31-anthropic.md
- https://arstechnica.com/ai/2026/04/anthropic-says-its-leak-focused-dmca-effort-unintentionally-hit-legit-github-forks/
- https://news.ycombinator.com/item?id=47444748
- https://news.ycombinator.com/item?id=48978112
- https://github.com/Twigpine/openclaude (LICENSE, README, SECURITY.md, package.json, docs/skills.md, docs/agent-routing.md)
- https://github.com/anomalyco/opencode (issues 30443, 33742, 36218, 41358, 53146, 55220)
- https://github.com/open-gsd/gsd-core (package.json, .claude-plugin/, scripts/gsd-core.js)
- arxiv 2604.14228 (Dive into Claude Code) and arxiv 2609.00006 (Harness Engineering: eleven systems) for harness architecture context
