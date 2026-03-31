# Stability Analysis and Bode Plots — Interview Preparation

## Overview

Understanding loop stability requires working with transfer functions, constructing and reading Bode plots, and applying the Nyquist stability criterion. This section covers the control-to-output and line-to-output transfer functions, key plant features (LC double pole, ESR zero, RHP zero), and practical stability measurement. These concepts are tested thoroughly at senior power electronics engineering roles.

---

## Key Equations Reference

### Buck Converter Plant (Voltage Mode, CCM)
```
Control-to-output:
  Gvd(s) = Vin × (1 + s/ωz_ESR) / (1 + s/(Q·ω0) + s²/ω0²)

  where:
    ω0 = 1/√(LC)               — LC resonant frequency (rad/s)
    ωz_ESR = 1/(ESR × C)       — ESR zero (rad/s)
    Q = R_load / √(L/C)        — quality factor (load-dependent)
    DC gain = Vin (ideal buck)
```

### Line-to-Output Transfer Function (Audio Susceptibility)
```
Open-loop:  Gvg_OL(s) = D × (1 + s/ωz_ESR) / (1 + s/(Q·ω0) + s²/ω0²)
Closed-loop: Gvg_CL(s) = Gvg_OL(s) / (1 + T(s))
```

### Current Mode Control Plant (Buck, CCM)
```
Gvd_cm(s) ≈ Vin × Rs / (sL + Rs) × 1/(1 + s·R_load·C)   [simplified]
         → single dominant output pole at fp = 1/(2π·R_load·C)
```

### RHP Zero (Boost/Flyback, CCM)
```
f_RHP = (1-D)² × R_load / (2π × L)
      = Vout × (1-D)² / (2π × L × Iout)
```

### Asymptotic Bode Slope Rules
```
Pole at fp:      -20 dB/dec above fp,   -45°/dec phase transition over 2 decades
Zero at fz:      +20 dB/dec above fz,   +45°/dec phase transition over 2 decades
Double pole f0:  -40 dB/dec above f0,   phase from 0° to -180° through resonance
RHP zero f_rhp:  +20 dB/dec (gain, same as LHP zero), -90° phase (opposite sign)
Integrator:      -20 dB/dec at all freq, -90° phase at all freq
```

### Gain and Phase Margin
```
Phase Margin:  PM = 180° + ∠T(jωc)   at gain crossover (|T|=0 dB)
Gain Margin:   GM = -20·log|T(jωpm)| at phase crossover (∠T=-180°)

Targets: PM ≥ 45°, GM ≥ 10 dB
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Derive the control-to-output transfer function for a voltage-mode buck converter in CCM.

**Answer:**

**Small-signal averaged model derivation:**

Starting from the averaged switched model, perturb around the operating point (D, IL, VC):
- Perturbation variables: d̂ (duty cycle), î_L (inductor current), v̂_C (capacitor voltage)
- Input Vin treated as constant for control-to-output derivation

**Averaged small-signal equations (ignoring winding resistance for clarity):**
```
L × s × î_L = Vin × d̂ - v̂_C
C × s × v̂_C = î_L - v̂_C / R_load
```

**Solving for v̂_C / d̂:**

From the first equation:
```
î_L = (Vin × d̂ - v̂_C) / (sL)
```

Substitute into the second:
```
C·s·v̂_C = (Vin·d̂ - v̂_C)/(sL) - v̂_C/R

v̂_C · [Cs + 1/(sL) + 1/R] = Vin·d̂/(sL)

Gvd(s) = v̂_C/d̂ = Vin / (s²LC + sL/R + 1)
```

**Including output capacitor ESR (r_C):**
The output voltage is measured at the load terminals:
```
v̂_out = v̂_C + r_C × (î_L - v̂_out/R)   [ESR carries ripple current difference]
```

After substitution and algebra:
```
Gvd(s) = Vin × (1 + s·r_C·C) / (s²LC + s(L/R + r_C·C + r_C²·C/R) + 1)

Simplified (r_C << R):
Gvd(s) = Vin × (1 + s/ωz) / (1 + s/(Q·ω0) + s²/ω0²)

where:
  ω0 = 1/√(LC)
  ωz = 1/(r_C·C)
  Q ≈ R·√(C/L)   [load-dependent]
```

**Key observations:**
- DC gain = Vin (voltage conversion ratio of 1 at D=1)
- Double pole at ω0 produces -40 dB/dec roll-off and up to -180° phase lag
- ESR zero at ωz partially recovers phase above its frequency
- Q is load-dependent: light load → high Q → sharp resonance peaking → more phase lag near f0

---

### Q2. What is the LC double pole and how does it affect Bode plot shape and phase margin?

**Answer:**

The LC double pole is a second-order resonance at:
```
f0 = 1 / (2π × √(L × C))
```

**Gain effects:**
- Below f0: gain is flat (determined by DC gain = Vin for a buck)
- At f0: gain peaks by 20·log(Q) dB if Q > 0.707 (underdamped)
  - High Q (light load): sharp tall peak
  - Q = 0.707 (Butterworth): maximally flat, no peak
  - Q < 0.5 (overdamped): smooth monotonic roll-off
- Above f0: -40 dB/decade slope (two poles contribute -20 dB/dec each)

**Phase effects:**
- Below f0/10: phase ≈ 0°
- At f0: phase = exactly -90° (for any Q)
- Above 10×f0: phase → -180°
- Transition rate depends on Q: high Q = rapid transition through -180° in a narrow frequency band

**Phase at arbitrary frequency for the second-order system:**
```
φ(ω) = -arctan( (ω/ω0) / (Q × (1 - (ω/ω0)²)) )
```

**Design implication:**

If fc (crossover) is chosen above f0, the plant has already delivered significant phase lag. For example, with f0 = 200 Hz and fc = 20 kHz (100:1 ratio), the plant phase at 20 kHz approaches -180°. The compensator must boost phase by at least 135° (to achieve PM = 45°). This is only achievable with a Type III compensator.

**Danger of high-Q resonance:**
With Q = 10 (light load, ceramic capacitors, low ESR), the plant gain at f0 is +20 dB. Even with a compensator that achieves PM = 60° at fc, the loop gain at f0 might still exceed 0 dB with insufficient phase margin. Check the Bode plot at f0 — if gain is positive and phase is near -180° at f0, the resonance is unstable.

---

### Q3. Explain the physical origin and frequency-domain effect of the ESR zero.

**Answer:**

**Physical origin:**

A real capacitor has an equivalent series resistance (ESR). When AC ripple current iL flows through the capacitor, the output voltage has two components:
1. **Capacitive voltage:** integral of current through C — 90° phase lag, impedance = 1/(sC)
2. **Resistive voltage:** directly proportional to current through ESR — in-phase, impedance = ESR

Below f_ESR = 1/(2π × ESR × C): the capacitive impedance dominates (ESR is small in comparison)
Above f_ESR: the resistive impedance dominates

**Effect on transfer function:**

The ESR creates a numerator factor (zero):
```
Gvd(s) ∝ (1 + s × ESR × C)

Zero at:  f_ESR = 1 / (2π × ESR × C)
```

Above f_ESR:
- Gain slope changes from -40 dB/dec to -20 dB/dec (zero cancels one pole)
- Phase recovers from approaching -180° back toward -90°

**Capacitor type comparison:**

| Capacitor Type  | Typical ESR      | 100 µF f_ESR     | Impact on Design             |
|-----------------|------------------|------------------|------------------------------|
| Aluminum elect. | 50–500 mΩ        | 3–32 kHz         | ESR zero often below fc      |
| Tantalum        | 100–500 mΩ       | 3–16 kHz         | Similar to aluminum          |
| Polymer elec.   | 10–50 mΩ         | 32–160 kHz       | Moderate impact              |
| MLCC (ceramic)  | 1–10 mΩ          | 160 kHz – 1.6 MHz| ESR zero well above fc       |

**Design consequence:**

With high-ESR capacitors (electrolytic): the ESR zero is often within the frequency range of interest. The compensator's HF pole should be placed at f_ESR to prevent the ESR zero from causing gain to rise at high frequency (which could cause a second gain crossover with poor phase).

With MLCCs: ESR is negligible for stability purposes. The double pole is unmitigated, requiring a full Type III compensator. The benefit of MLCC is lower output voltage ripple.

---

### Q4. Define gain margin and phase margin. Explain how to read both from a Bode plot.

**Answer:**

**Phase Margin (PM):**

Measured at the gain crossover frequency ωc where |T(jωc)| = 0 dB:
```
PM = 180° + ∠T(jωc)
```

If phase at the 0 dB crossing = -125°: PM = 180° - 125° = 55°.

**Reading PM from a Bode plot:**
1. Locate the frequency where the magnitude plot crosses 0 dB — this is ωc
2. Drop vertically to the phase plot
3. Read the phase value (e.g., -120°)
4. PM = 180° + (phase value) = 180° - 120° = 60°

**Gain Margin (GM):**

Measured at the phase crossover frequency ωpm where ∠T(jωpm) = -180°:
```
GM = -20 × log₁₀|T(jωpm)|   [dB]
```

If magnitude at the -180° phase crossing = -12 dB: GM = 12 dB.

**Reading GM from a Bode plot:**
1. Locate the frequency where the phase plot crosses -180° — this is ωpm
2. Rise vertically to the magnitude plot
3. Read the gain value (e.g., -15 dB)
4. GM = -(-15) = 15 dB

**Physical meaning:**
- PM = how much additional phase lag before instability (at the operating gain)
- GM = how much additional gain before instability (at the operating phase)

**Multiple crossovers:**
Some plants cross 0 dB or -180° multiple times. Always check PM at every 0 dB crossing and GM at every -180° crossing. The minimum values determine the actual stability margins.

**Minimum acceptable values:**
```
PM ≥ 45°  (minimum), ≥ 60° (recommended for well-damped transient response)
GM ≥ 10 dB (minimum), ≥ 12–15 dB (recommended for component tolerance margin)
```

---

### Q5. What is the RHP zero, where does it arise physically, and what does it look like on a Bode plot?

**Answer:**

**Physical origin in a boost converter:**

In a boost converter operating in CCM, increasing duty cycle D means:
- On-time increases → inductor stores more energy
- Off-time (1-D) decreases → inductor delivers energy to output for less time
- Net immediate effect: diode current (and thus output current) decreases
- Longer-term effect: more stored energy eventually increases output current

This "wrong-way" initial response is non-minimum phase behavior. The transfer function includes a numerator factor:
```
(1 - s/ωRHP)   →  RHP zero at s = +ωRHP
```

**Transfer function:**
```
Gvd_boost(s) = (Vout/D) × (1 - s/ωRHP) / (1 + s/(Q·ω0) + s²/ω0²)

where:
  ωRHP = (1-D)²·R / L = Vout·(1-D)² / (L·Iout)
```

**Bode plot appearance:**

This is the key insight: the RHP zero looks identical to an LHP zero on the gain (magnitude) Bode plot, but opposite on the phase plot.

| Quantity           | LHP Zero (normal)      | RHP Zero               |
|--------------------|------------------------|------------------------|
| Gain above fz      | +20 dB/decade          | +20 dB/decade (same)   |
| Phase above fz     | +90° (phase lead)      | -90° (phase lag)       |
| Time-domain step   | Normal minimum-phase   | Initial wrong direction |

**Why this is dangerous:**

A designer looking only at the gain Bode plot sees a zero (gain increases, which helps crossover) and may assume it helps phase margin. It does not — it subtracts phase. At the RHP zero frequency:
```
Phase lag from RHP zero = -arctan(fc/fRHP)
```

At fc = fRHP: the RHP zero contributes -45° of lag. At fc = 3 × fRHP: -71° of lag.

**Design rule:** Keep fc < fRHP/3 (some references say fRHP/5 for margin). The RHP zero limits maximum achievable bandwidth.

**Boost converter numerical example:**
```
Vout=48V, D=0.75, R=48Ω, L=100µH:
fRHP = (1-0.75)² × 48 / (2π × 100µH) = 0.0625 × 48 / (628µ) = 4.8 kHz
→ fc must be < 1.6 kHz
```

---

### Q6. What is the audio susceptibility of a power supply and how is it specified and measured?

**Answer:**

**Definition:**

Audio susceptibility (also called line-to-output transfer function or input audio susceptibility) quantifies how input voltage variations appear at the output:
```
Gvg(s) = v̂_out(s) / v̂_in(s)|_{load=constant, d=closed-loop regulated}
```

**Open-loop vs closed-loop:**
```
Open-loop:   Gvg_OL(s) = D × (1 + s/ωz) / (1 + s/(Q·ω0) + s²/ω0²)   [buck]
Closed-loop: Gvg_CL(s) = Gvg_OL(s) / (1 + T(s))
```

At low frequencies (within loop bandwidth, |T| >> 1):
```
Gvg_CL ≈ Gvg_OL / |T|  → very small (good input rejection)
```

At high frequencies (above bandwidth, |T| << 1):
```
Gvg_CL ≈ Gvg_OL → filter attenuation only
```

**Specification:**
Expressed in dB. A specification of -60 dB at 100 Hz means: 1 V RMS of input ripple at 100 Hz → 1 mV at the output.

**Why "audio":**
The term comes from audio equipment, where 50/60 Hz mains ripple coupling into a DC supply creates audible hum. Modern usage extends to any ripple frequency. Typical specifications:
```
PSRR (Power Supply Rejection Ratio) = -Gvg_CL in dB
Audio-grade supply: PSRR ≥ 80 dB at 50/60 Hz, ≥ 60 dB at 1 kHz
Industrial supply: PSRR ≥ 40–60 dB at ripple frequencies
```

**Measurement method:**
1. Operate converter at rated load with nominal Vin
2. Inject a known AC voltage (variable frequency) onto Vin using a series transformer or by modulating the Vin source
3. Measure V_out_AC / V_in_AC ratio vs frequency
4. The closed-loop rejection = 20·log(V_out_AC / V_in_AC)

**Important:**
Line regulation (a static spec) and audio susceptibility (a dynamic spec) are different. A converter can have excellent static line regulation (0.01% Vout change per 1% Vin change) but poor audio susceptibility at 1 kHz if the loop bandwidth is only 500 Hz. Always specify both.

---

## Intermediate (Questions 7–13)

---

### Q7. How do you construct an asymptotic Bode plot for a voltage-mode buck plant, including the ESR zero?

**Answer:**

**Given parameters:**
```
Vin = 12V, Vout = 5V, L = 22µH, C = 100µF, ESR = 100mΩ, R_load = 5Ω (at 1A)
```

**Step 1: Calculate key frequencies**
```
f0 = 1/(2π√LC) = 1/(2π × √(22µH × 100µF)) = 1/(2π × 1.483ms) = 107 Hz

f_ESR = 1/(2π × ESR × C) = 1/(2π × 100mΩ × 100µF) = 15.9 kHz

Q = R_load × √(C/L) = 5 × √(100µF/22µH) = 5 × 2.132 = 10.7  [at 1A — high Q]
```

**Step 2: Construct gain plot**
- 0 Hz to 107 Hz: flat at 20·log(12) = +21.6 dB
- At 107 Hz: gain peaks by 20·log(10.7) = 20.6 dB → peak at 21.6 + 20.6 = 42.2 dB
  (Note: the asymptote continues flat then drops — the peak is above the asymptote)
- 107 Hz to 15.9 kHz: -40 dB/decade
  - At 1 kHz (1 decade above 107 Hz): 21.6 - 40 = -18.4 dB
  - At 10 kHz: 21.6 - 80 = -58.4 dB (approximate — ESR zero at 15.9 kHz begins to modify)
- Above 15.9 kHz: ESR zero kicks in, slope changes to -20 dB/decade

**Step 3: Construct phase plot**
- Below f0/10 ≈ 10 Hz: phase ≈ 0°
- At f0 = 107 Hz: phase = -90°
- At 10 × f0 ≈ 1 kHz: phase approaching -180° (with high Q, this transition is sharp)
- At f_ESR/10 ≈ 1.6 kHz: ESR zero begins adding positive phase
- At f_ESR = 15.9 kHz: phase from zero = +45°; combined with double-pole lag ≈ -180° + 45° = -135°
- Net phase at 15.9 kHz ≈ -135° (approximate)

**Reading off PM possibility:**
If a compensator targets fc = 10 kHz, the plant phase at 10 kHz is approximately -165° (mostly double pole, small ESR correction). The compensator must boost by >120° to achieve PM = 45°. A Type III is necessary.

---

### Q8. Explain the Nyquist stability criterion and when simple Bode-based analysis is insufficient.

**Answer:**

**Nyquist criterion statement:**

For a feedback system with open-loop transfer function T(s):
```
Z = N + P
```
- Z = number of unstable closed-loop poles (RHP)
- N = net clockwise encirclements of (-1, 0) by the Nyquist contour T(jω) as ω sweeps -∞ to +∞
- P = number of open-loop RHP poles

**Stability condition:** Z = 0. If P = 0 (stable open-loop plant), then N must equal 0 — no net clockwise encirclements.

**Relationship to Bode:**

For minimum-phase systems (no RHP zeros, no delays) with a single gain crossover and single phase crossover:
- N = 0 is equivalent to PM > 0° and GM > 0 dB
- Bode-based analysis is sufficient and equivalent to Nyquist

**When Bode analysis is insufficient:**

1. **Non-minimum-phase systems (RHP zeros):**
   A RHP zero shifts the Nyquist contour differently than an LHP zero. The simple Bode PM/GM check can indicate stability when the system is actually marginally stable or unstable.

2. **Systems with delays:**
   A pure delay e^(-sτ) wraps the phase indefinitely. The Nyquist contour spirals inward. Multiple phase crossings occur. Bode shows multiple -180° crossings; each must be checked individually.

3. **Conditionally stable systems:**
   The Nyquist contour encircles (-1) in both clockwise and counterclockwise directions. The net count N = 0, so Z = 0 — stable. But reducing gain by a factor causes one counterclockwise encirclement to drop out while the clockwise one remains: N = 1, Z = 1 — unstable. Bode analysis of this situation requires checking ALL gain crossings.

4. **Resonant plants with multiple crossings:**
   An LLC resonant converter or a converter with underdamped input filter can have multiple gain crossovers. Bode shows PM > 45° at the primary crossover, but poor stability exists at a secondary crossover.

**Practical rule:**
Always plot the full Bode magnitude and phase from 10 Hz to 10× fsw. If the phase ever goes below -180° while gain is still above 0 dB — or the gain is above 0 dB while phase is below -180° at any frequency — instability exists regardless of the PM at the primary crossover.

---

### Q9. What is conditional stability and how does it manifest in a power supply?

**Answer:**

**Definition:**
A system is conditionally stable if it is stable only within a specific range of loop gain. Reducing the gain below a lower threshold OR increasing it above an upper threshold causes instability.

**Signature on Bode plot:**
The phase plot crosses -180° at a frequency where the gain is still above 0 dB (primary crossover), but the gain is also above 0 dB at another frequency where the phase again crosses -180°. The system is stable overall (Nyquist count = 0), but if gain is reduced, the second phase crossing region has insufficient gain margin.

**Bode plot sequence (conditionally stable):**
```
Gain plot:  >0dB [crosses 0dB at f1] <0dB [crosses 0dB again at f2] <0dB
Phase plot: >-180° [nominally]  <-180° [at f between f1 and f2, but gain<0dB OK]
            Then phase recovers above -180° for higher frequencies
But if gain reduces: gain remains above 0dB in the trouble region → unstable
```

**How it arises in power supplies:**

1. **Over-aggressive Type III compensation:** A compensator with very high phase boost creates a "hump" in the phase that briefly exceeds -180° at a frequency where the loop gain is below but close to 0 dB. If the plant gain decreases (light load, lower Vin), that region may go above 0 dB → conditional instability at light load.

2. **Input filter resonance:** An LC input filter resonates. Below the resonance, the plant is normal. Above it, the input filter impedance peaks, interacting with the converter's negative input impedance to create phase inversion. The Nyquist contour wraps around -1 at the resonance frequency.

3. **LLC resonant converters:** The gain characteristic has gain peaks and valleys. Moving the operating frequency away from resonance can change the effective loop gain sign — conditional stability in the gain-vs-frequency sense.

**Detection and fix:**
- Simulate Bode plot over full frequency range and all operating conditions (light load, heavy load, Vin min, Vin max)
- Verify that gain < 0 dB at ALL frequencies where phase < -180°
- If conditional stability exists: add damping (higher-Q input filter, snubbers), reduce compensator phase boost, or add a notch filter at the problem frequency

---

### Q10. How does the output capacitor type (electrolytic vs ceramic) change the stability analysis?

**Answer:**

The output capacitor type fundamentally changes the plant transfer function by shifting the ESR zero location and modifying the Q factor.

**Electrolytic capacitors (high ESR):**
```
Typical: 100µF, ESR = 150mΩ
f_ESR = 1/(2π × 0.15 × 100µF) = 10.6 kHz
Q = R_load × √(C/L) × 1/(1 + r_C/R) ≈ low due to ESR damping
```

Effect on stability:
- ESR zero at 10.6 kHz is within or below a typical 20–50 kHz crossover → helps phase
- High ESR damps the LC resonance (lower Q) → less phase peaking at f0
- Bode plot shows -40 dB/dec from f0, transitioning to -20 dB/dec at f_ESR

Compensator implication: A Type II compensator may be sufficient. Place the HF compensator pole at f_ESR to prevent loop gain from rising above the crossover.

**Ceramic capacitors (ultra-low ESR):**
```
Typical: 10µF × 10 = 100µF effective, ESR = 5mΩ per capacitor total ≈ 0.5mΩ
f_ESR = 1/(2π × 0.5mΩ × 100µF) = 3.18 MHz
Q = R_load × √(C/L) = very high (minimal ESR damping)
```

Effect on stability:
- ESR zero is at 3.18 MHz — irrelevant for a 50 kHz crossover
- Very high Q at resonance — sharp phase transition through -180°
- Plant phase at fc is close to -180° with no ESR zero relief

Compensator implication: Must use Type III. Both zeros placed near f0. The LC resonance is the dominant stability challenge.

**Mixed capacitor banks (common in practice):**
A mix of electrolytic + ceramic creates a more complex impedance profile. The ESR zero splits — the electrolytic's zero appears at a moderate frequency, the ceramic's at very high frequency. The effective impedance must be modeled as a parallel combination. Simulate the actual capacitor bank in SPICE with manufacturer-provided SPICE models.

**Temperature effects:**
Electrolytic capacitance and ESR drift significantly with temperature. At -40°C, ESR can be 4× higher than at 25°C — this raises f_ESR and alters Q. Verify stability at temperature extremes.

---

### Q11. What is the output impedance of a converter and why does it matter to system designers?

**Answer:**

**Definition:**
The output impedance Zout(s) is the small-signal impedance seen looking into the converter output terminals with the input voltage held constant:
```
Zout(s) = v̂_out(s) / î_load(s)|_{v̂_in=0}
```

**Open-loop output impedance (buck):**
Without feedback, the output impedance is that of the LC filter with load:
```
Zout_OL(s) ≈ (sL + r_L) || (1/(sC) + r_C) || R_load
```

At low frequency: Zout_OL ≈ r_L (winding resistance)
At resonance f0: Zout peaks at √(L/C) / (r_L/√(L/C) + ...) — Q × √(L/C)
At high frequency: Zout → r_C (ESR)

**Closed-loop output impedance:**
```
Zout_CL(s) = Zout_OL(s) / (1 + T(s))
```

Within loop bandwidth (|T|>>1): Zout_CL << Zout_OL → good load regulation.
At and above fc: Zout_CL → Zout_OL.

**Zout and load transient response:**
A load current step ΔI produces an output voltage deviation:
```
ΔV(s) = Zout_CL(s) × ΔI(s)
```
In the time domain, the peak voltage deviation is approximately:
```
ΔV_peak ≈ ΔI × Zout_CL(fc)  [rough estimate]
```

**Why system designers care:**

1. **Point-of-load converters:** Multiple POL converters sharing a bus. If a POL's Zout peaks at a frequency where a downstream load's input impedance dips, oscillation can occur (Middlebrook's stability criterion).

2. **Audio equipment:** Zout directly determines how much the supply voltage changes when audio current pulses are drawn. High Zout = audible hum or distortion.

3. **Microprocessor power:** Modern CPUs draw 100A load steps in nanoseconds. The effective Zout at 10–100 MHz must be below the allowed voltage droop / current step.

4. **Middlebrook criterion:** For a source-load system to be stable, the source output impedance must be less than the load input impedance at all frequencies. If |Zout_source(jω)| > |Zin_load(jω)| in some frequency band, oscillation can occur.

---

### Q12. How do you measure the loop gain Bode plot experimentally?

**Answer:**

**Standard method — frequency response analysis:**

Equipment needed:
- Frequency Response Analyser (FRA) or Vector Network Analyser (VNA): e.g., Bode 100, AP Instruments 300, Keysight E5061B
- Injection transformer: Picotest J2100A or equivalent (10 Hz to 10 MHz)
- Differential probe (optional but recommended)

**Setup:**

1. Identify a suitable injection point in the feedback loop. Preferred locations:
   - Between the error amplifier output and the PWM comparator input
   - Between the output voltage feedback divider and the error amplifier input
   - A 50Ω resistor can be inserted in series at the chosen point

2. Connect the injection transformer secondary across the injection resistor (or into the loop break). Primary driven by the FRA source output.

3. Channel A of FRA: measure signal at the point BEFORE the injection (V1)
   Channel B of FRA: measure signal at the point AFTER the injection (V2)

4. The FRA computes:
   ```
   T(jω) = -V2(jω) / V1(jω)   [sign depends on injection point orientation]
   ```

**Injection amplitude:**
Set to 10–50 mVpp at the injection point. Too small: poor SNR. Too large: converter goes nonlinear (clips, hits protection thresholds).

**Frequency sweep range:**
Start at ~10 Hz (to capture the integrator region), sweep to fsw/2. Use log frequency spacing (10–20 points per decade).

**What to record and verify:**
1. Gain crossover frequency fc: where |T| = 0 dB
2. Phase at fc: PM = 180° + phase
3. Phase crossover frequency: where phase = -180°
4. Gain at phase crossover: GM = negative of that gain value

**Common measurement errors:**
- Ground loops between FRA and converter: use transformer-isolated measurement
- Insufficient injection amplitude at high frequency (transformer coupling drops): verify amplitude with oscilloscope
- Measuring outside the feedback loop (open loop): the measured response will not show the characteristic integrator roll-off at DC

---

### Q13. What is the effect of a time delay on the Bode plot and stability margins?

**Answer:**

**Transfer function of a pure time delay:**
```
G_delay(s) = e^(-s·τ)
```

**Magnitude:** |e^(-jω·τ)| = 1 for all frequencies — no gain change.

**Phase:** ∠e^(-jω·τ) = -ω·τ radians = -360°·f·τ degrees

The delay adds phase lag linearly proportional to frequency. At high frequencies, the phase lag becomes arbitrarily large.

**Effect on phase margin:**

If the undelayed loop has PM = 60° at crossover fc, a delay τ reduces it:
```
PM_reduced = 60° - 360° × fc × τ
```

For τ = 1µs (1 microsecond computational delay) at fc = 50 kHz:
```
Phase reduction = 360° × 50,000 × 1µs = 18°
PM_reduced = 60° - 18° = 42° (marginally acceptable)
```

For τ = 1µs at fc = 100 kHz:
```
Phase reduction = 36°
PM_reduced = 60° - 36° = 24° (inadequate)
```

**Maximum crossover frequency with delay:**
To maintain PM ≥ 45°:
```
360° × fc × τ ≤ (PM_undelayed - 45°)
fc ≤ (PM_undelayed - 45°) / (360° × τ)
```

**Sources of delay in power supplies:**
1. Analog PWM controller propagation delay: 50–200 ns
2. Gate drive propagation: 50–100 ns
3. Current sense filtering (RC filter): equivalent delay = RC time constant
4. Digital controller computational delay: 0.5–2 switching periods
5. Optocoupler propagation delay: 1–10 µs

**Compensation for delay:**
In analog systems, minimize physical delays. In digital systems, use a predict-ahead control (reduce effective delay to 0.5 Ts) or limit fc to keep phase lag manageable. The bilinear transform (Tustin method) implicitly compensates half a sample delay.

---

## Advanced (Questions 14–17)

---

### Q14. Derive the control-to-output transfer function for a boost converter in CCM and identify all poles and zeros.

**Answer:**

**Boost converter state-space averaging:**

States: iL (inductor current), vC (capacitor voltage)

**On-state (switch closed, D × Ts):**
```
L × diL/dt = Vin
C × dvC/dt = -vC/R   [diode off, capacitor supplies load]
```

**Off-state (switch open, (1-D) × Ts):**
```
L × diL/dt = Vin - vC
C × dvC/dt = iL - vC/R   [diode conducts]
```

**Averaged equations (with duty cycle d = D + d̂):**
```
L × dIL/dt = Vin - (1-D)·VC + small-signal: L·s·î_L = Vin·d̂·0 + (1-D)·...
```

Working through the small-signal perturbation (setting DC terms to zero):
```
L·s·î_L = VC·d̂ - (1-D)·v̂C        [VC·d̂ = Vin/(1-D) × d̂]
C·s·v̂C = (1-D)·î_L - IL·d̂ - v̂C/R
```

Note: IL·d̂ appears in the capacitor equation — this is the source of the RHP zero.

**Solving for v̂C/d̂:**

Eliminate î_L:
```
î_L = [VC·d̂ - (1-D)·v̂C] / (s·L)

C·s·v̂C = (1-D)/sL × [VC·d̂ - (1-D)·v̂C] - IL·d̂ - v̂C/R
```

After collecting terms:
```
Gvd(s) = v̂C/d̂ = [Vin/(1-D)² × (1 - s·L·IL/(Vin/(1-D)²·...)] / [second order denominator]

Simplified form:
Gvd_boost(s) = (Vout/D) × (1 - s/ωRHP) / (1 + s/(Q·ω0) + s²/ω0²)

where:
  ωRHP = (1-D)²·R/L          — RHP zero (positive real axis of s-plane)
  ω0 = (1-D)/√(LC)           — LC resonant frequency (modified by (1-D))
  Q = (1-D)·R·√(C/L)         — quality factor
  DC gain = Vin/(1-D)² = Vout/D·1/(1-D)  [boost conversion ratio]
```

**Key features:**
- DC gain higher than buck (boosted by 1/(1-D)²)
- LC double pole shifted down in frequency by factor (1-D)
- RHP zero: gain slope is +20 dB/dec above ωRHP (like LHP zero), phase is -90° (lag, like a pole)
- No ESR zero in this simplified analysis; it appears when capacitor ESR is included in the same way as for the buck

---

### Q15. Explain the Middlebrook source-load stability criterion and when it applies to distributed power systems.

**Answer:**

**Problem statement:**

In a distributed power system, a regulated source (converter A) feeds a load (converter B or a complex load). Converter A has output impedance Zs(s). Converter B presents input impedance Zin(s). The interconnected system has an additional loop gain term:

```
Additional loop gain = Zs(s) / Zin(s)
```

**Middlebrook criterion:**

If the magnitude of Zs exceeds Zin at any frequency, the interconnected system may be unstable even though each converter is individually stable:
```
|Zs(jω)| < |Zin(jω)|   for all ω  →  stable
|Zs(jω)| > |Zin(jω)|   at some ω  →  potentially unstable
```

This is a sufficient but not necessary condition for stability — it is conservative.

**Physical interpretation:**

A regulated converter presents a negative incremental input impedance:
```
Zin(s)|DC = -Vin²/Pin   [negative for a regulated load]
```

As Vin increases, the converter draws less current (to maintain constant power). This negative impedance can cancel the positive output impedance of the source, creating a zero in the denominator of the interconnected system → instability.

**Application in distributed power:**

- 48V bus systems (data centers, telecom): 48V bus converter → multiple POL converters
- Each POL presents negative input impedance; the 48V bus converter must have Zout < |Zin_POL|
- Midbus architecture: if multiple POL converters are on the same bus and their negative Zin values sum, the effective load impedance can be very low or negative

**Mitigation strategies:**

1. **Reduce source output impedance:** Use larger output capacitors on the source converter, or higher loop bandwidth
2. **Input filter on the load converter:** An LC input filter provides positive input impedance at its resonance, masking the negative impedance of the regulated converter
3. **Routh-Hurwitz analysis:** More precise stability analysis computing the combined loop gain of the full system
4. **Benchmark test:** Apply a load step to converter B and observe whether converter A's output rings or oscillates

---

### Q16. How does gain and phase plot measurement change when an optocoupler is in the feedback path?

**Answer:**

The optocoupler introduces gain, phase, and bandwidth uncertainty that must be accounted for in the Bode plot measurement.

**Optocoupler model:**
```
Gopt(s) = CTR / (1 + s/ωopt)

where:
  CTR = current transfer ratio (Ic/If) — varies 2:1 or more
  ωopt = 1/(R_collector × C_parasitic) — bandwidth pole
```

**Effect on measured Bode plot:**

1. **DC gain shift:** If CTR varies from 0.5 to 1.5 (typical spread), the total loop gain shifts by:
   ```
   20 × log(1.5/0.5) = 9.5 dB
   ```
   This changes fc by approximately half a decade. PM and GM shift accordingly.

2. **Phase lag at crossover:** The opto pole at fopt contributes:
   ```
   φ_opto = -arctan(fc/fopt)
   ```
   For fopt = 30 kHz and fc = 10 kHz: φ = -arctan(0.33) = -18.4° of additional lag.

3. **Temperature shift:** CTR decreases with temperature. At 85°C, a PC817 CTR may drop to 60% of the 25°C value. The measured Bode plot at room temperature does not represent worst-case.

**Best measurement practice:**

- Measure at 25°C and 85°C — verify PM ≥ 45° at both temperatures
- Measure with the slowest (lowest CTR) optocoupler in the population — use CTR min from the datasheet
- If adjusting R_collector to compensate for CTR variation (gain scheduling), verify at all CTR values
- Inject at the point just after the optocoupler on the primary side, and measure on both sides to characterize the opto's contribution separately from the rest of the loop

**Design verification across CTR range:**
```
Loop gain at nominal CTR:   T_nom(s) = T_base(s) × CTR_nom
Loop gain at min CTR:       T_min(s) = T_base(s) × CTR_min   → lower gain, lower fc, potentially lower GM
Loop gain at max CTR:       T_max(s) = T_base(s) × CTR_max   → higher gain, higher fc, potentially lower PM
```

The compensator must provide adequate PM at CTR_max and adequate GM at CTR_min.

---

### Q17. What analysis tools are available beyond Bode plots for stability verification?

**Answer:**

**1. Nyquist plot:**
Plots Im{T(jω)} vs Re{T(jω)} as ω sweeps. Directly applies Nyquist criterion by counting encirclements of (-1, 0). Required for non-minimum-phase systems and systems with multiple gain crossings.

**2. Nichols chart:**
Plots open-loop gain (dB) vs open-loop phase (degrees). Closed-loop gain contours are overlaid. Advantages:
- Directly reads closed-loop peak response (M_p circles)
- Shows forbidden region for excessive peaking
- Crossover frequency and PM readable simultaneously
Primarily used in aerospace and military power applications.

**3. Root locus:**
Plots the locations of closed-loop poles as loop gain K varies from 0 to ∞. Shows which poles move toward the RHP (instability) as gain increases. Useful for conditional stability analysis — the root locus crosses the imaginary axis at the gain crossover gains.

**4. Describing function analysis:**
For systems with nonlinearities (limiters, saturation), the describing function approximates the effective gain and phase of the nonlinearity as a function of amplitude. Combined with the linear Bode plot, it predicts limit cycle amplitude and frequency.

**5. Eigenvalue analysis:**
For multi-converter systems (e.g., paralleled converters with current sharing), the full system matrix eigenvalues determine stability. If any eigenvalue has a positive real part, the system is unstable. More general than loop gain analysis.

**6. Hardware-in-the-loop (HIL) simulation:**
A real-time digital simulator (e.g., Typhoon HIL, OPAL-RT) simulates the power stage while the actual controller hardware controls it. The Bode plot can be measured on the HIL setup, giving accurate results that include all software delays, quantization, and nonlinearities.

**7. Impedance measurement:**
Measure the converter output impedance (Zout) and the load input impedance (Zin). Apply Middlebrook criterion or full Nyquist analysis of Zout/Zin to verify system stability in a distributed power system.

---

## Quick Reference: Stability Margin Targets

| Parameter            | Minimum  | Recommended | Rationale                              |
|----------------------|----------|-------------|----------------------------------------|
| Phase Margin         | 45°      | 60°         | 45° → 20% overshoot; 60° → 8% overshoot|
| Gain Margin          | 10 dB    | 12–15 dB    | 6 dB CTR variation + 6 dB tolerance    |
| Crossover freq (fc)  | —        | fsw/10      | Avoids sampling artefacts              |
| fc relative to f_RHP | —        | < fRHP/3    | Limits RHP zero phase lag at crossover |
| High-freq gain       | < 0 dB   | < -10 dB    | Above fc to prevent HF instability     |
