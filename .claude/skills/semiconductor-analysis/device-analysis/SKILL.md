# Sub-Skill #3: Device Analysis

> **Load when the task involves extracting device parameters from IV/CV measurement data.**

## Scope

Parameter extraction and analysis from electrical measurement data: Id-Vg, Id-Vd, CV, and related sweeps.

## Threshold Voltage (Vth) Extraction

### Method 1: Constant Current Method
Most common in production. Define a reference current, find the Vg where Id crosses it.
```python
# Typical reference: Id = 100nA * W/L (for NMOS)
vth_ref_current = 100e-9 * W / L
# Interpolate Vg at that current
from scipy.interpolate import interp1d
f = interp1d(id_array, vg_array)
vth = f(vth_ref_current)
```

### Method 2: Linear Extrapolation
Fit the linear region of Id-Vg (in linear regime), extrapolate to Id=0.
```python
# Find max gm point, fit line around it
gm = np.gradient(id_array, vg_array)
max_gm_idx = np.argmax(gm)
# Linear fit around max gm
```

### Method 3: Max-gm (Transconductance) Method
Vth = Vg at peak transconductance minus (Id_at_peak / gm_peak).

**Choose method based on:** team convention, spec document, or measurement context. When in doubt, ask the user.

## IV Curve Analysis

### Id-Vg Analysis
- **Subthreshold slope (SS):** Extract from log(Id) vs. Vg in subthreshold region. Ideal = 60mV/dec at 25C.
- **Ion/Ioff ratio:** On-current at Vg=Vdd vs. off-current at Vg=0 (NMOS) or Vg=Vdd (PMOS)
- **DIBL:** Vth difference between low-Vd and high-Vd sweeps. DIBL = (Vth_lin - Vth_sat) / (Vd_sat - Vd_lin)
- **Gate leakage (Ig):** Check Ig column if present — should be orders of magnitude below Id

### Id-Vd (Output Characteristics)
- **Saturation behavior:** Does Id flatten at high Vd?
- **Output resistance (Ro):** Slope of Id-Vd in saturation = 1/Ro
- **Self-heating:** Id droop at high Vd*Id (power). Look for negative slope in saturation.
- **Breakdown:** Sudden Id increase at high Vd

## CV Analysis

- **Cox:** Maximum capacitance in accumulation
- **Flatband voltage (Vfb):** Inflection point in C-V curve
- **Dit (interface trap density):** From conductance method or high-low frequency comparison
- **EOT (equivalent oxide thickness):** From Cox = epsilon_0 * epsilon_ox * A / EOT

## Common Pitfalls

| Pitfall | How to Detect | What to Do |
|---------|--------------|------------|
| Contact resistance | Id-Vd doesn't pass through origin | Flag, may need 4-point measurement |
| Self-heating | Id drops at high power | Note in report, consider pulsed measurements |
| Gate leakage masking | Ig comparable to Id in subthreshold | Check Ig separately, flag if > 1% of Id |
| Measurement noise | Jagged curves, especially at low currents | Check compliance settings, may need averaging |
| Wrong sweep direction | Hysteresis between up/down sweeps | Report both, note hysteresis magnitude |

## Output

Extracted parameters should be reported as:
```
Device: NMOS W=1u L=0.1u, Die (5,3), Wafer 03, Lot 2024A
Condition: 25C, Vd=50mV (linear), Vd=1.0V (saturation)
─────────────────────────────────
Vth_lin:    0.342 V  (constant current method)
Vth_sat:    0.318 V
DIBL:       25.3 mV/V
SS:         68.2 mV/dec
Ion:        485 uA/um
Ioff:       12.3 pA/um
Ion/Ioff:   3.9e7
Ig @ Vg=1V: 2.1 pA/um
```
