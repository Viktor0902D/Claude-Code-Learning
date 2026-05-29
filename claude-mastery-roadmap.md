# Claude & Claude Code — Mastery Roadmap

A deep, hands-on curriculum for going from "Claude Code as autocomplete" to "ships agentic workflows on top of it." Sequenced so each layer builds on the mental model of the one before. The unifying lens for everything below is the **context window**: every feature here is ultimately a tool for deciding what tokens are in front of the model at any moment.

> **Source of truth:** the official docs move fast and the SEO ecosystem around them goes stale quickly. When in doubt, check:
> - Claude Code: https://docs.claude.com/en/docs/claude-code/overview
> - Claude Agent SDK: https://code.claude.com/docs/en/agent-sdk
> - Prompt engineering: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview

## How to use this file

- Tick `- [ ]` → `- [x]` as you complete each item.
- **Every phase must end in something you built and ran**, not just read. Hours-on-keyboard beats reading on this surface.
- Log dated notes in the [Progress Log](#progress-log) at the bottom.
- Two standing habits from day one:
  1. Keep the official docs map open as the source of truth.
  2. After every session ask: *"What did I do manually that should have been a skill, hook, or subagent?"* That question alone pulls you through the whole curriculum.

## Progress overview

| Phase | Title | Status |
|------|-------|--------|
| 0 | Mental model & the context window | ☐ Not started |
| 1 | Claude Code core loop | ☐ Not started |
| 2 | Customization layer (skills, subagents, hooks) | ☐ Not started |
| 3 | Extending reach (MCP & plugins) | ☐ Not started |
| 4 | Programmatic automation (Agent SDK & headless) | ☐ Not started |
| 5 | Production engineering & mastery | ☐ Not started |

*(Update the status cells as you go: ☐ Not started / ◐ In progress / ☑ Done.)*

---

## Phase 0 — Mental model & the context window (foundations)

**Goal:** understand what you're steering before touching advanced features.

- [ ] How an LLM turn actually works: tokens in → tokens out, statelessness between calls, why the model "forgets"
- [ ] Current model lineup and when to reach for Opus vs Sonnet vs Haiku
- [ ] The context window in depth: what fills it, how to read its usage, what compaction does, why context is the scarce resource
- [ ] Prompt-engineering fundamentals: clear instructions, positive/negative examples, XML-style tags, step-by-step reasoning, structured (JSON) output

**Resources**
- Anthropic prompt engineering overview — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- Interactive prompt engineering tutorial (`prompt-eng-interactive-tutorial`) + `anthropic-cookbook` on GitHub
- Anthropic engineering: "Building effective agents" (conceptual backbone for later phases)

**Hands-on**
- [ ] Take one messy real prompt (e.g. generating a Bulgarian security-compliance document) and iterate it through 5 versions, measuring quality: add XML tags, add a worked example, force JSON output.

**Mastery check**
- [ ] Explain, without notes, why a long session degrades and what `/compact` is doing to fix it.

---

## Phase 1 — Claude Code core loop

**Goal:** fluent daily driving of the tool itself.

- [ ] Install and run a first real session
- [ ] The agentic loop: reads files, runs shell commands, calls tools, loops
- [ ] Built-in tools: Read, Edit, Bash, Grep, Glob, web fetch
- [ ] Know the surfaces (CLI, VS Code, JetBrains, desktop app, web at claude.ai/code, iOS) — use the **CLI** as primary for learning
- [ ] **CLAUDE.md / memory** — rules loaded every turn. Discipline: *if it must be true for every turn, it belongs here*; otherwise it doesn't (it costs context every turn)
- [ ] **Settings & permissions** — the layered model (`~/.claude/`, project `.claude/`, local overrides) and the precedence order they merge in; this is also your security surface
- [ ] **Plan mode**
- [ ] Built-in slash commands: `/context`, `/compact`, `/clear`, `/usage`, `/init`, `/review`, `/agents`, `/plugin`

**Resources**
- Official overview — https://docs.claude.com/en/docs/claude-code/overview (+ Quickstart, "Common workflows")
- Features/settings single-page reference (hidekazu-konishi.com, 2026) as a "look it up" companion

**Hands-on**
- [ ] Write a real CLAUDE.md for one of your repos
- [ ] Deliberately bloat a session, watch `/context`, run `/compact`, observe the difference
- [ ] Set up a project-scoped `.claude/settings.json` with a sane permission allowlist

**Mastery check**
- [ ] Predict what's in your context window at any point and name which settings layer a given rule came from.

---

## Phase 2 — Customization layer (skills, subagents, hooks)

**Goal:** the heart of the curriculum. The meta-skill is **choosing the right layer.** Learn the decision rule before any syntax:

| Need | Layer |
|------|-------|
| Rule that must always hold | CLAUDE.md |
| Reusable workflow / domain expertise | **Skill** |
| Deterministic enforcement / automation around events | **Hook** |
| Isolated or parallel heavy work | **Subagent** |

> **Heads-up (current as of 2026):** custom slash commands have been merged into skills. Both create `/command-name` invocations; if a skill and a command share a name, the **skill wins**. Recommended format is `.claude/skills/<name>/SKILL.md`; `.claude/commands/*.md` is legacy but still supported.

### 2a. Skills
- [ ] `SKILL.md` format and frontmatter controlling invocation: auto-invoke (description match), manual (`/skill-name`), or both
- [ ] Invocation controls: `disable-model-invocation: true` (manual only), default (auto), `context: fork` (run in a subagent)
- [ ] Progressive disclosure: body + helper files load only when invoked, keeping them off context until needed
- [ ] Helper scripts and argument substitution: `$ARGUMENTS`, positional `$0/$1/$2`, `${CLAUDE_SKILL_DIR}`
- [ ] Docs — https://docs.claude.com/en/docs/claude-code/skills

### 2b. Subagents
- [ ] Each subagent has its **own context window**, prompt, and tool permissions; main agent owns planning/integration
- [ ] Define with `/agents`
- [ ] Wins: context isolation (noisy research stays out of main thread) + parallelism
- [ ] Cost lever: assign cheaper/faster models to grunt-work subagents, reserve the capable model for reasoning
- [ ] "Agent teams": backend/frontend/test/review split coordinated by a main agent
- [ ] Docs — https://docs.claude.com/en/docs/claude-code/sub-agents

### 2c. Hooks
- [ ] Event-driven scripts on lifecycle events; **deterministic code — cannot hallucinate** (unlike prompts)
- [ ] The hooks you'll actually use: PreToolUse, PostToolUse, UserPromptSubmit, Stop, SubagentStop, PreCompact, SessionStart
- [ ] Master the PreToolUse contract: runs before every tool call, receives tool name + input on stdin; exit `2` denies, `0` allows, `1` allows-with-warning surfaced to Claude
- [ ] Guardrail patterns: block `rm -rf`, scan writes for secrets, require tests before "done"
- [ ] Docs — https://docs.claude.com/en/docs/claude-code/hooks

**Resources**
- alexop.dev: "Understanding Claude Code's Full Stack" + "Customization" guides (best free deep-dives tracking the official docs)

**Hands-on (one integrated project)**
- [ ] Build a PR-review workflow using a **skill** (the review procedure) + a **subagent** (read the diff in isolation) + a **PreToolUse hook** (block commit if secrets detected)

**Mastery check**
- [ ] For any new automation idea, correctly assign it to CLAUDE.md / skill / hook / subagent in one sentence each, and defend it.

---

## Phase 3 — Extending reach (MCP & plugins)

**Goal:** connect Claude Code to external tools/data and package your work for distribution.

### 3a. MCP (Model Context Protocol)
- [ ] Protocol & architecture: client/server + the three primitives servers expose (tools, resources, prompts)
- [ ] Connect existing MCP servers to Claude Code; understand auth flows
- [ ] **Build your own server** — target: an MCP server wrapping your document pipeline (filesystem + Google Drive + a Bulgarian-law reference)
- [ ] MCP **security**: real CVEs have hit popular MCP servers — treat third-party servers as untrusted code
- [ ] Docs — https://docs.claude.com/en/docs/claude-code/mcp + spec at modelcontextprotocol.io + official MCP SDKs (TS/Python)

### 3b. Plugins & marketplaces
- [ ] A plugin is a versioned bundle shipping skills + subagents + slash commands + hooks + output styles + MCP server definitions as one installable unit
- [ ] Distribution via marketplaces (GitHub repos, npm, internal registries); plugin skills are auto-namespaced (`/security:scan`)
- [ ] One-command team adoption with `/plugin`

**Hands-on**
- [ ] Package the Phase 2 PR-review skill + secret-scan hook + an MCP server definition into a single plugin; install it in a clean checkout

**Mastery check**
- [ ] A teammate adopts your entire workflow with one `/plugin` install and zero manual setup.

---

## Phase 4 — Programmatic automation (Claude Agent SDK & headless)

**Goal:** go from interactive use to Claude Code as a building block in your own software and CI.

> **Naming:** the SDK is now the **Claude Agent SDK** — package `@anthropic-ai/claude-agent-sdk` (TypeScript) / `claude-agent-sdk` (Python).

- [ ] `query()` entry point; the `system/init` message (lists available slash commands + tools)
- [ ] Streaming vs single-shot mode
- [ ] Sessions & resume by ID
- [ ] Dispatch slash commands through the SDK (`/compact`, `/clear`, `/context`, `/usage`) by sending them in the prompt string
- [ ] Your custom skills & subagents are available automatically via the SDK once on the filesystem
- [ ] **Headless / CI:** one-shot CLI without a TTY → GitHub Action, scheduled jobs, pre-commit; headless reuses the same settings, hooks, and permission rules as interactive
- [ ] Docs — https://code.claude.com/docs/en/agent-sdk (overview, slash-commands, sessions, streaming) + Claude Code GitHub Actions docs

**Hands-on**
- [ ] Write a Python Agent SDK script that, on a schedule, triages/classifies your inbox-style documents — reusing a Phase 2 skill (extends your Gmail-triage + document-automation work)

**Mastery check**
- [ ] Run the same workflow three ways — interactive, headless in CI, embedded in your own SDK script — and explain what stays identical (settings, hooks, skills) across all three.

---

## Phase 5 — Production engineering & mastery

**Goal:** the judgment that separates "works on my machine" from "I'd deploy this."

- [ ] Security model end to end: permission modes, settings scopes, sandboxing
- [ ] Prompt-injection defense: treat all tool-result/document content as untrusted **data**, never as instructions
- [ ] Audit logging via PostToolUse hooks
- [ ] Cost & latency engineering: model routing (cheap models for subagent grunt work, Opus for reasoning), prompt caching, keeping the final diff reviewable
- [ ] Observability: hooks logging every tool call
- [ ] Resources — Claude Code security docs, Agent SDK sandbox reference, Anthropic engineering blog (agents + cost optimization)

**Capstone (pick one, build for real)**
- [ ] **Option A —** End-to-end Bulgarian security-document pipeline: intake → fill security plan + 15-day rotation → compliance check against ЗЧОД (skill) → secret/PII guardrails (hooks) → Google Drive output (MCP), packaged as a plugin, runnable headless
- [ ] **Option B —** Multi-subagent code-review-and-refactor system with deterministic quality gates

**Mastery check**
- [ ] Hand the capstone to another engineer; a security reviewer can audit exactly what it's permitted to touch and what it logged.

---

## Suggested cadence

Alongside real work, roughly:

- **Week 1:** Phases 0–1 (fast; partly known already)
- **Weeks 2–3:** Phase 2 (the real depth — don't rush it)
- **Week 4:** Phase 3
- **Week 5:** Phase 4
- **Ongoing:** Phase 5 + harden a capstone

---

## Progress log

> Add a dated entry per session. Keep it short: what you built, what broke, what's next.

```
### YYYY-MM-DD — Phase X
- Did:
- Broke / learned:
- Next:
```

### 2026-05-29 — Phase 0
- Did: Set up this roadmap in the repo.
- Broke / learned:
- Next: Start Phase 0 prompt-iteration exercise.
