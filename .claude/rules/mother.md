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

## Skill Routing — Ask First, Load Second

**At the start of a new task, ask the user which domain they're working in.** Present the available domains and their sub-skills so the user can choose what to load. This keeps context usage minimal — only load what's needed.

### Available Domains and Sub-Skills

**1. Semiconductor Analysis** (`/semiconductor-analysis`)
   - `/data-ingestion` — CSV/Excel parsing, file structure inspection, pandas workflows
   - `/condition-logic` — Valid comparisons, normalization, grouping by lot/wafer/die
   - `/device-analysis` — Vth, SS, DIBL, Ion/Ioff extraction from IV/CV data
   - `/reliability` — NBTI/HCI/TDDB stress data, degradation trends, lifetime projection
   - `/statistical` — Cp/Cpk, control charts, wafer maps, lot acceptance

**2. Cadence SKILL Code** (`/cadence-skill`)
   - `/gds-to-skill` — GDS/GDSII to SKILL conversion via gdspy/klayout
   - `/code-organization` — File structure, naming conventions, coding standards
   - `/pcell-patterns` — Parameterized cells, CDF parameters, callbacks, device generators

**3. iOS App Building** (`/ios-app`)
   - `/firebase-auth` — Google/Apple/Email sign-in, token management
   - `/revenuecat` — Subscriptions, paywalls, entitlements
   - `/firestore` — Data models, CRUD, real-time listeners, security rules
   - `/swiftui-patterns` — MVVM with @Observable, navigation, reusable views
   - `/app-store` — Submission checklist, rejection handling, metadata

**4. Backend** (`/backend`)
   - `/firebase` — Cloud Functions, Firestore triggers, security rules
   - `/supabase` — Postgres, RLS policies, Edge Functions, real-time

### Routing Workflow

1. **Ask the user:** "What domain are you working in? Here are the available skills: [list relevant ones]"
2. **User picks** a domain and/or specific sub-skills
3. **Tell the user** to run the slash commands (e.g., "Run `/data-ingestion` then `/device-analysis`")
4. **Only then** proceed with the task using the loaded skill knowledge

**If no domain matches:** Apply the Decision Hierarchy above using general engineering judgment. No skills needed.

## Rules

1. **Never generate code as a first response.** First identify the domain, then check the hierarchy.
2. **If a library exists, recommend it.** Don't build a custom version.
3. **Ask the user which skills to load.** Don't auto-load everything.
4. **Sub-skills are manual-invoke only.** The user must run `/skill-name` to load them.
5. **When in doubt, ask the user** rather than assuming you should build.
