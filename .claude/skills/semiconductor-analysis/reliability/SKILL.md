---
description: analyzing stress testing data, degradation trends, NBTI/HCI/TDDB, or projecting device lifetime.
---

# Sub-Skill #4: Reliability Analysis

> **Load when the task involves stress testing, degradation tracking, or lifetime projection.**

## Scope

Analysis of device reliability data: stress-measure-stress sequences, parameter drift over time, and lifetime estimation.

## Common Stress Types

| Stress Type | What Degrades | Key Parameter to Track |
|------------|---------------|----------------------|
| NBTI (Negative Bias Temperature Instability) | PMOS Vth shift | ΔVth vs. stress time |
| PBTI (Positive Bias Temperature Instability) | NMOS Vth shift (high-k) | ΔVth vs. stress time |
| HCI (Hot Carrier Injection) | Id degradation, Vth shift | ΔId/Id0, ΔVth vs. stress time |
| TDDB (Time-Dependent Dielectric Breakdown) | Gate oxide failure | Time-to-breakdown distribution |
| Electromigration (EM) | Metal line failure | Time-to-failure distribution |
| HTOL (High-Temperature Operating Life) | General parametric drift | Multiple parameters vs. time |

## Data Structure

Reliability data typically has:
- **Readout intervals:** t0 (fresh), then stress times (100s, 1000s, 10000s, etc.)
- **Pre-stress (fresh) measurement:** The baseline — this is always your reference
- **Post-stress measurements:** Same measurement conditions as pre-stress, taken after stress intervals
- **Stress conditions:** Voltage, temperature, and duration of the stress phase

## Analysis Workflow

### 1. Establish Baseline
```python
# t0 (fresh) measurement is always the reference
baseline = df[df['stress_time'] == 0]
```

### 2. Calculate Parameter Shifts
```python
# Absolute shift
delta_vth = vth_stressed - vth_fresh

# Relative shift (percentage)
delta_id_pct = (id_stressed - id_fresh) / id_fresh * 100
```

### 3. Plot Degradation Trends
- X-axis: stress time (log scale)
- Y-axis: parameter shift (ΔVth, ΔId/Id0 %)
- One line per device/die
- Include spec limit as horizontal line

### 4. Power Law Fitting (NBTI/PBTI/HCI)
```python
# ΔVth = A * t^n
# Log-log fit: log(ΔVth) = log(A) + n * log(t)
from scipy.optimize import curve_fit
def power_law(t, A, n):
    return A * t**n
popt, pcov = curve_fit(power_law, time_array, delta_vth_array)
```
- **Time exponent (n):** Typical ~0.15-0.25 for NBTI, ~0.5 for interface trap generation
- Unusual exponents may indicate recovery artifacts or measurement issues

### 5. Lifetime Projection
```python
# Extrapolate to use-condition voltage/temperature
# Using acceleration factors:
# Voltage acceleration: AF_V = exp(gamma * (V_stress - V_use))
# Temperature acceleration: AF_T = exp(Ea/k * (1/T_use - 1/T_stress))
# Lifetime at use condition = measured_time * AF_V * AF_T
```

## TDDB / EM Analysis

For time-to-failure distributions:
- Use Weibull distribution (TDDB) or lognormal (EM)
- Plot on Weibull paper (ln(-ln(1-F)) vs. ln(t))
- Extract shape parameter (β) and scale parameter (η)
- Project to failure rate at use conditions

## Red Flags

| Red Flag | What It Means |
|----------|--------------|
| Non-monotonic degradation | Recovery during measurement, or measurement artifact |
| Time exponent > 0.5 | Unusual mechanism, check for stress condition issues |
| Large device-to-device spread | Process variation issue, or contact/measurement problem |
| Sudden parameter jump | Possible hard breakdown or contact failure |
| Baseline drift in unstressed monitor | Measurement setup issue — recalibrate |

## Output

Report should include:
- Fresh parameter summary (mean, sigma, range)
- Degradation trend plots (log-log)
- Fitted power law parameters (A, n) with confidence intervals
- Projected lifetime at use conditions (if enough data points)
- Pass/fail vs. spec limits
- Any anomalies flagged
