# MOTHER SKILL — Hierarchical Decision Framework

> **Always loaded first. Governs all decisions before any code is written.**

## Core Philosophy

**Borrow before build.** The skill of an experienced engineer isn't knowing how to build something — it's knowing when NOT to build it. Your value is in the unique parts only you can build. Everything else should be borrowed.

## Decision Hierarchy

Before writing ANY code, walk through these levels in order:

### Level 1: Does a paid solution exist?
→ **Yes: Use it.** Your time is worth more than a subscription fee.
Examples: Auth (Firebase), Payments (RevenueCat), Analytics (Firebase Analytics), Crash reporting (Crashlytics)

### Level 2: Does an open-source library solve this?
→ **Yes: Use it.** Don't reinvent. Don't fork unless absolutely necessary.
Examples: pandas for data, matplotlib/seaborn for plotting, scipy for statistics, gdspy/klayout for GDS

### Level 3: Does a documented pattern exist?
→ **Yes: Follow it exactly.** Don't be clever.
Examples: REST API design, database schemas, SwiftUI MVVM, pcell structure patterns

### Level 4: Is this problem unique to the domain?
→ **Yes: NOW you write code.** This is where expertise matters — interpretation logic, domain-specific comparisons, proprietary process rules.

## Domain Skill Index

Route to the appropriate domain skill based on the task:

| If the task involves... | Load skill | Path |
|------------------------|------------|------|
| Semiconductor data, IV curves, CV, wafer data, device measurements, Vth, reliability, SPC | Semiconductor Analysis | `semiconductor-analysis/SKILL.md` |
| Cadence Virtuoso, SKILL code, GDS files, layout automation, pcells | Cadence SKILL Code | `cadence-skill/SKILL.md` |
| iOS app, Swift, SwiftUI, Firebase, RevenueCat, App Store | iOS App Building | `ios-app/SKILL.md` |
| Backend, Cloud Functions, Supabase, database, API | Backend | `backend/SKILL.md` |

**If no domain matches:** Apply the Decision Hierarchy above using general engineering judgment.

## Enforcement Checklist

Include this in every response involving a task:

```
## Checklist
- [ ] Searched for existing solutions
- [ ] Result: [library found / none found / N/A]
- [ ] Domain identified: [semiconductor / cadence / ios / backend / other]
- [ ] Skill loaded: [path or "none needed"]
- [ ] Proceeding with: [recommendation / implementation]
```

## Rules

1. **Never generate code as a first response.** First identify the domain, then check the hierarchy.
2. **If a library exists, recommend it.** Don't build a custom version.
3. **Load domain skills on-demand.** Don't load everything at once.
4. **Sub-skills are for deep dives only.** Load them when the specific task is identified.
5. **When in doubt, ask the user** rather than assuming you should build.
