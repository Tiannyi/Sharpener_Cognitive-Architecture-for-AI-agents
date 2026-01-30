---
description: Load when task involves semiconductor device data, IV/CV curves, wafer measurements, reliability testing, or statistical process control.
---

# Semiconductor Device Measurement Analysis

> **Domain skill — load when task involves device data, IV/CV curves, wafer measurements, reliability, or SPC.**

## Scope

This skill covers analysis of semiconductor device measurement data from fab/test environments:
- IV curves (Id-Vg, Id-Vd), CV measurements, S-parameters
- Reliability and stress testing data
- Statistical process control and lot acceptance
- Cross-wafer, cross-lot, and cross-condition comparisons

## What NOT to Build

| Need | Use This | Never Build Custom |
|------|----------|--------------------|
| Data loading/parsing | pandas (`read_csv`, `read_excel`) | Custom file parsers |
| Plotting | matplotlib, seaborn | Custom plotting engines |
| Statistics | scipy.stats, numpy | Custom stat functions |
| Curve fitting | scipy.optimize.curve_fit | Custom fitting algorithms |
| Data tables | pandas DataFrame | Custom data structures |

## What TO Build (Domain-Specific)

- Interpretation logic specific to your process/technology
- Comparison rules for your device types and test structures
- Alert thresholds tied to your specs
- Naming convention parsers for your team's file formats
- Condition-aware grouping and normalization logic

## Sub-Skill Routing

| Task | Load Sub-Skill |
|------|---------------|
| Reading files, parsing filenames, understanding data structure | `data-ingestion/SKILL.md` |
| Understanding test conditions, deciding what to compare | `condition-logic/SKILL.md` |
| Extracting Vth, analyzing IV/CV curves, parameter extraction | `device-analysis/SKILL.md` |
| Stress testing, degradation, lifetime projection | `reliability/SKILL.md` |
| Lot/wafer variation, Cp/Cpk, control charts | `statistical/SKILL.md` |

## Workflow Order

Always follow this sequence:
1. **Ingest** — parse files, extract metadata (Sub-skill #1)
2. **Understand conditions** — identify what's being compared (Sub-skill #2)
3. **Analyze** — apply the appropriate analysis (Sub-skills #3–5)
4. **Report** — summarize findings with plots and tables
