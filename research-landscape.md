# Landscape Research: AI Agent Skills Ecosystem

> Research conducted Jan 2026. Goal: identify repositories to fork, clean up, and integrate into Sharpener as the one-stop skill hub.

---

## Key Finding

**SKILL.md is now an open standard.** Anthropic released it, OpenAI adopted it for Codex CLI and ChatGPT (Dec 2025), and GitHub Copilot uses a similar folder-based format. Skills written in SKILL.md are portable across Claude Code, Codex CLI, ChatGPT, and Copilot.

**The ecosystem is large but fragmented.** 87,000+ skills indexed by third-party marketplaces, but no single curated, quality-controlled collection exists for domain-specific expert knowledge. Most skills are general-purpose developer productivity tools. This is the gap Sharpener fills.

---

## Repositories Worth Forking / Studying

### Tier 1: Fork and integrate

| Repository | Stars | What It Has | Why Fork |
|-----------|-------|------------|----------|
| [anthropics/skills](https://github.com/anthropics/skills) | Official | 50+ skills across 9 categories (doc processing, data analysis, dev tools, etc.) | Official reference implementation. Cherry-pick well-structured skills. |
| [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) | — | Complete config: agents, skills, hooks, commands, rules, MCP configs | Battle-tested over 10+ months. Study architecture decisions. |
| [PatrickJS/awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | Large | .cursorrules for many frameworks (Next.js 15, Angular, Astro, etc.) | Convert best Cursor rules into SKILL.md format. Framework-specific knowledge. |

### Tier 2: Study and selectively adopt

| Repository | What It Has | Value |
|-----------|------------|-------|
| [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) | 18k stars. Links to skills, hooks, slash-commands, agent orchestrators | Discovery — find individual high-quality skills to adopt |
| [github/awesome-copilot](https://github.com/github/awesome-copilot) | Official GitHub collection. Community agents, instructions, skills | Cross-pollinate Copilot instructions into SKILL.md format |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | 100+ specialized subagents by development use case | Subagent patterns we could adopt |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | Curated practical Claude skills | Vetted quality — good source for cherry-picking |
| [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) | Curated skills, resources, tools | Another curation layer |

### Tier 3: Reference / learn from architecture

| Repository | What It Has | Value |
|-----------|------------|-------|
| [EliFuzz/awesome-system-prompts](https://github.com/EliFuzz/awesome-system-prompts) | System prompts from Augment, Claude Code, Cursor, Devin, Kiro, Codex, etc. | Understand how different agents are instructed internally |
| [instructa/ai-prompts](https://github.com/instructa/ai-prompts) | Cross-tool prompts for Cursor, Copilot, Zed, Windsurf, Cline | Multi-tool conversion patterns |
| [sparesparrow/cursor-rules](https://github.com/sparesparrow/cursor-rules) | Agentic workflows, cognitive architectures | Similar philosophy to Sharpener |
| [blefnk/awesome-cursor-rules](https://github.com/blefnk/awesome-cursor-rules) | Rules for multiple AI IDEs | Cross-IDE compatibility patterns |

---

## Third-Party Marketplaces

| Platform | Scale | Notes |
|----------|-------|-------|
| [SkillsMP.com](https://skillsmp.com/) | 87,000–96,000+ skills | Searchable, category filtering, quality indicators. Not affiliated with Anthropic. |
| [SkillHub](https://www.skillhub.club/) | Unknown | Another marketplace for Claude Code skills |
| [quemsah/awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins) | 3,717 repos indexed, 243 plugins | Automated metrics collection |

---

## What's Missing (Sharpener's Opportunity)

1. **No domain-expert curation.** Marketplaces index everything but don't evaluate quality. Nobody is saying "this semiconductor analysis skill is wrong about Vth extraction."

2. **No hierarchical architecture.** Every repo is a flat list of skills. None implement a MOTHER → domain → sub-skill routing system that manages context window budget.

3. **No cross-tool normalization.** Cursor rules, Copilot instructions, and Claude skills exist in separate silos. Nobody is converting between formats systematically.

4. **No "borrow before build" enforcement.** Most skills tell the AI what TO do. None tell it what NOT to do — which is the core Sharpener philosophy.

5. **No niche domain coverage.** The ecosystem is dominated by web dev (React, Next.js, Angular). Semiconductor analysis, Cadence SKILL, hardware design — zero coverage.

---

## Recommended Reading

| Source | Link | Summary |
|--------|------|---------|
| Anthropic Engineering | [Equipping agents for the real world](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | Official deep dive on Agent Skills architecture |
| Anthropic Best Practices | [Claude Code best practices](https://www.anthropic.com/engineering/claude-code-best-practices) | Official guidance on CLAUDE.md, skills, configuration |
| Lee Han Chung | [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/) | First-principles technical analysis |
| Hugging Face / Sionic AI | [1000+ ML Experiments/Day](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) | Production use case with skills |

---

## Action Plan

1. **Fork `anthropics/skills`** — cherry-pick the best general-purpose skills, clean up, add to Sharpener
2. **Fork `PatrickJS/awesome-cursorrules`** — convert top framework rules (Next.js, React, etc.) to SKILL.md format
3. **Study `affaan-m/everything-claude-code`** — adopt architectural patterns (hooks, MCP config, subagent setup)
4. **Scan `hesreallyhim/awesome-claude-code`** — identify high-quality individual skills to adopt
5. **Build conversion tooling** — automate .cursorrules → SKILL.md and .github/instructions → SKILL.md conversion
6. **Expand domain coverage** — add skills for underserved domains (hardware, EDA, scientific computing, etc.)
7. **Submit Sharpener to awesome lists** — get listed on awesome-claude-code, awesome-claude-skills, SkillsMP
