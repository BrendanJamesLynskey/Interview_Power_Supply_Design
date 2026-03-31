# Transformer Design — Interview Preparation

## Overview

Transformers in switching power supplies provide galvanic isolation, voltage conversion, and energy storage (in flyback designs). Transformer design involves turns ratio calculation, magnetising inductance, leakage inductance management, winding techniques, insulation for safety, and minimising common-mode noise. These topics appear extensively in interviews for isolated converter design roles.

---

## Key Equations Reference

### Turns Ratio
```
n = Np / Ns = (Vin × D) / Vout_ideal   [flyback, CCM, ignoring drops]
n = Np / Ns = Vin / (Vout + Vf)         [forward converter, simplified]
```

### Faraday's Law (Volt-Second Balance)
```
Np × dΦ/dt = Vp
ΔB = Vp × ton / (Np × Ae)  →  must not exceed ΔB_max = B_max - B_min

For full-bridge (bipolar excitation): ΔB = 2 × Vp × ton / (Np × Ae) ≤ 2 × B_sat
For flyback: ΔB = Vin × D × Ts / (Np × Ae) ≤ B_sat (unipolar)
```

### Magnetising Inductance (referred to primary)
```
Lm = µ0 × µr × Np² × Ae / le   [no gap, approximation]
Lm = Np² / (le/(µ0 × µr × Ae) + Lg/(µ0 × Ae))   [with gap]

In flyback: Lm stores the energy each cycle:
Energy = ½ × Lm × I_peak² = Pout × Ts / η   [in DCM]
```

### Leakage Inductance (simplified, two-winding)
```
Ll ≈ µ0 × (N_total)² × bw × d_ins / (3 × lw)

where:
  bw = winding breadth (dimension along the core window width)
  d_ins = insulation spacing between primary and secondary
  lw = winding length (along the core leg)
  N_total = effective total turns (function of winding geometry)
```

### Voltage Spike from Leakage (Flyback)
```
V_spike = Vin + n × Vout + I_peak × √(Ll / Coss)   [LC resonance]
or with snubber: V_spike = Vin + n × Vout + V_clamp
```

### Creepage and Clearance (IEC 62368 / IEC 60950)
```
Creepage (reinforced insulation, mains to secondary, pollution degree 2):
  ≥ 8 mm for 250V mains (IEC 62368-1, Table 11)

Clearance (air gap):
  ≥ 4 mm for 250V mains (IEC 62368-1)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. How do you calculate the turns ratio for a flyback converter?

**Answer:**

**Steady-state volt-second balance:**

In a flyback converter, during the on-time, the primary stores energy in the magnetising inductance. During the off-time, the secondary delivers this energy to the output. The turns ratio relates primary and secondary voltages during the off-time:

```
When switch is OFF:
  Vin (primary reflected through turns ratio) = Vout + Vf  [from secondary perspective]
  → n × (Vout + Vf) = Vout_reflected_to_primary
  → During off-time: voltage across primary = n × (Vout + Vf)
```

**Volt-second balance on the magnetising inductance:**

During on-time: `V_Lm = Vin → ΔB_on = Vin × D × Ts / (Np × Ae)`
During off-time: `V_Lm = -(n × Vout + Vf) → ΔB_off = (n × Vout + Vf) × (1-D) × Ts / (Np × Ae)`

Balance: `Vin × D = n × (Vout + Vf) × (1-D)`

**Solving for n:**
```
n = Vin × D / ((Vout + Vf) × (1-D))
```

**In practice:**
- Choose D at the minimum Vin (to ensure regulation at all input voltages)
- Include the diode forward voltage Vf (typically 0.5–1 V for Schottky, 0.8–1.2 V for standard)
- Design conservatively: D_max = 0.45–0.48 (leave margin for duty cycle limit in the controller)

**Example:**
```
Vin_min = 85V (rectified 85Vac → DC ≈ 85 × 1.414 × 0.9 = 108V — let's use Vin=100V for simplicity)
Vout = 12V, Vf = 0.5V (Schottky), D_max = 0.45

n = 100 × 0.45 / ((12 + 0.5) × 0.55) = 45 / 6.875 = 6.55 → choose n = 6 or 7

With n = 6.5 → Np/Ns = 13/2 (impractical) → Np/Ns = 7/1 (n=7)

Verify: D_required = n × (Vout + Vf) / (Vin_min + n × (Vout + Vf))
      = 7 × 12.5 / (100 + 87.5) = 87.5 / 187.5 = 0.467  [< D_max = 0.48 ✓]
```

---

### Q2. What is magnetising inductance and how does it affect converter operation?

**Answer:**

**Definition:**

The magnetising inductance Lm is the inductance of the primary winding when the secondary is open-circuited. It represents the energy storage in the core's magnetic circuit.

**Physical basis:**

When primary current flows, it establishes magnetic flux in the core:
```
Φ = N × I / Reluctance = µ × Ae × N × I / le

Lm = N × dΦ/dI = µ × N² × Ae / le
```

**Role in a flyback converter:**

Lm is the energy storage element:
```
Energy per cycle = ½ × Lm × I_peak² = Pout / (fsw × η)
```

A smaller Lm → higher I_peak for the same power → larger peak switch current, higher copper losses.
A larger Lm → smaller I_peak → operates in DCM at light loads (if Lm is very large), losing efficiency.

**In other topologies (forward, full-bridge):**

Lm is a parasitic element that carries the "magnetising current" — a triangular current waveform that does not contribute to power transfer. The magnetising current must reset to zero each cycle (volt-second balance), accomplished by the reset winding or active clamp.

If Lm is too small:
- Magnetising current becomes large (especially at low fsw or high duty cycle)
- Core may saturate due to magnetising current alone
- The magnetising current represents wasted reactive energy

If Lm is too large:
- Magnetising current is negligible (ideal transformer)
- Core is large → physical size and cost increase

**Measurement:**
LCR meter, open secondary, measure at the operating frequency. Compare to datasheet specification.

---

### Q3. What is leakage inductance and why is it problematic in flyback converters?

**Answer:**

**Definition:**

Leakage inductance is the portion of the winding inductance that does not couple to other windings — it represents flux that leaks out of the core and travels through air paths rather than through the core.

For a two-winding transformer:
```
Total primary inductance = Lm + Ll_primary
Total secondary inductance = (Lm/n²) + Ll_secondary

Coupled model: total leakage referred to primary:
Ll_total = Ll_primary + n² × Ll_secondary
```

**Physical origin:**

Leakage flux path is through the insulation layers and winding spaces between primary and secondary. The leakage inductance is approximately:
```
Ll ≈ µ0 × N²_p × Aw_ins / lw

where:
  Aw_ins = cross-sectional area of insulation between windings
  lw = winding length (along the bobbin axis)
```

**Why it is problematic:**

**1. Voltage spike at switch turn-off:**

When the MOSFET opens, the leakage inductance current cannot change instantaneously. The energy stored in Ll (½ × Ll × I_peak²) must be dissipated or absorbed:
```
E_leakage = ½ × Ll × I_peak²

Voltage spike: V_spike = Vin + n × Vout + √(2 × E_leakage / Coss)
```

Without a snubber, this spike can exceed the MOSFET breakdown voltage (BVDSS) → MOSFET failure.

**2. Efficiency reduction:**

Energy stored in leakage inductance is lost each switching cycle:
```
P_leakage = ½ × Ll × I_peak² × fsw
```

For Ll = 2 µH, I_peak = 5A, fsw = 100kHz:
`P_leakage = ½ × 2µH × 25 × 100k = 2.5W` — significant loss at moderate power.

**3. Converter regulation:**

The leakage inductance causes the output voltage to depend on load current more than ideal. Under heavy load, the leakage inductance "steals" more volt-seconds, reducing the effective duty cycle at which energy transfers to the output.

**Mitigation:**

1. **Interleave windings (P-S-P sandwich):** Reduces leakage significantly (can reduce by 4–10×)
2. **Tight winding:** Minimise the insulation gap between primary and secondary
3. **Snubber circuits:** Absorb leakage energy (RCD snubber, active clamp)
4. **LLC topology:** Uses leakage inductance as part of the resonant network — leakage becomes beneficial

---

### Q4. What is the "volt-second balance" constraint on transformer design and what happens if it is violated?

**Answer:**

**Volt-second balance:**

A magnetic core's operating flux density is determined by the integral of voltage applied to the winding:
```
ΔB = (1/N × Ae) × ∫ V(t) dt   [over one half-cycle for symmetrical waveforms]
```

In steady state, the flux must return to its starting point each cycle. This requires the volt-seconds applied during the positive half-cycle to equal those during the negative half-cycle:
```
∫(+) V dt = ∫(-) V dt   → ΔB_pos = ΔB_neg → net ΔB over one full cycle = 0
```

**What happens if violated — flux walking:**

If the positive and negative volt-seconds do not balance exactly:
- Each cycle, the flux shifts slightly in one direction
- After N cycles, the flux has "walked" N × ΔB away from its nominal operating point
- When flux reaches B_sat, the inductance collapses → peak current rises dramatically → core loss increases → potentially catastrophic

**Causes of volt-second imbalance:**

1. **Duty cycle offset:** In a push-pull or full-bridge converter, if the two switches have slightly different on-times (due to dead time asymmetry or PWM resolution), there is a net DC volt-second.

2. **Gate drive delay asymmetry:** Slightly different propagation delays for the two switches.

3. **Switch voltage drop mismatch:** Different Vds(on) for the two switches creates different applied voltages.

4. **Half-bridge DC bias:** Capacitor voltage in a half-bridge is not exactly Vin/2 due to component tolerances.

**Solutions:**

1. **Volt-second balance sensing (current transformer):** Monitor the current waveform — if it grows each cycle, the balance is off. Adjust duty cycle accordingly. Used in digital controllers.

2. **DC-blocking capacitor:** Series capacitor in the primary winding blocks DC bias. The capacitor voltage adjusts to restore balance automatically.

3. **Current-mode control:** Peak current mode control naturally limits each switch's on-time to when the current reaches the threshold — if one switch builds up current faster, it turns off sooner → inherent flux balance.

4. **Flux balance circuit:** Sense the average magnetising current directly and use it to trim the duty cycle symmetry.

---

### Q5. Explain primary-secondary isolation requirements and how they relate to transformer construction.

**Answer:**

In mains-connected power supplies, the transformer provides galvanic isolation between the high-voltage primary (connected to mains) and the low-voltage secondary (connected to user-accessible output). This isolation is a safety requirement specified by standards (IEC 62368-1, UL 60950-1).

**Creepage and clearance:**

- **Clearance:** Shortest distance through air between two conductors
- **Creepage:** Shortest path along the surface of the insulating material

Both must meet minimum values based on:
- Working voltage (mains: 250V or 400V AC)
- Overvoltage category (typically CAT II for household equipment)
- Pollution degree (PD 2 for typical indoor equipment)
- Type of insulation (basic, supplementary, reinforced)

**IEC 62368-1 minimum values (pollution degree 2, reinforced insulation, 250V):**
```
Clearance: ≥ 4 mm
Creepage: ≥ 8 mm
```

**Transformer construction for safety:**

1. **Separate bobbin sections:** Primary and secondary wound in separate sections of the bobbin, separated by physical barriers. The distance between the outer winding surface of the primary and the inner winding surface of the secondary must meet creepage requirements.

2. **Triple-insulated wire (TIW):** Wire with three layers of insulation (each rated for 1000V). TIW allows primary and secondary to be wound on the same bobbin layer (interleaved) without separate physical barriers. Requires ≥ 3 layers of insulation on the wire itself.

3. **Tape insulation:** Multiple layers of insulating tape (0.1mm polyester tape, each rated for 1000V) between primary and secondary. Typically 3 layers of tape for reinforced insulation (IEC requirement: insulation withstand 3kV for 1 minute).

4. **Margin tape:** Additional tape along the edges of the bobbin to extend creepage distance along the bobbin flange.

**Partial discharge:**
High-voltage gradients in transformer insulation can cause partial discharge (ionisation of air pockets) that degrades insulation over time. Design for electric field uniformity; use impregnation varnish to fill voids.

---

### Q6. What is a sandwich (interleaved) winding and how does it reduce leakage inductance?

**Answer:**

**Standard (non-interleaved) winding:**
```
Core | Primary (all layers) | Insulation | Secondary (all layers) |
```
The magnetic field builds across the full height of the primary winding, then crosses the insulation gap, then drives the secondary. The "height" of the H-field peak (which determines leakage flux) equals the full winding height.

**Sandwich (interleaved) winding:**
```
Core | Primary (half) | Insulation | Secondary (all) | Insulation | Primary (half) |
```
Or P-S-P-S for more layers.

**Why it reduces leakage:**

Leakage inductance is proportional to the integral of H² over the cross-section (stored magnetic energy in leakage field):
```
Ll ∝ ∫ H² dV  (integrated over winding space)
```

In an interleaved winding, the H-field peaks at half the value of the non-interleaved case (because each section of primary is linked with adjacent secondary). The integral of H² is reduced by approximately 1/N_interleave²:

```
Ll_interleaved ≈ Ll_non_interleaved / (N_sections)²
```

For a P-S-P sandwich (N_sections = 2): Ll is reduced by 4×.
For P-S-P-S-P (N_sections = 4): Ll reduced by 16×.

**Quantitative formula for leakage (interleaved):**
```
Ll = µ0 × Np² × bw × (d_ins/3 + d_primary/(3×N_sections²)) / lw

where d_ins = total insulation thickness, d_primary = total conductor height
```

**Trade-offs:**

1. **Common-mode EMI increases:** More overlap between primary and secondary creates more inter-winding capacitance → more CM EMI. Shield winding (Faraday shield) can be added to intercept this capacitive coupling.

2. **Manufacturing complexity:** More winding steps, harder to automate.

3. **Safety compliance:** Interleaved with TIW allows closer winding placement while maintaining safety — but TIW wire costs more and is harder to wind tightly.

---

## Intermediate (Questions 7–13)

---

### Q7. Design the turns ratio and calculate the number of turns for a flyback converter with given specifications.

**Answer:**

**Given specifications:**
```
Vin_min = 100V DC, Vin_max = 375V DC (rectified from 85-265Vac)
Vout = 15V, Iout_max = 1A
Vf = 0.5V (Schottky rectifier)
fsw = 100 kHz
η = 85% (assumed efficiency)
D_max = 0.45 at Vin_min
Core: EE25 ferrite, Ae = 40 mm², B_max = 250 mT (operating limit)
```

**Step 1: Turns ratio**
```
n = Np/Ns = Vin_min × D_max / ((Vout + Vf) × (1 - D_max))
  = 100 × 0.45 / (15.5 × 0.55)
  = 45 / 8.525
  = 5.28 → choose n = 5 (Np/Ns = 5:1)

Verify D at Vin_min with n=5:
D = n × (Vout + Vf) / (Vin_min + n × (Vout + Vf))
  = 5 × 15.5 / (100 + 77.5) = 77.5 / 177.5 = 0.437 < 0.45 ✓
```

**Step 2: Magnetising inductance (CCM design)**
```
Target: converter operates at boundary of CCM/DCM at minimum load (10%)
Pout = 15V × 0.1A = 1.5W (minimum)

In DCM flyback: Lm = Vin² × D² / (2 × Pout × fsw)
At Vin_min, D = 0.437:
Lm = 100² × 0.437² / (2 × 1.5 × 100k)
   = 10000 × 0.191 / 300,000
   = 1910 / 300,000
   = 6.37 µH — too small, would be DCM at all loads for 1A output

For CCM at full load (15W output):
Lm = Vin_min² × D² / (2 × Pout_max × fsw) × K_CCM  [K_CCM adjustment]
= 100² × 0.437² × (η-efficiency correction) / ...

Use direct approach: target 20% ripple in magnetising current at full load
Ipeak = 2 × Pout / (Vin_min × D × η) = 2 × 15 / (100 × 0.437 × 0.85) = 0.81 A
ΔILm = 0.2 × 0.81 = 0.162 A

Lm = Vin_min × D / (fsw × ΔILm) = 100 × 0.437 / (100kHz × 0.162) = 43.7 / 16200 = 2.7 mH
```

**Step 3: Primary turns from Faraday's law**
```
Np = Vin × D_max × Ts / (ΔB_max × Ae)
   = 100 × 0.45 × 10µs / (250mT × 40mm²)
   = 4.5×10⁻⁴ / (250×10⁻³ × 40×10⁻⁶)
   = 4.5×10⁻⁴ / 10⁻⁵
   = 45 turns
```

**Step 4: Secondary turns**
```
Ns = Np / n = 45 / 5 = 9 turns
```

**Step 5: Verify secondary voltage**
```
Vs = Vin_min × D / (n × (1-D))  ... this checks out from turn ratio derivation.
Actual Vout = n_secondary/n_primary × Vin × D/(1-D) - Vf = 15.5V - 0.5V = 15V ✓
```

---

### Q8. Describe the RCD snubber for flyback converters and how to calculate its components.

**Answer:**

**Purpose:**

When the MOSFET turns off in a flyback converter, the leakage inductance energy cannot transfer immediately to the output. The energy causes a voltage spike:
```
V_spike = Vin + Vclamp ≥ Vin + n × Vout + Vleakage_spike
```

Without a snubber, V_spike may reach 2–5× Vin + n×Vout, destroying the MOSFET.

**RCD snubber operation:**

```
Schematic:
  Switch drain ──┬── D_snubber ── C_snubber ── R_snubber ── (back to drain or Vin)
                 │
               L_leakage
```

When the MOSFET turns off:
1. Leakage inductance current flows through D_snubber and charges C_snubber
2. C_snubber clamps the drain voltage to `V_clamp = Vin + V_C_snubber`
3. R_snubber discharges C_snubber between cycles

**Clamp voltage:**
```
V_clamp = √(Vin × (Vin + n × Vout_max + some_margin))
A common choice: V_clamp = 1.5 × (Vin_max + n × Vout)
```

**Component calculations:**

The power dissipated in the snubber resistor:
```
P_R = ½ × Ll × I_peak² × fsw   [energy per cycle × switching frequency]
```

The capacitor voltage is held at V_clamp. The charge/discharge balance:
```
R_snubber = V_clamp² / P_R    [steady state: power in R = power from inductor]

For example: Ll=2µH, I_peak=1A, fsw=100kHz, V_clamp=200V:
P_R = ½ × 2µH × 1² × 100k = 0.1W
R_snubber = 200² / 0.1 = 400 kΩ   [very high — for small leakage energy]
```

The capacitor must hold the voltage approximately constant:
```
ΔV_C / V_clamp ≤ 10% (typical)
C_snubber = I_peak × D × Ts / (R_snubber × ΔV_ratio)
or: C_snubber ≥ 10 × fsw × R_snubber  [time constant >> switching period]

More precisely: C_snubber = P_R × D_max × Ts / (V_clamp × ΔV_C)
```

**Typical values:** R_snubber = 50–500 kΩ, C_snubber = 1–10 nF.

**Efficiency impact:**
All the leakage energy is dissipated in R_snubber. This is acceptable when leakage is small. For large Ll or high power, an active clamp (instead of RCD snubber) recycles leakage energy → higher efficiency.

---

### Q9. What is an active clamp circuit and how does it improve on the passive RCD snubber?

**Answer:**

**Active clamp circuit:**

An active clamp replaces the RCD snubber with an additional MOSFET (M_clamp) in series with a clamp capacitor (C_clamp):

```
Primary circuit:
  Vin ─── T_primary ─── M_main (SW)
                │
                └─── M_clamp (SW, complementary) ─── C_clamp ─── Vin
```

M_clamp turns ON when M_main turns OFF (complementary switching with dead time).

**Operation during dead time:**

1. M_main turns OFF: leakage inductance current continues to flow, charges drain voltage
2. M_clamp turns ON: C_clamp absorbs the leakage inductance energy (LC resonance between Ll and C_clamp)
3. Energy stored in C_clamp is returned to the primary circuit when M_clamp turns OFF and M_main turns ON next cycle (zero-voltage switching opportunity)

**Advantages over RCD snubber:**

1. **Leakage energy recovery:** Instead of dissipating leakage energy in a resistor, active clamp stores it in C_clamp and returns it to Vin at the beginning of the next on-time. This improves efficiency by:
   ```
   η_improvement = P_leakage / Pout × 100%  [typical: 1–3%]
   ```

2. **Zero-voltage switching (ZVS):** The LC resonance between Ll and C_clamp can discharge the MOSFET drain capacitor Coss before M_main turns ON. This enables ZVS → eliminates capacitive turn-on losses.

3. **Voltage clamping:** The clamp voltage is determined by C_clamp voltage (more controlled and predictable than an RCD snubber).

**Trade-off:**
- Requires an additional MOSFET, driver, isolated gate drive supply
- Control complexity: must synchronise main and clamp gate signals with precise dead time
- More expensive and larger than RCD snubber

**When to use active clamp:**
- High power (P > 50W) where leakage energy is significant
- High frequency (fsw > 100 kHz) where leakage losses are large
- Efficiency targets > 90% that RCD snubber cannot achieve

---

### Q10. How does inter-winding capacitance create common-mode EMI and how is it mitigated?

**Answer:**

**Inter-winding capacitance:**

The primary and secondary windings of a transformer are separated by insulation. This insulation has a capacitance (inter-winding capacitance, C_iw) determined by:
```
C_iw = ε0 × εr × A_overlap / d_insulation
```

For a typical EE25 transformer with 3 layers of polyester tape (d=0.3mm, εr=3.5):
```
A_overlap ≈ 25 mm × 15 mm = 375 mm² (approximate winding overlap area)
C_iw = 8.85×10⁻¹² × 3.5 × 375×10⁻⁶ / 0.3×10⁻³ = 39 pF
```

**How common-mode EMI is created:**

The primary side switch node has high dV/dt (typically 1–10 V/ns). This fast voltage change couples through C_iw to the secondary ground:
```
I_CM = C_iw × dV/dt

For dV/dt = 5 V/ns, C_iw = 40 pF:
I_CM = 40pF × 5×10⁹ = 200 mA peak common-mode current
```

This common-mode current flows through the secondary ground, through the safety earth connection (Y capacitor), back to the primary earth. It appears as common-mode conducted EMI on both input and output cables.

**Mitigation techniques:**

1. **Faraday shield (electrostatic shield):**
   A thin copper layer (one turn) wound between primary and secondary, connected to primary ground (or a clean reference). This shield intercepts the capacitive current from the primary and returns it to the primary side, preventing it from flowing to the secondary:
   ```
   C_primary_to_shield × dV/dt → current to primary ground (not secondary)
   C_shield_to_secondary: much smaller if shield is close to secondary
   ```
   The shield must NOT form a shorted turn (do not connect both ends together).

2. **Reduce dV/dt:** Slower switching transitions reduce I_CM. Trade-off with switching losses.

3. **Y capacitors:** Small (1–10 nF, class Y safety-rated) capacitors from secondary ground to primary ground provide a low-impedance path for CM current → returns it without flowing through long cables. Y capacitors cannot be too large (leakage current safety requirement: typically < 3.5 mA for IT equipment).

4. **Balanced winding:** If the secondary has equal capacitance to the upper and lower rail of the primary simultaneously, the CM currents cancel partially. Difficult to achieve in practice.

5. **Common-mode choke on input/output:** Attenuates CM currents that do reach the cables.

---

### Q11. What is transformer demagnetisation and how is it implemented in a forward converter?

**Answer:**

**Need for demagnetisation:**

In a forward converter, the primary winding applies Vin to the transformer core during the on-time. This magnetises the core:
```
ΔB_on = Vin × D × Ts / (Np × Ae)
```

The core must demagnetise (return to B=0 or B_reset) before the next on-time, otherwise flux builds up cycle by cycle → saturation.

**Three reset methods:**

**1. Demagnetising (reset) winding:**
A third winding Nreset (typically equal to Np) is wound bifilar with the primary. A reset diode connects it so that when M1 turns OFF, the reset winding applies -(Vin × Nreset/Np) to the core.

```
Reset: V_reset = Vin → B resets at same rate as it magnetised
Reset time = D × Ts (same duration) → limits D to ≤ 0.5

Switch voltage stress: V_DS = 2 × Vin (during reset, Vin adds to reflected reset voltage)
```

Limitation: Duty cycle ≤ 0.5 (reset takes as long as magnetisation).

**2. Resonant reset:**
A capacitor C_reset in series with the reset winding creates LC resonance. The flux resets via a half-sine oscillation, allowing D > 0.5 (reset happens faster than linear reset).

```
Reset time ≈ π × √(Lm × C_reset) / 2   [half period of LC]
→ allows D up to ~0.7
```

**3. Active clamp:**
As described in Q9 — the clamp capacitor absorbs and returns energy, enabling volt-second balance through controlled voltage clamping.

**Volt-second balance verification:**
After reset, the flux must return to its initial value:
```
∫ V_on dt = ∫ V_reset dt
Vin × D × Ts = V_reset × (1-D) × Ts

For reset winding: V_reset = Vin × (Nreset/Np)
→ D_max = Nreset/Np / (1 + Nreset/Np) = 0.5 for Nreset = Np
```

---

### Q12. What is the difference between a flyback and forward converter transformer from a design perspective?

**Answer:**

**Flyback transformer:**

The flyback "transformer" is actually a coupled inductor that stores energy during the on-time and releases it during the off-time. The secondary never conducts simultaneously with the primary.

Design characteristics:
- **Gap:** Must be gapped to store energy (like an inductor). Ungapped ferrite cannot store enough energy for normal operation without saturating.
- **Inductance:** Large magnetising inductance is critical — it IS the energy storage element.
- **Leakage:** Undesirable — causes voltage spikes, losses, regulation issues.
- **Core excitation:** Unipolar (B swings from 0 to B_peak in CCM, or with partial reset in DCM) — uses only half the B-H characteristic. Core loss is lower (smaller ΔB) but saturation margin is tighter.

**Forward converter transformer:**

The forward transformer transfers energy instantaneously from primary to secondary. Both windings conduct simultaneously during the on-time.

Design characteristics:
- **Gap:** No intentional gap needed (only to avoid saturation from residual flux and magnetising current). Very small or no gap.
- **Inductance:** Magnetising inductance should be large (to minimise magnetising current, which is wasted). Large Lm = less reactive energy.
- **Leakage:** Still undesirable but less catastrophic (no large spike if reset winding is used). Affects load regulation.
- **Core excitation:** Bipolar (B swings between +B_max and -B_max for push-pull/full-bridge). Uses full B-H loop → potentially higher core losses but better utilisation.

**Summary comparison:**

| Feature             | Flyback                        | Forward                         |
|---------------------|--------------------------------|---------------------------------|
| Energy transfer     | Stored then released           | Instantaneous (DC transformer)  |
| Gap                 | Required (energy storage)      | Minimal (if any)                |
| Magnetising L       | Critical parameter             | Parasitic (want it large)       |
| Core utilisation    | Unipolar (50% of B-H curve)    | Bipolar (full B-H curve)        |
| Power range         | < 150W typical                 | 50W to multi-kW                 |
| Design complexity   | Simpler (fewer windings)       | More complex (reset winding)    |
| Leakage spike risk  | High (large gap → energy)      | Moderate (smaller gap)          |

---

### Q13. How do you manage multiple output windings in a flyback converter?

**Answer:**

**Multiple output challenge:**

A flyback with outputs V1, V2, V3 uses a single primary and multiple secondary windings. Only one output (the one connected to the feedback loop) is tightly regulated. The other outputs depend on:
1. Their turns ratio relationship to the regulated output
2. Cross-regulation: how their voltage changes with load on other outputs

**Cross-regulation mechanism:**

During the flyback interval, the reflected secondary voltage is:
```
V_reflected = n × Vout_regulated
```

All secondary windings share the same core flux. If the secondary rectifiers have different forward voltages or the windings have different leakage resistances, the actual output voltages deviate from the ideal turns-ratio relationship under loading.

**Cross-regulation quantification:**
```
ΔVout2 / ΔIout1 = (n2/n1) × (n1 × ESR1 + n2 × ESR2 + Ll_total × fsw × ...)
[complex expression; simulate in SPICE for exact result]
```

**Mitigation:**

1. **Magnetic amplifier (magamp) on secondary outputs:** A saturable reactor on the output controls each secondary output independently without opto-feedback from each output.

2. **Linear post-regulator:** Add an LDO or switching post-regulator on the loosely regulated outputs. Simple and precise but reduces efficiency.

3. **Tight winding coupling (low leakage):** Interleave windings so all secondaries are tightly coupled to each other → better cross-regulation.

4. **Feedback from the most sensitive output:** Regulate the critical output (e.g., 5V digital rail) directly. Accept 5–10% variation on less critical rails (e.g., 12V bias rail).

5. **Multiple-stage regulation:** Use the flyback as a first stage (bulk regulation) and add dedicated regulators for each output.

---

## Advanced (Questions 14–17)

---

### Q14. Derive the peak current and primary turns for a flyback converter in DCM.

**Answer:**

**DCM flyback operation:**

In DCM (discontinuous conduction mode), the inductor current returns to zero before the end of the switching period. The converter operates in three intervals per cycle:
- Interval 1 (D×Ts): Primary switch ON, inductor current ramps up from 0 to I_peak
- Interval 2 (D_off×Ts): Primary switch OFF, secondary conducts, current ramps down to 0
- Interval 3 (D_idle×Ts): Both switches OFF, current = 0

**Energy balance:**

Energy stored during Interval 1:
```
E_stored = ½ × Lm × I_peak²
```

All this energy is delivered to the output in Interval 2:
```
E_out = E_stored = ½ × Lm × I_peak²
Pout = E_out × fsw = ½ × Lm × I_peak² × fsw / η
```

**Peak current from power balance:**
```
I_peak = √(2 × Pout / (fsw × Lm × η))
```

**Primary turns from Faraday's law:**
```
During Interval 1: Vin = Lm × dI/dt → I_peak = Vin × D × Ts / Lm

→ Lm = Vin × D × Ts / I_peak = Vin × D × Ts / √(2 × Pout / (fsw × Lm × η))

Solving for Lm:
Lm = Vin² × D² × η / (2 × Pout × fsw)

Then I_peak = √(2 × Pout / (fsw × Lm × η)) = Vin × D / Lm × Ts
```

**Primary turns:**
```
Np = Vin × D × Ts / (ΔB × Ae) = Vin_min × D_max / (ΔB_max × Ae × fsw)

Substituting Ts = 1/fsw:
Np = Vin_min × D_max / (B_max × Ae × fsw)
```

**Example:**
```
Vin_min = 100V, D_max = 0.45, B_max = 0.3T, Ae = 40mm², fsw = 100kHz

Np = 100 × 0.45 / (0.3 × 40×10⁻⁶ × 100×10³)
   = 45 / 1.2
   = 37.5 → choose Np = 38 turns
```

---

### Q15. What is transformer loss partitioning and how do copper and core losses trade off with each other?

**Answer:**

**Loss components:**

**Copper loss (winding loss):**
```
P_Cu = I_p_rms² × DCR_primary + I_s_rms² × DCR_secondary + ...

For a flyback (DCM):
I_p_rms = I_peak / √3 × √D   [triangular waveform during on-time]
I_s_rms = I_peak/n / √3 × √(D_off)   [triangular, secondary]
```

**Core loss:**
```
P_core = Cm × f^α × ΔB^β × Ve   [Steinmetz]
ΔB = Vin × D × Ts / (Np × Ae)   [flux swing]
```

**Trade-off:**

Both losses depend on the number of primary turns Np:

- **Increasing Np:** Reduces ΔB → reduces P_core (flux swing reduces as turns increase for same volt-seconds). BUT: more turns → longer wire → higher DCR → higher P_Cu.
- **Decreasing Np:** Increases ΔB → more P_core. Shorter wire → lower DCR → lower P_Cu.

**Optimal turns number:**

At the optimum, the marginal reduction in core loss equals the marginal increase in copper loss:
```
dP_core/dNp + dP_Cu/dNp = 0

At optimum: P_Cu_winding ≈ P_core × (β) / (α × 2)  [approximate]
```

For typical ferrite (α ≈ 1.7, β ≈ 2.7): the optimal condition is approximately:
```
P_Cu ≈ (2.7 / (1.7 × 2)) × P_core = 0.79 × P_core
→ P_Cu ≈ P_core at the optimum (roughly equal copper and core losses)
```

This "equal loss" heuristic is a useful starting point. Refine with simulation.

**Practical approach:**
1. Start with a core size and assume 50/50 copper/core loss split
2. Set B_max to achieve target P_core at the chosen frequency
3. Calculate Np and wire size for the copper loss budget
4. Iterate if the losses don't balance well (usually 2–3 iterations)

---

## Quick Reference: Transformer Design Checklist

```
TRANSFORMER DESIGN CHECKLIST

Electrical:
1. Calculate turns ratio n = Np/Ns from topology equation
2. Calculate primary turns: Np from Faraday's law (volt-second / B_max / Ae)
3. Calculate secondary turns: Ns = Np/n (round to nearest integer, adjust n)
4. Calculate magnetising inductance Lm and required gap (if flyback)
5. Estimate leakage inductance, plan winding geometry (interleaved?)
6. Calculate I_peak, I_rms for each winding

Physical:
7. Select core: Ap = Ae × Aw ≥ required from energy/current density
8. Choose wire gauges: J ≤ 4 A/mm² for each winding
9. Verify windings fit in bobbin window (fill factor η ≤ 0.4-0.5)
10. Plan winding order: inner primary, insulation, secondary, insulation, outer primary (P-S-P)

Safety:
11. Verify creepage ≥ 8mm (reinforced), clearance ≥ 4mm (250Vac)
12. Count insulation layers (3× tape or 3× TIW layers for reinforced)
13. Specify hipot test voltage (3kV/1min for reinforced insulation)

Verification:
14. Calculate P_core and P_Cu at full load and maximum frequency
15. Verify temperature rise < spec (typically ΔT ≤ 40°C)
16. Check MOSFET voltage stress: Vds = Vin_max + n × Vout + V_spike
17. Prototype, measure Lm (open secondary), Ll (shorted secondary) with LCR meter
```
