---
disable-model-invocation: true
description: determining valid comparisons between measurements, normalizing by device geometry, or grouping data by lot/wafer/die conditions.
---

# Sub-Skill #2: Condition Logic & Meaningful Comparisons

> **Load after data ingestion. Determines what conditions exist and which comparisons are valid.**

## Purpose

Raw semiconductor data contains measurements under many different conditions. This skill ensures Claude understands what those conditions mean and only makes comparisons that are physically and statistically meaningful.

## Step 1: Condition Parsing

Extract and classify conditions from the data/filenames:

### Bias Conditions
- Gate voltage (Vg), Drain voltage (Vd), Substrate voltage (Vb)
- Sweep ranges vs. fixed bias points
- Linear region (low Vd) vs. saturation region (high Vd)

### Environmental Conditions
- Temperature: 25C (room temp), -40C (cold), 85C/125C/150C (hot)
- Humidity (for some reliability tests)

### Stress Conditions (Reliability)
- Stress type: NBTI, PBTI, HCI, TDDB, EM
- Stress voltage, stress temperature
- Stress duration / readout intervals (e.g., 0s, 100s, 1000s, 10000s)

### Device Geometry
- Channel width (W) and length (L)
- Number of fingers (nf)
- Device flavor (core, IO, HV, SRAM, etc.)

## Step 2: Valid Comparison Rules

**Only compare like with like.** These rules are non-negotiable:

| Comparison | Requirement |
|-----------|-------------|
| Device A vs. Device B | Same geometry (W/L), same measurement condition |
| Wafer-to-wafer | Same device, same die location, same condition |
| Lot-to-lot | Same device, same wafer position (or average), same condition |
| Pre-stress vs. post-stress | Same device, same die, same measurement condition, different stress time |
| Temperature comparison | Same device, same bias, different temperature |
| Geometry scaling | Same device type, same condition, vary W or L systematically |

### Red Flags — Invalid Comparisons

**Stop and warn the user if:**
- Comparing devices with different W/L without normalization
- Mixing linear-region and saturation-region measurements
- Comparing different die locations without acknowledging within-wafer variation
- Averaging across different device types
- Comparing stress data without a time-zero baseline
- Mixing data from different measurement tools without calibration check

## Step 3: Normalization Logic

### When to Normalize
| Situation | Normalization |
|-----------|--------------|
| Comparing different W | Divide current by W (Id/W) |
| Comparing different W/L | Divide by W/L ratio |
| Capacitance comparison | Normalize by area (C/A) |
| Current density | J = I/A or I/(W*nf) |
| Sheet resistance | Rs = R * W/L |

### When NOT to Normalize
- Same geometry devices — use raw values
- Threshold voltage (Vth) — already geometry-independent
- When tracking absolute spec limits

## Step 4: Grouping Logic

Determine how to aggregate data for meaningful analysis:

```python
# Common grouping patterns:
# By lot → wafer → die → device → condition (full hierarchy)
df.groupby(['lot', 'wafer', 'die', 'device', 'condition'])

# For wafer maps:
df.groupby(['wafer', 'die_x', 'die_y'])

# For lot-level summary:
df.groupby(['lot', 'device']).agg(['mean', 'std', 'min', 'max'])

# For reliability trends:
df.groupby(['device', 'die', 'stress_time'])
```

## Step 5: Baseline Identification

For any comparison, establish the reference:
- **Process monitoring:** Use spec target or historical mean as baseline
- **Reliability:** Time-zero (t0) measurement is always the baseline
- **Lot comparison:** Use a known-good reference lot
- **Split lot experiments:** Use the control wafer as baseline

## Output

After this sub-skill, you should know:
1. What conditions exist in the dataset
2. Which comparisons are valid (and which are not)
3. How to normalize (if needed)
4. How to group the data
5. What the baseline/reference is

**Next:** Route to the appropriate analysis sub-skill (`device-analysis`, `reliability`, or `statistical`).
