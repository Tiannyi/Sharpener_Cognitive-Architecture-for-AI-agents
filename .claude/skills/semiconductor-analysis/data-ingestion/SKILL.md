---
description: parsing CSV/Excel measurement files, inspecting file structures, or ingesting raw semiconductor data via pandas.
---

# Sub-Skill #1: File Structure & Data Ingestion

> **Priority sub-skill — always load this first. Claude must understand the raw data before any analysis.**

## Core Philosophy

**Claude cannot directly read CSV/Excel files reliably.** Always use pandas via the terminal to inspect and process data. Never attempt to raw-read large data files. Work through the terminal, read the output, then reason about it.

## Step 1: Inspect File Names First

Before opening any file, extract metadata from the filename itself. Typical naming patterns encode:
- **Lot ID** (e.g., `LOT2024A`, `N12345`)
- **Wafer number** (e.g., `W03`, `wafer_12`)
- **Die coordinates** (e.g., `X5Y3`, `die_0503`)
- **Device type** (e.g., `NMOS`, `PNP_5x10`, `MIM_cap`)
- **Test condition** (e.g., `25C`, `125C`, `stress_1000s`)
- **Date/timestamp** (e.g., `20240115`, `2024-01-15`)
- **Measurement type** (e.g., `IdVg`, `IdVd`, `CV`, `Spar`)

**Action:** List the files first (`ls` or `glob`), then parse the filenames to understand what you're working with before opening anything.

## Step 2: Inspection-First Workflow

For every data file, run these pandas commands in the terminal **before making any analysis decisions**:

```python
import pandas as pd

# For CSV:
df = pd.read_csv('path/to/file.csv', nrows=5)  # Read only 5 rows first
# For Excel:
df = pd.read_excel('path/to/file.xlsx', nrows=5)

# Mandatory inspection sequence:
print("Shape:", df.shape)
print("\nColumns:", df.columns.tolist())
print("\nData types:\n", df.dtypes)
print("\nFirst 5 rows:\n", df.head())
print("\nBasic stats:\n", df.describe())
print("\nNull counts:\n", df.isnull().sum())
```

**Never skip this step.** The inspection output tells you everything you need to plan the analysis.

## Step 3: Large File Strategy

Files can be massive (100K+ rows). Rules:

1. **Read headers first** — use `nrows=0` or `nrows=5` to understand structure without loading everything
2. **Check file size** — use `df.shape` on a small read, or `wc -l` in terminal for CSVs
3. **Never print full DataFrames** — always use `.head()`, `.tail()`, or slicing
4. **Filter before displaying** — select specific columns/rows/conditions before printing
5. **Summarize, don't dump** — use `.value_counts()`, `.nunique()`, `.describe()`, `.groupby().agg()`
6. **Chunked reading** for truly massive files:
   ```python
   chunks = pd.read_csv('file.csv', chunksize=10000)
   for chunk in chunks:
       # process each chunk
   ```
7. **For Excel with multiple sheets:**
   ```python
   xls = pd.ExcelFile('file.xlsx')
   print("Sheet names:", xls.sheet_names)
   # Read one sheet at a time
   df = pd.read_excel(xls, sheet_name='Sheet1', nrows=5)
   ```

## Step 4: Data Structure Recognition

After inspection, classify the data type based on columns and filename:

| If you see columns like... | It's likely... |
|---------------------------|---------------|
| Vg, Id, Ig (or Gate Voltage, Drain Current) | Id-Vg sweep |
| Vd, Id (or Drain Voltage, Drain Current) | Id-Vd sweep |
| Frequency, Capacitance, Conductance | CV measurement |
| Frequency, S11, S21, S12, S22 | S-parameter data |
| Time, Voltage, Current with stress labels | Reliability/stress data |
| Lot, Wafer, Die, Parameter, Value | Summary/parametric data |

## Step 5: Metadata Extraction Pipeline

Systematically extract from both filename and file contents:

```python
# From filename
import re
filename = "LOT2024A_W03_X5Y3_NMOS_5x10_IdVg_25C.csv"
# Parse with regex or split

# From file contents (headers, first rows, or dedicated metadata rows)
# Some measurement tools put metadata in the first N rows before the actual data
# Check if the first few rows are metadata vs. column headers
```

## Step 6: Data Cleaning Rules

- **Missing values:** Identify and decide — drop, fill, or flag? (Domain-dependent)
- **Malformed rows:** Measurement tools sometimes insert status lines mid-file. Detect and skip.
- **Unit inconsistencies:** Check if currents are in A, mA, or uA. Voltages in V or mV. Normalize.
- **Duplicate measurements:** Check for repeated die/condition combinations. Decide: keep all, average, or flag.

## Step 7: Cross-Check and Confirmation

**Never trust a single pass.** After any parsing or transformation:

1. **Print a summary** of the result (shape, head, dtypes)
2. **Spot-check** — pick 2-3 raw values and verify they match the processed output
3. **Row count audit** — confirm no silent row drops (compare raw count vs. processed count)
4. **Group count check** — verify grouping/filtering didn't accidentally exclude data
5. **Show the user a sanity-check summary** before proceeding:
   ```
   Parsed: 3 lots, 12 wafers, 5 device types, 2 conditions
   Total measurements: 14,400 rows
   Any nulls: 23 (0.16%) — in column 'Ig'
   Ready to proceed? [show summary table]
   ```

## Output

After completing ingestion, you should have:
- A clean pandas DataFrame (or set of DataFrames)
- A metadata dictionary (lot, wafer, die, device, condition info)
- A classification of the measurement type
- Confidence that the data is correctly parsed (verified via cross-checks)

**Next:** Route to `condition-logic/SKILL.md` to determine what comparisons are meaningful.
