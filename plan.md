# Execution Plan: Hierarchical Skill Architecture

---

## Module 0: Project Scaffolding
1. Create the directory structure under `/mnt/skills/user/` (or chosen root)
2. Set up a version-controlled repo for the skill files
3. Define a consistent template for all `SKILL.md` files (header, purpose, decision logic, what-to-build/not-build, sub-skill index)

---

## Module 1: MOTHER.md (Foundation — Do First)
This is the root of everything. All other modules depend on it.

1. **Draft the philosophy section** — encode "borrow before build" as the governing principle
2. **Build the 4-level decision tree** — Buy → Library → Pattern → Build, with concrete trigger conditions for each level
3. **Create the skill index/router** — a table mapping task keywords/domains to skill file paths, with clear routing rules (e.g., "if task mentions IV curve, Vth, wafer → load `semiconductor-analysis/SKILL.md`")
4. **Write the enforcement checklist** — the markdown checklist Claude must include in every response
5. **Add escape hatches** — rules for when none of the domain skills apply, and fallback behavior
6. **Token budget audit** — verify the file stays within ~1,500 tokens; trim aggressively

**Deliverable:** A single `MOTHER.md` file that can be loaded standalone and still provide value.

---

## Module 2: Semiconductor Device Measurement Analysis Skill
1. **Write the domain SKILL.md** (~1,000 tokens)
   - Define scope: IV curves, CV, S-parameters, reliability, SPC
   - List the "do not build" items (pandas, matplotlib, scipy handle these)
   - List the "do build" items (interpretation logic, comparison rules, alert thresholds)
   - Add routing table to sub-skills
2. **Sub-skill #1 (Priority): File Structure & Data Ingestion (`data-ingestion/SKILL.md`)**
   This is the foundation — Claude must understand raw data before it can analyze anything.

   **Core Philosophy: Claude cannot directly read CSV/Excel files reliably.** Always use pandas in the terminal to inspect and process data, then read the terminal output. Never try to raw-read large data files.

   - **File naming conventions** — extract embedded metadata from filenames (lot ID, wafer number, die coordinates, device type, test condition, date, etc.)
   - **Inspection-first workflow** — always start by running pandas commands in the terminal to understand the data before making any analysis decisions:
     - `df.shape` — how big is the file? (row/column count)
     - `df.head()`, `df.tail()` — preview first/last rows
     - `df.columns.tolist()` — get all column names
     - `df.dtypes` — understand data types
     - `df.describe()` — summary statistics
     - `df.info()` — memory usage, null counts
   - **Large file strategy** — files can be massive; never try to print or load everything at once:
     - Read only headers first (`nrows=0` or `nrows=5`) to understand structure
     - Use chunked reading (`chunksize`) for files too large to fit in memory
     - Filter/slice before displaying (specific columns, specific rows, specific conditions)
     - Summarize rather than dump (value counts, unique values, aggregations)
   - **File format parsing** — rules for reading CSV and Excel files from measurement tools (column headers, units, delimiter conventions, multi-sheet layouts)
   - **Data structure recognition** — identify what type of measurement a file contains based on column names, headers, and filename patterns
   - **Metadata extraction pipeline** — systematically pull lot, wafer, die, device, and condition info from filenames and file contents
   - **Data cleaning rules** — handling missing values, malformed rows, unit inconsistencies, duplicate measurements
   - **Cross-check and confirmation workflow** — never trust a single pass:
     - After parsing/transforming, always print a summary of the result to verify correctness
     - Spot-check: compare a few raw values against the processed output
     - Confirm row counts match expectations (no silent drops)
     - Double-check that grouping/filtering didn't accidentally exclude data
     - Show the user a sanity-check summary before proceeding to analysis
   - **Organizing raw data into analysis-ready structures** — standard DataFrame schemas for downstream sub-skills
3. **Sub-skill #2: Condition Logic & Meaningful Comparisons (`condition-logic/SKILL.md`)**
   Claude must understand what different test conditions mean and which comparisons are valid.
   - **Condition parsing** — identify bias conditions, temperature, stress type, and timing from data/filenames
   - **Valid comparison rules** — what can be compared to what (same device geometry, same die location, same condition, etc.)
   - **Normalization logic** — when and how to normalize (by W/L, by area, by perimeter, etc.)
   - **Grouping logic** — how to group data for meaningful aggregation (by lot, wafer, die, device type, condition)
   - **Baseline vs. stressed comparisons** — identifying pre/post stress pairs, time-zero references
   - **Red flags** — detect when a comparison is invalid or misleading (mismatched geometries, wrong normalization, mixed conditions)
4. **Sub-skill #3: Device Analysis (`device-analysis/SKILL.md`)**
   - Vth extraction methods (constant current, linear extrapolation, max-gm)
   - IV curve analysis and parameter extraction
   - CV analysis patterns
   - Common pitfalls (contact resistance artifacts, self-heating)
5. **Sub-skill #4: Reliability (`reliability/SKILL.md`)**
   - Stress test patterns (HTOL, NBTI, HCI)
   - Degradation tracking templates
   - Lifetime projection methods
6. **Sub-skill #5: Statistical Process Control (`statistical/SKILL.md`)**
   - Lot-to-lot, wafer-to-wafer, within-wafer variation analysis
   - Cp/Cpk calculation patterns
   - Control chart generation
   - Lot acceptance criteria
7. **Create `examples/` directory**
   - Sample raw data files (sanitized) showing typical naming conventions
   - Reference code snippets for each sub-skill
   - Sample Jupyter notebook showing the full pipeline: filename parsing → data ingestion → condition grouping → analysis

**Validation:** Given a folder of raw CSV/Excel files, can Claude correctly parse filenames, extract metadata, identify valid comparisons, and route to the appropriate analysis sub-skill?

---

## Module 3: Cadence Virtuoso SKILL Code Skill
1. **Write the domain SKILL.md** (~1,000 tokens)
   - Define scope: layout automation, GDS↔SKILL, pcell creation
   - "Do not build" list (gdspy/klayout for GDS parsing, SKILL built-ins for geometry)
   - "Do build" list (team-specific patterns, DRC integration, parameterization)
   - Sub-skill routing table
2. **Sub-skill: GDS-to-SKILL Conversion (`gds-to-skill/SKILL.md`)**
   - GDS hierarchy extraction workflow (using gdspy/klayout Python bindings)
   - Layer/purpose mapping rules (GDS layer numbers → PDK layer/purpose pairs)
   - SKILL code generation templates for rectangles, paths, vias, instances
   - Handling of array refs and parameterized placement
3. **Sub-skill: Code Organization (`code-organization/SKILL.md`)**
   - File naming conventions (`pcc_<device>_<variant>.il`)
   - Standard file structure: header (let block, args), body (geometry), footer (DRC)
   - Team coding style rules (indentation, commenting, variable naming)
4. **Sub-skill: Pcell Patterns (`pcell-patterns/SKILL.md`)**
   - Parameterized cell skeleton template
   - CDF parameter definitions
   - Callback function patterns
   - Common parameterization logic (finger count, width scaling, guard ring options)
5. **Sub-skill: DRC Integration (`drc/SKILL.md`)** (optional, add if needed)
   - In-layout DRC check patterns
   - Common rule encoding in SKILL
6. **Create `examples/` directory**
   - Reference SKILL code files from existing codebase (sanitized)
   - Before/after examples of GDS → SKILL conversions

**Validation:** Given a GDS file description or layout requirement, does Claude produce SKILL code matching team conventions?

---

## Module 4: iOS App Building Skill
1. **Write the domain SKILL.md** (~1,000 tokens)
   - Embed the component matrix table (Auth→Firebase, Payments→RevenueCat, etc.)
   - Philosophy: 90% borrowed, 10% unique
   - "Never build" list with specific alternatives
   - Sub-skill routing table
2. **Sub-skill: Firebase Auth (`firebase-auth/SKILL.md`)**
   - FirebaseUI setup for Google/Apple/GitHub sign-in
   - Account linking flow
   - Token management patterns
   - Common gotchas (keychain, app delegate setup)
3. **Sub-skill: RevenueCat (`revenuecat/SKILL.md`)**
   - SDK integration steps
   - Subscription product setup
   - Paywall UI configuration
   - Receipt validation flow
   - Testing with sandbox accounts
4. **Sub-skill: Firestore (`firestore/SKILL.md`)**
   - Data modeling patterns for common app structures
   - Security rules templates
   - Offline persistence configuration
   - Real-time listener patterns
5. **Sub-skill: SwiftUI Patterns (`swiftui-patterns/SKILL.md`)**
   - Recommended architecture (MVVM with `@Observable`)
   - Navigation patterns (NavigationStack)
   - State management approach
   - Common reusable view patterns
6. **Sub-skill: App Store Submission (`app-store/SKILL.md`)**
   - Pre-submission checklist
   - Common rejection reasons and how to avoid them
   - Required metadata/screenshots
7. **Create `templates/` directory**
   - Starter project template with Firebase + RevenueCat pre-wired
   - Boilerplate `AppDelegate`, `ContentView`, etc.

**Validation:** When asked "build me an iOS app with auth and subscriptions," does Claude use FirebaseUI + RevenueCat instead of building from scratch?

---

## Module 5: Backend Skill (Lightweight)
1. **Write the domain SKILL.md**
   - Routing: Firebase vs Supabase decision criteria
   - "Do not build" (custom REST APIs for CRUD, custom auth servers)
2. **Sub-skill: Firebase backend (`firebase/SKILL.md`)** — Cloud Functions patterns, Firestore triggers
3. **Sub-skill: Supabase (`supabase/SKILL.md`)** — Postgres patterns, edge functions, row-level security

---

## Module 6: Testing & Iteration
1. **Smoke test each skill in isolation** — give Claude a task in each domain and verify it routes correctly through the hierarchy
2. **Test cross-domain tasks** — e.g., "build an iOS app that visualizes semiconductor data" — does MOTHER correctly invoke multiple skills?
3. **Test the "don't build" enforcement** — ask Claude to build auth from scratch, verify it pushes back
4. **Measure token usage** — confirm each task stays within the ~3,000-3,500 token budget for loaded skills
5. **Collect failure cases** — where does the routing break? Where does Claude ignore the philosophy?
6. **Iterate on MOTHER.md** — refine routing rules and decision tree based on failures
7. **Iterate on domain skills** — add missing patterns, remove unused sub-skills

---

## Execution Order Summary

| Order | Module | Dependency | Priority |
|-------|--------|-----------|----------|
| 1 | Module 0: Scaffolding | None | Immediate |
| 2 | Module 1: MOTHER.md | Scaffolding | Immediate |
| 3 | Module 2: Semiconductor | MOTHER | High |
| 4 | Module 3: Cadence SKILL | MOTHER | High |
| 5 | Module 4: iOS App | MOTHER | High |
| 6 | Module 5: Backend | MOTHER + iOS | Medium |
| 7 | Module 6: Testing | All above | Ongoing |

Modules 2, 3, and 4 are independent of each other and can be developed in parallel or in whichever order feels most natural. Module 5 is a lighter-weight companion to Module 4.
