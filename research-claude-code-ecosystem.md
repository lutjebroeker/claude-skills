# Claude Code Ecosystem — Research Summary

Research of 10+ repos from the Claude Code ecosystem, evaluated for practical value and integration potential with this `claude-skills` repo.

---

## Tier 1 — High Value, Worth Integrating

### 1. Superpowers (`obra/superpowers`)
**What:** Composable skills framework with a 7-phase dev workflow. Blocks code generation until a spec/design is approved.

**3 most valuable things:**
- **Brainstorming + design gate** — hard-stops implementation until a design is reviewed and approved. Prevents building the wrong thing.
- **TDD skill** — enforces RED-GREEN-REFACTOR cycle per task.
- **Subagent-driven development** — breaks work into discrete tasks, delegates to parallel agents, integrates results.

**Structure:** Skills live in `skills/` as self-contained `SKILL.md` files (brainstorming, TDD, systematic-debugging, writing-plans, code-review). Agents in `agents/`. Hooks in `hooks/`.

**Use today:** Before any feature, run the brainstorming skill to explore 2-3 approaches with tradeoffs, get approval, then generate a plan before touching code.

---

### 2. Everything Claude Code (`affaan-m/everything-claude-code`)
**What:** Production-ready plugin with 28 agents, 125+ skills, 60+ slash commands, and AgentShield security scanning (1,282 tests).

**3 most valuable things:**
- **28 specialized agents** — language-specific code reviewers (TS, Python, Go, Rust, etc.), build resolvers, planner, architect, security auditor, DB specialist.
- **AgentShield** — detects secrets in prompts (sk-, ghp_, AKIA patterns), blocks `--no-verify`, prevents config misuse.
- **Continuous learning** — auto-extracts patterns from sessions into reusable "instincts" with confidence scoring.

**Structure:** `rules/` (copy to `~/.claude/rules/`), `agents/` (markdown agent definitions), `skills/` (organized by language/domain), `hooks/`, `commands/`, `mcp-configs/`.

**Use today:** Install via plugin, copy `rules/` to `~/.claude/rules/`, use `/plan` + `/code-review` + `/security-scan` workflow.

---

### 3. Get Shit Done (`gsd-build/get-shit-done`)
**What:** Spec-driven development system that solves context rot across long projects. Uses phased workflow: Initialize → Discuss → Plan → Execute → Verify → Ship.

**3 most valuable things:**
- **Atomic XML task planning** — max 2-3 tasks per plan with `<action>`, `<verify>`, `<done>` sections. Prevents context window overflow.
- **State persistence** — PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md keep decisions across 50+ sessions.
- **Parallel agent coordination** — concurrent research agents (stack, features, architecture, pitfalls) synthesize findings before execution.

**Structure:** Templates in `.claude/` (config.json, settings.json, context.md, continue-here.md). Runtime files: PROJECT.md, REQUIREMENTS.md, STATE.md, PLAN.md at project root.

**Use today:** `/gsd:new-project` → `/gsd:discuss-phase 1` → `/gsd:plan-phase 1` → `/gsd:execute-phase 1`. Each task gets a fresh context window + clean git commit.

---

## Tier 2 — Valuable for Specific Use Cases

### 4. UI UX Pro Max (`nextlevelbuilder/ui-ux-pro-max-skill`)
**What:** 161 design rules that auto-generate context-aware design systems based on product type + industry vertical.

**Key value:** Industry-specific reasoning (67 UI styles, 161 color palettes, 57 font pairings) organized in 10 priority tiers. Includes anti-patterns like "avoid AI purple/pink gradients for banking."

**Structure:** Single `SKILL.md` + `scripts/search.py` (BM25 search engine) + `design-system/MASTER.md` (persisted output).

**Best for:** Anyone building user-facing UIs. Pairs well with `prototype-builder` skill already in this repo.

---

### 5. Claude-Mem (`thedotmack/claude-mem`)
**What:** Persistent memory via MCP server with vector search (ChromaDB). Captures tool usage observations, generates semantic summaries, injects them into future sessions.

**Key value:** ~10x token savings through 3-layer progressive filtering. No more "hey Claude, remember when we..." — it actually remembers.

**Structure:** MCP server implementation + ChromaDB config. Runs as a service alongside Claude Code.

**Best for:** Long-running projects where you frequently switch sessions and lose context.

---

### 6. Obsidian Skills (`kepano/obsidian-skills`)
**What:** 5 skills that make Claude Code a native Obsidian power user — markdown with wikilinks, Bases databases, JSON Canvas, CLI interaction, web-to-markdown extraction.

**Key value:** The `obsidian-cli` skill bridges Claude Code with Obsidian's plugin ecosystem for vault-level operations.

**Best for:** Anyone using Obsidian as their knowledge base alongside Claude Code.

---

## Tier 3 — Reference & Discovery

### 7. Awesome Claude Code (`hesreallyhim/awesome-claude-code`)
**What:** Curated directory of production-ready Claude Code resources. Hand-picked skills, agents, hooks, slash commands, CLAUDE.md examples.

**Use as:** Starting point before building anything from scratch. Check here first.

### 8. LightRAG
**What:** Knowledge graphs + vector RAG for codebases too big for context windows.

**Use when:** Your codebase exceeds what Claude can hold in context.

### 9. n8n-MCP
**What:** Bridges Claude Code to 1,084 n8n workflow nodes directly.

**Use when:** You need Claude to trigger/manage automation workflows.

### 10. Context Engineering Intro
**What:** Teaches how to write a CLAUDE.md that actually works. "Context engineering > prompt engineering."

**Use as:** Reference when setting up any new project's CLAUDE.md.

---

## Patterns Worth Adopting

Based on the research, these patterns appear consistently across the best repos:

| Pattern | Used By | Description |
|---------|---------|-------------|
| **Spec-first gate** | Superpowers, GSD | Block code generation until design is approved |
| **Atomic tasks** | GSD, ECC | Max 2-3 tasks per plan, each with verify + done criteria |
| **Language-specific agents** | ECC | Dedicated reviewer/resolver per language |
| **State files** | GSD | PROJECT.md + STATE.md + DECISIONS.md for cross-session persistence |
| **Security hooks** | ECC (AgentShield) | Block secrets in commits, prevent `--no-verify` |
| **Priority-tiered rules** | UI UX Pro Max | Rules ranked CRITICAL → LOW so AI applies them in order |
| **Fresh context per task** | GSD, Superpowers | Each task runs in a clean agent to avoid context rot |

---

## Recommendation for This Repo

The `claude-skills` repo currently has 2 skills (prototype-builder, notebooklm-youtube). The most impactful additions would be:

1. **Spec-first workflow skill** (inspired by Superpowers + GSD) — force a discuss → design → plan → execute flow
2. **Design system skill** (inspired by UI UX Pro Max) — pairs naturally with the existing prototype-builder
3. **Session persistence pattern** (inspired by GSD) — STATE.md / DECISIONS.md for multi-session projects

These three would make the biggest difference in output quality without adding complexity.
