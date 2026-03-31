# Buck Converter Basics — Interview Preparation

## Overview

The buck converter (step-down converter) is the most widely used switching regulator topology. It converts a higher DC input voltage to a lower DC output voltage with high efficiency. Understanding its operation, key equations, and design trade-offs is essential for any power electronics interview.

---

## Key Equations Reference

### Duty Cycle (Ideal, CCM)
```
D = Vout / Vin
```
Derivation: volt-second balance on inductor. During on-time: V_L = Vin - Vout. During off-time: V_L = -Vout.
```
(Vin - Vout) × D × Ts = Vout × (1-D) × Ts
D = Vout / Vin
```

### Inductor Current Ripple (CCM)
```
ΔIL = (Vin - Vout) × D × Ts / L  =  Vout × (1-D) / (fsw × L)
```

### Critical Inductance (CCM/DCM boundary)
```
Lcrit = Vout × (1-D) / (2 × fsw × Iout)
```

### Output Voltage Ripple (capacitor-dominated)
```
ΔVout_cap = ΔIL / (8 × fsw × Cout)
```

### Output Voltage Ripple (ESR-dominated)
```
ΔVout_ESR = ΔIL × ESR
```

### Output Voltage Ripple (ESL contribution, at switching edges)
```
ΔVout_ESL = ESL × (dI/dt)
```

### Efficiency (simplified)
```
η = Pout / Pin = Pout / (Pout + Psw + Pcond + Pgate + Pcore + ...)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Describe the operation of an asynchronous buck converter through one complete switching cycle.

**Answer:**

An asynchronous buck converter consists of a high-side switch (MOSFET Q1), a freewheeling diode D1, an output inductor L, and output capacitor Cout.

**Phase 1 — Switch ON (duration: D × Ts):**
- Q1 closes, connecting Vin to the switch node (Vsw = Vin, ignoring drops).
- Voltage across inductor: `VL = Vin - Vout` (positive).
- Inductor current ramps up linearly: `dIL/dt = (Vin - Vout)/L`.
- Diode D1 is reverse biased (blocking).
- Energy is stored in the inductor core.

**Phase 2 — Switch OFF (duration: (1-D) × Ts):**
- Q1 opens. Inductor current cannot change instantaneously (Lenz's law) — it forces the switch node negative until D1 conducts.
- Vsw ≈ -Vf (diode forward voltage, ~0.3–0.5 V for Schottky).
- Voltage across inductor: `VL = -(Vout - Vf) ≈ -Vout` (negative).
- Inductor current ramps down linearly: `dIL/dt = -(Vout - Vf)/L`.
- Inductor delivers energy to the load and charges Cout.

**Steady state (volt-second balance):**
The inductor current returns to its starting value each cycle:
```
(Vin - Vout) × D × Ts = Vout × (1-D) × Ts
→  D = Vout / Vin
```

**Key insight**: The inductor acts as an energy storage element and current source. The capacitor filters the AC component of the inductor current. Together, the LC filter converts the pulsating switch-node voltage into a smooth DC output.

---

### Q2. Derive the duty cycle equation for an ideal buck converter in CCM.

**Answer:**

**Assumption**: Ideal switches, continuous conduction mode (inductor current never reaches zero).

**Volt-second balance principle**: In steady state, the average voltage across the inductor must be zero (otherwise the average current would drift to infinity or zero). This gives us the steady-state condition.

**Switch ON (time = D × Ts):**
```
VL_on = Vin - Vout
```

**Switch OFF (time = (1-D) × Ts):**
```
VL_off = 0 - Vout = -Vout    [switch node is at ground through freewheeling path]
```

**Volt-second balance:**
```
VL_on × D × Ts + VL_off × (1-D) × Ts = 0
(Vin - Vout) × D + (-Vout) × (1-D) = 0
Vin × D - Vout × D - Vout + Vout × D = 0
Vin × D = Vout
D = Vout / Vin
```

**Verification**: For Vin = 12 V, Vout = 3.3 V: D = 3.3/12 = 0.275 (27.5%). The switch is on for 27.5% of each period.

**Important caveats:**
- This is an ideal result. Real duty cycle is slightly higher due to conduction losses (switch and diode voltage drops).
- In DCM, the duty cycle equation changes — D ≠ Vout/Vin (it depends on load current too).
- In synchronous buck, the freewheeling "diode" is replaced by a MOSFET; same voltage balance if ideal, but body diode drop during dead time affects the result slightly.

---

### Q3. What is the difference between CCM and DCM? Describe the inductor current waveform in each mode.

**Answer:**

**CCM (Continuous Conduction Mode):**
- Inductor current never reaches zero during the switching cycle.
- The inductor current waveform is a triangle wave oscillating around the average value (Iout).
- Peak current: `IL_peak = Iout + ΔIL/2`
- Valley current: `IL_valley = Iout - ΔIL/2`
- Valley > 0 always (definition of CCM).

**DCM (Discontinuous Conduction Mode):**
- Inductor current reaches zero before the switch turns on again.
- The switching cycle has three phases: switch on (current ramps up), diode on (current ramps down to zero), idle (both switch and diode off, inductor current = 0, switch node floats).
- In the idle phase, the switch node can ring (LC resonance between the inductor and switch node capacitance) — this "ringing" is the characteristic signature of DCM on an oscilloscope.

**CCM/DCM boundary condition:**
```
ΔIL = 2 × Iout  (peak-to-peak ripple equals twice the average current)
Lcrit = Vout × (1-D) / (2 × fsw × Iout)
```
Operation: L > Lcrit → CCM; L < Lcrit → DCM.

**Key behavioural differences:**

| Property             | CCM                         | DCM                          |
|----------------------|-----------------------------|------------------------------|
| Duty cycle           | D = Vout/Vin (independent of load) | Varies with load current    |
| Transfer function    | Simple: single-pole at high f | More complex; load-dependent|
| Ripple               | Lower                        | Higher                       |
| Switch peak current  | Lower                        | Higher                       |
| Diode reverse recovery | Concern                   | Less concern (current hits zero)|
| Efficiency at light load | Can be worse (fixed switching losses) | Can be better             |
| Control loop         | Easier to compensate         | Harder (gain varies with load)|

---

### Q4. Calculate the inductor current ripple and critical inductance for a buck converter with Vin = 12 V, Vout = 3.3 V, Iout = 1 A, fsw = 500 kHz, L = 10 µH.

**Answer:**

**Duty cycle:**
```
D = Vout / Vin = 3.3 / 12 = 0.275
```

**Inductor current ripple:**
```
ΔIL = (Vin - Vout) × D × Ts / L
    = (Vin - Vout) × D / (fsw × L)
    = (12 - 3.3) × 0.275 / (500,000 × 10×10⁻⁶)
    = 8.7 × 0.275 / 5
    = 2.3925 / 5
    = 0.479 A
```

Alternatively, using the off-time formula (numerically identical):
```
ΔIL = Vout × (1-D) / (fsw × L)
    = 3.3 × (1 - 0.275) / (500,000 × 10×10⁻⁶)
    = 3.3 × 0.725 / 5
    = 2.3925 / 5 = 0.479 A ✓
```

**Critical inductance:**
```
Lcrit = Vout × (1-D) / (2 × fsw × Iout)
      = 3.3 × 0.725 / (2 × 500,000 × 1)
      = 2.3925 / 1,000,000
      = 2.39 µH
```

**Mode determination**: L = 10 µH > Lcrit = 2.39 µH → Operating in **CCM** at 1 A.

The converter would enter DCM at:
```
Iout_crit = Vout × (1-D) / (2 × fsw × L)
           = 3.3 × 0.725 / (2 × 500,000 × 10×10⁻⁶)
           = 0.239 A
```
Below ~240 mA load, this converter enters DCM.

**Ripple ratio**: ΔIL/Iout = 0.479/1 = 47.9% — a typical design target is 20–40%, so 10 µH is near the lower end of typical inductance for this application.

---

### Q5. Explain the input current waveform of a buck converter and why it matters.

**Answer:**

**Input current waveform (asynchronous buck):**

- When Q1 is ON: input current equals the inductor current (ramping, AC component present)
- When Q1 is OFF: input current = 0 (freewheeling path through diode, not through Vin)

The input current is therefore a **pulsating** current: rectangular pulses with amplitude ≈ Iout during the ON time, dropping to zero during OFF time.

**RMS input current:**
```
Iin_rms = Iout × √D  (for ideal rectangular pulses)
```

**Why this matters:**

1. **Input capacitor sizing**: The pulsating input current must be supplied by the input capacitor during the off-time. The capacitor must handle significant RMS ripple current. This is one of the most common causes of premature input capacitor failure.

2. **EMI/conducted noise**: The fast-switching pulsating current creates differential-mode EMI on the input lines. The input EMI filter must attenuate this before the power line.

3. **Input voltage ripple**: The voltage ripple on the input capacitor:
   ```
   ΔVin = Iout × D × (1-D) / (fsw × Cin)
   ```
   Must be kept small to avoid duty cycle modulation and efficiency degradation.

4. **Trace inductance matters**: The inductance in the loop formed by Vin, Q1, and Cin must be minimised to avoid large voltage spikes when Q1 turns off. Place Cin as close as physically possible to Q1.

**Contrast with output current**: The output current (from inductor to load) is the inductor current — a smooth triangular waveform with low AC content, much easier to filter.

---

### Q6. Compare synchronous and asynchronous (diode) rectification. When would you choose each?

**Answer:**

**Asynchronous rectification:**
- Freewheeling element is a diode (typically Schottky for low forward voltage drop).
- Power loss in diode: `P_diode = Vf × Iout × (1-D)` ≈ 0.3–0.5 V × Iout × (1-D)
- At Vout = 3.3 V, Iout = 2 A, D = 0.275, Vf = 0.4 V: `P_diode = 0.4 × 2 × 0.725 = 0.58 W`

**Synchronous rectification:**
- Freewheeling diode replaced by a low-side MOSFET (Q2), controlled to conduct when Q1 is off.
- Conduction loss: `P_Q2 = Iout² × Rds_on_Q2 × (1-D)` ≈ much less than diode at high current
- At 10 mΩ Rds_on: `P_Q2 = 4 × 0.01 × 0.725 = 0.029 W` — 20× less than the diode loss
- But adds gate drive loss: `Pgate_Q2 = Vgs × Qg × fsw`

**Comparison table:**

| Criterion            | Asynchronous (diode)        | Synchronous (MOSFET)        |
|----------------------|-----------------------------|-----------------------------|
| Efficiency (high current) | Lower (Vf loss)         | Higher (low Rds_on)        |
| Efficiency (low current)  | Better (no Qg loss)     | Worse (gate charge overhead)|
| Reverse current      | Naturally blocked by diode  | Can conduct reverse — needs dead-time control |
| Cost/complexity      | Simple; one fewer switch    | More complex; needs LS driver |
| DCM at light load    | Automatic (diode blocks)    | Must implement active DCM or use body diode |
| Short-circuit risk   | None (diode is passive)     | Shoot-through if dead time wrong |

**Choose asynchronous when:**
- Low power (< 5 W), where simplicity wins
- Always-DCM design (diode naturally blocks reverse current)
- Cost is primary driver

**Choose synchronous when:**
- Output current > ~0.5 A (efficiency gain outweighs added cost)
- Vout is low (e.g., 1 V) where even 0.3 V diode drop is 30% of output
- Need a complete integrated solution (most modern buck ICs are synchronous)

---

## Intermediate (Questions 7–13)

---

### Q7. What is dead time in a synchronous buck converter and what are the consequences of incorrect dead time?

**Answer:**

Dead time is the brief period between turning off one switch (Q1 or Q2) and turning on the other, during which both switches are off. It prevents simultaneous conduction of Q1 and Q2 (shoot-through), which would short Vin to GND.

**Shoot-through**: If both switches conduct simultaneously even for nanoseconds, the current spike through them is limited only by parasitic inductance. This causes:
- Extreme instantaneous power dissipation
- Switch destruction
- EMI spike
- In severe cases, immediate MOSFET failure

**Minimum dead time requirement:**
```
tdead_min > max(t_off_Q1, t_on_Q2) including gate drive delays and gate charge time
```
Typically 20–100 ns depending on gate driver speed and MOSFET capacitances.

**Consequences of excessive dead time:**
- During dead time, current flows through the body diode of Q2 (for falling inductor current) or Q1's body diode
- Body diode Vf ≈ 0.7 V (much higher than Q2's Vds when conducting at Rds_on)
- Power loss: `P_body = Vf × Iout × 2 × tdead × fsw`
- At Iout = 5 A, Vf = 0.7 V, tdead = 50 ns, fsw = 500 kHz:
  ```
  P_body = 0.7 × 5 × 2 × 50×10⁻⁹ × 500,000 = 175 mW
  ```
- At heavy load and high switching frequency, body diode conduction loss becomes significant.

**Adaptive dead time**: Many modern gate driver ICs monitor the body diode voltage and turn on the synchronous MOSFET immediately after the body diode starts conducting, minimising dead time loss while preventing shoot-through.

---

### Q8. Explain the bootstrap circuit in a high-side gate driver. Why is it needed?

**Answer:**

**Problem**: The high-side MOSFET (Q1) is an NMOS device with its source connected to the switch node (Vsw). To turn on Q1, the gate must be driven to `Vsw + Vgs_th + margin` — which is above Vin. There is no simple supply rail above Vin available.

**Bootstrap solution:**

A small capacitor (Cboot, typically 100 nF) is connected between the BOOT pin (gate driver's floating supply rail) and the SW node. A charging diode (Dboot) connects from a fixed supply (Vcc, typically 5–15 V) to the BOOT pin.

**Operation:**

1. **Q1 OFF, Q2 ON (SW ≈ 0 V)**: Dboot conducts, charging Cboot to approximately Vcc: `Vboot ≈ Vcc - Vf_Dboot`.
2. **Q1 turns ON**: SW rises to ≈ Vin. Cboot floats up with the SW node. Boot pin voltage rises to `Vin + Vcc` (approximately). Gate driver uses Cboot energy to drive Q1 gate above Vin.
3. **Q1 ON**: Cboot slowly discharges through the gate driver's supply current. Must be refreshed before next cycle.

**Bootstrap capacitor sizing:**
```
Cboot > Qg_Q1 / ΔVboot_allowed
```
Where ΔVboot_allowed is the maximum droop in boot voltage during Q1 on-time:
- Qg_Q1 = 10 nC, ΔVboot = 0.5 V: `Cboot > 20 nF` → use 100 nF for margin.

**Limitations:**
- Cannot operate at 100% duty cycle (boot cap must refresh, requires Q2 on-time each cycle)
- Maximum boot voltage rating must not be exceeded (`Vin + Vcc < Vboot_max`)
- For true 100% duty cycle, an alternative is needed (charge pump, separate isolated gate supply, or depletion-mode FET)

---

### Q9. How does output voltage ripple arise, and what are the ESR, ESL, and capacitance contributions?

**Answer:**

The inductor current ripple (ΔIL, a triangular waveform) flows through the output capacitor and creates voltage ripple through three mechanisms:

**1. Capacitance contribution:**
The charge/discharge of Cout by the triangular ripple current:
```
ΔVout_C = ΔIL / (8 × fsw × Cout)
```
This is the integral of the triangular current waveform. It has a parabolic shape, with peak deviation at the midpoint of each half-cycle.

**2. ESR contribution:**
The series resistance causes a voltage drop proportional to the instantaneous current. The ESR-induced ripple is triangular (same shape as IL):
```
ΔVout_ESR = ΔIL × ESR
```
This is in phase with the inductor current ripple.

**3. ESL contribution:**
The series inductance causes a voltage spike proportional to dI/dt at switching transitions:
```
ΔVout_ESL = ESL × dIL/dt = ESL × (Vin - Vout) / L  [during switch-on transition]
```
This appears as narrow spikes at the switching edges — most visible on high-bandwidth oscilloscope captures.

**Total ripple (approximate, for dominant ESR case):**
```
ΔVout ≈ ΔIL × √(ESR² + (1/(8×fsw×Cout))²)
```
The ESR and capacitance components are 90° phase-shifted, so they add in quadrature, not directly.

**Which dominates:**
- Low-ESR ceramics: capacitance-limited ripple dominates
- Electrolytics: ESR-limited ripple dominates
- Polymers: intermediate, often ESR-limited
- At very high fsw (> 1 MHz): ESL spikes become visible and may dominate peak-to-peak measurements

---

### Q10. What is the right-half-plane (RHP) zero, and does the buck converter have one?

**Answer:**

A right-half-plane (RHP) zero is a zero of the converter's control-to-output transfer function located in the right half of the s-plane (positive real part). It has a magnitude response that increases with frequency (like a left-half-plane zero — phase advance expected) but actually causes **phase lag** (180° worse than expected). This combination of gain increase with phase lag makes it fundamentally difficult to control.

**Physical explanation (boost converter, not buck):**

In a boost converter, when the duty cycle increases (switch on-time increases), the diode is blocked for longer. This momentarily **reduces** the output current to the load, even though more energy is being stored. Only after the switch turns off does the inductor deliver the extra stored energy. The response is initially in the wrong direction — this is the RHP zero behaviour.

**Does the buck converter have an RHP zero?**

**No.** The buck converter does NOT have a right-half-plane zero in CCM.

The control-to-output transfer function of a CCM buck converter (voltage mode control) is:
```
Gvd(s) = Vin / (1 + s/(Q×ω0) + (s/ω0)²)   × (1 + s×ESR×Cout) / (1 + s×ESR×Cout)
```
This is a second-order low-pass function with an ESR zero — all zeros are in the left half plane (or on the imaginary axis).

**RHP zero appears in:**
- Boost converter (CCM): `fRHP = Rload × (1-D)² / (2π × L)`
- Buck-boost (CCM): Similar expression
- Flyback (CCM): RHP zero from the transformer inductor behaviour

**Practical consequence for buck converters**: No bandwidth limitation from RHP zero — you can push the bandwidth high (up to fsw/5 to fsw/10). This is one reason buck converters can achieve faster transient response than boost or buck-boost converters.

---

### Q11. Describe how a synchronous buck converter achieves ZVS (Zero Voltage Switching) and why it matters.

**Answer:**

Zero Voltage Switching (ZVS) means turning on a MOSFET at the instant its drain-source voltage is zero (or near zero), eliminating the capacitive switching loss.

**Capacitive switching loss without ZVS:**
When a MOSFET turns on with Vds = Vin, the energy stored in its output capacitance Coss is dissipated:
```
Psw_cap = 0.5 × Coss × Vin² × fsw
```
At Vin = 12 V, Coss = 100 pF, fsw = 1 MHz: `Psw_cap = 0.5 × 100×10⁻¹² × 144 × 10⁶ = 7.2 mW` per switch.

**Achieving ZVS in a synchronous buck:**

During the dead time after Q2 turns off, the inductor current (if flowing in reverse, into the converter output — this requires DCM or forced reverse current) charges/discharges the switch node capacitance. If the inductor current is made sufficiently negative (reverse), it can fully ring the switch node from 0 V to Vin before Q1 turns on.

**Requirements for ZVS in synchronous buck:**
```
0.5 × L × IL_reverse² > 0.5 × Coss_total × Vin²
IL_reverse > Vin × √(Coss/L)
```

**Practical limitation**: Forcing significant reverse current through the inductor at light load (to achieve ZVS) wastes conduction energy. The ZVS gain must outweigh the reverse current loss. This trade-off is why ZVS synchronous bucks typically operate in ZVS only above a threshold current.

**More common in**: Resonant converters (LLC) where ZVS is a natural consequence of the resonant waveform shape, not a forced condition.

---

### Q12. How do you select the switching frequency, and what are the trade-offs?

**Answer:**

Switching frequency (fsw) is one of the most important design decisions, affecting component size, efficiency, cost, and transient response.

**Trade-offs:**

| Higher fsw benefits                         | Higher fsw penalties                         |
|---------------------------------------------|---------------------------------------------|
| Smaller inductor (L ∝ 1/fsw)               | Higher switching losses (Psw ∝ fsw)          |
| Smaller output capacitor                    | Higher gate drive losses (Pgate ∝ fsw)       |
| Faster transient response (higher bandwidth)| Core losses increase (for gapped ferrite)    |
| Smaller filter for EMI                      | Higher EMI frequency (may hit regulations)   |
| More compact design                         | MOSFET selection more critical (fast FETs)   |

**Practical guidelines:**

- **Consumer electronics (5–12 V systems)**: 200 kHz–2 MHz. Balances PCB area vs efficiency.
- **CPU/GPU VRMs (1 V, high current)**: 300 kHz–1 MHz. Compromise between core loss and inductor size.
- **Automotive (12 V input)**: 100–400 kHz. Switching losses increase faster with Vin.
- **High-efficiency server supplies**: 100–300 kHz. Magnetics quality justifies lower fsw.
- **GaN-based designs**: 1–5 MHz. GaN switches eliminate much of the switching loss penalty.

**Design rule of thumb**: Loop bandwidth cannot exceed fsw/5 to fsw/10 (Nyquist-like constraint). If fast transient response is required, push fsw higher.

**Efficiency optimum**: Maximum efficiency occurs where switching losses equal conduction losses. Optimal fsw can be found by:
```
fsw_opt = √(Pcond / (Psw_per_unit_freq))
```

---

### Q13. What is the cause of audible noise in a buck converter, and how is it mitigated?

**Answer:**

**Cause 1 — Switching frequency in the audio band (20 Hz–20 kHz):**

If fsw falls in the audio range, magnetic components (inductor, input capacitor) vibrate mechanically at the switching frequency due to magnetostrictive forces and Maxwell stress in the core material. This produces audible buzzing or whining.

**Mitigation**: Design fsw > 30 kHz (out of audible range). Most designs use > 100 kHz.

**Cause 2 — PFM/burst mode at light load:**

In light-load efficiency modes, the converter enters PFM (Pulse Frequency Modulation) or burst mode. The burst frequency drops with decreasing load:
```
fpulse ≈ (Vout × Iout) / (0.5 × L × Ipeak²)
```
At very light loads, fpulse can fall into the audible range (1–20 kHz), causing intermittent buzzing.

**Mitigation:**
- Forced PWM mode at the cost of efficiency (no burst mode)
- Minimum burst frequency clamped above 25 kHz (some ICs implement this)
- Spread spectrum modulation (dithers the burst frequency)
- Use a lower peak current in burst mode to keep frequency higher

**Cause 3 — Ceramic capacitor piezoelectric effect:**

X5R and X7R MLCC capacitors are mildly piezoelectric. High AC current at the switching frequency causes physical vibration of the ceramic dielectric. The vibration is transferred to the PCB, which acts as a speaker diaphragm.

**Mitigation:**
- C0G ceramics (non-piezoelectric, but low capacitance density)
- Polymer or electrolytic capacitors for bulk output filtering
- Mount ceramic caps in a sandwich configuration (mechanical cancellation)
- Mount on component side away from customer interface surface

---

## Advanced (Questions 14–18)

---

### Q14. Derive the control-to-output transfer function Gvd(s) for a voltage-mode CCM buck converter.

**Answer:**

**Circuit model (small-signal):**

In small-signal analysis, we perturb the duty cycle by d̂ and find the resulting output voltage perturbation v̂out. The switch is modelled by its average small-signal source: `v̂sw = Vin × d̂`.

**LC filter with load:**

The switch node voltage v̂sw drives an LC low-pass filter:
```
Gvd(s) = v̂out / d̂
```

For an LC filter with Cout having ESR (resr) and load Rload:

Step 1: Define the output impedance Zout:
```
Zout(s) = Rload || (resr + 1/(s×Cout))
         = Rload × (1 + s×resr×Cout) / (1 + s×(Rload + resr)×Cout)
         ≈ Rload × (1 + s×resr×Cout) / (1 + s×Rload×Cout)  [when Rload >> resr]
```

Step 2: Voltage divider between L and Zout:
```
v̂out = v̂sw × Zout / (s×L + Zout)
```

Step 3: Substitute and simplify:
```
Gvd(s) = Vin × Zout(s) / (s×L + Zout(s))
```

**Resulting standard form:**
```
Gvd(s) = Vin × (1 + s/ωz) / (1 + s/(Q×ω0) + s²/ω0²)

where:
  ω0 = 1/√(L×Cout)           [LC resonant frequency]
  Q = Rload × √(Cout/L) / (Rload + resr) ≈ Rload/√(L/Cout)  [quality factor]
  ωz = 1/(resr × Cout)       [ESR zero]
```

**Key observations:**
- Double pole at ω0: -40 dB/decade slope and -180° phase shift above this frequency
- ESR zero at ωz: +20 dB/decade, +90° phase boost (moves the second decade to -20 dB/decade)
- DC gain: Vin (at 0 dB reference, the gain in V/V-duty is Vin)
- No RHP zero — both poles are LHP, zero is LHP

**Phase at ω0**: Approximately -180° from the double pole. The compensator must provide sufficient phase boost to achieve the required phase margin at the crossover frequency.

---

### Q15. Explain slope compensation in peak current mode control for buck converters. Why is it required?

**Answer:**

Peak current mode control (PCMC) uses the inductor current directly as part of the control loop: a comparator turns off the high-side switch when the sensed inductor current ramp reaches a reference level set by the voltage error amplifier.

**Sub-harmonic oscillation (without slope compensation):**

For duty cycles D > 0.5, PCMC becomes unstable with a sub-harmonic oscillation at half the switching frequency. This is a fundamental instability of the current control inner loop, not the voltage loop.

**Physical explanation:**

Consider a perturbation ΔI0 added to the inductor current at the start of cycle n:
- Current ramp slope during ON time: `m1 = (Vin - Vout)/L` (positive)
- Current ramp slope during OFF time: `m2 = Vout/L` (positive magnitude, current decreasing)

The perturbation grows from cycle to cycle when:
```
ΔI(n+1) / ΔI(n) = m2 / m1 = Vout / (Vin - Vout) = D / (1 - D)
```

When D > 0.5: `m2/m1 > 1` → perturbation grows → sub-harmonic oscillation.
When D < 0.5: `m2/m1 < 1` → perturbation decays → stable.

**Slope compensation solution:**

Add an artificial ramp (slope `mc`) to the current sense signal (or subtract it from the reference). The stability criterion becomes:
```
mc ≥ m2 - m1 = (Vout - (Vin - Vout))/L = (2×Vout - Vin)/L
```

**Critical compensation**: `mc = m2/2 = Vout/(2L)` → stabilises for all D, eliminates sub-harmonic oscillation.

**Over-compensation (mc >> m2)**: The converter behaves more like voltage mode control — the current information is swamped by the slope, reducing the benefits of current mode. Use minimum necessary slope compensation.

**Minimum required slope:**
```
mc_min = m2 × (2D - 1) / 2  [for D > 0.5]
```
For D ≤ 0.5: no slope compensation required.

---

### Q16. How does the buck converter's efficiency change across load current? Explain the loss mechanisms.

**Answer:**

**Loss mechanisms and their load-current dependence:**

**1. Conduction losses (Pcond ∝ Iout²):**
```
Pcond = Iout² × (Rds_on_Q1 × D + Rds_on_Q2 × (1-D) + DCR)
```
Dominates at high load. Strong quadratic dependence.

**2. Switching losses (Psw ≈ constant + small dependence on Iout):**
```
Psw = 0.5 × Vin × Iout × (tr + tf) × fsw + 0.5 × Coss × Vin² × fsw
```
The first term is linear in Iout (cross-conduction during switching); the second is constant. Together, switching losses are roughly constant with load.

**3. Gate drive losses (Pgate ≈ constant):**
```
Pgate = Qg_total × Vgate × fsw
```
Fixed regardless of load current.

**4. Quiescent/bias losses (Pq ≈ constant):**
Controller IC current draw, housekeeping supplies — fixed.

**Efficiency curve shape:**

- At **heavy load**: conduction losses dominate (rising steeply with Iout²), efficiency decreases.
- At **medium load**: optimal balance — all loss terms roughly equal — peak efficiency.
- At **light load**: fixed losses (switching, gate, bias) become large relative to Pout, efficiency drops sharply.

**Efficiency calculation example** (12 V → 3.3 V, 500 kHz, D = 0.275):
```
At Iout = 2 A:
  Pcond = 4 × (10mΩ × 0.275 + 8mΩ × 0.725 + 5mΩ) = 4 × (2.75 + 5.8 + 5) mΩ = 54 mW
  Psw = 0.5 × 12 × 2 × 20ns × 500k + 0.5 × 100pF × 144 × 500k = 120 + 3.6 mW = 124 mW
  Pgate = 15nC × 10V × 500k = 75 mW
  Pq = 5 mA × 5V = 25 mW
  Ptotal_loss ≈ 280 mW
  Pout = 3.3 × 2 = 6.6 W
  η = 6.6 / (6.6 + 0.28) = 95.9%
```

---

### Q17. Describe the design of a synchronous buck converter for a CPU core voltage (Vout = 1 V, Vin = 12 V, Iout = 50 A, fsw = 500 kHz).

**Answer:**

This is a high-current, low-voltage application. Key challenges: duty cycle is D = 1/12 = 8.3% (very narrow ON pulse), extreme rms currents, thermal management.

**Step 1: Inductor selection**
```
ΔIL target = 30% of Iout = 15 A peak-to-peak ripple
L = Vout × (1-D) / (ΔIL × fsw) = 1 × 0.917 / (15 × 500,000) = 122 nH
```
Use a 100 nH or 150 nH inductor. Must handle 50 A + 7.5 A = 57.5 A saturation current.
DCR target: < 0.5 mΩ (conduction loss: 50² × 0.5 mΩ = 1.25 W — already significant).

**Step 2: Multiphase consideration**
At 50 A, a single-phase solution requires enormous MOSFETs and inductors. Real CPU VRMs use 4–12 phases. A 4-phase design divides current to 12.5 A per phase, making component selection manageable.

**Step 3: High-side MOSFET**
D = 8.3% — high-side is on for only 8.3% of the time. Dominant loss is switching loss, not conduction.
Select MOSFET with minimum Qg and Coss, even if Rds_on is moderate.

**Step 4: Low-side MOSFET**
(1-D) = 91.7% — low-side conducts most of the time. Dominant loss is conduction.
Select MOSFET with minimum Rds_on. Rds_on < 2 mΩ for 50 A single-phase.

**Step 5: Output capacitor**
Fast transient response requires high capacitance. CPU specifications typically require < 50 mV overshoot/undershoot for 50 A step:
```
Cout ≈ ΔI × tresponse / ΔVout
     ≈ 50 × 1µs / 0.05 = 1000 µF
```
Use multiple MLCCs (ceramic) for low ESL, with electrolytic bulk capacitors.

**Step 6: PCB considerations**
At 50 A, even 1 mΩ of trace resistance = 2.5 W loss. Use wide copper pours (2 oz minimum), multiple vias for current sharing, thermal management for both MOSFETs.

---

### Q18. How do you measure the efficiency of a buck converter accurately? What are the common sources of measurement error?

**Answer:**

**Basic setup:**
```
[DC Source] → [Power Analyser (Ch1: Vin, Iin)] → [Buck Converter] → [Power Analyser (Ch2: Vout, Iout)] → [Load]
```

**Efficiency:**
```
η = (Vout × Iout) / (Vin × Iin) × 100%
```

**Required instrumentation:**
- DC power analyser (e.g., Yokogawa WT series, Keysight N6705): simultaneous, synchronised measurement of V and I on input and output.
- Electronic load: programmable, low inductance — avoids lead resistance errors.
- Thermal equilibrium: measure efficiency only after temperature stabilises (10–20 min warm-up).

**Common measurement errors:**

1. **Four-wire (Kelvin) sensing not used**: Measuring voltage at the cable connection point rather than at the converter terminals includes cable resistance. A 10 mΩ cable with 2 A load introduces 20 mV error — 0.6% error at 3.3 V output.

2. **Power analyser bandwidth too low**: At high switching frequencies, the current waveform harmonics extend to 10× fsw. An analyser with insufficient bandwidth misses these harmonics, underreporting power. Use analysers with > 500 kHz bandwidth (e.g., Yokogawa WT3000E).

3. **Thermal equilibrium not reached**: Efficiency increases as temperature rises (Rds_on decreases, but magnetic losses increase). Measuring cold gives optimistic results. Always measure at thermal equilibrium.

4. **Ground current paths**: Oscilloscope ground clips connected to the converter create ground loops, injecting current that appears as a load. Use floating measurements or an isolated supply.

5. **Load wiring resistance**: Resistance in cables from converter to load creates voltage drop. Measure Vout directly at the converter's output terminals (Kelvin sense).

6. **Input capacitance pre-charging**: The input cap on the bench power supply acts as an energy source during transients, making efficiency appear higher. Use a long connection cable or add series resistance to characterise the converter's true input current.

**Loss breakdown method**: Measure efficiency, then identify each loss mechanism separately:
- Conduction: measure Rds_on at operating temperature using curve tracer
- Switching: calculate from Qg and switching waveform rise/fall times
- Magnetics: calorimetric measurement or Steinmetz calculation
- This allows targeted optimisation of the dominant loss mechanism
