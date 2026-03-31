# Multiphase Converters — Interview Preparation

## Overview

Multiphase (interleaved) converters are the dominant architecture for high-current, low-voltage DC-DC conversion — most notably in CPU voltage regulator modules (VRMs), GPU power stages, server power stages, and battery chargers. Multiple converter phases operate in parallel but with evenly spaced switching instants, providing ripple cancellation at the input and output, improved transient response, and distributed thermal management. Understanding interleaving mathematics, current sharing, phase shedding, and coupled inductor design is essential for roles at semiconductor companies, cloud computing hardware teams, and EV powertrain engineers.

---

## Key Equations Reference

### Ripple Cancellation

```
N-phase interleaved input ripple frequency:   f_ripple_in  = N × fsw
N-phase interleaved output ripple frequency:  f_ripple_out = N × fsw

Output ripple reduction factor (N phases, equal D):
  r_N / r_1 = (N × (1 - N×D) × (N×D - (N-1))) / ((1 - D) × D × N²)
  [valid when (k-1)/N ≤ D ≤ k/N, for k-th ripple region]

Ripple-cancellation null points at D = k/N (integer k from 1 to N-1)
  → Zero ripple theoretically at these duty cycle values
```

### Phase Timing

```
Phase offset (radians): θ_k = 2π × (k-1) / N   for phase k = 1, 2, ..., N
Phase offset (time):    t_k  = (k-1) × Ts / N
```

### Coupled Inductors

```
Coupled inductor ripple reduction:
  ΔIL_coupled = ΔIL_uncoupled × (1 - k_coupling)/(1 + (N-1) × k_coupling)
  [for inverse coupling, k_coupling < 0]
```

### Transient Response

```
Maximum inductor current slew rate per phase: dI/dt = Vin / L
Total N-phase slew rate: (dI/dt)_total = N × Vin / L_per_phase
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Explain the principle of interleaving in a multiphase buck converter. Why does it reduce input and output ripple?

**Answer:**

**Interleaving principle:**

In an N-phase interleaved buck converter, N identical buck converter stages share the same input and output but switch at equal time offsets of Ts/N (one period divided by N). Each phase produces a triangular inductor current ripple at frequency fsw. The output current is the sum of all phase currents.

**Output ripple cancellation:**

The triangular current ripples from each phase are phase-shifted by 360°/N. Because they are phase-shifted versions of the same waveform, they partially cancel when summed:
- At duty cycle D = 1/N, 2/N, 3/N, ..., (N-1)/N: the ripples cancel completely (theoretical zero ripple)
- At other duty cycles: partial cancellation — ripple is significantly reduced vs. single phase

**Mathematical explanation:**

For a 2-phase converter at D = 0.5:
- Phase 1 current: triangular wave peaking at t = 0.25Ts
- Phase 2 current: same triangular wave, shifted by Ts/2, peaking at t = 0.75Ts
- Sum: the rising slope of phase 1 coincides with the falling slope of phase 2 → cancellation

**Input ripple cancellation:**

Each phase draws pulsed input current during its on-time. The input capacitor must supply these pulses. With N-phase interleaving, the input current pulses overlap and the residual ripple is at N × fsw with much smaller amplitude.

**For a single-phase converter (N=1):**
```
I_rms_cin = Iout × √(D × (1-D))   [worst case at D=0.5: I_rms = Iout/2]
```

**For an N-phase converter:**
```
I_rms_cin_N ≈ Iout × √(D/N × (1 - D/N))   [much smaller than single phase]
```

For N=4, D=0.25: `I_rms_cin = 0.25 × √(0.0625 × 0.9375) ≈ Iout × 0.061`
vs. single phase at D=0.25: `I_rms_cin = Iout × √(0.25 × 0.75) ≈ Iout × 0.433`

A 7:1 reduction in input capacitor RMS current — the input capacitor can be dramatically reduced in size and cost.

---

### Q2. Derive the output ripple reduction factor for a 4-phase interleaved buck converter at D = 0.25.

**Answer:**

At D = 0.25 with 4 phases, the duty cycle equals exactly 1/N. This is a ripple null point.

**Single-phase ripple:**
```
ΔIL_1phase = Vout × (1-D) / (fsw × L) = Vout × 0.75 / (fsw × L)
```

**4-phase analysis at D = 0.25:**

Phase offsets: 0, Ts/4, Ts/2, 3Ts/4

Each phase is ON for D × Ts = Ts/4. During a full period Ts:
- Phase 1 on: [0, Ts/4]
- Phase 2 on: [Ts/4, Ts/2]
- Phase 3 on: [Ts/2, 3Ts/4]
- Phase 4 on: [3Ts/4, Ts]

At any instant, exactly one phase is on (since N × D = 1 exactly). The total inductor current consists of four triangular waves whose peaks and valleys interleave perfectly.

**Summation at D = 1/N (exact nulling):**

When the sum is taken:
```
I_total(t) = Σ IL_k(t)
```
Because the waveforms are uniformly spaced and at D=1/N exactly one is rising while others fall at the same rate, the net sum is a constant (no ripple):
```
ΔI_total = 0  (theoretical)
```

**In practice:**
- Component tolerances (L mismatch, timing jitter) create residual ripple
- Ripple appears at 4 × fsw with very small amplitude
- The ripple null condition is fragile — small D changes move away from null rapidly

**At D ≠ 0.25:**

Using the general formula for N=4, 0.25 < D < 0.5 (second ripple region):
```
ripple_ratio = (N × (1 - N×D) × (N×D - (N-1))) / ((1-D) × D × N²)
             = (4 × (1 - 4D) × (4D - 3)) / ((1-D) × D × 16)
```

At D = 0.375 (midpoint of second region):
```
= (4 × (1 - 1.5) × (1.5 - 3)) / ((0.625) × 0.375 × 16)
= (4 × (-0.5) × (-1.5)) / (3.75)
= 3 / 3.75 = 0.8
```

So the 4-phase output ripple at D=0.375 is 80% of the single-phase ripple (only 20% reduction). The effectiveness depends strongly on duty cycle proximity to the null points.

---

### Q3. How is current sharing achieved in a multiphase converter and why does it matter?

**Answer:**

**Why current sharing matters:**

If current sharing is not maintained, one phase carries more than its fair share of the total load current. This phase overheats, its inductor may saturate, and it may fail — even though the overall current is within specification.

**Sources of current imbalance:**

1. **Inductor mismatch:** If L1 ≠ L2, the phase with smaller L has higher current ripple → higher average current for the same switch timing.
2. **MOSFET Rds_on mismatch:** Phase with lower Rds_on carries more current (lower voltage drop).
3. **Gate drive timing mismatch:** Different gate drive propagation delays change effective duty cycle.
4. **Temperature differences:** Rds_on is temperature-dependent; hotter phases carry less current (self-balancing via positive temperature coefficient of Rds_on).

**Current balancing methods:**

**1. Droop current sharing (resistive droop):**
Each phase has a sense element (DCR or sense resistor). The sensed current is fed back to adjust the effective duty cycle of that phase:
```
V_control_phase_k = V_reference - R_droop × I_phase_k
```
Phases with higher current reduce their duty cycle → lower current → balance achieved.

**2. Active current sharing with control loop:**

Controller monitors each phase current individually. An averaging network finds the mean current. Each phase is adjusted toward the average:
```
D_phase_k = D_nominal + Kcc × (I_average - I_phase_k)
```
Kcc is the current sharing loop gain. This is used in digital VRM controllers (e.g., Renesas ISL69259, Intersil ISL68201).

**3. DCR sensing for current measurement:**

Each inductor's DCR is used as a lossless current sensor by placing an RC network in parallel:
```
R_sense parallel with L:  V_C = I_phase × DCR  (when RC = L/DCR)
```
The measured voltage is proportional to phase current without any power loss. Temperature compensation is needed since DCR changes with temperature (use NTC thermistor in the RC network).

**Steady-state current balance specification:**
Most VRM standards (Intel VR14, JEDEC) require current sharing within ±5% of the average phase current across all phases at full load.

---

### Q4. What is phase shedding and when should it be used?

**Answer:**

**Phase shedding (phase dropping):** At light load, some converter phases are turned off (shed), leaving fewer active phases to supply the load. The active phases continue operating at their normal switching frequency and duty cycle.

**Why phase shedding improves light-load efficiency:**

At light load, each phase carries less current. The fixed losses per phase (gate drive, switching losses, controller power, standby bias) remain constant. If 4 phases each have 50 mW of fixed loss, the total is 200 mW regardless of load current.

By shedding 3 phases and operating with only 1 active phase:
- Fixed losses drop to 50 mW (only 1 phase)
- The remaining phase carries the full light load current (say 1A instead of 0.25A per phase)
- The single-phase efficiency at 1A is higher than 4-phase at 0.25A each

**Trade-off — increased output ripple:**

With fewer phases, ripple cancellation is reduced. Shedding from 4 phases to 1 phase increases output ripple by up to 4×. This requires:
- Larger output capacitance (to maintain ripple spec), or
- Accepting larger ripple at light load (if processor voltage tolerance allows), or
- Burst mode on the single remaining phase at very light load

**Phase shedding thresholds:**

Set by controller IC or firmware based on total output current measurement:
```
Shed to N-1 phases when: Iout < I_shed_threshold_N
  where I_shed_threshold_N is chosen so efficiency improvement from shedding > loss from higher ripple
```

Example for a 4-phase VRM with each phase rated at 30A:
- 4 phases: 0–120A
- 3 phases: 0–60A (shed at 60A)
- 2 phases: 0–30A (shed at 30A)
- 1 phase: 0–15A (shed at 15A)
- Burst mode: below 5A

**Hysteresis in shedding decisions:**

The threshold for shedding a phase must be lower than the threshold for re-enabling it (hysteresis). Without hysteresis, the controller would oscillate between N and N-1 phases as load current hovers near the threshold:
```
Shed threshold: I_shed
Re-enable threshold: I_shed + I_hysteresis
```

---

### Q5. Describe the transient response advantage of multiphase converters versus single-phase.

**Answer:**

**Inductor current slew rate:**

When a load step (ΔIload) occurs, the inductors must ramp their current to supply the new load demand. The rate is:
```
dI/dt_per_phase = (Vin - Vout) / L   [positive step]
dI/dt_per_phase = Vout / L          [negative step]
```

For N phases in parallel:
```
Total dI/dt = N × (Vin - Vout) / L_per_phase
```

**Time to respond to a load step:**
```
t_respond = ΔIload / (N × dI/dt_per_phase) = ΔIload × L_per_phase / (N × (Vin - Vout))
```

**Example:** 4-phase VRM, each L = 300 nH, Vin = 12V, Vout = 1.0V, ΔIload = 50A:
```
dI/dt_total = 4 × (12 - 1) / 300e-9 = 4 × 36.67 A/µs = 146.7 A/µs
t_respond = 50 / 146.7 = 0.34 µs
```

A single-phase design with the same L would take 4× longer (1.36 µs).

**Output voltage undershoot:**

During t_respond, the output capacitor supplies the load current difference:
```
ΔVout = ΔIload × t_respond / (2 × Cout)
```

Faster response (smaller t_respond from more phases) → smaller ΔVout undershoot.

**Why multiphase inductors can be smaller:**

Each phase in an N-phase design handles Iout/N average current but must have enough inductance for good current sharing and ripple characteristics. The inductance can often be reduced proportionally to N:
```
L_per_phase = L_single / N  (approximately, for same per-phase ripple)
```

With smaller per-phase inductance, the per-phase current slew rate is higher:
```
dI/dt_per_phase_new = (Vin - Vout) / (L/N) = N × dI/dt_original
Total dI/dt = N × N × dI/dt_original = N² × improvement
```

This N² improvement in response rate (through both more phases AND smaller inductors) is why multiphase designs are universally used in modern high-current VRMs.

---

### Q6. What are coupled inductors in a multiphase converter and what advantage do they provide?

**Answer:**

**Standard multiphase inductors:**

Each phase has an independent inductor. The inductance seen by each phase equals its individual inductance L. Phase currents are magnetically independent.

**Coupled inductors:**

Multiple inductor windings share a common magnetic core with controlled coupling. For a 2-phase coupled inductor with coupling coefficient k_mutual (defined such that k_mutual > 0 for inverse coupling — windings wound to oppose each other):

The effective inductance seen during steady-state current ripple (transient ripple inductance):
```
L_AC = L × (1 - k_mutual)
```

The effective inductance seen during a transient where only one phase changes (other phases unchanged):
```
L_tran = L × (1 + (N-1) × k_mutual) / (1 + (N-1) × k_mutual)
```

Wait — more precisely, the two-phase coupled inductor has:
- Steady-state (ripple) inductance: `L_ss = L × (1 - k_mutual)` — smaller, so ripple is larger
- Transient (step response) inductance: `L_tran ≈ L` — full inductance, limits transient slew rate

**Wait — the key benefit:**

For inverse coupling (windings wound opposing each other in the core):
- Steady-state: current in each winding creates flux that partially cancels the other → reduced effective inductance → higher ripple per phase
- But the output current is the sum, and the out-of-phase high ripples partially cancel → net output ripple is actually reduced

More precisely, inverse-coupled inductors provide:
1. **Lower per-phase effective inductance for ripple purposes:** Each phase "sees" L_ripple = L(1-k). But the output sees L_ripple/(1 at null D) — the output ripple can be near zero at D=1/N.
2. **Higher effective inductance during transients:** When all phases slew together (load step), the cross-coupling fields add and the effective inductance is L_transient = L(1+k). But since all phases slew simultaneously in a load step, the total slew rate is N × (Vin-Vout)/L_transient — similar to uncoupled.

**Key benefit — decoupled ripple and transient performance:**

With uncoupled inductors:
- To reduce ripple: increase L → slows transient response
- To improve transient response: decrease L → increases ripple

With inversely-coupled inductors:
- L_ripple is set by the coupling factor: L_ripple = L(1-k) → can be small
- But per-phase ripple, when summed, cancels further at the output
- Transient performance uses full L → better than uncoupled at same volume

This allows the designer to achieve low output ripple AND fast transient response in a smaller overall magnetics volume. Widely used in Intel VRM designs (Microsemi/Microsemi coupled inductors, Vishay Dale series).

---

## Intermediate (Questions 7–12)

---

### Q7. How do you design a 4-phase VRM for a CPU with the following specs: Vin=12V, Vout=1.0V, Iout=100A, fsw=400kHz?

**Answer:**

**Step 1 — Duty cycle:**
```
D = Vout / Vin = 1.0 / 12 = 0.0833  (8.3%)
```

This is a very low duty cycle. The high-side MOSFET is on for only 8.3% of each cycle. The low-side MOSFET carries most of the current (91.7% duty cycle).

**Step 2 — Current per phase:**
```
Iout_per_phase = 100 / 4 = 25 A average
```

**Step 3 — Inductor selection:**

Choose ripple ratio r = 0.4 per phase:
```
ΔIL_per_phase = r × Iout_per_phase = 0.4 × 25 = 10 A

L = Vout × (1-D) / (fsw × ΔIL)
  = 1.0 × 0.9167 / (400e3 × 10)
  = 0.9167 / 4e6
  = 229 nH → select 220 nH or 270 nH
```

Use L = 220 nH per phase.

**Step 4 — Output ripple after interleaving:**

At D = 0.0833 ≈ 1/12, which is not near any null point for N=4 (nulls at D=0.25, 0.5, 0.75). Use the general formula:

For D = 0.0833, k = 1 (D < 1/N = 0.25):
```
ripple_ratio = 4 × (1 - 4×D) / ((1-D)×D×16×N/(N-1))
             ≈ 4 × (1 - 0.333) / (0.9167 × 0.0833 × 16)
             = 4 × 0.667 / 1.222
             = 2.183 → ripple is 218% of single-phase!?
```

Wait — this seems backwards. At very low duty cycle, the 4-phase ripple is actually WORSE in some formulations. Let's recalculate properly.

For N=4, D=0.0833, the output ripple (absolute, not ratio) is:
```
ΔI_4phase = ΔI_1phase × |sin(N × π × D)| / (N × |sin(π × D)|)
           ... [complex formula]
```

More practically, simulate or use design software. The key insight: at D << 1/N, ripple cancellation is minimal. The output ripple for 4 phases at D=0.0833 is approximately:
```
ΔI_out ≈ 3 × ΔI_per_phase = 3 × 10 = 30 A  [rough worst case for D far from null]
```

This requires a large output capacitor. VRMs address this with many output capacitors (often 20–40 MLCC 100µF caps + OSCon polymer capacitors).

**Step 5 — MOSFET selection:**

High-side (on for 8.3%): Must handle fast switching, lower Rds_on stress.
Low-side (on for 91.7%): Dominates conduction loss.

For 12V bus, use 25V N-channel MOSFETs (Rds_on ≈ 1.5–3 mΩ per phase):
```
P_cond_LS_per_phase = (25)² × 0.002 × 0.917 = 625 × 0.002 × 0.917 = 1.15 W
P_cond_HS_per_phase = (25)² × 0.003 × 0.083 = 625 × 0.003 × 0.083 = 0.156 W
```

Total HS+LS conduction per phase: 1.31 W × 4 phases = 5.24 W at full load.

**Step 6 — Controller selection:**

Use a dedicated multiphase VRM controller (e.g., Renesas ISL69269, Intersil ISL68201, ON Semi NCP81239). These integrate:
- Phase interleaving timing
- Current sensing and sharing
- Phase shedding logic
- Adaptive dead time
- Intel/AMD VR specification compliance (SVID, VR14.0, etc.)

---

### Q8. How does adaptive voltage positioning (AVP) work in a VRM design?

**Answer:**

**The problem with fixed output voltage:**

Processors define a load-line (also called droop resistance) specification. Without AVP, the output voltage is regulated to a fixed setpoint Vout_nom. A large load step from light to heavy load causes:
1. Vout droops below Vout_nom (undershoot) during the transient
2. Capacitors charge back up to Vout_nom at steady state

The processor voltage must always remain within a window [Vout_min, Vout_max]. A fixed setpoint means the steady-state voltage headroom above Vout_min is "wasted" — because the transient droop takes Vout below Vout_min if the setpoint is at nominal.

**AVP principle:**

Program the converter's output impedance to match the processor's specified load line:
```
Vout(Iout) = Vout_nom - Iout × Z_droop
```
where Z_droop is the target droop resistance (e.g., 2 mΩ for a high-current CPU VRM).

**Effect of AVP:**

At light load (Iout = 0): `Vout = Vout_nom` (maximum voltage)
At full load (Iout = 100A): `Vout = Vout_nom - 100 × 0.002 = Vout_nom - 0.2V`

A load step from 0 to 100A:
- Without AVP: Vout starts at Vout_nom, droops during transient, returns to Vout_nom
- With AVP: Vout starts high (Vout_nom), droops during transient, settles at (Vout_nom - 0.2V)

The final value with AVP is lower than without AVP. Since Vout_min must not be violated, the starting voltage with AVP can be set higher (Vout_nom can be higher) while still maintaining regulation:
```
Vout_nom_AVP = Vout_min + I_transient × Z_droop + margin
```

**AVP allows 2× smaller output capacitor** because the steady-state droop "pre-positions" the output voltage, leaving a larger voltage window for the transient.

**Implementation:**

AVP is achieved by creating an output impedance in the control loop:
```
Vout_setpoint = Vout_nom - Gm_droop × I_total
```
where Gm_droop × I_total implements the current-dependent droop. This is implemented via the FB voltage and current sense inputs in the VRM controller IC.

---

### Q9. What is the impact of inductor current sensing method on multiphase converter performance?

**Answer:**

Accurate per-phase current sensing is essential for:
1. Current sharing balance between phases
2. Overcurrent protection
3. AVP implementation (total current for droop)
4. Phase shedding decisions

**Method 1 — Sense resistor (Rsense) in series:**

A small resistor (5–20 mΩ) in series with each phase inductor.
```
V_sense = I_phase × Rsense
```
Pros: Simple, accurate, linear.
Cons: Power loss `P = Iout² × Rsense` = 625 × 0.01 = 6.25W for 25A phase with 10mΩ — too lossy for high-current designs.
Use at: Low current (<5A) or low duty cycle applications.

**Method 2 — DCR sensing (lossless):**

Parallel RC network across the inductor:
```
R_filter = R_dcr_sense (selected resistor)
C_filter = L / (R_filter × DCR)  [when RC = L/DCR, V_C = I × DCR]
```

When the time constants match:
```
V_C(s) = I_L(s) × DCR
```
No power loss (only V × I in DCR of inductor). Accuracy depends on DCR matching between components.

Pros: Zero additional power loss, integrates with standard inductors.
Cons: DCR temperature coefficient must be compensated (DCR changes 40% from 25°C to 100°C). Matching RC = L/DCR requires inductor DCR tolerance (±20% typical).

**Method 3 — Current transformer (CT):**

A small toroidal CT with primary = 1 turn (the switching node trace) and secondary = 50–100 turns feeds current into a burden resistor.
```
I_secondary = I_primary / N
V_sense = I_primary × Rsense_burden / N
```

Pros: Galvanic isolation, can handle high current.
Cons: Only senses AC component (needs reconstruction of DC); bulky; adds cost.

**Method 4 — Rds_on sensing:**

Measure voltage across the low-side MOSFET during its on-time:
```
I_phase = V_Rds_LS / Rds_on_LS
```

Pros: No additional components.
Cons: Rds_on varies ±30% with temperature and part-to-part variation. Poor accuracy without calibration.

**Practical choice for high-current VRM:**

DCR sensing with NTC temperature compensation (to correct for DCR temperature coefficient) is the standard approach in commercial VRM designs. The cost is negligible and accuracy is sufficient for 5% current balance tolerance.

---

### Q10. Describe the frequency spectrum of a 4-phase interleaved converter's input and output currents.

**Answer:**

**Single-phase converter input current spectrum:**

Fundamental: fsw
Harmonics: 2×fsw, 3×fsw, 4×fsw, ...

The amplitudes decrease at higher harmonics but can still be significant, creating a wide-bandwidth noise source for the input EMI filter.

**4-phase interleaved input current spectrum:**

With 4 phases at 90° offsets, the fundamental and harmonics at non-multiples of 4×fsw cancel:
```
Cancelled components: 1×fsw, 2×fsw, 3×fsw
Remaining components: 4×fsw, 8×fsw, 12×fsw, ...
```

The first significant input current harmonic is at 4×fsw = 4×400kHz = 1.6 MHz.

**Design implication:**

The input EMI filter only needs to attenuate frequencies starting at 4×fsw rather than at fsw. For a 400 kHz converter:
- 1-phase: must attenuate at 400 kHz (CISPR limit at 150 kHz requires very good filter)
- 4-phase: must attenuate at 1.6 MHz (4× easier, much smaller filter inductors and capacitors)

The input filter size is approximately (1/N²) of the single-phase filter (both L and C reduce because the frequency is higher and the amplitude is lower).

**Output current spectrum:**

Similar cancellation occurs at the output:
- 1-phase: output ripple at fsw
- 4-phase: output ripple at 4×fsw (ripple frequency multiplied by N)

The output capacitor reactance is lower at 4×fsw, so the same capacitor provides better filtering. This allows the output capacitance to be reduced approximately (1/N) for the same output ripple voltage.

**Practical significance:**

This spectral multiplication is a primary motivation for multiphase design beyond current rating — it directly enables smaller input and output filtering for the same noise performance, which is critical in space-constrained applications.

---

### Q11. What is droop current sharing and how does it compare to active current sharing?

**Answer:**

**Droop current sharing (passive, autonomous):**

Each phase generates a feedback voltage proportional to its current. This feedback reduces the effective duty cycle of phases carrying more current and increases the duty cycle of phases carrying less current.

Implementation: DCR sense voltage or Rsense voltage is subtracted from the phase's reference:
```
V_error_k = V_ref - (V_out + I_k × R_droop)
```

When I_k is large, V_error_k is small → lower duty cycle → lower current. Natural self-balancing.

**Properties:**
- No communication between phases required
- Simple analogue implementation
- Response speed limited by the current sense filter bandwidth
- Residual current imbalance due to component mismatches (R_droop tolerance, inductor DCR mismatch)
- Typical steady-state balance: ±5–10%

**Active current sharing:**

A centralised controller measures all phase currents, calculates the average, and adjusts each phase's reference voltage or pulse width to drive currents toward the average.

```
I_avg = (I_1 + I_2 + ... + I_N) / N
ΔD_k = Kcc × (I_avg - I_k)
```

**Properties:**
- Requires inter-phase communication (can be analogue bus or digital)
- Tighter balance: ±1–3%
- Can compensate for large component mismatches
- More complex but widely used in digital multiphase controllers (Renesas, Intersil, MPS)
- Phase shedding logic easier to integrate (controller knows exact phase currents)

**Comparison:**

| Aspect | Droop sharing | Active sharing |
|--------|--------------|----------------|
| Complexity | Low | Medium-High |
| Current balance | ±5–10% | ±1–3% |
| Communication needed | No | Yes (bus or digital) |
| Fault detection | Limited | Individual phase monitoring |
| Response to mismatch | Slow (filter limited) | Fast (digital loop) |
| Typical applications | Simple multi-phase ICs | High-end VRM, server PSU |

---

### Q12. How do you calculate the minimum output capacitance for a multiphase VRM to meet a given transient specification?

**Answer:**

**Transient specification (Intel-style):**
- Load step: ΔIload = 80A (0 to 80A in 100 ns)
- Allowed undershoot: ΔVout ≤ 50 mV
- Droop specification: Vout droops to Vout_set - 80A × 2mΩ = setpoint - 160 mV (AVP target)

With AVP, the allowable undershoot relative to the AVP setpoint is 50 mV.

**Transient analysis:**

During the load step, before the inductors can respond:
1. Capacitors supply the full ΔIload
2. The inductor current eventually ramps up to meet the load

**Response time (N-phase, L per phase):**
```
t_response = ΔIload × L_per_phase / (N × (Vin - Vout))
```
For N=4, L=220nH, Vin=12V, Vout=1.0V:
```
t_response = 80 × 220e-9 / (4 × 11) = 17.6e-6 / 44 = 400 ns
```

**Capacitor-supplied charge during response:**
```
ΔQ = ΔIload × t_response / 2 = 80 × 400e-9 / 2 = 16 µC
```

**Minimum capacitance from charge delivery:**
```
Cout_min = ΔQ / ΔVout = 16e-6 / 0.050 = 320 µF
```

**ESR contribution:**
Any ESR in the capacitor bank adds `ΔV_ESR = ΔIload × ESR`. For ΔV_ESR ≤ 10mV:
```
ESR_max = 10e-3 / 80 = 0.125 mΩ
```

This extremely low ESR requires many ceramic capacitors in parallel. For 100µF X5R with 1mΩ ESR:
```
N_caps_ESR = 1e-3 / 0.125e-3 = 8 caps minimum for ESR
N_caps_cap = 320e-6 / (100e-6 × 0.75) = 4.3 → 5 caps minimum  [75% derating at 1V bias]
```

Take the larger requirement: 8 × 100µF X5R MLCCs + additional polymer capacitors for low-frequency energy.

**Final design:**
- 10× 100µF, 2.5V X5R MLCC (1.8mm height allows dense placement)
- 4× 560µF polymer (Panasonic OS-CON or similar)
- Total effective capacitance: ~750µF + 2240µF = ~3000µF (note: X5R derating at 1V reduces MLCC effective value significantly — recalculate with actual datasheet curves)

This is a realistic VRM output filter — modern CPU VRMs use 30–50 MLCCs plus several polymer capacitors for this reason.
