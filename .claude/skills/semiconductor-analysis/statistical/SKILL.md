---
description: Load when calculating Cp/Cpk, building control charts, analyzing wafer maps, or performing lot acceptance.
---

# Sub-Skill #5: Statistical Process Control

> **Load when the task involves lot/wafer variation analysis, Cp/Cpk, control charts, or lot acceptance.**

## Scope

Statistical analysis of semiconductor parametric data across lots, wafers, and dies for process monitoring and quality control.

## Key Metrics

### Cp / Cpk (Process Capability)
```python
import numpy as np

def calc_cpk(data, usl, lsl):
    mean = np.mean(data)
    std = np.std(data, ddof=1)
    cp = (usl - lsl) / (6 * std)
    cpu = (usl - mean) / (3 * std)
    cpl = (mean - lsl) / (3 * std)
    cpk = min(cpu, cpl)
    return cp, cpk
```
- **Cp ≥ 1.33:** Process is capable
- **Cpk ≥ 1.33:** Process is capable AND centered
- **Cp good but Cpk poor:** Process has good spread but is off-center

### Variation Decomposition
```python
# Lot-to-lot variation
lot_means = df.groupby('lot')['param'].mean()
lot_to_lot_var = lot_means.var()

# Wafer-to-wafer (within lot)
wafer_means = df.groupby(['lot', 'wafer'])['param'].mean()
within_lot_var = wafer_means.groupby('lot').var().mean()

# Within-wafer
within_wafer_var = df.groupby(['lot', 'wafer'])['param'].var().mean()

# Total
total_var = lot_to_lot_var + within_lot_var + within_wafer_var
```

## Control Charts

### X-bar and R chart (lot/wafer level)
```python
import matplotlib.pyplot as plt

lot_means = df.groupby('lot')['param'].mean()
lot_stds = df.groupby('lot')['param'].std()
grand_mean = lot_means.mean()
ucl = grand_mean + 3 * lot_means.std()
lcl = grand_mean - 3 * lot_means.std()

plt.figure(figsize=(12, 5))
plt.plot(lot_means.index, lot_means.values, 'bo-')
plt.axhline(grand_mean, color='g', linestyle='-', label='Mean')
plt.axhline(ucl, color='r', linestyle='--', label='UCL')
plt.axhline(lcl, color='r', linestyle='--', label='LCL')
plt.xlabel('Lot')
plt.ylabel('Parameter')
plt.legend()
plt.title('X-bar Control Chart')
```

### Out-of-Control Rules (Western Electric)
Flag if any of:
1. One point beyond 3σ
2. Two of three consecutive points beyond 2σ (same side)
3. Four of five consecutive points beyond 1σ (same side)
4. Eight consecutive points on same side of center line

## Wafer Maps

```python
# Visualize within-wafer distribution
plt.figure(figsize=(8, 8))
scatter = plt.scatter(df['die_x'], df['die_y'], c=df['param'], cmap='RdYlGn_r', s=100)
plt.colorbar(scatter, label='Parameter Value')
plt.xlabel('Die X')
plt.ylabel('Die Y')
plt.title('Wafer Map')
plt.axis('equal')
```

Look for:
- **Center-to-edge gradient** — common, process-related
- **Radial symmetry** — typical for deposition/etch uniformity
- **Cluster defects** — localized process issue
- **Edge exclusion effects** — expected, usually excluded from stats

## Lot Acceptance Criteria

Typical acceptance flow:
1. Calculate mean and sigma for each parameter
2. Check all parameters against spec limits (LSL, USL)
3. Calculate Cpk — require ≥ 1.33 (or per team standard)
4. Check for outlier dies (beyond 3σ or per exclusion rules)
5. Generate pass/fail summary

```
Lot: 2024A | Wafers: 25 | Dies per wafer: 200
─────────────────────────────────────────────
Parameter    Mean    Sigma   LSL    USL    Cpk    Status
Vth_n       0.340   0.012   0.28   0.42   1.85   PASS
Vth_p      -0.355   0.015  -0.44  -0.28   1.78   PASS
Idsat_n     490     22      420    ---    1.06   WARN
Ron          85     4.2      ---   100    1.19   WARN
```

## Output

- Variation breakdown (lot-to-lot, wafer-to-wafer, within-wafer)
- Cp/Cpk per parameter
- Control charts with out-of-control flags
- Wafer maps for spatial analysis
- Lot acceptance summary (pass/warn/fail per parameter)
