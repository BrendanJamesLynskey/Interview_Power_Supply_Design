# LLC Resonant Converter — Interview Preparation

## Overview

The LLC resonant converter has become the dominant topology for high-efficiency isolated DC-DC conversion in the 100W–10kW range, particularly in telecom rectifiers, data-centre power supplies, EV on-board chargers, and laptop adapters. It achieves zero-voltage switching (ZVS) on the primary switches and near-zero-current switching (ZCS) on the secondary rectifiers, dramatically reducing switching losses. Understanding its gain curve, operating modes, design methodology, and light-load behaviour is a top-tier interview requirement for power electronics roles.

---

## Key Equations Reference

### Resonant Parameters

```
Resonant frequency:  fr = 1 / (2π × √(Lr × Cr))
Magnetising freq:    fm = 1 / (2π × √((Lr + Lm) × Cr))

Quality factor:      Q = √(Lr/Cr) / (n² × Rload)   [or Q = Lr × ω_r / (n² × Rload)]
Inductance ratio:    k = Lm / Lr  (typical 3–10)

FHA voltage gain:    M(fn, Q) = fn² × k / √{[(fn²(k+1) - 1)² + fn² × (fn² - 1)² × Q² × (k+1)²]}
  where fn = fsw / fr (normalised frequency)
```

### Gain Limits

```
Maximum gain (below resonance, ZVS, approaching ZCS on secondary):
  M_max ≈ (k+1)/(2k+1) × (1/Q_max)  [approximate, from FHA]

Unity gain crossing: occurs at fsw = fr regardless of Q (for LLC, always M=1 at fn=1)

Minimum gain (well above resonance):
  M_min → 1/(1+1/k) as fsw → ∞
```

### ZVS Condition

```
ZVS requires magnetising current Imag at end of half-cycle to charge/discharge switch capacitances:
  0.5 × Lm × Imag² > 2 × Coss × Vin²   (for each leg)

  Imag = Vin / (2 × fsw × Lm)  (peak-to-peak triangle waveform, half-period)
  Imag_peak = Vin / (4 × fsw × Lm)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Describe the operating principle of an LLC resonant converter. What are the three reactive elements and what does each contribute?

**Answer:**

An LLC resonant converter has three reactive elements that together determine its gain characteristic:

**Lr (resonant inductor):** A discrete inductor in series with the transformer primary. Together with Cr, it sets the series resonant frequency `fr = 1/(2π√(Lr×Cr))`. During operation, Lr resonates with Cr to produce sinusoidal-like current waveforms in the primary.

**Lm (magnetising inductance):** The transformer's magnetising inductance, which appears in parallel with the secondary reflected load. At the series resonant frequency fr, Lm is effectively in parallel with the series tank (Lr-Cr) and participates in a second resonance with Cr at a lower frequency fm. This two-resonance characteristic gives LLC its unique gain curve.

**Cr (resonant capacitor):** A capacitor in series with Lr. Its primary role is to block DC and resonate with Lr. It also prevents DC flux buildup in the transformer (important for balanced operation).

**Operation summary:**

The two primary switches operate at 50% duty cycle (complementary), creating a square wave voltage across the resonant tank (Lr-Cr in series with Lm). The tank filters this square wave, and the approximately sinusoidal current through the transformer primary magnetises the core and transfers power to the secondary.

The key insight: unlike a PWM converter, the LLC converter regulates output voltage by changing the switching frequency relative to the resonant frequency, not by changing the duty cycle. This is called frequency modulation (FM) control.

**Frequency regions:**
1. `fsw < fm`: Both Lr and Lm participate in resonance. Operation below fm causes capacitive input impedance — ZVS is lost on primary switches. This region is avoided.
2. `fm < fsw < fr`: Lm is clamped by the secondary voltage (diodes conducting). Only Lr resonates with Cr. Gain > 1 possible. ZVS maintained.
3. `fsw = fr`: Unity voltage gain regardless of load. This is the nominal operating point for many designs.
4. `fsw > fr`: Gain < 1. Inductive input impedance maintained — ZVS preserved. Gain decreases with frequency.

---

### Q2. Draw and explain the LLC voltage gain curve. Why does it have a different shape from a standard LC resonant circuit?

**Answer:**

**Standard series LC resonant circuit gain:** Peak at fr, gain → 0 at DC and at high frequency. Only one resonant peak.

**LLC voltage gain curve characteristics:**

The LLC has two resonant frequencies:
```
fr = 1/(2π√(Lr×Cr))    [series resonance — Lm clamped by secondary]
fm = 1/(2π√((Lr+Lm)×Cr)) [parallel resonance — Lm unclamped at no-load]
```
Since Lm >> Lr typically, fm << fr.

**Gain curve description (qualitative):**

At fn = 0 (DC): Open circuit secondary, infinite gain theoretically
At fn = fm/fr: The gain peaks sharply (Q-dependent; larger Q → lower peak)
At fn = 1 (fsw = fr): Gain = 1 always, regardless of Q or load (this is the unity gain crossover)
At fn > 1 (above resonance): Gain < 1, decreases approximately with 1/fn²

**Q dependence:**

Higher Q (higher load, lower Rload) flattens and lowers the gain peak:
- Heavy load (high Q): gain curve has low, broad peak; limited regulation range
- Light load (low Q): gain curve has high, sharp peak; wide regulation range

**The crucial design implication:**

For regulation from full load to some minimum load (e.g., 10% to 100%), the converter must achieve all required gain values within a feasible frequency range. The design must ensure:
```
M(fn_min, Q_max) ≥ M_required_at_Vinmax   [gain at high frequency, full load]
M(fn_max, Q_min) ≤ M_required_at_Vinmin   [gain at low frequency, light load]
```
The converter cannot regulate below the minimum gain point on the curve, which is why LLC converters typically require burst mode or pulse skipping at very light load.

---

### Q3. What is the First Harmonic Approximation (FHA) and how is it used in LLC design?

**Answer:**

**What is FHA:**

The FHA assumes that the resonant tank filters out all harmonics of the switching frequency, leaving only the fundamental component. Under this assumption:
- The square wave voltage applied to the tank is replaced by its fundamental sine component: `V_primary_1st = (4/π) × (Vin/2) × sin(ω_sw × t)`
- The tank current is sinusoidal
- The rectifier and load are replaced by an equivalent AC resistance: `Rac = (8/π²) × n² × Rload`

This transforms the LLC into a simple linear AC circuit (series Lr-Cr with parallel Lm-Rac) that can be solved analytically.

**Voltage gain from FHA:**
```
M(fn, Q, k) = |Vout_AC / Vin_AC|

M = k × fn² / √[(k × fn² × (k+1) × (fn² - 1))² + (k × fn² - (k+1) × (fn² - 1))² × Q²]
```

Simplified for fn = 1 (at series resonance): M = 1 (as expected)

**Accuracy of FHA:**

FHA is accurate near the resonant frequency where harmonics are indeed strongly attenuated by the tank. Accuracy degrades:
- Below resonance (above M=1): significant current harmonics → FHA underestimates gain
- Well above resonance (M << 1): tank no longer sinusoidal → FHA overestimates gain

In practice, FHA provides sufficient accuracy for initial design. Verification is done with SPICE simulation or dedicated LLC design tools (e.g., TI's Webench, PSIM).

**Using FHA for design:**

1. Choose target operating frequency range [fn_min, fn_max]
2. Choose k = Lm/Lr (larger k → wider gain range but less ZVS energy)
3. Solve for Q that places the gain curve to cover the required Vout range over the Vin range
4. Compute Lr, Lm, Cr from the resonant frequency and Q

---

### Q4. How does the LLC converter achieve ZVS on the primary switches?

**Answer:**

**ZVS mechanism:**

ZVS occurs when the MOSFET's body diode begins to conduct before the gate signal turns on the MOSFET. This requires the switch node voltage to reach 0V (or Vin) before gate turn-on.

**In the LLC, ZVS is enabled by the magnetising current (Imag):**

During the half-cycle transition:
1. Both primary switches are off (dead time)
2. The magnetising current Imag (which was flowing in one direction) must now charge/discharge the output capacitances (Coss) of both switches to swing the leg voltage from 0V to Vin (or vice versa)
3. If Imag is large enough, it fully charges/discharges Coss within the dead time
4. Once the switch node reaches Vin, the body diode of the MOSFET that's about to turn on starts conducting
5. The gate signal turns on the MOSFET — at zero voltage → ZVS

**Energy balance for ZVS:**
```
Energy in Lm: E_Lm = 0.5 × Lm × Imag_peak²
Energy needed: E_Coss = 2 × 0.5 × Coss × Vin² = Coss × Vin²

ZVS condition: E_Lm > E_Coss
0.5 × Lm × Imag² > Coss × Vin²
```

**Imag_peak in terms of design parameters:**
```
Imag_peak = Vin / (4 × fsw × Lm)
```

**ZVS condition restated:**
```
0.5 × Lm × (Vin / (4 × fsw × Lm))² > Coss × Vin²
Vin² / (32 × fsw² × Lm) > Coss × Vin²
1 / (32 × fsw² × Lm) > Coss
Lm < 1 / (32 × fsw² × Coss)
```

This means for a given Coss and fsw, there is a maximum Lm that allows ZVS. Smaller Lm → more magnetising current → better ZVS but:
- More circulating current (winding copper loss)
- Smaller k = Lm/Lr → narrower gain range
- Lower efficiency from Imag-induced conduction losses

**Why LLC maintains ZVS at all loads:**

Unlike PSFB, where ZVS depends on primary current (load-dependent), LLC's ZVS relies on the magnetising current, which is independent of load (it is set by Lm and frequency). The LLC therefore maintains ZVS from full load down to zero load — a significant advantage.

---

### Q5. Describe the LLC converter's behaviour in burst mode at light load. Why is burst mode used?

**Answer:**

**Why burst mode is needed at light load:**

At very light loads, the LLC must operate at high frequency to reduce gain below the minimum required. However:
1. Very high frequency operation increases switching losses and gate drive power
2. Fixed losses (core loss, driver bias) become significant relative to output power
3. Below some frequency limit, the gain curve cannot provide sufficient reduction in Vout

**When burst mode activates:**

As load decreases toward zero, the control loop increases frequency to reduce gain. When the frequency reaches a defined maximum (`fsw_max`), the converter cannot reduce gain further. Beyond this point, burst mode is used.

**Burst mode operation:**

The converter alternates between:
- **Active bursts:** Operates at a fixed high frequency (or resonant frequency) for a burst duration
- **Idle periods:** All switches off; output capacitor maintains Vout

The output voltage droop during idle periods triggers the next burst:
```
Burst duration ∝ Pout (more output power → longer bursts)
Period (bursts + idle) varies to maintain average output power
```

**Efficiency in burst mode:**

During active bursts, the converter operates near resonance (high efficiency). During idle periods, losses are minimal (just leakage currents). Average efficiency is:
```
η_burst ≈ η_burst_active × (T_active / (T_active + T_idle))
```

At very light load, T_active << T_idle, and the fixed losses per burst are spread over a long period → high efficiency achievable.

**Audible noise concern:**

If the burst frequency falls in the audible range (20 Hz – 20 kHz), a high-pitched whine may be heard. This is a regulatory concern for consumer products (EV chargers, laptop adapters). Solutions:
1. Push burst frequency above 20 kHz (maintain silence)
2. Use random burst timing (spread spectrum of burst frequency)
3. Variable burst size (maintain burst frequency while varying burst length)

---

### Q6. What is ZCS in the LLC converter's secondary rectifier and why does it matter?

**Answer:**

**ZCS (Zero Current Switching):**

In the LLC operating at exactly fr (series resonant frequency), the primary current is sinusoidal and the secondary diode current naturally reaches zero at the end of each half-cycle — the diode turns off at zero current. This eliminates reverse recovery charge (Qrr) in the secondary diodes.

**Why it matters:**

Reverse recovery in high-voltage diodes (e.g., 100V fast-recovery diodes for 48V output from 400V primary) is a significant loss mechanism:
```
P_rr = 0.5 × Qrr × V_reverse × fsw
```
For a fast-recovery diode with Qrr = 50 nC, V_reverse = 100V, fsw = 150kHz:
```
P_rr = 0.5 × 50e-9 × 100 × 150e3 = 0.375 W per diode
```

With ZCS, Qrr = 0 → this loss is eliminated.

**ZCS conditions:**

At fn = 1 (fsw = fr): True ZCS — secondary current reaches zero at exactly the diode commutation point.

At fn > 1 (above resonance): Secondary current still reaches zero naturally (the tank current is sinusoidal and returns to zero) → approximate ZCS maintained.

At fn < fr (below resonance): The secondary current may not reach zero before forced commutation → non-ZCS, reverse recovery losses return.

**Design implication:**

For highest efficiency, operate near or above the series resonant frequency where both ZVS (primary) and ZCS (secondary) are achieved simultaneously. This is the "sweet spot" of LLC operation.

**Synchronous rectification in LLC:**

SR MOSFETs replace secondary diodes for lowest conduction loss. However, SR timing is critical:
- MOSFET must turn off before secondary current reverses (or current flows backwards into transformer, creating overcurrent)
- Zero-crossing detection of secondary current needed (gate driver IC or comparator)
- Popular SR driver ICs: TI LM5046, NXP TEA1792, ON Semi NCP4306

---

## Intermediate (Questions 7–12)

---

### Q7. How do you design LLC resonant tank components (Lr, Cr, Lm) given a specification?

**Answer:**

**Given:** Vin = 380–420V (narrow range, PFC output), Vout = 48V, Pout = 1kW, fr = 150kHz.

**Step 1 — Select turns ratio n:**

At nominal Vin = 400V, fn = 1 (at resonance, M = 1 always):
```
n = Vin / (2 × Vout) = 400 / (2 × 48) = 4.17 → use n = 4 (8:2 winding ratio)
```
With n=4: at fn=1, Vout_no_loss = 400/(2×4) = 50V. Losses bring this to ~48V at full load — acceptable.

**Step 2 — Select k = Lm/Lr and calculate Q:**

Gain range needed: 48V to 50V at resonance = 0.96 to 1.0 (very narrow for tight Vin). For Vin variation ±5%:
```
M_min = 0.96, M_max = 1.04 (to handle Vin variation)
```

Choose k = 7 (typical starting point). Calculate maximum Q for full load:
```
Rload = Vout²/Pout = 48²/1000 = 2.304 Ω  (secondary side)
Rac = (8/π²) × n² × Rload = 0.811 × 16 × 2.304 = 29.9 Ω  (primary-referred)
```

Choose Lr first from desired fr:
```
If we target Lr = 100 µH:
Cr = 1 / ((2π×fr)² × Lr) = 1 / ((2π×150e3)² × 100e-6) = 1 / (8.88e11 × 100e-6) = 11.3 nF
```

Q at full load:
```
Q = Lr × ω_r / Rac = 100e-6 × (2π×150e3) / 29.9 = 100e-6 × 942,477 / 29.9 = 3.15 → too high
```

High Q limits gain range. Reduce Lr:
```
Try Lr = 30 µH:
Cr = 37.6 nF → use 39 nF (standard value)
Q_full = 30e-6 × 942,477 / 29.9 = 0.945  (more reasonable)
```

**Step 3 — Select Lm:**
```
Lm = k × Lr = 7 × 30 = 210 µH
```

**Step 4 — Verify fm:**
```
fm = 1/(2π×√((Lr+Lm)×Cr)) = 1/(2π×√(240e-6×39e-9))
   = 1/(2π×√(9.36e-12)) = 1/(2π×3.06e-6) = 1/(19.23e-6) = 52 kHz
```

Operating range 52 kHz to 150 kHz — wide enough.

**Step 5 — ZVS verification:**
```
Imag_peak = Vin / (4 × fr × Lm) = 400 / (4 × 150e3 × 210e-6)
           = 400 / 126 = 3.17 A

Coss (for 600V MOSFET ≈ 200 pF effective):
E_ZVS = 0.5 × Lm × Imag² = 0.5 × 210e-6 × 10.05 = 1.055 mJ
E_needed = 2 × Coss × Vin² = 2 × 200e-12 × 160,000 = 64 µJ

E_ZVS >> E_needed → ZVS achieved easily
```

---

### Q8. How does frequency change with load in an LLC converter, and why does this complicate the design?

**Answer:**

**Load-frequency relationship:**

At a fixed Vout regulation target, as load changes:
- Heavy load → high Q → gain curve flattened → converter must operate closer to fr to achieve needed gain → lower frequency
- Light load → low Q → gain curve peaked → converter operates further above fr → higher frequency

**Direction:**
- Increase load → decrease frequency (toward fr)
- Decrease load → increase frequency (away from fr)

This is the opposite of what many engineers expect (in most regulators, more load → wider pulses or more action, not less frequency).

**Design complications:**

1. **Maximum frequency constraint:**
   At very light load, frequency may become excessively high. Common limit: 2–3× fr. Above this:
   - Gate drive and switching losses dominate
   - Gain curve insensitivity (flat region above resonance)
   - LLC switches to burst mode

2. **Minimum frequency constraint:**
   Below fm, the LLC loses ZVS and the secondary current waveform becomes non-sinusoidal. Operating below fm is generally prohibited. The minimum frequency is bounded by fm.

3. **Gain at minimum frequency:**
   The maximum achievable gain (at fm, light load) must exceed the required gain at lowest Vin:
   ```
   M_max(fn=fm/fr, Q→0) = (k+1)/(k) = 1 + 1/k
   ```
   For k=7: M_max = 1.143 → can boost output by 14.3%.

4. **Non-linear control characteristic:**
   The relationship between fsw and Vout is non-linear (it follows the gain curve). Simple PID controllers may be unstable or slow if the gain characteristic changes sharply. Digital control or variable-gain analogue control is preferred.

**Practical operating range:**

Most LLC designs operate:
- Nominal (Vin nominal, full load): fsw slightly above fr (fn = 1.0–1.1)
- High Vin, full load: fsw increases to reduce gain
- Low Vin, full load: fsw decreases toward fr (potentially below for boost gain)
- Light load: fsw increases toward fsw_max, then burst mode

---

### Q9. What is the importance of the dead time in an LLC converter and how do you select it?

**Answer:**

Dead time in LLC is the interval between one switch turning off and the complementary switch turning on. Unlike PWM converters where dead time is a brief protective interval, LLC dead time is a functional element of ZVS.

**Functions of dead time:**

1. **ZVS transition:** During dead time, Imag must swing the switch node from 0 to Vin (or vice versa). The time required is approximately:
   ```
   t_dead_min = π × √(2 × Coss × Vin / Imag_peak) × (1/(2π))...
   More practically: t_dead ≈ 4 × Coss × Vin / Imag_peak
   ```
   For Coss=200pF, Vin=400V, Imag=3A:
   ```
   t_dead = 4 × 200e-12 × 400 / 3 = 106 ns
   ```

2. **Preventing shoot-through:** Dead time prevents simultaneous conduction of HS and LS switches.

**Selection criteria:**

```
t_dead > 4 × Coss_eff × Vin / Imag_peak    [sufficient for resonant transition]
t_dead < (T/2 - t_active)                   [must be less than remaining half-period]
```

**Effect of dead time that's too long:**

If dead time exceeds the resonant transition time, the body diode conducts for excess time after the switch node reaches its final value. This causes:
- Additional conduction loss (body diode Vf > MOSFET Vds)
- Reverse recovery of body diode when MOSFET finally turns on
- For GaN: no body diode (Qoss only), so extended dead time means the Coss begins to charge back up — the ZVS window is lost

**GaN in LLC:**
GaN FETs have no body diode (gate-controlled channel only). The dead time must be precisely controlled to coincide with the resonant transition window. Too short: shoot-through. Too long: ZVS lost as Coss recharges. Many LLC designs use GaN with adaptive dead time control.

**Fixed vs. adaptive dead time:**
- Fixed: Simple controller, optimise dead time for worst case (minimum Imag = worst case)
- Adaptive: Measure when body diode starts and stops conducting; adjust dead time each cycle
- Adaptive required for wide-range designs (GaN, high efficiency targets)

---

### Q10. How does the LLC converter differ in controllability from a PWM converter?

**Answer:**

**PWM converter control:**
- Single control variable: duty cycle D
- Linear relationship: Vout ∝ D (approximately)
- Direct, well-understood transfer function
- Wide variety of analogue and digital controllers available

**LLC frequency modulation control:**

- Single control variable: switching frequency fsw
- Non-linear relationship: Vout = f(fsw, Q, k) — follows gain curve
- Transfer function is non-linear and changes with operating point (Q changes with load)
- Different small-signal behaviour above vs. below resonance

**Small-signal model of LLC:**

The LLC has a complex control-to-output transfer function that depends on the operating point. Near resonance (fn ≈ 1), the small-signal gain is relatively well-behaved. Far from resonance, the gain changes rapidly with frequency.

For designs operating above resonance:
```
Gvf(s) ≈ -G0 / (1 + s/ωp)   [dominant pole behaviour]
```
The negative gain (decrease frequency → increase gain → higher Vout) means the control sense is inverted compared to PWM.

**Compensation challenges:**

1. The plant gain changes 2–4× across the load range (Q varies).
2. The gain has a different sign relationship depending on operating region.
3. The resonant tank introduces complex poles that can destabilise compensation.

**Solutions:**

1. **Type II or Type III compensation with conservative bandwidth:** Cross over well below the resonant frequency to avoid the tank poles. Typical crossover: 1–5% of fr.

2. **Gain-scheduled compensation:** Different compensator gains for different operating regions (implemented in digital control).

3. **CLLC or improved variants:** Bidirectional resonant converter adds resonant elements on both sides, making the transfer function more symmetric.

4. **Digital control:** Microcontrollers (DSP or ARM Cortex-M4) with LLC control algorithms (e.g., TI's Digital Power SDK for LLC) manage gain scheduling, burst mode, frequency limits, and dead time automatically.

---

### Q11. Compare LLC with PSFB for a 400V bus, 12V/100A output power supply. Which is better and why?

**Answer:**

**Application:** 400V bus → 12V, 100A = 1200W isolated converter

**LLC analysis:**

Turns ratio: n = Vin/(2×Vout) = 400/(2×12) = 16.7 → use 17:1 (34:2 winding)

Secondary peak current: Iout = 100A → very high secondary current density, requires many parallel diodes or SRs, thick copper windings. The transformer secondary winding (2 turns of very heavy gauge copper foil) is challenging to build.

Secondary diode/SR voltage: Each SR MOSFET sees Vout + rectifier spike ≈ 15–20V; use 30V MOSFETs.

The 100A secondary current and high turns ratio make the transformer design challenging but feasible.

**PSFB analysis:**

Turns ratio: n = Vin×D_max/Vout = 400×0.45/12 = 15 → n=15:1 (30:2 winding)

Primary current: Iout/n × efficiency = 100/15/0.95 = 7A average.

The PSFB has similar transformer challenges (17:1 or 15:1 turns ratio, heavy secondary winding).

**Key differences at this operating point:**

| Factor | LLC | PSFB |
|--------|-----|------|
| Primary switching loss | Very low (ZVS always) | Low (ZVS at full load) |
| Secondary switching loss | Very low (ZCS) | Moderate (SR timing) |
| Full-load efficiency | 96–97% | 94–95% |
| Light-load efficiency | 90–94% (burst) | 88–92% |
| Transformer design | Harder (higher turns ratio) | Moderate |
| Control complexity | High (non-linear FM) | Moderate (PWM) |
| Input range | Narrow preferred | Wide range OK |

**Recommendation:**

For this application (well-regulated 400V PFC output feeding a 12V high-current rail):
- **LLC is preferred** for highest efficiency (1% gain at 1200W = 12W saved = significant in high-density designs)
- Input range is narrow (PFC output ≈ ±5%), which plays to LLC's strength
- The 100A secondary current is challenging for both topologies equally

If the input range were wide (e.g., 200–400V) or the output voltage needed to be programmable, PSFB would be the better choice.

---

### Q12. What safety protections are critical in an LLC converter and how are they implemented?

**Answer:**

**1. Over-frequency / under-voltage protection:**

At no-load, LLC operates at high frequency. If frequency exceeds safe limit (e.g., gate driver maximum frequency, or fsw > 3fr where ZVS may be lost), the converter must shut down or burst.
- Implementation: Frequency comparator in controller IC triggers fault flag above fsw_max.

**2. Over-current protection:**

LLC does not inherently limit current (unlike peak-current mode PWM). If the secondary is overloaded or shorted:
- Primary current can grow very large near resonance (Q → ∞ at short circuit)
- Primary MOSFET current must be monitored
- Implementation: Current transformer on primary or Rds_on sensing; threshold set below primary switch rating.

**3. Below-resonance operation prevention:**

Operating below fm causes ZVS loss and potentially capacitive input impedance (negative input resistance → instability). Controller must prevent steady-state operation below fm.
- Implementation: Minimum frequency clamp in frequency control loop (set fsw_min > fm + margin).

**4. Startup sequence:**

At startup, Cout is discharged (Vout = 0). The LLC would see a short-circuit load, driving it into heavy CCM-like operation with very high current.
- Implementation: Soft-start by sweeping frequency from high to low. Start at fsw >> fr (very low gain) and ramp frequency down to setpoint slowly. Limits inrush current.

**5. Resonant capacitor voltage:**

Cr sees AC voltage that can exceed supply voltage during startup or fault. Must be rated accordingly:
- Implementation: Use film capacitors (lower ESR, stable, high peak voltage capability) rather than ceramics for Cr. Rating typically 2× nominal Vcr.

**6. Transformer primary short protection:**

A shorted transformer winding creates a direct short on Cr-Lr, producing extremely high resonant currents.
- Implementation: Fast overcurrent protection with sub-microsecond response (comparator-based, not through MCU).
