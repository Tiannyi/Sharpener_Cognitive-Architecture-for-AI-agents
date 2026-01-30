# Hierarchical Skill Architecture for AI Agents

## Executive Summary

This document captures a brainstorming session on building a hierarchical skill system for AI coding assistants (Claude Code, GitHub Copilot, etc.). The core insight is that AI assistants should encode **decision-making philosophy** — knowing what NOT to build — as the primary skill, with domain-specific skills called on-demand.

---

## The Problem We Identified

### Current AI Behavior
- AI defaults to generating code when asked
- Doesn't ask "should we build this at all?"
- Misses the experienced developer's instinct: **borrow before build**
- Treats all problems as implementation problems

### What Experienced Developers Actually Do
1. Assume someone smarter already solved this
2. Search for existing libraries/services
3. Only build if truly novel or domain-specific
4. Spend creative energy on the **unique** part of the problem

### The Gap
AI knows *how* to build things, but doesn't know *when* not to build.

---

## The Solution: Hierarchical Skill Architecture

### Core Concept

```
┌─────────────────────────────────────────────────────────┐
│                    MOTHER SKILL                          │
│         (Always loaded, governs all decisions)           │
│                                                          │
│  • Philosophy: Borrow before build                       │
│  • Decision tree: Buy → Use library → Follow pattern →   │
│                   Only then: Build                       │
│  • Index of all domain skills                            │
│  • Instructions for when to call each skill              │
└─────────────────────────────────────────────────────────┘
                          │
                          │ Calls appropriate skill based on task
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   ┌─────────┐      ┌──────────┐     ┌───────────┐
   │ Domain  │      │ Domain   │     │ Domain    │
   │ Skill A │      │ Skill B  │     │ Skill C   │
   └────┬────┘      └────┬─────┘     └─────┬─────┘
        │                │                 │
        ▼                ▼                 ▼
   ┌─────────┐      ┌──────────┐     ┌───────────┐
   │Sub-skill│      │Sub-skill │     │Sub-skill  │
   └─────────┘      └──────────┘     └───────────┘
```

### Key Principles

1. **MOTHER skill is always loaded first** — small footprint (~1,500 tokens)
2. **Domain skills loaded on-demand** — only when task requires
3. **Sub-skills for deep dives** — specific implementation patterns
4. **Skills encode decisions, not just implementations**

---

## The Meta-Lesson (Most Important Insight)

The most valuable skill isn't knowing *how* to build auth, payments, or databases. It's knowing:

- **What NOT to build** (most things)
- **Which tools to use** (specific recommendations)
- **Where to focus effort** (the unique parts)

This meta-lesson should be the foundation of the MOTHER skill.

---

## The Hierarchy of Action

Encoded in the MOTHER skill:

```
Level 1: Does this problem have a paid solution?
         → Yes: Buy it. Your time is worth more than $20/month.
         → Examples: Auth (Firebase), Payments (RevenueCat)

Level 2: Does this problem have an open-source solution?
         → Yes: Use it. Don't fork unless necessary.
         → Examples: UI components, networking libraries

Level 3: Does this problem have a documented pattern?
         → Yes: Follow the pattern exactly. Don't be clever.
         → Examples: REST API design, database schemas

Level 4: Is this problem unique to my domain?
         → Yes: NOW you write code.
         → This is where your expertise matters.
```

---

## Three Domain Skills to Develop

### 1. Semiconductor Device Measurement Analysis

**Purpose:** Analyze device data from fab measurements

**Domain Knowledge to Encode:**
- Data structure patterns (IV curves, CV, S-parameters)
- Device naming conventions and parameter extraction
- Meaningful comparisons (normalize by W/L, compare same die location)
- Statistical analysis norms (lot-to-lot, wafer-to-wafer variation)
- Red flags and anomaly patterns (Vth shift, Ion/Ioff degradation)

**What NOT to Build:**
- Data parsers (use pandas)
- Plotting (use matplotlib/seaborn)
- Statistics (use scipy)

**What TO Build:**
- Interpretation logic specific to your process
- Comparison rules for your device types
- Alert thresholds for your specs

**Potential Sub-skills:**
- IV curve analysis and Vth extraction
- Reliability analysis (stress patterns)
- Statistical process control
- Cross-wafer/lot comparison

---

### 2. Cadence Virtuoso SKILL Code (GDS ↔ SKILL Conversion)

**Purpose:** Layout automation, code organization, GDS-to-SKILL conversion

**Domain Knowledge to Encode:**
- SKILL++ coding conventions
- Naming patterns (`pcc_<device>_<variant>.il`)
- Common layout primitives → SKILL function mappings
- GDS layer/purpose mappings to PDK
- Parameterized cell (pcell) structure patterns
- Code organization (header, body, footer with DRC checks)

**Key Capability:**
- Convert GDS file → SKILL code (reverse engineering layouts)
- Understand existing codebase patterns
- Generate code that matches team conventions

**What NOT to Build:**
- GDS parsing (use existing libraries like gdspy, klayout)
- Basic geometry operations (use SKILL built-ins)

**What TO Build:**
- Your team's specific layout patterns
- Process-specific DRC integration
- Parameterization logic for your device types

**Potential Sub-skills:**
- GDS parsing and hierarchy extraction
- SKILL code generation patterns
- DRC integration
- Pcell parameterization

---

### 3. iOS App Building (Software Engineering)

**Purpose:** Build iOS apps efficiently by using existing solutions for common components

**Core Philosophy:**
90% of your app should be borrowed/bought/copied.
10% is your unique value — spend all creative energy there.

**The Component Matrix:**

| Component | Solution | Effort | Build Custom? |
|-----------|----------|--------|---------------|
| Auth (Google/Apple/GitHub) | FirebaseUI | 1-2 hours | NEVER |
| Account Management | Follows auth provider | 1 hour | NEVER |
| Payments/Subscriptions | RevenueCat | 2-3 hours | NEVER |
| Paywall UI | RevenueCat Paywalls | 1 hour | NEVER |
| Database | Firestore or Supabase | 2 hours | NEVER |
| Push Notifications | Firebase Cloud Messaging | 1 hour | NEVER |
| Analytics | Firebase Analytics | 30 min | NEVER |
| Crash Reporting | Crashlytics | 30 min | NEVER |
| **Your Unique Feature** | **You build this** | **Your time** | **YES** |

**What User Should Focus On:**
- Understanding users
- Designing unique feature
- Building unique feature
- Polishing UX of unique feature

**What User Should NOT Do:**
- Build custom auth screens
- Build custom payment flows
- Build generic CRUD operations
- Build push notification infrastructure
- Debate architecture before having users
- Optimize before having performance problems

**Potential Sub-skills:**
- Firebase Auth integration patterns
- RevenueCat subscription setup
- Firestore data modeling
- SwiftUI architecture patterns
- App Store submission checklist

---

## File Structure

```
/mnt/skills/user/
│
├── MOTHER.md                              ← Always loaded first (~1,500 tokens)
│
├── semiconductor-analysis/
│   ├── SKILL.md                           ← Main skill (~1,000 tokens)
│   ├── iv-analysis/SKILL.md               ← Vth extraction, curve fitting
│   ├── reliability/SKILL.md               ← Stress analysis patterns
│   ├── statistical/SKILL.md               ← SPC, lot acceptance
│   └── examples/                          ← Reference code snippets
│
├── cadence-skill/
│   ├── SKILL.md                           ← Main skill (~1,000 tokens)
│   ├── gds-to-skill/SKILL.md              ← Conversion patterns
│   ├── code-organization/SKILL.md         ← Team conventions
│   ├── pcell-patterns/SKILL.md            ← Parameterized cells
│   └── examples/                          ← Reference code from your codebase
│
├── ios-app/
│   ├── SKILL.md                           ← Main skill (~1,000 tokens)
│   ├── firebase-auth/SKILL.md             ← OAuth setup, account linking
│   ├── revenuecat/SKILL.md                ← Subscriptions, paywalls
│   ├── firestore/SKILL.md                 ← Data modeling, security rules
│   ├── swiftui-patterns/SKILL.md          ← Architecture, state management
│   └── templates/                         ← Starter code snippets
│
└── backend/
    ├── SKILL.md
    ├── firebase/SKILL.md
    └── supabase/SKILL.md
```

---

## Context Window Management

| What's Loaded | Tokens | When |
|---------------|--------|------|
| MOTHER.md | ~1,500 | Always |
| Domain SKILL.md | ~1,000 | When domain identified |
| Sub-skill | ~500-1,000 | When specific task identified |
| **Total per task** | **~3,000-3,500** | Manageable |

**Key Strategy:**
- MOTHER.md is compact but governs everything
- Domain skills loaded on-demand via `view` tool
- Sub-skills only when diving into specific implementation
- Never load everything at once

---

## Enforcement Mechanism

The MOTHER skill includes a checklist that Claude must show in responses:

```markdown
## Checklist (Include in Every Response)

- [ ] Searched for existing solutions
- [ ] Result: [library found / none found / N/A]
- [ ] Domain identified: [semiconductor / cadence / ios / backend / other]
- [ ] Skill loaded: [path or "none needed"]
- [ ] Proceeding with: [recommendation / implementation]
```

This creates accountability and helps the user see that the reasoning process was followed.

---

## Connection to AI/ML Research

What we're describing aligns with several research directions:

| Concept | Our Implementation |
|---------|-------------------|
| Hierarchical reasoning | MOTHER → Domain → Sub-skill |
| Meta-cognition | MOTHER skill knows when NOT to act |
| Tool use | Skills called via `view` tool |
| Task decomposition | Breaking into build-vs-borrow decision first |
| Multi-agent (lite) | Different skills act like specialized agents |

This is a **practical, user-implementable** version of what AI labs are researching with more complex multi-agent systems.

---

## Next Steps

### Phase 1: Create MOTHER.md
- Philosophy of problem-solving
- Decision tree (buy → library → pattern → build)
- Skill index with routing logic
- Enforcement checklist

### Phase 2: Create Semiconductor Analysis Skill
- Leverage Tianyi's domain expertise
- Encode meaningful comparisons
- Document what NOT to build
- Add sub-skills for specific analyses

### Phase 3: Create Cadence SKILL Code Skill
- Based on existing codebase patterns
- GDS-to-SKILL conversion logic
- Team conventions and naming patterns
- Code organization standards

### Phase 4: Create iOS App Building Skill
- Component matrix (what to use for each need)
- Integration guides for FirebaseUI, RevenueCat, etc.
- Sub-skills for specific implementations
- Templates and starter code

### Phase 5: Test and Iterate
- Use skills in real projects
- Measure: Does Claude follow the hierarchy?
- Refine based on where it breaks down
- Add more domain skills as needed

---

## Key Quotes from Discussion

> "The skill of an experienced developer isn't knowing *how* to build auth — it's knowing **not to build auth**."

> "Your job is to build what ONLY YOU can build. Everything else should be borrowed."

> "AI tends to show off by generating code, when the right answer is often 'don't write code, use this library.'"

> "The meta-skill should be the first thing loaded, not an afterthought."

> "90% of your app should be borrowed/bought/copied. 10% is your unique value."

---

## Conclusion

This architecture represents a shift from "AI as code generator" to "AI as experienced colleague" — one that knows when to say "don't build that, use this instead" before diving into implementation.

The MOTHER skill encodes the wisdom that takes years for developers to learn: most problems are already solved, and your value is in the unique parts only you can build.

---

*Document created: January 2025*
*Project: Hierarchical Skill Architecture for AI Agents*
*Next: Move to dedicated project for detailed skill development*