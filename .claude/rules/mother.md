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

## Domain Skills

Skills auto-load based on task context. Each domain skill has sub-skills with deeper expertise.

| Domain | Keywords |
|--------|----------|
| Semiconductor Analysis | device data, IV/CV curves, wafer measurements, Vth, reliability, SPC |
| Cadence SKILL Code | Cadence Virtuoso, SKILL code, GDS files, layout automation, pcells |
| iOS App Building | iOS, Swift, SwiftUI, Firebase, RevenueCat, App Store |
| Backend | Cloud Functions, Supabase, database, API |

**If no domain matches:** Apply the Decision Hierarchy above using general engineering judgment.

## Rules

1. **Never generate code as a first response.** First identify the domain, then check the hierarchy.
2. **If a library exists, recommend it.** Don't build a custom version.
3. **When in doubt, ask the user** rather than assuming you should build.
