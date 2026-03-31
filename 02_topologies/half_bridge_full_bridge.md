# Half-Bridge and Full-Bridge Converters — Interview Preparation

## Overview

Half-bridge and full-bridge topologies are the workhorses for isolated DC-DC converters in the 200W–10kW power range. They apply alternating voltage to the transformer primary, enabling full utilisation of the B-H curve in both directions, lower transformer core size, and natural volt-second balancing. Understanding their operation, rectifier options, soft-switching techniques, and design trade-offs is essential for server power, EV chargers, industrial supplies, and UPS systems.

---

## Key Equations Reference

### Half-Bridge

```
Vout = (Vin/2) × D × (Ns/Np)   [CCM, full-wave rectified output]

Primary voltage: Vin/2 applied to each half (split capacitors or leg midpoint)
Switch voltage stress: Vin each (vs 2×Vin for single-switch topologies)
Peak transformer current: Ipri_peak = Iout × Np/Ns + ΔIL/(2×(Ns/Np))
```

### Full-Bridge

```
Vout = Vin × D × (Ns/Np)   [CCM, full-wave rectified output]
Peak primary voltage: Vin (2× the half-bridge)
Switch voltage stress: Vin each
Transformer utilisation: better than half-bridge (full Vin across primary)
```

### Phase-Shifted Full Bridge (PSFB) ZVS Conditions

```
ZVS condition: 0.5 × (Llk + Lext) × Ipri² > 2 × Coss × Vin²
Minimum current for ZVS: Izys = Vin × √(2 × Coss / (Llk + Lext))
ZVS range: Iload_min(ZVS) = (Izys × Np) / Ns - ΔIL/2
```

### Rectifier Options

```
Full-wave centre-tap: V_diode_stress = 2 × Vout/η_transformer
Full-wave bridge:     V_diode_stress = Vout (lower stress, 4 diodes instead of 2)
Synchronous rectifier (SR): Replace diodes with MOSFETs, gate driven by gate-driver IC
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Explain the operation of a full-bridge converter through a complete switching cycle.

**Answer:**

A full-bridge consists of four switches (Q1–Q4) arranged in an H-bridge on the primary side, a transformer, and a rectifier with LC filter on the secondary.

**Switches are controlled in diagonal pairs:**
- Pair A: Q1 (high-side left) + Q4 (low-side right)
- Pair B: Q2 (high-side right) + Q3 (low-side left)

**Phase 1 — Pair A conducts (D × Ts/2 duration):**
- Q1 and Q4 on; Q2 and Q3 off
- Positive Vin applied across primary: `V_primary = +Vin`
- Secondary voltage: `V_secondary = +Vin × Ns/Np`
- Secondary diodes D1 and D4 conduct (or SR top switches)
- Output inductor current ramps up: `dIL/dt = (V_secondary - Vout) / L`

**Phase 2 — Freewheeling (deadtime, brief):**
- All four switches off
- Output inductor freewheels through secondary rectifier

**Phase 3 — Pair B conducts (D × Ts/2 duration):**
- Q2 and Q3 on; Q1 and Q4 off
- Negative Vin across primary: `V_primary = -Vin`
- Secondary: `V_secondary = -Vin × Ns/Np` → after rectification, same polarity as Phase 1
- Output inductor continues ramping up (or remains in CCM)

**Phase 4 — Freewheeling again**

**Key result:**
The secondary sees a full-wave rectified AC waveform at 2× fsw (two pulses per period). The effective output duty cycle is:
```
Vout = Vin × Ns/Np × D
```
where D is the duty cycle of each pair (0 ≤ D ≤ 0.5 for full bridge, referenced to total period).

**Volt-second balance:**
Because the primary voltage alternates positive and negative symmetrically, the net volt-seconds on the transformer core is zero — no DC flux buildup. This eliminates transformer saturation from volt-second imbalance (a key advantage over single-ended topologies).

---

### Q2. How does a half-bridge differ from a full-bridge? What are the trade-offs?

**Answer:**

**Structural difference:**

Half-bridge:
- Two switches (Q1 high-side, Q2 low-side)
- Two large split capacitors (C1, C2) from Vin to the primary midpoint
- Or one capacitor plus bootstrap — the capacitor midpoint serves as the reference

Full-bridge:
- Four switches (Q1–Q4 in H-bridge)
- Direct connection to positive and negative rails

**Voltage applied to transformer primary:**
- Half-bridge: ±Vin/2 (capacitor divider limits voltage to half supply)
- Full-bridge: ±Vin (full supply voltage)

**For the same output power (Pout):**
```
P = Vprimary × Iprimary × η
Half-bridge: Iprimary = 2 × Pout / (Vin × η)   [twice the current of full-bridge]
Full-bridge: Iprimary = Pout / (Vin × η)
```

**Turns ratio comparison (for same Vout):**
```
Half-bridge turns ratio:  n = (Vin/2 × D) / Vout → n = Vin × D / (2 × Vout)
Full-bridge turns ratio:  n = (Vin × D) / Vout
```
The full-bridge uses twice as many primary turns (or half the secondary turns) for the same output.

**Comparison table:**

| Parameter | Half-Bridge | Full-Bridge |
|-----------|------------|------------|
| Number of switches | 2 | 4 |
| Primary voltage swing | ±Vin/2 | ±Vin |
| Primary RMS current | 2× | 1× |
| Transformer utilisation | Lower | Higher |
| Switch voltage stress | Vin | Vin |
| Gate drive complexity | 2 channels (one high-side) | 4 channels (two high-side) |
| Common for power levels | 100–500W | 300W–10kW |
| Susceptibility to flux walking | Higher (capacitors allow some) | Balanced, but need CM monitoring |

**Selection guidance:**
- Half-bridge: lower power, lower component count, adequate for 100–500W.
- Full-bridge: higher power, better transformer utilisation, justified at 500W+.

---

### Q3. What is the transformer utilisation factor and why does it matter?

**Answer:**

The transformer utilisation factor (TUF) measures how effectively the core and windings are used relative to their theoretical maximum. A higher TUF means a smaller (cheaper, lighter) transformer for the same power.

**Definition:**
```
TUF = Pout / (VA_primary_RMS × VA_secondary_RMS)^0.5
```

More practically, it is proportional to `Bmax / (Bmax + Bmin)` for unidirectional topologies:
- Flyback/forward (unipolar): core works in one quadrant only → TUF = 0.5 relative to bidirectional
- Push-pull/half-bridge/full-bridge (bipolar): core works in both quadrants → higher TUF

**Core utilisation:**
```
Maximum flux density (bipolar):  ΔB = 2 × Bmax  (swing from -Bmax to +Bmax)
Maximum flux density (unipolar): ΔB = Bmax       (swing from 0 to Bmax)
```

A bipolar topology uses the full B-H curve, allowing a given core to handle twice the volt-second product of a unipolar topology. For the same Bmax limit (to avoid saturation), the required core cross-sectional area is half:
```
Ae_bipolar = Vin × D / (4 × Np × Bmax × fsw)   [vs Vin × D / (2 × Np × Bmax × fsw) for unipolar]
```

**Practical implication:**

A full-bridge converter can use a transformer approximately half the size of a forward converter for the same power level, at the same frequency. This is why full-bridge topologies are preferred for high-power designs where transformer size and weight are significant constraints.

---

### Q4. Describe synchronous rectification in a full-bridge converter. Why is it used and what are the challenges?

**Answer:**

**Standard rectification:** Output diodes (typically Schottky or fast-recovery) convert the secondary AC to DC. Forward voltage drop (0.3–0.8V) causes conduction loss proportional to output current.

**Synchronous rectification (SR):** Replaces diodes with MOSFETs timed to turn on and off in synchronisation with the transformer secondary voltage. MOSFET Rds_on is much lower than diode forward voltage for the same current rating.

**Loss comparison (at 12V, 10A output):**

Diode rectification: `P_diode = 2 × Vf × Iout/2 = 2 × 0.5 × 5 = 5W` (both diodes share load, duty cycle)

SR MOSFET: `P_SR = Iout² × Rds_on = 100 × 0.005 = 0.5W` (for 5 mΩ MOSFET)

Efficiency improvement: 4.5W = 4.5/120W = 3.75% improvement at this operating point.

**SR control methods:**

1. **Self-driven SR (transformer-driven gate):**
   - Secondary winding voltage directly drives gate of SR MOSFET
   - Simple, no control signal required
   - Problem: gate voltage follows transformer voltage; may not be optimal for all duty cycles

2. **Synchronous rectifier controller IC (e.g., Texas Instruments UCC27714):**
   - Dedicated IC senses transformer secondary voltage or body diode voltage
   - Turns MOSFET on when body diode starts conducting (Vds detects body diode voltage)
   - Turns MOSFET off before reverse current flows
   - Prevents cross-conduction (current flowing backwards through SR MOSFET into transformer)

**Key challenges:**

1. **Shoot-through/cross-conduction:** If both SR MOSFETs conduct simultaneously, they short the output. Requires dead time management.

2. **Gate drive delay:** The SR MOSFET must turn on within nanoseconds of the diode starting to conduct. Propagation delay in gate driver + PCB parasitics can cause the body diode to conduct for longer than necessary, reducing efficiency gain.

3. **Ringing during discontinuous conduction:** During light load (DCM), the secondary ringing can falsely trigger SR MOSFET turn-on. Advanced SR controllers use blanking time to prevent this.

4. **Isolation of gate signals:** In a full-bridge, the SR MOSFETs are on the secondary (isolated) side. Their gate signals can come from secondary-side control IC, simplifying isolation requirements vs. primary-side control.

---

### Q5. What is volt-second imbalance (flux walking) in a push-pull or full-bridge converter and how is it prevented?

**Answer:**

**The problem:**

Even small asymmetries in switch timing, gate drive delays, or device characteristics can cause unequal volt-seconds to be applied to the transformer in alternate half-cycles:
```
Half-cycle 1: V_primary × t1 applied
Half-cycle 2: V_primary × t2 applied
```
If t1 ≠ t2, the net volt-seconds per cycle is non-zero → flux accumulates each cycle → transformer saturates.

**Why it's dangerous:**

Core saturation causes inductance collapse → primary current rises rapidly → switch failure. This is particularly insidious because it may not appear during brief lab testing but manifests under sustained load or temperature variations.

**Sources of asymmetry:**
- Different propagation delays in gate drivers for Q1 vs Q2 paths
- Threshold voltage mismatch between MOSFETs
- Dead time not perfectly symmetric
- Transformer core operating at different temperatures in each half

**Prevention techniques:**

1. **Current-mode control:**
   The inner current loop terminates each half-cycle when primary current reaches the programmed peak. Both half-cycles automatically have the same peak current → same magnetic energy stored → natural volt-second balance. This is the standard solution in high-quality full-bridge designs.

2. **Flux balance (DC current measurement):**
   Monitor the average primary current (DC component). Any non-zero DC component indicates volt-second imbalance. Feed this information back to adjust duty cycle timing.

3. **Series capacitor on primary:**
   A DC-blocking capacitor in series with the transformer primary prevents DC current flow. It also blocks volt-second buildup — the capacitor charges to whatever DC voltage is needed to maintain zero net current. The downside: reactive power stored in the capacitor affects circuit operation.

4. **Matched gate drive delays:**
   Use gate driver ICs with symmetric propagation delay (HCPL-314J, etc.) or measure and trim dead time.

---

### Q6. What types of output rectification are used in bridge converters and when do you choose each?

**Answer:**

**Full-wave bridge rectifier (4-diode bridge on secondary):**
```
V_diode_stress = Vout + Vf ≈ Vout
Number of diodes: 4
Diode current: Iout/2 average per diode
```
- Each diode conducts for 50% of the cycle
- Lower voltage stress on diodes (only Vout across each reverse-biased diode)
- Better for higher output voltages (fewer turns on secondary)
- Requires a single secondary winding

**Centre-tap rectifier (2 diodes, centre-tapped secondary):**
```
V_diode_stress = 2 × Vout/n   [full secondary voltage across reverse-biased diode]
Number of diodes: 2
Diode current: Iout/2 average per diode
```
- Only two diodes = two voltage drops instead of four (more efficient)
- Higher voltage stress — diode must withstand 2× the output voltage reflected to secondary
- Requires centre-tapped secondary winding (twice the copper vs bridge)
- Preferred at low output voltages (1.2V, 1.8V, 3.3V) where diode drop is a significant fraction of Vout

**Current doubler rectifier:**
```
Two inductors (L1, L2) and two diodes
Each inductor handles Iout/2 (at twice the ripple frequency of the switching frequency)
```
- Attractive for high-current, low-voltage outputs (e.g., 5V at 50A)
- Each inductor is physically half the size of a single-inductor design
- Two diodes only (same as centre-tap)
- Natural current sharing between the two inductors
- Used in VRM (voltage regulator modules) and server power stages

**Synchronous rectifier variants:**
All three above can replace diodes with MOSFETs. The synchronous rectifier matches the chosen topology:
- Bridge rectifier: 4 SR MOSFETs in H-bridge configuration
- Centre-tap: 2 SR MOSFETs, drain to centre-tap, source to output ground
- Current doubler: 2 SR MOSFETs (simpler than bridge, lower gate charge)

---

## Intermediate (Questions 7–12)

---

### Q7. Describe the phase-shifted full-bridge (PSFB) converter and explain how it achieves zero-voltage switching.

**Answer:**

The phase-shifted full-bridge is a full-bridge converter where zero-voltage switching (ZVS) is achieved by controlling the phase shift between the leading and lagging leg switch transitions rather than using constant frequency PWM.

**Operation principle:**

In a standard full-bridge, Q1/Q4 and Q2/Q3 switch hard (with full Vin across each switch at turn-on). In PSFB:
1. Leading leg (Q1/Q2): switches first, at ZVS using resonance with transformer leakage inductance and switch output capacitance.
2. Lagging leg (Q3/Q4): switches after the leading leg, during the freewheeling period, using stored magnetic energy for ZVS.

**ZVS mechanism for the leading leg:**

After Q1 turns off, the energy stored in the transformer leakage inductance (plus any external inductor Lext) resonates with Q1's and Q2's output capacitances (Coss):
```
Resonant action: Lres × (dI/dt) = (Vc1 - Vc2) → capacitor voltages ring
Q2 body diode starts to conduct → Q2 is turned on at zero voltage
```

**ZVS condition:**
```
0.5 × (Llk + Lext) × Ipri_min² ≥ 2 × Coss × Vin²
```

where Ipri_min is the minimum primary current when the leg transition occurs. The leakage inductance and/or external inductance must store enough energy to fully swing the leg capacitors from Vin to 0V.

**Lagging leg ZVS:**

The lagging leg switches during the freewheeling period when output current circulates through the primary. The primary current at this point is the reflected output inductor current:
```
Ipri_freewheeling = Iout × Ns/Np
```

At full load, this current is large enough to achieve ZVS in the lagging leg. At light load, the current decreases and the lagging leg loses ZVS first (turn-on with partial hard switching).

**Effective duty cycle loss:**

The PSFB has an inherent effective duty cycle loss because:
- During the leading leg transition, primary current still flows through transformer leakage inductance (not through the secondary load)
- This "lost" time (Δt = Llk × Iout × Np/Ns / Vin) reduces the available secondary duty cycle
- At high load or high leakage inductance, duty cycle loss becomes significant
```
D_effective = D_control - 2 × fsw × Llk × Iout_secondary_reflected / Vin
```

**Benefits of PSFB:**
- ZVS on all four switches (at sufficient load)
- Eliminating turn-on switching loss significantly improves efficiency at high fsw
- Common in 1–10kW telecom rectifiers, EV on-board chargers, data center power supplies

---

### Q8. How do you prevent transformer saturation in a full-bridge converter?

**Answer:**

Transformer saturation occurs when the core flux density exceeds Bsat. In a full-bridge, the most common cause is volt-second imbalance from asymmetric switch timing.

**Detection:**
Monitor primary current waveform. Signs of saturation:
- Unequal peak currents in alternate half-cycles
- Sudden increase in peak current per cycle (inductance decrease)
- Asymmetric waveform on oscilloscope

**Prevention Method 1 — Current-mode control:**

Use peak current mode control with a current sense transformer or resistor in the primary. Each switching half-cycle terminates when primary current reaches the programmed threshold. Since each half-cycle terminates at the same peak current, magnetic reset is symmetric:
```
Each half-cycle: ΔB = (V_primary × t_on) / (Np × Ae) = bounded by current limit
```

Current-mode control is the most robust solution and is universally used in high-power bridge converters.

**Prevention Method 2 — DC-blocking series capacitor:**

A capacitor (Cs) in series with the transformer primary blocks DC current:
- Any volt-second imbalance causes a DC flux shift
- The DC component charges Cs, creating a voltage that corrects the imbalance
- The circuit self-corrects without sensing circuitry

Downside: Cs sees full primary current, must be rated accordingly, adds reactive power.

**Prevention Method 3 — Active flux balancing:**

Monitor the primary current DC component (average) with a low-pass filter. If non-zero, trim the duty cycle of one half-cycle to correct. Used in precision applications.

**Primary current waveform during normal operation:**

In a properly balanced full-bridge with current-mode control:
```
Positive half-cycle: current ramps from -Ioffset to +Ipeak
Negative half-cycle: current ramps from +Ioffset to -Ipeak
Net DC = 0
```

Any persistent positive or negative bias indicates volt-second imbalance requiring investigation.

---

### Q9. What is the duty cycle loss mechanism in the PSFB and how does it affect design?

**Answer:**

**Duty cycle loss explained:**

During the transition of the lagging leg in a PSFB, there is a period where:
- Leading leg has switched (current changing direction in primary)
- But lagging leg has not yet switched (primary is still "freewheeling")

During this interval, the transformer primary current is circulating through the leakage inductance and does not transfer power to the secondary. The secondary voltage is zero during this interval even though the primary is switching.

**Quantitative duty cycle loss:**

The time for the primary current to slew from +Ip to -Ip through the leakage inductance:
```
Δt_loss = 2 × Llk × Ip / Vin
```

The effective duty cycle (what the secondary sees):
```
D_eff = D_control - fsw × Llk × (n × Iout) / Vin
      = D_control - fsw × Llk × Ipri_peak / Vin
```

For Llk = 3 µH, Iout = 20A, n = 0.25, Vin = 400V, fsw = 100kHz:
```
Δ = 100e3 × 3e-6 × (0.25 × 20) / 400 = 0.3 × 5 / 400 = 0.00375 → 0.375%
```

This is small at 100 kHz, but becomes significant at higher frequencies or higher leakage inductance.

**Impact on design:**

1. Transformer turns ratio must be adjusted for duty cycle loss:
   ```
   n_actual = Vin × D_eff_max / Vout
   ```
   If design ignores duty cycle loss, Vout will be lower than expected.

2. At light load, the secondary current is smaller → less Ipri → less ZVS energy. The lagging leg may lose ZVS, increasing switching loss.

3. An external inductor (Lext) can be added to increase the ZVS energy reserve:
   - More Lext → better ZVS → but larger duty cycle loss
   - Trade-off: choose Lext for full ZVS at minimum load, accept some duty cycle loss

**Extended phase shift for deep light load:**

At very light load, increase the phase shift angle to maintain some minimum primary current. This reduces output voltage (must be compensated by control loop) but maintains ZVS on the leading leg.

---

### Q10. How do you select the transformer turns ratio for a full-bridge converter given a wide input voltage range?

**Answer:**

**Challenge:** A wide Vin range requires the duty cycle to vary over a wide range to regulate output voltage. Extreme duty cycles create problems:
- Very high D (close to 1): converter cannot reset; transformer volt-seconds may not balance
- Very low D: higher peak current, higher secondary current stress

**Design procedure:**

**Step 1 — Define operating range:**
```
Vin_min = 36V, Vin_max = 72V (2:1 range)
Vout = 12V, Iout = 20A
```

**Step 2 — Select maximum duty cycle:**

D_max = 0.45 (leave margin for dead time and volt-second balance)

**Step 3 — Calculate turns ratio from minimum input voltage:**
```
n = Np/Ns = Vin_min × D_max / Vout
  = 36 × 0.45 / 12
  = 16.2 / 12
  = 1.35 → use n = 1.25 (standard value)
```

**Step 4 — Calculate duty cycle at maximum input:**
```
D_min = n × Vout / Vin_max = 1.25 × 12 / 72 = 15 / 72 = 0.208
```

This wide duty cycle range (20% to 45%) is acceptable.

**Step 5 — Check peak primary current:**

At maximum input, minimum load, in CCM:
```
Ipri_peak = (Iout/n) + ΔIL/(2n)
```

At minimum input, maximum load:
```
Ipri_peak = Iout/n + ΔIL_max/(2n) = 20/1.25 + ΔIL/2.5
```

**Step 6 — Verify transformer utilisation:**

Primary turns (for Bmax = 0.2T, ETD34 core Ae = 97 mm²):
```
Np = Vin_min × D_max / (4 × Bmax × Ae × fsw)
   = 36 × 0.45 / (4 × 0.2 × 97e-6 × 100e3)
   = 16.2 / 7.76
   = 2.1 → use 3 turns (round up)
```

Secondary turns:
```
Ns = Np / n = 3 / 1.25 = 2.4 → use 2 turns (adjust n to 3/2 = 1.5)
```

Recheck: D_at_Vinmin = 1.5 × 12 / 36 = 0.5 → at D_max=0.5, core will just balance.

---

### Q11. What is the difference between a current-fed and voltage-fed bridge converter?

**Answer:**

**Voltage-fed bridge (standard):**

The primary is connected directly to the DC bus capacitor (low source impedance → voltage source). This is the common full-bridge topology.

Characteristics:
- Switch node voltage changes rapidly (hard switching unless ZVS achieved)
- High dV/dt at switching transitions → EMI
- Transformer leakage inductance must be managed (snubbers or ZVS)
- Good for wide duty cycle range
- Most common in industrial and telecom converters

**Current-fed bridge:**

An input inductor (L_series) is placed in series with the DC bus before the bridge. The converter now sees a current source at its primary input.

Characteristics:
- Primary current is continuous (smoother waveform)
- Transformer always has current flowing → no current interruption issues
- Higher transformer utilisation (current source)
- Switches must not open simultaneously (or they must overlap) — unlike voltage-fed where shoot-through is forbidden
- Natural boost action from the series inductor
- More complex control (shoot-through timing during transitions is mandatory)
- Used in some high-power DC-DC designs (e.g., 10kW+ bi-directional converters, BESS)

**Key difference in switch control:**
```
Voltage-fed: D1+D2 < 0.5 each (no overlap, dead time required)
Current-fed: D1+D2 > 0.5 (overlap mandatory; no switches can all be open simultaneously)
```

Opening all switches in a current-fed bridge while current is flowing would cause infinite voltage spike across the transformer (L × dI/dt with I forced to zero instantaneously). This is the complementary safety concern to shoot-through in voltage-fed topologies.

---

### Q12. What is shoot-through in a bridge leg and how is it prevented?

**Answer:**

**Definition:**

Shoot-through occurs when both the high-side and low-side MOSFETs in a bridge leg are conducting simultaneously, creating a direct short circuit from Vin to ground. The current is limited only by MOSFET Rds_on and wiring impedance:
```
I_shoot = Vin / (Rds_HS + Rds_LS + R_wiring) → potentially thousands of amperes
```

This instantaneously destroys both MOSFETs.

**Causes of shoot-through:**

1. **Insufficient dead time:** Controller commands HS turn-on before LS turn-off is complete. Even 1 µs overlap can cause shoot-through.

2. **Gate signal propagation delay mismatch:** LS turn-off command travels faster than it reaches the gate (long PCB traces, slow gate driver) → apparent dead time is shorter than intended.

3. **Miller effect re-triggering:** When the switch node transitions from 0V to Vin rapidly, the dV/dt × Cgd charges the gate capacitance of the opposite MOSFET above Vth, causing it to begin conducting.

**Prevention:**

1. **Dead time insertion:** All gate drivers introduce minimum dead time between HS and LS gate signals. Minimum dead time:
   ```
   t_dead ≥ t_off(worst) + t_on(worst)   [with margin]
   ```
   Gate driver ICs automatically enforce dead time (adjustable via resistor or internally fixed).

2. **Adaptive dead time:** Detect when the switch node reaches its final voltage (HS drain at 0V, LS drain at Vin) and only then allow the other MOSFET to turn on. Prevents unnecessarily long dead time that would cause body diode conduction.

3. **Gate resistor optimisation:** Reduce dV/dt (slightly longer fall time) by increasing gate pull-down resistance → less Miller current into opposite MOSFET's gate.

4. **Low-Vth MOSFETs are more susceptible:** Use MOSFETs with Vth > 2V for power bridge applications. Logic-level MOSFETs (Vth ≈ 1V) are dangerous in this regard.

5. **Bootstrap and supply voltage:** Ensure HS gate driver supply (bootstrap or isolated bias) is within specifications. Under-voltage on HS driver can cause incomplete turn-on → excessive Rds_on → heating but not shoot-through. Over-voltage on driver can exceed Vgs_max and rupture gate oxide.

---

## Advanced (Questions 13–15)

---

### Q13. Design a phase-shifted full-bridge for 400V input, 48V output, 2kW. Verify ZVS conditions.

**Answer:**

**Specification:**
- Vin = 400V, Vout = 48V, Pout = 2000W
- Target fsw = 100 kHz, η = 95%

**Step 1 — Turns ratio:**

Select D_max = 0.42 (leaving headroom for dead time and duty cycle loss):
```
n = Np/Ns = Vin × D_max / Vout = 400 × 0.42 / 48 = 168 / 48 = 3.5
```
Use n = 3.5 (or 7:2 winding ratio).

**Step 2 — Primary RMS current:**
```
Ipri_rms ≈ Pout / (Vin × η × D_max × √2) ≈ 2000 / (400 × 0.95 × 0.42 × 1.414)
         ≈ 2000 / 226 ≈ 8.85 A
Ipri_peak ≈ Iout × Ns/Np = (2000/48) × (1/3.5) = 41.67 × 0.286 = 11.9 A
```

**Step 3 — Select MOSFET (500V, low Qoss):**

Use 500V SJ MOSFET (e.g., IPP60R160CFD7: Rds_on = 160 mΩ, Coss = 60 pF at 200V, Qg = 38 nC):
```
Coss_effective = 60 pF at 200V (use Coss_eff = Qoss/Vin for accurate ZVS calculation)
```

**Step 4 — ZVS condition:**

Each leg must swing 400V using energy stored in Llk + Lext:
```
E_required = 2 × Coss_eff × Vin² = 2 × 60e-12 × 400² = 2 × 60e-12 × 160000 = 19.2 µJ
```

Energy available from leakage + external inductance at minimum load current (30% = 600W):
```
Ipri_min = (600W/48V) × (1/3.5) / 0.95 = 12.5 × 0.286 = 3.57 A

E_available = 0.5 × Lres × Ipri_min² ≥ E_required
Lres_min = 2 × 19.2e-6 / (3.57)² = 38.4e-6 / 12.7 = 3.02 µH
```

The resonant inductance (leakage + external) must be at least 3.02 µH for ZVS down to 30% load.

**Typical transformer leakage:** Llk ≈ 1–2 µH for this power level. Add external series inductor Lext ≈ 2 µH.

**Step 5 — Duty cycle loss:**
```
Δ_loss = 2 × fsw × Lres × Ipri_peak / Vin
       = 2 × 100e3 × 3e-6 × 11.9 / 400
       = 7.14 / 400
       = 0.0179 → 1.8%
```

Effective duty cycle at full load:
```
D_eff = D_control - Δ_loss = 0.42 - 0.018 = 0.402
Vout_check = Vin × D_eff / n = 400 × 0.402 / 3.5 = 45.9 V
```

Controller will increase D to compensate (up to D_max = 0.45).

**Step 6 — Efficiency estimate:**
```
P_conduction = 4 × Ipri_rms² × Rds_on / 2 [each switch conducts ~D/2 of cycle]
             = 4 × 78.5 × 0.160 × 0.21 = 10.6 W
P_switching ≈ 0 (ZVS achieved)
P_gate = 4 × 38e-9 × 15 × 100e3 = 2.28 W
P_transformer ≈ 15 W (estimate, includes copper + core)
P_output_rectifier ≈ 48 × (2000/48) × 0.007 × 2 = 2.8 W [SR, Rds=7mΩ]
Total losses ≈ 31 W → η = 2000/2031 = 98.5%  [optimistic without ZVS loss details]
Realistic: η ≈ 95–96%
```

---

### Q14. What is the effect of transformer leakage inductance on output voltage regulation in a full-bridge converter?

**Answer:**

Leakage inductance acts as a series impedance in the power transfer path. During each half-cycle, the primary current must ramp through Llk before reaching the reflected secondary current. This causes:

**1. Duty cycle loss (as covered in Q9):**
```
D_lost = 2 × fsw × Llk × Iout × n / Vin
```

**2. Cross-regulation problems with multiple outputs:**
Each secondary has its own leakage inductance. Changes in one output's load current affect the leakage voltage drop on all outputs, causing cross-regulation errors.

**3. Output voltage drop under load:**
The effective output voltage includes a leakage-induced drooping term:
```
Vout_loaded = Vout_unloaded - ΔV_llk
ΔV_llk = 2 × fsw × Llk × Iout × (Ns/Np)²
```

For a design with Llk = 3 µH, Iout = 10A, n = 2, fsw = 100kHz:
```
ΔV_llk = 2 × 100e3 × 3e-6 × 10 × 0.25 = 1.5 V
```

This is a 3% drop for a 48V output — significant but manageable with a control loop.

**4. Ringing and snubber requirements:**
When the primary switch turns off (or the rectifier commutates), Llk resonates with junction capacitances, creating voltage ringing. This must be absorbed by snubbers to prevent overstress.

**Minimising leakage inductance:**

Transformer winding techniques to reduce Llk:
1. **Interleaving:** Alternate layers of primary and secondary winding. Each pair of interleaved layers cancels part of the leakage flux. Dramatically reduces Llk.
2. **Tight coupling:** Wind primary and secondary bifilar (side by side). Very low Llk, but insulation challenges.
3. **Width matching:** Ensure winding widths (bobbin fills) are equal across primary and secondary.
4. **Reduce number of layers:** Fewer layers mean less proximity effect and less leakage inductance.

---

### Q15. Compare full-bridge converter efficiency to LLC resonant converter for a 400V bus, 48V output application. What are the key trade-off points?

**Answer:**

**Full-bridge (PSFB) characteristics:**
- ZVS on all four primary switches (at sufficient load)
- Hard switching on secondary rectifier (unless ZVS-ZCS achieved)
- Switching frequency: typically 50–150 kHz
- Wide regulation range via duty cycle
- Duty cycle loss at high leakage inductance or high load
- Well-understood design methodology

**LLC resonant converter characteristics:**
- ZVS primary switches (above resonant frequency)
- ZCS secondary rectifier (below resonant frequency — actually at resonant frequency for optimal)
- Switching frequency: varies over load and line (frequency modulation)
- Fixed duty cycle (50%) — simplifies magnetics
- Near-lossless switching transitions at resonant point
- Non-linear gain curve makes design more complex
- Narrow gain range makes it better suited for narrow Vin range

**Efficiency comparison (typical for 400V in, 48V out, 1kW):**

| Load | Full Bridge (PSFB) | LLC Resonant |
|------|-------------------|--------------|
| 100% | 95–96% | 96–97.5% |
| 50%  | 94–95% | 95–97% |
| 10%  | 88–92% | 90–94% |
| Peak efficiency | ~96% at 40–60% load | ~97% at 30–50% load |

**Key decision factors:**

Choose PSFB when:
- Wide input voltage range required (>2:1)
- Fixed frequency required (for synchronous rectification timing)
- Output voltage regulation over wide range
- Designers have more familiarity/experience with PWM control

Choose LLC when:
- Input voltage is relatively narrow (universal AC input with PFC → well-regulated 400V bus)
- Maximum efficiency is the primary goal
- Switching losses are dominant (>500 kHz desired)
- Resonant topology experience is available on team

**Hybrid approaches:**
Some designs use LLC for the main power transfer with an additional regulation stage (post-regulation). This achieves LLC efficiency while providing output voltage regulation independent of load/line.
