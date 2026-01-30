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

## Skill Routing — Planning-First Workflow

Skills are identified naturally during the planning phase, not by asking upfront. Sub-skills are manual-invoke only (`/skill-name`) to keep context minimal.

### How Skills Get Selected

**Phase 1: Planning conversation**
The user discusses their project, goals, and approach. During this natural conversation, you identify which domain and sub-skills are relevant. Don't interrupt to ask — listen, plan, then recommend.

**Phase 2: Document in CLAUDE.md**
When the planning produces a plan (often saved to an MD file), also write the selected skills into the project's `CLAUDE.md` or `.claude/CLAUDE.md`. This persists across sessions:

```markdown
## Active Skills
This project uses semiconductor analysis. Load at session start:
- `/data-ingestion` — for parsing measurement CSVs
- `/device-analysis` — for Vth/SS extraction
- `/statistical` — for Cpk and control charts
```

**Phase 3: Subsequent sessions**
When returning to the project, read `CLAUDE.md`. If active skills are listed, remind the user: "This project uses `/data-ingestion`, `/device-analysis`, and `/statistical`. Run these to load them."

**Phase 4: Ongoing work**
Once skills are loaded, proceed with the task. If the work evolves and new skills are needed, suggest them and update `CLAUDE.md`.

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

### If No Domain Matches

Apply the Decision Hierarchy above using general engineering judgment. No skills needed.

## Rules

1. **Never generate code as a first response.** First identify the domain, then check the hierarchy.
2. **If a library exists, recommend it.** Don't build a custom version.
3. **Identify skills during planning, not by asking upfront.** Let the conversation reveal what's needed.
4. **Persist skill selections in CLAUDE.md** so they carry across sessions.
5. **Sub-skills are manual-invoke only.** Remind the user to run `/skill-name` to load them.
6. **When in doubt, ask the user** rather than assuming you should build.
