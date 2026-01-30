# Sharpener

## What This Project Is

This project defines a **cognitive architecture** for AI agents.

In academic/industry terms, it touches on:
- Cognitive architectures (like ACT-R, SOAR in AI research)
- Hierarchical task planning
- Meta-reasoning (thinking about how to think)
- Knowledge engineering (encoding expert knowledge)
- Agentic workflows

But implemented practically using existing skill systems (Claude Code, GitHub Copilot, etc.).

### The One-Sentence Version

> "I'm building a cognitive architecture that encodes expert judgment into
> AI decision-making, ensuring domain knowledge guides every action."

---

## Why This Exists

AI coding assistants are powerful, but they default to building everything from scratch. An experienced engineer knows the opposite — **the best code is code you don't write.**

This project encodes that judgment into a structured skill system so that AI agents:
- Know what NOT to build (use Firebase Auth, not a custom auth system)
- Know what libraries to reach for (pandas, gdspy, RevenueCat)
- Know domain-specific patterns (semiconductor measurement analysis, Cadence pcells, SwiftUI MVVM)
- Only write custom code for problems that are truly unique

## Architecture

```
.claude/
├── rules/
│   └── mother.md                          ← Always loaded. Decision hierarchy + routing.
└── skills/
    ├── semiconductor-analysis/
    │   ├── SKILL.md                       ← Domain overview, what to borrow vs. build
    │   ├── data-ingestion/SKILL.md        ← File parsing, pandas via terminal, large file handling
    │   ├── condition-logic/SKILL.md       ← Valid comparisons, normalization, grouping
    │   ├── device-analysis/SKILL.md       ← Vth extraction, IV/CV analysis
    │   ├── reliability/SKILL.md           ← NBTI/HCI/TDDB, degradation trends, lifetime projection
    │   └── statistical/SKILL.md           ← Cp/Cpk, control charts, wafer maps, lot acceptance
    ├── cadence-skill/
    │   ├── SKILL.md                       ← Domain overview, SKILL language essentials
    │   ├── gds-to-skill/SKILL.md          ← GDS parsing with gdspy/klayout, SKILL code generation
    │   ├── code-organization/SKILL.md     ← File structure, naming conventions, coding style
    │   └── pcell-patterns/SKILL.md        ← Pcell skeletons, CDF callbacks, parameterization
    ├── ios-app/
    │   ├── SKILL.md                       ← Domain overview, component matrix (buy vs. build)
    │   ├── firebase-auth/SKILL.md         ← Google/Apple/Email sign-in, token management
    │   ├── revenuecat/SKILL.md            ← Subscriptions, paywall UI, entitlements
    │   ├── firestore/SKILL.md             ← Data modeling, repository pattern, security rules
    │   ├── swiftui-patterns/SKILL.md      ← MVVM with @Observable, navigation, reusable views
    │   └── app-store/SKILL.md             ← Submission checklist, rejection handling, ASO
    └── backend/
        ├── SKILL.md                       ← Firebase vs. Supabase decision framework
        ├── firebase/SKILL.md              ← Cloud Functions, security rules
        └── supabase/SKILL.md              ← Postgres/RLS, Edge Functions, real-time
```

## How It Works

1. **MOTHER loads first** — establishes the "borrow before build" decision hierarchy
2. **Task is routed** to the appropriate domain skill based on keywords
3. **Domain skill loads** — provides scope, what-not-to-build, and routes to sub-skills
4. **Sub-skills load on demand** — only the relevant ones, keeping context usage low (~3,000 tokens per sub-skill)

Each sub-skill contains:
- Scope and trigger conditions
- Code patterns and templates
- Domain-specific rules and red flags
- Output format expectations

## Core Principle: The Decision Hierarchy

Before writing any code, the agent walks through four levels:

| Level | Question | Action |
|-------|----------|--------|
| 1 | Does a paid solution exist? | Use it (Firebase, RevenueCat, etc.) |
| 2 | Does an open-source library solve this? | Use it (pandas, gdspy, etc.) |
| 3 | Does a documented pattern exist? | Follow it exactly |
| 4 | Is this problem truly unique? | NOW write code |

## Usage

Copy the `.claude/` folder into your project:

```bash
cp -r .claude/ your-project/.claude/
```

Claude Code automatically loads rules from `.claude/rules/` and discovers skills from `.claude/skills/`. No additional configuration needed.

## Current Domains

| Domain | Sub-Skills | Focus |
|--------|-----------|-------|
| Semiconductor Analysis | 5 | Device measurement data parsing, parameter extraction, reliability, SPC |
| Cadence SKILL | 3 | Layout automation, pcell creation, GDS conversion |
| iOS App | 5 | SwiftUI apps with Firebase, RevenueCat, Firestore |
| Backend | 2 | Firebase Cloud Functions, Supabase Postgres/Edge Functions |

## Roadmap

- [ ] Add `examples/` directories with sample data and code for each domain
- [ ] Convert skill files to GitHub Copilot custom instructions format
- [ ] Test and iterate skills against real projects
- [ ] Measure and optimize token usage per skill load