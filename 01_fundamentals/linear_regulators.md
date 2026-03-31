# Linear Regulators — Interview Preparation

## Overview

Linear regulators are the simplest form of voltage regulation. A pass element (transistor) placed in series between input and output is controlled by a feedback loop to maintain a constant output voltage regardless of line and load variations. Their simplicity, ultra-low noise, and fast transient response make them indispensable in mixed-signal, RF, and precision applications despite their inherent power inefficiency.

---

## Key Concepts Reference

### Dropout Voltage
The minimum differential (Vin - Vout) at which the regulator maintains regulation:
- Standard NPN: ~2 V (Vce_sat + Vbe overhead)
- PMOS LDO: as low as 100–300 mV (limited by Rds_on × Iout)

### Efficiency
```
Efficiency = (Vout × Iout) / (Vin × Iin) ≈ Vout / Vin  (ignoring Iq)
```
Power dissipated as heat: `P_diss = (Vin - Vout) × Iout + Vin × Iq`

### PSRR (Power Supply Rejection Ratio)
```
PSRR(dB) = 20 × log10(ΔVin / ΔVout)
```
Higher PSRR = better rejection. Degrades with frequency, typically 6 dB/decade above a few kHz.

### Pass Element Comparison

| Type  | Vdropout        | Gate/Base drive            | Notes                        |
|-------|-----------------|----------------------------|------------------------------|
| NPN   | ~2 V            | Base above Vout (bootstrap) | Older standard regulators    |
| PNP   | ~0.5–1 V        | Base pulled to GND          | Simple drive; moderate Vdo   |
| NMOS  | ~1–2 V          | Gate boosted above Vout     | Low Rds_on; needs charge pump|
| PMOS  | ~0.1–0.5 V      | Gate pulled to GND          | Dominant in modern LDOs      |

### Output Voltage Setting
```
Vout = Vref × (1 + R1/R2)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Explain how a PMOS LDO regulator maintains a constant output voltage.

**Answer:**

A PMOS LDO uses a negative feedback loop with three main elements:

1. **Feedback network (R1, R2)**: Resistor divider samples Vout and produces a divided voltage at the error amplifier's inverting input.
2. **Error amplifier**: Compares the divided output against an internal bandgap reference (typically 0.8–1.25 V). The difference (error) drives the gate of the PMOS pass element.
3. **PMOS pass element**: Acts as a voltage-controlled current source in series with the load. Reducing the gate voltage (Vg) increases |Vgs| and increases conduction; raising Vg reduces conduction.

**Negative feedback action**:
- If Vout rises → Vfb rises → error amp output rises → PMOS gate rises → |Vgs| decreases → less current → Vout falls back.
- The loop drives the error to zero in steady state.

**Common mistake**: Candidates reverse the direction of PMOS control. For PMOS: source is at Vout (top rail), gate is driven low to turn on. Pulling the gate UP reduces |Vgs| and reduces conduction. The error amplifier's output directly controls gate voltage.

---

### Q2. What is dropout voltage and why does it matter in battery-powered applications?

**Answer:**

Dropout voltage (Vdo) is the minimum (Vin - Vout) at which the pass element can maintain regulation. Below this threshold, the pass element is fully saturated and the output is simply Vin minus a small saturation drop — the regulator has lost control.

**For a PMOS LDO:**
```
Vdo = Iout × Rds_on(PMOS)
```
Vdo increases with output current and with temperature (Rds_on has a positive temperature coefficient).

**Practical impact on batteries:**

A Li-ion cell discharges from ~4.2 V to ~3.0 V. If regulating a 3.3 V output:
- Standard regulator (Vdo = 2 V): needs 5.3 V minimum — cannot operate from this battery at all.
- LDO with Vdo = 300 mV: needs 3.6 V minimum — usable across 4.2 V to 3.6 V (roughly 60–80% of charge capacity).
- Ultra-LDO with Vdo = 100 mV: usable down to 3.4 V — accesses more of the battery.

**Design rule**: Always verify that `Vin_min ≥ Vout + Vdo_max`, where Vdo_max accounts for maximum Iout and worst-case (maximum) Rds_on at the highest operating temperature.

---

### Q3. What is quiescent current (Iq) and how do you estimate standby battery life?

**Answer:**

Quiescent current is the current consumed by the regulator's internal circuits (bandgap reference, error amplifier, bias chains) when no external load is connected. This current flows from Vin to GND through internal paths, not through the load.

**Total input current:**
```
Iin = Iout + Iq
```

**Standby battery life estimate:**
```
Battery life (hours) = Battery capacity (mAh) / Iq (mA)
```

**Example**: 1 µA Iq LDO with a 200 mAh coin cell:
```
Life = 200 mAh / 0.001 mA = 200,000 hours ≈ 22 years
```

**Trade-off**: Achieving ultra-low Iq (< 1 µA) requires slow error amplifier bias, which degrades:
- Transient response speed
- PSRR at mid-to-high frequencies
- Load regulation at fast load steps

**Practical implication**: For IoT devices sleeping for long intervals, Iq dominates total energy consumption. The LDO Iq may exceed the microcontroller's sleep current, making it the primary target for optimisation.

---

### Q4. Explain PSRR and how it varies with frequency. Why does it matter when an LDO follows a switching regulator?

**Answer:**

PSRR quantifies how well the LDO attenuates noise/ripple appearing at its input from reaching the output.

```
PSRR(dB) = 20 × log10(ΔVin / ΔVout)
```

A PSRR of 60 dB means 100 mV of input ripple produces only 0.1 mV at the output.

**Frequency behaviour:**

- **DC to ~1 kHz**: PSRR dominated by open-loop gain of the error amplifier. Typically 60–80 dB.
- **1 kHz to ~100 kHz**: PSRR rolls off at ~20 dB/decade as the error amplifier gain rolls off. The loop can no longer correct fast disturbances.
- **Above 100 kHz to 1 MHz**: PSRR may be only 20–40 dB. The output capacitor's impedance relative to the load becomes the dominant rejection mechanism.

**When feeding from a switching regulator (pre-regulator architecture):**

The buck converter produces switching ripple at fsw (e.g., 500 kHz). The LDO must reject this ripple. If the LDO has only 30 dB PSRR at 500 kHz, and the buck produces 50 mV ripple:
```
ΔVout = 50 mV / 10^(30/20) = 50 mV / 31.6 = 1.6 mV
```
This may be acceptable for digital loads but unacceptable for RF PLLs (which may require < 10 µV ripple). In that case, either increase buck's output filter or choose an LDO with better high-frequency PSRR.

---

### Q5. Calculate the power dissipation and junction temperature for the following LDO application.

**Given**: Vin = 5 V, Vout = 3.3 V, Iout = 500 mA, Iq = 2 mA, package Rth_ja = 50°C/W (SOT-223), Ta = 25°C. Is this thermally safe?

**Answer:**

**Power dissipation:**
```
P_diss = (Vin - Vout) × Iout + Vin × Iq
P_diss = (5 - 3.3) × 0.5 + 5 × 0.002
P_diss = 1.7 × 0.5 + 0.01
P_diss = 0.85 + 0.01 = 0.86 W
```

**Junction temperature:**
```
Tj = Ta + P_diss × Rth_ja
Tj = 25 + 0.86 × 50 = 25 + 43 = 68°C
```

**Assessment**: Safe — well below the typical 125°C limit with 57°C of margin. However, if Vin were raised to 12 V:
```
P_diss = (12 - 3.3) × 0.5 = 4.35 W
Tj = 25 + 4.35 × 50 = 242.5°C  ← far beyond safe limit
```
At 12 V input, this application would require a heatsink (reducing Rth to ~3°C/W), a larger package (TO-220), or a switching pre-regulator to reduce the voltage headroom dissipated in the LDO.

**Common mistake**: Using Rth_jc instead of Rth_ja. For a device without a heatsink, Rth_ja (junction to ambient, including PCB spreading) governs. With a heatsink: Rth = Rth_jc + Rth_cs + Rth_sa.

---

### Q6. What is the difference between line regulation and load regulation? Give typical specifications.

**Answer:**

**Line regulation**: Change in Vout per unit change in Vin, with load current constant.
```
Line Reg = ΔVout / ΔVin  [mV/V]
```
Caused by finite open-loop gain — the error amplifier cannot perfectly reject input variation.

**Load regulation**: Change in Vout per unit change in Iout, with Vin constant.
```
Load Reg = ΔVout / ΔIout  [mV/A or mΩ]
```
Caused by finite loop bandwidth and the effective output impedance of the regulator.

**Expressed as percentage:**
```
Load Reg (%) = (Vout_noload - Vout_fullload) / Vout_nominal × 100
```

**Typical specifications:**
- Good line regulation: < 1 mV/V
- Good load regulation: < 5 mV/A
- Precision LDOs: < 0.5 mV/V line, < 2 mV/A load

**Example**: An LDO with 2 mV/A load regulation producing 3.3 V at 0–1 A load will vary:
```
ΔVout = 2 mV/A × 1 A = 2 mV  → Vout ranges from 3.300 V to 3.302 V
```
This is 0.06% variation — excellent for most applications.

---

## Intermediate (Questions 7–13)

---

### Q7. Why do some LDOs require a minimum output capacitor ESR, and what happens if you use a low-ESR ceramic capacitor with them?

**Answer:**

A classical PMOS LDO has two significant poles in its control loop:

1. **Dominant pole (p1)**: At the error amplifier output / PMOS gate node, set by the amplifier's output impedance and the gate capacitance. Typically 1–10 kHz.
2. **Output pole (p2)**: At the output node: `fp2 = 1/(2π × Rload × Cout)`. Varies with load from ~1 kHz (full load) to lower frequencies at light load.

The output capacitor's ESR introduces a zero:
```
fz_ESR = 1 / (2π × ESR × Cout)
```

This zero adds phase boost before the unity-gain crossover, maintaining adequate phase margin (typically > 45°).

**With a low-ESR ceramic capacitor (ESR < 5 mΩ)**:
- The ESR zero moves to extremely high frequency (beyond 10 MHz)
- No phase boost is provided near crossover
- Both p1 and p2 contribute phase lag
- If the total phase lag exceeds 180° before gain drops below 0 dB, the loop oscillates
- Symptom: high-frequency output oscillation, often at 100 kHz–2 MHz

**Solution options:**
1. Use the specific capacitor type/value recommended in the datasheet
2. Choose a "ceramic-stable" LDO (internal compensation provides the zero)
3. Add a small series resistor (10–100 mΩ) in series with a ceramic capacitor to create a controlled ESR zero

---

### Q8. Explain the operation and stability implications of the output pole varying with load current.

**Answer:**

The output pole frequency is:
```
fp2 = 1 / (2π × Rload × Cout)
Rload = Vout / Iout
```

**At heavy load (Iout = 1 A, Vout = 3.3 V):**
```
Rload = 3.3 Ω
fp2 = 1 / (2π × 3.3 × 100µF) = 482 Hz
```

**At light load (Iout = 1 mA):**
```
Rload = 3300 Ω
fp2 = 1 / (2π × 3300 × 100µF) = 0.48 Hz
```

The output pole moves three decades in frequency across the load range.

**Stability implications:**
- **Heavy load**: fp2 is at a low frequency, but the loop bandwidth (limited by p1) is typically well above fp2, so the two-pole system has limited phase margin. The ESR zero must provide phase boost.
- **Light load**: fp2 moves very low, below the loop crossover. With two poles below crossover and no zero (or an insufficient zero), phase margin collapses.

**Worst-case condition**: Minimum load (or no load). Always verify stability with maximum output capacitor, minimum ESR, and zero load.

**LDO designs that address this:**
- Pole-splitting compensation: pushes p1 to even lower frequency so crossover is below fp2 at all loads
- Adaptive biasing: increases amplifier bandwidth with load current
- Internal zero network: provides phase boost independent of external ESR

---

### Q9. How do you select the feedback resistor values? What are the trade-offs?

**Answer:**

**Setting the output voltage:**
```
Vout = Vref × (1 + R1/R2)
```
Where R1 is from Vout to the feedback pin, R2 is from the feedback pin to GND.

**Divider current (Idiv):**
```
Idiv = Vout / (R1 + R2) = Vref / R2
```

**Trade-offs in resistor value selection:**

| Consideration                | Smaller R values             | Larger R values              |
|------------------------------|------------------------------|------------------------------|
| Divider quiescent current    | Higher (wastes battery power)| Lower (better for low-Iq)   |
| Bias current error           | Smaller effect               | Larger error: ΔVout = Ibias×R|
| Johnson noise                | Lower                        | Higher                       |
| Accuracy vs component cost   | Requires tight tolerance     | Same, but noise dominates    |

**Design rule:**
```
Idiv >> Ibias_error_amp  (typically 10–100× to minimise bias current error)
```

If Ibias = 10 nA and we want < 0.1% error on 3.3 V (= 3.3 mV):
```
ΔVout = Ibias × (R1 + R2) < 3.3 mV
R1 + R2 < 3.3 mV / 10 nA = 330 kΩ
```

**Typical practical range**: 10 kΩ–500 kΩ, with preference for 100 kΩ range for general-purpose LDOs.

---

### Q10. Describe the noise output of an LDO and the most effective technique to reduce it.

**Answer:**

**Noise sources in order of importance:**

1. **Bandgap reference noise**: Dominates. Generates both 1/f (flicker) noise below ~1–10 kHz and thermal noise above. The error amplifier amplifies this by the noise gain `(1 + R1/R2)`.
2. **Error amplifier input-referred noise**: Multiplied by `(1 + R1/R2)`.
3. **Resistor Johnson noise**: Thermal noise of R1, R2: `vn = √(4kTRΔf)`.
4. **Pass transistor noise**: Usually small contribution.

**Output noise:**
```
Vnoise_out ≈ √[(Vnoise_ref × (1+R1/R2))² + (Vn_amp × (1+R1/R2))² + Vn_R1R2²]
```

**Most effective reduction technique — Noise Bypass Capacitor:**

Most LDOs expose a NOISE or BYP pin connected to the internal reference. A small capacitor (10 nF–100 nF) from this pin to GND rolls off the reference noise at:
```
froll-off = 1 / (2π × Rref_internal × Cbyp)
```

This single capacitor can reduce output noise by 10–20 dB across the critical 100 Hz–100 kHz band. It is the highest-impact, lowest-cost noise reduction technique available.

**Secondary techniques:**
- Use a lower output voltage setpoint (reduces R1/R2 ratio, reduces noise gain)
- Add an LC post-filter (adds impedance — use only if source impedance is acceptable)
- Select an LDO specified for low noise (e.g., 10 nV/√Hz input-referred reference noise)

---

### Q11. When should you choose a switching regulator vs an LDO, and when does a hybrid architecture make sense?

**Answer:**

**Choose an LDO when:**
- Vin - Vout is small (< 1 V): efficiency penalty is minor; e.g., 3.3/3.6 V = 91.7% vs switching regulator ~92% — comparable.
- Output noise is critical: RF, PLLs, ADC references, audio DACs require µV-level noise.
- Fast transient response is needed: no inductor means no bandwidth limitation from LC filter.
- Simplicity/cost: one IC, two resistors, one capacitor.
- EMI must be zero: linear regulators produce no switching noise.

**Choose a switching regulator when:**
- Large Vin-Vout differential: efficiency of 5 V → 1.2 V LDO is only 24%; a buck at 90% uses 3.75× less energy.
- High current: at 3 A with 5 V input drop, P_diss = 15 W — requires large heatsink or forces switching.
- Voltage inversion or step-up: boost, inverting, SEPIC are impossible with a linear.
- Battery life: every percentage point of efficiency directly extends runtime.

**Hybrid architecture** (switching pre-regulator + LDO):
```
Vin → [Buck: 12 V → 3.6 V] → [LDO: 3.6 V → 3.3 V, Vdo = 0.3 V] → Clean Vout
```

- Buck efficiency: ~90%, LDO efficiency: 3.3/3.6 = 91.7%, total: ~82.5%
- Compare to LDO alone from 12 V: 3.3/12 = 27.5%
- The LDO rejects the buck's switching ripple

**Binding constraint of hybrid**: LDO PSRR at the buck's switching frequency must be sufficient. Verify at fsw, not just DC.

---

### Q12. How does temperature affect LDO output accuracy? Explain the bandgap reference.

**Answer:**

**Temperature effects on LDO output accuracy:**

1. **Reference voltage temperature drift**: A well-designed bandgap reference is largely compensated, but residual tempco remains: typically 10–100 ppm/°C.
2. **Feedback resistor tempco**: Standard thick-film resistors: ±100–200 ppm/°C. If R1 and R2 have different tempcos, the ratio R1/R2 drifts.
3. **Error amplifier offset drift**: Differential pair mismatch causes Vos drift with temperature, directly adding to Vout error.
4. **PMOS Rds_on drift**: Dropout voltage increases ~0.5–1% per °C, limiting regulation headroom at high temperature.

**Bandgap reference operation:**

Silicon BJTs exhibit two complementary temperature behaviours:
- **Vbe**: Approximately −2 mV/°C (Complementary To Absolute Temperature — CTAT)
- **ΔVBE** = (kT/q) × ln(n): Positive temperature coefficient (Proportional To Absolute Temperature — PTAT)

By summing: `Vref = Vbe + K × ΔVBE`

Choosing K so the PTAT and CTAT components cancel at first order:
```
Vref ≈ 1.25 V  (silicon bandgap voltage — fundamental physics, not a design choice)
```

**Higher-order correction (curvature correction)**: Even with first-order cancellation, a parabolic term remains (Vbe is not perfectly linear in T). High-precision references add curvature correction to achieve < 5 ppm/°C.

---

### Q13. Explain the concept of conditional stability in LDO designs and why it rarely appears in linear regulators.

**Answer:**

**Conditional stability** occurs when a feedback system is stable with its nominal loop gain but becomes unstable if the loop gain is reduced (e.g., during startup, fault conditions, or gain rolloff at high temperature). This happens when the phase plot crosses −180° at a frequency where the gain is below 0 dB — but then crosses back above −180° at a lower frequency.

**Why rare in LDOs:**

Most LDO loops are designed as two-pole (p1 dominant, p2 output), occasionally with one zero (ESR zero). The phase characteristic of such a system:
- Starts at 0° phase shift at DC
- Rolls through −90° near p1
- Approaches −180° as p2 takes effect
- The ESR zero prevents it from reaching −180° before crossover

This monotonic phase rolloff is a **unconditionally stable** Bode characteristic. As loop gain decreases (e.g., light load shifts p2, temperature effects), the phase margin typically stays positive.

**When conditional stability could appear:**
- If a Type II or Type III compensator is used inside the LDO loop (multiple zeros and poles)
- If an external filter creates additional phase lag at high frequency
- If a large capacitive load adds a high-frequency pole

**Practical implication**: Unconditional stability makes LDOs robust in the field — they tolerate capacitive loads, varying ESR, and load transients without oscillating. Switching regulator control loops are more susceptible to conditional stability due to more complex compensation networks.

---

## Advanced (Questions 14–18)

---

### Q14. Derive the small-signal open-loop transfer function of a PMOS LDO and identify all poles and zeros.

**Answer:**

**Circuit elements:**
- Error amplifier: gain Av, single pole at p1 = ωa/(2π), output resistance Roa
- PMOS: transconductance gm_p, gate capacitance Cg, drain-source capacitance Cds
- Load: Rload = Vout/Iout
- Output capacitor: Cout in series with ESR resr
- Feedback divider: attenuation factor β = R2/(R1+R2)

**Output impedance of converter:**
```
Zout(s) = (Rload || Zc) where Zc = resr + 1/(s×Cout)
```

At frequencies where Rload >> Zc:
```
Zout(s) ≈ Rload × (1 + s×resr×Cout) / (1 + s×Rload×Cout)
```

**Open-loop gain T(s):**
```
T(s) = β × Av(s) × gm_p × Zout(s)

where Av(s) = Av0 / (1 + s/ωa)
```

**Complete expression:**
```
T(s) = [β × Av0 × gm_p × Rload] × [(1 + s×resr×Cout)] / [(1 + s/ωa)(1 + s×Rload×Cout)]
```

**Poles:**
- p1: `fp1 = ωa/(2π)` — error amplifier dominant pole, set by internal compensation
- p2: `fp2 = 1/(2π×Rload×Cout)` — output pole, load-dependent

**Zero:**
- z1 (ESR zero): `fz1 = 1/(2π×resr×Cout)`

**DC loop gain:**
```
T0 = β × Av0 × gm_p × Rload
```

Typically T0 >> 1 (60–80 dB), ensuring good regulation accuracy.

**Phase margin** (simplified, assuming p1 << p2 and fz near crossover):
```
PM ≈ 90° - arctan(fcross/fp1_effective) + arctan(fcross/fz1) - arctan(fcross/fp2)
```

---

### Q15. How do you verify LDO stability in the lab using a loop injection method?

**Answer:**

Direct loop gain measurement requires breaking the loop and injecting a test signal — non-trivial for an LDO because breaking the output feedback loop changes the operating point.

**Method 1: Injection at the feedback divider (recommended)**

Insert a small series resistor (10–50 Ω) in the feedback path between Vout and the top of R1 (or between R1 and the feedback pin). Inject a small sinusoidal signal via a bias-T or transformer:

```
[Vin] → [LDO] → [Vout] → [Rinject (10Ω)] → [R1] → [Feedback pin] → ...
                                ↑
                        [Network Analyser Out]
                Measure V at both sides of Rinject
```

Measure V1 (at Vout side) and V2 (at R1 side). Loop gain:
```
T(jω) = V1/V2 - 1  ≈ V1/V2 for |T| >> 1
```

**Practical considerations:**
- Injection resistor must be << Zout to not disturb the operating point
- Use a network analyser (e.g., Bode 100, OMICRON) or an impedance analyser with injection capability
- Load the output with a realistic load — stability varies with load
- Sweep from 10 Hz to 10 MHz
- Read phase margin directly from the Bode plot at the 0 dB crossover frequency

**Method 2: Transient step response (indirect)**

Apply a fast load step (e.g., 10%–90% of rated current using a MOSFET switch). The ringing frequency and damping of the transient response reveal the loop bandwidth and phase margin:
- Critically damped (no overshoot): PM ≈ 70°+
- ~30% overshoot: PM ≈ 45°
- Sustained oscillation: PM ≈ 0° (oscillating)

This method is quick but less precise than direct loop gain measurement.

---

### Q16. What is the significance of the LDO's output impedance, and how does it affect system performance?

**Answer:**

The closed-loop output impedance of an LDO is:
```
Zout_cl(s) = Zout_ol(s) / (1 + T(s))
```

At DC and low frequency where T >> 1:
```
Zout_cl ≈ Zout_ol / T0  (very low — typically milliohms)
```

At high frequency where T → 0 (above loop bandwidth):
```
Zout_cl → Zout_ol = Rload || Zout_pass_element
```

The output impedance rises from milliohms at DC to several ohms or more above the loop bandwidth.

**Practical implications:**

1. **Decoupling capacitor sizing**: Fast load transients (high slew rate) with frequency content above the loop bandwidth see the open-loop output impedance. The output capacitor must supply the instantaneous current:
   ```
   ΔVout_peak = ΔIout × Zout_cl(fload_slew)
   ```

2. **Interaction with downstream loads**: A resonant load (e.g., an LC filter on a downstream circuit) can create a resonance with the LDO output impedance, causing peaking if the output impedance is inductive (phase lagging by 90°).

3. **Load regulation figure**: Load regulation = ΔVout/ΔIout = Zout_cl(0) — the DC output impedance. Lower T0 = higher output impedance = worse load regulation.

4. **Multiple LDOs in parallel**: If two LDOs share a load, their output impedances determine current sharing. Without ballast resistors, tiny Vout differences cause unequal sharing. LDOs are generally not paralleled.

---

### Q17. Describe modern LDO innovations: NMOS LDOs, recycling folded cascode, and adaptively biased architectures.

**Answer:**

**NMOS LDO (flipped voltage follower or source follower LDO):**
- NMOS pass element in source-follower configuration: Vout = Vg - Vgs
- Gate must be driven above Vout — requires an internal charge pump or a regulated supply rail
- Advantage: NMOS mobility is 2–3× higher than PMOS → smaller die area, lower Rds_on for same size
- Better high-frequency PSRR than PMOS because the gate-source voltage directly controls the channel
- Used in point-of-load regulators where a boosted supply is available

**Recycling folded cascode (RFC) LDO:**
- Addresses the quiescent current vs bandwidth trade-off
- Uses current recycling within the amplifier to achieve high slew rate and fast settling with very low static bias current
- The output stage uses only a fraction of the bias current while maintaining full slew capability
- Achieves low Iq (< 50 µA) with microsecond transient response
- Common in mobile/wearable SoC power management ICs

**Adaptively biased LDO:**
- Error amplifier bias current scales with output current or load transient magnitude
- At light load: minimal bias → ultra-low Iq
- During a fast load transient: detects the error rate (dV/dt) and temporarily boosts bias by 10–100×
- Provides near-instantaneous response for the initial transient without sustained high quiescent current
- Implemented with a differentiator circuit on the feedback node that generates transient current pulses

**Capacitor-less LDO:**
- Eliminates the external output capacitor entirely (saves board space)
- Uses a very high loop bandwidth (> 1 MHz) and internal compensation to achieve stability without Cout
- Typically requires a minimum load current or uses the gate capacitance of a subsequent MOSFET load as the stabilising capacitor
- Output impedance is higher (no bulk capacitance), limiting transient performance

---

### Q18. A precision reference application requires the LDO output to be stable within 0.01% across −40°C to +85°C and 3–4 V input. Detail your design process.

**Answer:**

**Step 1: Reference selection**
- Vref tempco must be < 10 ppm/°C to achieve 0.01% (100 ppm) over 125°C range
- Choose an LDO with curvature-corrected bandgap: e.g., TI TPS7A47, Analog Devices ADP7118
- Verify reference section noise specification for the application bandwidth

**Step 2: Dropout budget**
- Vdo must be < 3.0 V - Vout. If Vout = 2.5 V: Vdo budget = 0.5 V at Vin_min = 3 V
- Verify Rds_on × Iout < 0.5 V at Tmax = 85°C (Rds_on increases ~50% from 25°C to 85°C)

**Step 3: Feedback resistor selection**
- Use 0.1% tolerance resistors (vs 0.01% error budget — resistors will be the bottleneck)
- Specify matched tempco: < 5 ppm/°C tracking (use resistors from the same die — dual-matched resistors)
- Minimise value to reduce Johnson noise: 10–50 kΩ range
- Add bypass capacitor on the feedback pin if LDO supports it, to reduce noise

**Step 4: Output capacitor**
- Use C0G (NP0) ceramic: < 30 ppm/°C capacitance drift, no piezoelectric effect
- Verify stability with this capacitor type per datasheet recommendation
- X7R ceramics acceptable for bypass but not for precision applications (±15% variation with temperature and voltage)

**Step 5: PCB layout**
- Kelvin sense connection: connect feedback resistor R2 directly at the output sense point, not at the bulk capacitor
- Separate analog ground from power return paths
- Copper pour under the package for thermal spreading (reduces temperature gradient effects)
- Shield sensitive nodes from switching noise if in a mixed-signal environment

**Step 6: Verification**
- Temperature characterise: measure Vout at −40, 0, 25, 70, 85°C
- Calculate actual tempco: `tempco = (Vout_max - Vout_min) / (Vout_25 × ΔT) × 10^6 ppm/°C`
- Verify: total drift / Vout_25 < 0.01%

---

## Summary: LDO Selection Guide

| Application              | Key Parameter to Prioritise          | Typical LDO Choice           |
|--------------------------|--------------------------------------|------------------------------|
| Battery-powered IoT      | Iq < 1 µA, Vdo < 300 mV             | TI TPS7A02, Microchip MCP1811|
| RF PLL supply            | Noise < 10 µV_rms, PSRR > 60 dB    | TI TPS7A47, ADI ADP151       |
| High-current (> 1 A)     | Low Rds_on, thermal performance      | TI TPS7A57, Renesas ISL80010 |
| Precision reference      | Low tempco, good load regulation     | TI REF50xx (reference IC)    |
| Post-switching regulator | High PSRR at fsw, moderate noise    | TI TPS717, ADI ADP7182       |
| Automotive               | −40°C to 125°C, AEC-Q100            | TI TPS7B84, Infineon TLE4250 |
