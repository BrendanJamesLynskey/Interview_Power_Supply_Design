# Worked Problem 01: Type II Compensator Design for a Buck Converter

## Problem Statement

Design a Type II compensator for the following current-mode controlled buck converter. Target a gain crossover frequency of 10 kHz with at least 60° of phase margin.

**Converter Specifications:**
```
Vin  = 12 V
Vout = 3.3 V
Iout = 3 A (full load)
fsw  = 200 kHz
```

**Plant transfer function (current-mode control, simplified):**
```
Gplant(s) = (R_load / (1 + s × R_load × C)) × (Rs / L_eff)
```

**Component values:**
```
L   = 10 µH  (inductor)
C   = 100 µF (output capacitor, ESR = 80 mΩ)
Rs  = 10 mΩ  (current sense resistance)
R_load = Vout / Iout = 3.3 / 3 = 1.1 Ω  (full load)
```

**PWM modulator:**
```
Ramp amplitude Vm = 0.5 V
PWM gain:  GPWM = 1 / Vm = 2 V⁻¹  (= +6 dB)
```

**Feedback divider:**
```
H = Vref / Vout = 0.8 V / 3.3 V = 0.242  (= -12.3 dB)
```

---

## Step 1: Simplify and Analyse the Plant

**Current-mode plant (CCM, simplified to first-order):**

In current-mode control, the inner current loop converts the duty cycle input into an inductor current. The outer voltage loop sees an effective first-order plant:
```
Gplant(s) = Vout_dc / (1 + s × R_load × C)

DC gain = Vout_dc = R_load = 1.1 Ω  [referred to error amplifier input via Rs]
```

More precisely, including ESR zero:
```
Gplant(s) = R_load × (1 + s × ESR × C) / (1 + s × R_load × C)

Dominant output pole:  fp = 1 / (2π × R_load × C)
                           = 1 / (2π × 1.1 × 100µF)
                           = 1448 Hz ≈ 1.45 kHz

ESR zero:              fz_esr = 1 / (2π × ESR × C)
                              = 1 / (2π × 80mΩ × 100µF)
                              = 19.9 kHz ≈ 20 kHz
```

**Bode plot of plant (approximate):**
```
Frequency range      Gain slope      Phase
DC to 1.45 kHz:      flat            ≈ 0°
1.45 kHz to 20 kHz:  -20 dB/dec      -45° to -84°
Above 20 kHz:        flat (0 dB/dec) recovering toward 0°

DC gain: 20 × log(R_load) = 20 × log(1.1) = 0.83 dB ≈ 0 dB (approximated)
At fc = 10 kHz:
|Gplant(j·2π·10kHz)| = R_load × √(1 + (10k/20k)²) / √(1 + (10k/1.45k)²)
                      = 1.1 × 1.118 / √(1 + 47.6)
                      = 1.230 / 6.97
                      = 0.176 = -15.1 dB

Phase at fc = 10 kHz (pole contribution):
φ_pole = -arctan(10k/1.45k) = -arctan(6.9) = -81.8°

Phase at fc = 10 kHz (ESR zero contribution):
φ_ESR_zero = +arctan(10k/20k) = arctan(0.5) = +26.6°

Net plant phase at 10 kHz: -81.8° + 26.6° = -55.2°
```

---

## Step 2: Determine Total Phase Available

**Loop gain phase budget:**

To achieve PM = 60° at fc = 10 kHz:
```
Required loop phase at 10 kHz = -(180° - PM) = -(180° - 60°) = -120°
```

**Phase contributions at 10 kHz (other than compensator):**
```
Plant phase:  -55.2°
PWM modulator phase:  ≈ 0° (assumed negligible delay in analog PWM)
Feedback divider:  0° (purely resistive)

Total non-compensator phase at 10 kHz:  -55.2°
```

**Required compensator phase at 10 kHz:**
```
φ_comp = Required total phase - non-compensator phase
       = -120° - (-55.2°)
       = -120° + 55.2°
       = -64.8°
```

A Type II compensator has a pole at the origin (-90° contribution from integrator) and a zero (up to +90° contribution). Net phase from Type II:
```
φ_TypeII = -90° + arctan(fc/fz) - arctan(fc/fp)
```

Setting φ_TypeII = -64.8°:
```
-64.8° = -90° + arctan(10k/fz) - arctan(10k/fp)
arctan(10k/fz) - arctan(10k/fp) = 25.2°
```

This gives us the required phase boost from the zero and HF pole: 25.2°.

---

## Step 3: Place the Compensator Zero

**Guideline:** Place the zero at or below fc to add phase lead at the crossover. A common choice is fz = fc / 3 to fc / 5.

**Choice:** Place zero at fz = 3 kHz:
```
Phase from zero at 10 kHz: arctan(10k/3k) = arctan(3.33) = 73.3°
```

**Place HF pole:** The HF pole should be at or above fc to avoid reducing phase margin. Typical placement: at the ESR zero (to cancel its gain contribution above fc) or at fsw/2.

**Choice:** Place HF pole at fp = 20 kHz (at f_ESR_zero):
```
Phase from HF pole at 10 kHz: -arctan(10k/20k) = -arctan(0.5) = -26.6°
```

**Verify total compensator phase:**
```
φ_comp = -90° + 73.3° - 26.6° = -43.3°
```

**Verify total loop phase:**
```
Total phase at 10 kHz = φ_plant + φ_comp = -55.2° + (-43.3°) = -98.5°
PM = 180° - 98.5° = 81.5° > 60° ✓  (exceeds requirement)
```

---

## Step 4: Set Compensator Gain for 0 dB Loop Gain at 10 kHz

**Total loop gain must equal 0 dB at fc = 10 kHz:**
```
|T(j·2π·10kHz)| = |Gcomp| × |GPWM| × |Gplant| × |H| = 1
```

**Calculate each factor at 10 kHz:**

Plant gain: -15.1 dB → |Gplant| = 0.176

At 10 kHz, with zero at 3 kHz and pole at 20 kHz, the Type II compensator mid-band gain is:
```
|Gcomp(j·2π·10kHz)| = K × |1 + j·10k/3k| / (|j·10k/fp| × |1 + j·10k/20k|)
```

At mid-band between fz and fp, the gain is approximately:
```
|Gcomp|_mid = K / (2π × fz)  [for a simple integrator + zero + HF pole]
```

More precisely, at 10 kHz:
```
|Gcomp(j·2π·10kHz)| = K_dc × √(1 + (10k/3k)²) / ((10k/fz) × √(1 + (10k/20k)²))

Let K_dc = K for the integrator gain constant.
= K × 3k/10k × √(1 + 11.1) / √(1 + 0.25)
= K × 0.3 × √(12.1) / √(1.25)
= K × 0.3 × 3.478 / 1.118
= K × 0.3 × 3.112
= K × 0.934
```

**Required compensator gain at 10 kHz:**
```
|Gcomp| × |GPWM| × |Gplant| × |H| = 1

|Gcomp| = 1 / (|GPWM| × |Gplant| × |H|)
         = 1 / (2 × 0.176 × 0.242)
         = 1 / 0.0852
         = 11.7  (= +21.4 dB)
```

**Solving for K:**
```
K × 0.934 = 11.7
K = 12.5
```

---

## Step 5: Calculate Component Values

The Type II compensator is implemented with a standard op-amp topology:

```
         R2
   ┌─────┤├─────┐
   │      C2    │
   │             │
V_e─┤R1          ├──── V_comp (to PWM)
   │             │
   └──────┬──────┘
          │
         C1 (from inverting input to output — forms zero with R2)
          │
         GND? [depends on topology; see below]
```

**Standard Type II topology (inverting op-amp):**
```
Input resistor:  R1
Feedback network: R2 in series with C1 (zero), then C2 in parallel (HF pole)

Transfer function: Gcomp(s) = -(R2/R1) × (1 + s·R1·C1) / (1 + s·R2·C2)

Wait — this is a different topology. Let us use the conventional version:

Gcomp(s) = -(1/s·R1·C1) × (1 + s·R2·C1) / (1 + s·R2·C2_par)

where C2_par = C2 in parallel with C1 (≈ C2 if C2 << C1)
```

**Using the standard two-capacitor Type II:**

```
Gcomp(s) = -(Zf / Zi)

Zi = R1   (input resistor)
Zf = R2 + (C1 in series with R2) in ...
```

Let us use the most common practical form. Two components set the zero and one the HF pole:

**Zero frequency:** fz = 1 / (2π × R2 × C1)
**HF pole frequency:** fp = 1 / (2π × R2 × C2)
**DC gain (integrator):** determined by R1, C1

Choose R1 = 10 kΩ (standard input resistor).

**Calculate C1 from zero frequency:**
```
fz = 1 / (2π × R2 × C1)   → need R2 first

Calculate R2 from mid-band gain:
Mid-band gain = R2 / R1 = K ≈ 12.5 (for the integrator + zero + pole topology)
Actually for the inverting integrator topology:
  Mid-band gain ≈ (1/(2π × fz × C1)) / R1 = R2 / R1

So R2 = 12.5 × R1 = 12.5 × 10kΩ = 125 kΩ → choose R2 = 130 kΩ (nearest E24; slightly more gain)
```

**Calculate C1:**
```
fz = 1 / (2π × R2 × C1)
C1 = 1 / (2π × fz × R2)
   = 1 / (2π × 3000 × 130kΩ)
   = 1 / (2.45 × 10⁹)
   = 408 pF → choose C1 = 390 pF (standard E12 value)
```

**Verify zero frequency with chosen C1:**
```
fz_actual = 1 / (2π × 130kΩ × 390pF) = 3.14 kHz  (close to target 3 kHz ✓)
```

**Calculate C2 from HF pole frequency:**
```
fp = 1 / (2π × R2 × C2)
C2 = 1 / (2π × fp × R2)
   = 1 / (2π × 20kHz × 130kΩ)
   = 1 / (1.634 × 10¹⁰)
   = 61.2 pF → choose C2 = 56 pF (standard E12 value)
```

**Verify HF pole with chosen C2:**
```
fp_actual = 1 / (2π × 130kΩ × 56pF) = 21.9 kHz (close to target 20 kHz ✓)
```

---

## Step 6: Component Summary

| Component | Value       | Purpose                     |
|-----------|-------------|-----------------------------|
| R1        | 10 kΩ       | Input resistor (sets gain)  |
| R2        | 130 kΩ      | Feedback resistor           |
| C1        | 390 pF      | Zero with R2 at 3.14 kHz    |
| C2        | 56 pF       | HF pole with R2 at 21.9 kHz |

---

## Step 7: Verification Summary

Recalculate loop gain at fc = 10 kHz with actual component values:

**Compensator gain at 10 kHz (with actual fz=3.14kHz, fp=21.9kHz):**
```
|Gcomp(10kHz)| ≈ (R2/R1) × √(1+(10k/3.14k)²) / (10k/3.14k × √(1+(10k/21.9k)²))
              = 13 × √(1+10.14) / (3.185 × √(1+0.208))
              = 13 × √11.14 / (3.185 × √1.208)
              = 13 × 3.338 / (3.185 × 1.099)
              = 43.39 / 3.50
              = 12.4  (= +21.9 dB)
```

**Total loop gain at 10 kHz:**
```
|T(10kHz)| = 12.4 × 2 × 0.176 × 0.242
           = 12.4 × 0.0852
           = 1.06 ≈ 0 dB ✓  (within rounding error of component standard values)
```

**Phase margin verification:**
```
Plant phase at 10 kHz:     -55.2°
Compensator phase:         -90° + arctan(10k/3.14k) - arctan(10k/21.9k)
                         = -90° + arctan(3.18) - arctan(0.457)
                         = -90° + 72.5° - 24.6°
                         = -42.1°

Total loop phase at 10 kHz:  -55.2° + (-42.1°) = -97.3°
Phase margin:  PM = 180° - 97.3° = 82.7°

Achieved PM = 82.7° >> 60° requirement ✓
```

**Gain margin verification:**
To find GM, locate the phase crossover frequency (where total phase = -180°):

At higher frequencies, plant phase approaches -90° + 90° (ESR zero) = 0° (asymptotic).
Compensator phase approaches -90° + 90° - 90° = -90° (asymptotic).
Total asymptotic phase = -90° — in this simplified model the phase never reaches -180°.

Since the total phase approaches but may not cross -180° in this configuration (beneficial ESR zero), the gain margin is effectively infinite or very large. The gain must drop below 0 dB before phase reaches -180°.

In practice, verify using simulation software (LTspice, Python) to account for parasitic effects and component tolerances.

---

## Step 8: Simulation Checklist

Before building hardware, verify in LTspice or MATLAB:

1. Open the compensator schematic and measure its Bode plot separately — verify fz = 3.14 kHz and fp = 21.9 kHz are correct.

2. Include the full plant model (not the simplified first-order): use the actual buck converter averaged model with:
   - L = 10 µH, R_L = 50 mΩ (inductor DCR)
   - C = 100 µF, ESR = 80 mΩ
   - Full CCM current-mode model

3. Measure loop gain T(s) by breaking the loop and injecting a perturbation. Confirm:
   - fc ≈ 10 kHz
   - PM ≥ 60°
   - GM ≥ 10 dB

4. Run a load step transient: 0 → 3A step with the loop closed. Confirm:
   - Vout deviation < ΔV_spec
   - Settling time < t_settle_spec
   - No sustained oscillation

---

## Common Mistakes to Avoid

1. **Forgetting the feedback divider H:** The loop gain includes H = Vref/Vout. Omitting it underestimates the compensator gain needed, resulting in under-compensation.

2. **Using nominal ESR at room temperature:** ESR of electrolytic capacitors varies 3:1 from -40°C to +85°C. The ESR zero frequency shifts proportionally. Verify PM across temperature.

3. **Not checking gain margin:** A large PM does not guarantee large GM. If the ESR zero causes the loop gain to rise again above fc before falling below 0 dB at the phase crossover, GM may be small.

4. **Ignoring optocoupler if isolated:** An isolated flyback with this Type II compensation on the secondary must account for the optocoupler pole. Verify the opto pole is above 3×fc or add a zero to compensate it.

5. **Standard component value round-off:** After choosing E12/E24 standard values, always recompute fz, fp, and gain. Small deviations (5–10%) are acceptable; verify PM does not drop below 45°.
