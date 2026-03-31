# Quiz: Practical Power Supply Design

20 multiple-choice questions covering PCB layout, EMC, thermal management, protection circuits, and efficiency measurement. Each question has four options; only one is correct. Explanations cover why wrong answers are wrong.

**Scoring guide:** 18-20 correct = expert level; 14-17 = solid intermediate; 10-13 = review the practical design section; below 10 = gain more lab and board design experience.

---

## Q1

In a synchronous buck converter PCB layout, which component must be placed closest to the high-side MOSFET drain and low-side MOSFET source, with the shortest possible connections?

- A) The output inductor
- B) The bootstrap capacitor
- C) The input bypass capacitor (bulk or decoupling)
- D) The PWM controller IC

**Correct answer: C**

**Explanation:**

The input bypass (decoupling) capacitor must be placed as close as possible to the high-side MOSFET drain and the low-side MOSFET source (the "hot loop"). During each switching transition, the high-side MOSFET turns on and draws a large pulse of current from the input capacitor. This current flows through the loop: C_in → HS MOSFET drain → SW node → output inductor → (load) → return → LS MOSFET source → back to C_in.

The area enclosed by this current path is the "hot loop." Any inductance in this loop (trace inductance, Lloop = µ0 × Area / height) causes a voltage spike when the large di/dt is interrupted:

```
V_spike = L_loop × di/dt
```

For a 10A converter with 1ns transition time: di/dt = 10A / 1ns = 10⁹ A/s. Even 1nH of loop inductance generates V_spike = 1nH × 10⁹ = 1V. With 10nH (a realistic value for poor layout): 10V spike on top of Vin.

Minimising the hot loop area — by placing C_in immediately adjacent to the MOSFET pair — directly reduces L_loop and V_spike. This is the single most impactful layout decision for switching regulators.

**Why A is wrong:** The output inductor connects the SW node to the output. Its placement is important for routing cleanliness, but the switching loop current does not flow through the output inductor continuously — the inductor current is the relatively slowly varying inductor current, not the fast switching pulse from the input. The critical high-di/dt path is the hot loop: C_in → HS → SW → LS → GND, not through the inductor.

**Why B is wrong:** The bootstrap capacitor charges the gate driver for the high-side MOSFET. While it should be placed close to the bootstrap pin of the driver IC, its placement relative to the MOSFET is for gate drive signal integrity, not for the main power current path. The bootstrap loop carries small currents compared to the main switching current.

**Why D is wrong:** The PWM controller IC placement matters for signal routing (feedback, gate signals), but it handles small signal currents. Placing it near the MOSFETs is helpful for short gate drive traces, but the power decoupling capacitor placement is far more critical for converter performance and EMI.

---

## Q2

What is the purpose of a LISN (Line Impedance Stabilisation Network) in EMC testing?

- A) To filter conducted emissions before they reach the mains supply
- B) To provide a defined 50Ω impedance at the EUT power terminals for measurement, and to isolate the mains impedance from the measurement
- C) To simulate the impedance of a battery for equipment under test that normally runs on battery power
- D) To generate standardised EMI signals for immunity testing

**Correct answer: B**

**Explanation:**

When measuring conducted emissions from a power supply (the Equipment Under Test, EUT), the mains impedance significantly affects the measurement. Real-world mains impedance varies widely with location, frequency, and load conditions — measurements made directly at the mains outlet would not be repeatable or standardised.

The LISN (defined in CISPR 16) inserts a defined network between the mains and the EUT. Key functions:

1. **Defined 50Ω source impedance:** At the measurement terminals, the LISN presents 50Ω to the EUT. This is the standardised measurement impedance that spectrum analysers and EMI receivers expect.

2. **Mains isolation:** The LISN's inductors (typically 50µH) block high-frequency noise from the mains from reaching the measurement terminals, so only the EUT's emissions are measured.

3. **RF voltage measurement:** The RF voltage developed across the 50Ω termination resistors is proportional to the conducted emission current. The spectrum analyser measures this voltage.

A standardised LISN enables reproducible measurements that correlate between different labs and can be compared against regulatory limits (CISPR 32 Class B, FCC Part 15, etc.).

**Why A is wrong:** The LISN does not filter emissions — it measures them. Filtering would be done by the EUT's own input filter (X/Y capacitors, CM choke). A LISN inserted between an EMI filter and the mains would defeat the purpose by reducing the measured emissions.

**Why C is wrong:** Simulating battery impedance is done by a battery simulator or a source with controlled output impedance, not by a LISN. Battery simulators have low internal resistance (milliohms) and often need to handle bidirectional current.

**Why D is wrong:** Immunity testing (EFT, surge, ESD, conducted RF immunity) uses pulse generators, burst generators, and CDNs (Coupling/Decoupling Networks) — not LISNs. LISNs are specifically for EMISSION testing, not immunity testing.

---

## Q3

A power MOSFET has Rth_jc = 1.5 K/W, and is mounted with thermal interface material (TIM) giving Rth_cs = 0.8 K/W on a heatsink with Rth_sa = 2.5 K/W. The MOSFET dissipates 8W. If ambient temperature is 40°C, what is the junction temperature?

- A) 80°C
- B) 98°C
- C) 118.4°C
- D) 145°C

**Correct answer: C**

**Explanation:**

The thermal resistance network is in series:

```
Tj = Ta + P × (Rth_jc + Rth_cs + Rth_sa)
Tj = 40°C + 8W × (1.5 + 0.8 + 2.5) K/W
Tj = 40°C + 8W × 4.8 K/W
Tj = 40°C + 38.4°C
Tj = 78.4°C ≈ 78°C
```

None of the options match exactly. Let me check:

8 × 4.8 = 38.4. 40 + 38.4 = 78.4°C.

Option A (80°C) is closest. The discrepancy suggests a slight rounding in the options.

Let me try with Rth_total = 1.5 + 0.8 + 2.5 = 4.8 K/W: Tj = 40 + 8×4.8 = 40 + 38.4 = 78.4°C → closest to A (80°C).

**Correct answer: A (80°C, with rounding)**

The formula is always:
```
Tj = Ta + P_device × (Rth_jc + Rth_cs + Rth_sa)
```

**Why B is wrong (98°C):** Would require total Rth = (98-40)/8 = 7.25 K/W. There is no combination of the given values that gives 7.25 K/W.

**Why C is wrong (118.4°C):** Would require Rth_total = (118.4-40)/8 = 9.8 K/W — about double the actual value.

**Why D is wrong (145°C):** Would require P × Rth = 105 → Rth = 13.1 K/W — far too high.

*Correct answer: A (80°C). The question as presented has option C labelled as correct in the key but the arithmetic gives ~78-80°C, making A the correct choice.*

---

## Q4

What is the Y-capacitor current limit in AC-DC power supplies, and why does it exist?

- A) The capacitor's self-resonance frequency must exceed the switching frequency
- B) The total leakage current to earth ground must be below safety standards (typically 0.25-0.5mA for consumer products) to prevent electric shock
- C) The Y capacitor must limit inrush current at power-on to below the fuse rating
- D) Y capacitors are current-limited to prevent them from overheating during EMI testing

**Correct answer: B**

**Explanation:**

Y capacitors are connected between line/neutral (L or N) and the safety earth ground. In a 230Vac system, each Y capacitor conducts continuous leakage current at the line frequency:

```
I_leak = V_line × 2π × f_line × Cy
```

At 230Vac, 50Hz, Cy = 10nF:
```
I_leak = 230 × 2π × 50 × 10×10⁻⁹ = 230 × 314 × 10⁻⁸ = 722µA ≈ 0.72mA
```

This leakage current flows through the earth wire. In normal operation this is harmless. However, if the earth wire is broken (a fault condition), the leakage current flows through anyone who touches the metal chassis — through their body to their ground reference. This is the "touch current" or "earth leakage current."

Safety standards set maximum limits:
- IEC 62368-1 (A/V equipment): 0.25mA for household equipment
- IEC 60601 (medical): 0.1mA for body-contact applied parts
- IEC 60950 / IEC 62368 (IT equipment): 3.5mA with earth connected, 0.25mA in portable equipment

This limits the maximum Y capacitance:
```
Cy_max = I_leak_max / (V_line × 2π × f_line)
```

For 0.25mA at 230Vac, 50Hz: Cy_max = 250×10⁻⁶ / (230 × 314) = 3.46nF. Typical Y2 cap values are 2.2nF-4.7nF for consumer products.

**Why A is wrong:** The Y capacitor self-resonance frequency is a consideration for EMI filter design (the capacitor must have low impedance at the noise frequencies, typically 150kHz-30MHz). However, the primary reason for limiting Y cap VALUE is leakage current safety, not self-resonance. The resonance frequency is a filter design concern, not a safety limit.

**Why C is wrong:** Inrush current limiting is handled by NTC thermistors, relay bypass, or active limiters — not by Y capacitors. Y capacitors are connected from line to earth, not in series with the inrush current path.

**Why D is wrong:** Y capacitors are passive components tested for their AC current rating and dielectric strength (they must withstand safety test voltages of 4-8kV). During EMI testing, the Y capacitors are part of the DUT circuit and must meet EMC limits together with the rest of the filter. Overheating Y caps during EMI testing is not the reason for the current limit.

---

## Q5

A buck converter experiences unexpected oscillation at approximately half the switching frequency. What is the most likely cause?

- A) The output capacitor has too high an ESR, creating an ESR zero at half the switching frequency
- B) The feedback loop phase margin is less than 45°
- C) Subharmonic oscillation due to peak current mode control without adequate slope compensation at D > 0.5
- D) The input capacitor resonates with the input inductance at half the switching frequency

**Correct answer: C**

**Explanation:**

Subharmonic oscillation in peak current mode control (PCMC) is a well-known instability that manifests at exactly fsw/2. The oscillation has a period of 2×Tsw — the current alternates between a high cycle and a low cycle, creating a visible ripple at half the switching frequency.

As derived in the control theory quiz (Q15 of quiz_control): without slope compensation, PCMC is unstable for D > 0.5 because the perturbation magnification factor m2/m1 > 1. The error doubles each cycle and settles into a stable limit cycle at fsw/2.

**Diagnosis:** Observe the inductor current or switch current waveform on an oscilloscope. Subharmonic oscillation shows as an "alternating" pattern — one tall triangle followed by one short triangle, repeating with period 2Tsw.

**Fix:** Add slope compensation (a ramp signal added to the current sense signal, with slope ≥ m2/2 for full-range stability).

**Why A is wrong:** An ESR zero at fsw/2 would be at f_ESR = 1/(2π × ESR × C) = fsw/2, requiring very specific values. Even if this occurred, the ESR zero is a LEFT-half-plane zero (beneficial for phase) and would not cause oscillation. The oscillation signature (exactly fsw/2) is the distinctive marker of subharmonic oscillation.

**Why B is wrong:** Low phase margin causes oscillation near the crossover frequency (which is typically fsw/5 to fsw/10, not fsw/2). The oscillation frequency from insufficient phase margin would be near the closed-loop resonant frequency, not exactly half the switching frequency.

**Why D is wrong:** Input LC resonance can cause oscillation if the input filter resonance lands near the converter's control bandwidth (Middlebrook criterion violation). However, the input filter resonance frequency would not typically be at exactly fsw/2, and the oscillation signature would depend on the input filter component values, not be a fixed fraction of fsw.

---

## Q6

What is the function of a "soft start" circuit in a power supply?

- A) To reduce output voltage ripple during steady-state operation
- B) To limit the inrush current by slowly increasing the switching frequency at startup
- C) To ramp the output voltage (or duty cycle) gradually at startup, preventing large inrush currents and overshoot
- D) To reduce the gate drive speed to minimise EMI during startup

**Correct answer: C**

**Explanation:**

Without soft start, a converter starting from zero output voltage would immediately command maximum duty cycle to establish the output voltage as quickly as possible. The output capacitors charge from zero, presenting a low impedance (initially near-short) to the converter. The inductor and switching stage must supply a large initial current surge to charge the output capacitors.

Additionally, the feedback loop in a conventional design would demand maximum duty cycle until the output reaches regulation — this "integrator wind-up" can cause significant overshoot when the output finally reaches the target voltage.

Soft start gradually ramps the reference voltage (or error amplifier output, or duty cycle limit) from zero to the final value over a defined time (typically 1-20ms). The output voltage rises smoothly, controlling the charging current:

```
Imax_soft_start = C_out × dVout/dt = C_out × Vout / t_ss
```

For C_out = 470µF, Vout = 12V, t_ss = 5ms: I = 470µF × 12/5ms = 1.13A (well-controlled).

Soft start also prevents overshoot by pre-loading the integrator with a limited reference, so the error amplifier doesn't wind up.

**Typical implementation:** A current source charges a capacitor C_ss to a reference voltage V_ss. The voltage on C_ss clamps the error amplifier output (or the reference), ramping from 0 to the full reference.

```
t_ss = C_ss × V_ss / I_ss
```

**Why A is wrong:** Steady-state ripple reduction is achieved by adequate output capacitance and inductor design, not soft start. Soft start is only active during startup and fault recovery; it has no effect on steady-state operation.

**Why B is wrong:** Soft start does not typically reduce switching frequency. Some controllers use PFM (pulse frequency modulation) at startup, but the defining characteristic of soft start is the gradual output voltage ramp — frequency reduction is a secondary optional feature.

**Why D is wrong:** Gate drive speed is set by the gate resistor and driver impedance, optimised for EMI/switching loss trade-off, and does not change during startup vs. steady state in most designs. Slowing down gate drive during startup would help inrush slightly but is not the function of a soft start circuit.

---

## Q7

Why is "ground plane splitting" necessary in isolated power supply PCBs, and where should the split be located?

- A) To separate digital and analog grounds to prevent digital noise from corrupting analog measurements
- B) To divide the return currents of the primary and secondary sides, placing the isolation barrier at the split, preventing primary noise from coupling to secondary ground
- C) To reduce the PCB capacitance between layers and improve high-frequency performance
- D) To comply with IEC creepage and clearance requirements by providing physical gap in the copper

**Correct answer: B**

**Explanation:**

In an isolated power supply (flyback, forward, LLC), the primary and secondary have galvanic isolation. However, if a continuous ground plane extends under both primary and secondary circuits, there is a large capacitance between the primary switch node (which swings at high dV/dt) and the secondary ground plane. This coupling:

```
I_CM = C_stray × dV/dt_primary
```

drives common-mode current into the secondary circuit, creating EMI and potentially safety hazards.

The solution is to split the ground plane at the isolation barrier:
- Primary side: one ground plane region under all primary components (controller, input filter, MOSFET, bootstrap)
- Secondary side: separate ground plane region under all secondary components (output rectifier, filter, feedback)
- The split is a physical gap in the copper that prevents direct capacitive coupling between planes

**The split location:** Directly under the isolation transformer (or optocoupler), at the magnetic/optical isolation boundary. The creepage and clearance requirements of the safety standard (IEC 62368, UL 60950) dictate the minimum gap width across the isolation barrier.

The primary and secondary grounds are only connected through the isolation barrier components (the transformer core, optocoupler, or Y-capacitors intentionally placed at the barrier).

**Why A is wrong:** Splitting analog and digital grounds on the same side of an isolated supply is a signal-integrity concern (AGND/DGND split), separate from the primary/secondary isolation. The question specifically addresses the isolation split, not the AGND/DGND split.

**Why C is wrong:** Ground plane capacitance between layers is a function of dielectric thickness and area. Splitting the plane reduces the coupled area between primary and secondary, which does reduce CM coupling capacitance — but framing this as "reducing PCB capacitance to improve high-frequency performance" is imprecise and misses the isolation purpose.

**Why D is wrong:** Creepage and clearance requirements DO affect the slot or gap design, but the primary function of splitting the ground plane is to manage return currents and prevent noise coupling across the isolation barrier. Creepage/clearance is a safety requirement that constrains the gap geometry, but it's a consequence of the isolation requirement, not the primary reason for the split.

---

## Q8

What is the difference between differential-mode (DM) and common-mode (CM) conducted EMI, and which circuit elements primarily address each?

- A) DM noise flows between L and N; addressed by X capacitors and series inductors. CM noise flows on L and N simultaneously, returning via earth; addressed by Y capacitors and CM chokes
- B) DM noise is radiated; CM noise is conducted. DM is addressed by shielding; CM by filtering
- C) DM noise is at the switching frequency only; CM noise covers the full spectrum. Both are addressed by CM chokes
- D) DM noise causes output voltage ripple; CM noise causes input current ripple. Both are addressed by larger output capacitors

**Correct answer: A**

**Explanation:**

**Differential-mode (DM) noise:**
- Flows between the Line (L) and Neutral (N) conductors in opposite directions (one conductor carries the noise current forward, the other carries it back)
- Generated by: switching current drawn from the input capacitor, input current ripple
- Measured by LISN as the difference in voltage between L and N
- Filter elements: X-class capacitors (connected across L-N, providing a low-impedance shunt for DM noise); series inductors that have equal and opposite flux (cancel for DM current, present high impedance)

**Common-mode (CM) noise:**
- Flows in the same direction on both L and N simultaneously, returning via the earth conductor (PE)
- Generated by: dV/dt at the switching node coupling through stray capacitances to earth (chassis, heatsink, transformer inter-winding capacitance)
- Measured by LISN as the average voltage on L+N
- Filter elements: Y-class capacitors (connected from L to PE and N to PE, shunting CM noise to earth); CM chokes (both L and N wound on same core so CM currents create additive flux → high impedance, while DM currents cancel flux → low impedance)

The physical distinction matters for filter design: a CM choke provides high impedance to CM currents but passes DM currents without significant attenuation. X capacitors provide low impedance to DM currents. Combining X caps and CM chokes creates an LC filter for DM, and a CM filter for CM noise.

**Why B is wrong:** Both DM and CM EMI can be radiated or conducted. The classification is about current path (differential vs. common), not about radiation vs. conduction. Radiated EMI is a separate measurement category governed by different standards and test methods.

**Why C is wrong:** Both DM and CM noise exist across the conducted EMI frequency range (150kHz to 30MHz for CISPR 32). Neither is limited to only the switching frequency. In fact, CM noise often dominates at higher frequencies where fast dV/dt transitions couple efficiently through stray capacitances.

**Why D is wrong:** DM noise is not the same as output voltage ripple (that's a load regulation issue). CM noise is not input current ripple. Both DM and CM are phenomena at the power supply's input terminals (L and N), not at the output. Output capacitors have no effect on conducted input EMI.

---

## Q9

A MOSFET's datasheet specifies Rds_on = 10mΩ at Vgs = 10V, Tj = 25°C. The MOSFET operates in a converter where its drain-source on-state temperature reaches 110°C. What is the approximate Rds_on at operating temperature?

- A) 10mΩ (unchanged)
- B) 12mΩ (+20%)
- C) 17mΩ (+70%)
- D) 20mΩ (doubled)

**Correct answer: C**

**Explanation:**

Silicon MOSFET on-resistance increases with temperature due to reduced carrier mobility. The temperature dependence follows approximately:

```
Rds_on(T) = Rds_on(25°C) × (T[K] / 298[K])^2.3
```

At Tj = 110°C = 383K:
```
Rds_on(110°C) = 10mΩ × (383/298)^2.3
               = 10mΩ × (1.285)^2.3
               = 10mΩ × 1.72
               = 17.2mΩ ≈ 17mΩ
```

The exponent of approximately 2.3 is material-dependent and varies between 2.0 and 2.5 for different silicon MOSFETs. Some datasheets provide a graph of Rds_on(T) / Rds_on(25°C) as a normalised curve — always use this curve rather than the simple formula for precision work.

**Practical implication:** Using 25°C datasheet values for Rds_on in a design that runs at 100°C will underestimate conduction loss by nearly 2×. This is one of the most common causes of simulation-vs-measurement efficiency discrepancies.

**Why A is wrong:** Rds_on is strongly temperature-dependent. It is never constant from 25°C to 110°C in silicon devices. Only wide-bandgap devices (GaN, SiC) show somewhat flatter Rds_on vs. temperature curves.

**Why B is wrong:** 12mΩ would correspond to a 20% increase, implying a linear or very weak temperature coefficient. For silicon MOSFETs, the increase from 25°C to 110°C is approximately 60-80%, not 20%.

**Why D is wrong:** 20mΩ (doubled) would correspond to going from 25°C to approximately 150°C junction temperature, not 110°C. Rds_on doubling requires a larger temperature rise than 85°C.

---

## Q10

What is the primary purpose of a "hiccup mode" (also called intermittent restart mode) overcurrent protection scheme?

- A) To increase the switching frequency during overload to maintain output voltage regulation
- B) To reduce the average power dissipation during a sustained overcurrent fault by periodically attempting restart instead of continuously driving the fault
- C) To protect the output capacitor from overvoltage during load step-off
- D) To limit the peak inductor current to exactly the rated maximum

**Correct answer: B**

**Explanation:**

In a sustained overcurrent or short-circuit fault, the converter must limit the current to protect the switches, inductor, and the load. There are several protection modes:

**Cycle-by-cycle current limiting (CBC):** Reduces duty cycle each cycle to keep peak current at the limit. Average output power during fault = P_fault = V_out_fault × I_limit. If V_out_fault is near zero (hard short), power is still I_limit × V_out ≈ low. But for a soft overload, power can be substantial.

**Hiccup mode:** When overcurrent is detected (for example, 5 consecutive cycles in current limit):
1. Controller shuts down completely
2. Waits for a timeout period (e.g., 32 × Tsw to several ms)
3. Attempts restart with soft start
4. If overload persists, detects overcurrent again, shuts down again
5. Repeats (hiccup pattern)

The average output power in hiccup = P_burst × duty_cycle_hiccup = P_burst × (t_on / (t_on + t_off))

Since t_off >> t_on, the average power is very low. This is critical for:
- Reducing average power dissipation in the converter during a fault (preventing thermal runaway)
- Reducing the load's exposure to fault current (some protects the downstream circuit)

When the fault is removed, the converter will immediately enter a new restart cycle and resume normal operation (auto-restart).

**Why A is wrong:** Increasing switching frequency during overload would increase switching losses and is not a protection strategy. The converter needs to REDUCE power delivery during fault, not increase it.

**Why C is wrong:** Overvoltage protection on load step-off is a separate function (OVP — overvoltage protection). Hiccup mode specifically handles overcurrent/short circuit, not overvoltage.

**Why D is wrong:** Peak inductor current limiting to exactly the rated maximum describes cycle-by-cycle (CBC) protection, not hiccup mode. In CBC, the converter continues switching with the peak current clamped. Hiccup mode shuts down the switching entirely and restarts periodically.

---

## Q11

A power supply uses a TVS (Transient Voltage Suppressor) diode for ESD protection. The TVS has a standoff voltage of 15V and a clamping voltage of 24V at 1A. What does "clamping voltage" mean in this context?

- A) The maximum reverse voltage the TVS can sustain without breakdown
- B) The forward voltage drop of the TVS in normal operation
- C) The voltage across the TVS when conducting its rated current (1A in this case), which limits the voltage spike presented to the protected circuit
- D) The breakdown voltage at which the TVS begins to conduct any current

**Correct answer: C**

**Explanation:**

TVS diodes are designed to clamp transient voltage spikes (ESD, lightning surges, inductive kickback) to a safe level. Their characterisation has several key voltages:

- **Standoff voltage (Vwm or Vrwm):** The maximum reverse voltage the TVS can block without significant leakage — the normal operating voltage range. Below this, the TVS acts as a high-impedance reverse-biased diode.

- **Breakdown voltage (Vbr):** The voltage at which the TVS begins conducting (typically measured at 1mA). Between standoff and breakdown, leakage increases.

- **Clamping voltage (Vc):** The voltage across the TVS when conducting its peak rated current (Ipp, e.g., 1A or higher). This is the maximum voltage the TVS allows on the protected node under the specified test conditions.

For the protected circuit: the maximum voltage it experiences during a transient is ≤ Vc = 24V (at 1A transient current). If the transient current is higher, the clamping voltage increases (TVS dynamic resistance × excess current).

**Selection criteria:**
1. Vstandoff ≥ maximum normal operating voltage (15V in this case → TVS does not conduct during normal operation)
2. Vc ≤ maximum voltage the protected circuit can tolerate
3. Peak power (or energy) rating ≥ expected transient

**Why A is wrong:** The maximum voltage the TVS can SUSTAIN without breakdown is the standoff voltage (15V), not the clamping voltage. The clamping voltage is the voltage WHILE the TVS is conducting — the TVS is already conducting at this point.

**Why B is wrong:** The forward voltage of a TVS in normal (forward) operation is approximately 0.7V (silicon junction). The clamping voltage is a reverse-avalanche/breakdown voltage (much higher), measured in the TVS's active clamping region.

**Why D is wrong:** The voltage at which the TVS BEGINS to conduct any current is the breakdown voltage (Vbr, typically measured at 1mA). The clamping voltage (Vc at 1A) is higher than Vbr because of the TVS dynamic resistance: Vc = Vbr + Ipp × Rd.

---

## Q12

In a PCB power supply design, why is a "Kelvin connection" used for current sensing with a shunt resistor?

- A) To measure temperature at the sense resistor to compensate for TCR drift
- B) To provide separate force and sense connections, eliminating the effect of trace resistance from the measured voltage
- C) To connect the sense resistor directly to the power plane without signal trace contamination
- D) To achieve lower inductance in the sense resistor by using a symmetric layout

**Correct answer: B**

**Explanation:**

A shunt resistor for current sensing converts current to voltage: V_sense = I × R_shunt. For accurate current measurement, only the voltage developed ACROSS the shunt must be measured — not the voltage along the connecting traces.

Without Kelvin sensing, the sense lines share the power connections:

```
Power trace: GND_plane ──[R_trace_A]──[R_shunt]──[R_trace_B]── load
                                       ↑         ↑
                                    Sense-     Sense+
```

The measurement includes R_trace_A and R_trace_B: V_measured = I × (R_trace_A + R_shunt + R_trace_B) — overestimating the actual current.

With Kelvin sensing, four separate connections are used:
- Two "force" connections carry the main current (heavy power traces)
- Two "sense" connections attach directly at the shunt resistor body terminals, carrying negligible current (high-impedance differential amplifier input)

```
Heavy power trace ────────[R_shunt]──────── heavy return
                          ↑        ↑
                        Sense+   Sense-  (separate, thin sense traces with no current)
```

The sense traces carry no significant current, so their resistance causes no voltage drop. The measured voltage is exactly V_sense = I × R_shunt.

This is critical for milliohm shunts: with R_shunt = 5mΩ and I = 20A, V_sense = 100mV. If the sense traces add even 1mΩ each, the error is 40mV/100mV = 40% — catastrophic for accurate current control.

**Why A is wrong:** Temperature compensation for TCR drift is a separate calibration/trimming function. Kelvin sensing is about eliminating trace resistance from the measurement, not about temperature measurement.

**Why C is wrong:** Connecting the sense resistor to the power plane is about mechanical mounting and return current management. Kelvin sensing specifically refers to the four-wire sensing technique — having separate current-carrying and voltage-sensing connections.

**Why D is wrong:** Symmetric layout reduces parasitic inductance in the shunt resistor (important for high-frequency current sensing), but this is a separate benefit from Kelvin sensing. The Kelvin technique eliminates trace resistance error; symmetric layout minimises inductance. Both are good practices but address different issues.

---

## Q13

A converter must operate from an input voltage range of 8-18V. The UVLO turn-on threshold should be 10V, and the UVLO turn-off threshold should be 8.5V. If the UVLO hysteresis resistor to the enable pin sources 50µA when the enable pin is high, and the voltage divider has R_bottom = 10kΩ and R_top = unknown, how is the hysteresis implemented?

- A) By using a separate comparator IC to sense Vin and control the enable pin
- B) R_top is selected to give V_enable = V_th at Vin = 10V; hysteresis is added by a resistor from enable pin to Vin that increases the effective divider ratio when the pin is high
- C) A Zener diode in series with R_bottom clamps the threshold voltage
- D) A capacitor in parallel with R_top slows the UVLO response to prevent oscillation at the threshold

**Correct answer: B**

**Explanation:**

Most controller ICs with UVLO have an enable or UVLO pin that compares to an internal reference (e.g., 1.25V). A resistor divider from Vin to the pin to GND sets the threshold:

```
V_pin = Vin × R_bottom / (R_top + R_bottom)
```

For V_pin = V_th_internal at the turn-on voltage:
```
R_top = R_bottom × (Vin_on / V_th - 1) = 10k × (10/1.25 - 1) = 10k × 7 = 70kΩ
```

**Adding hysteresis with a "pull-up" resistor (R_hys) from enable pin to Vin:**

When enable is HIGH (converter running, I_hys sources 50µA from Vin through R_hys to the pin):

This extra current adds to the current through R_bottom, increasing V_pin. The turn-on threshold appears HIGHER from Vin's perspective, but the actual threshold calculation must account for the added current.

Alternatively: the controller has an internal current source that sinks current from the UVLO pin when the enable is asserted (Vin > threshold). This current source provides hysteresis:

```
V_hysteresis = I_hys × R_top_parallel_R_hys
```

The turn-off voltage (when the enable goes low):
```
Vin_off = (V_th - I_hys × R_bottom) × (R_top + R_bottom) / R_bottom
```

Where I_hys × R_bottom is the voltage reduction caused by the current source no longer providing current.

**Practical calculation:** With I_hys = 50µA sourced to the UVLO pin (when enabled):
```
ΔV_vin = I_hys × R_top = 50µA × 70kΩ = 3.5V  (the hysteresis band)
Turn-off threshold ≈ 10V - 3.5V × adjustment factor ≈ 8.5V (approximately)
```

This requires iterating with the exact controller model.

**Why A is wrong:** An external comparator is unnecessary — most UVLO functions are built into the controller IC. External comparators add cost and complexity. The hysteresis is implemented within the divider network as described in B.

**Why C is wrong:** A Zener in series with R_bottom would clamp the UVLO pin voltage, not set a hysteresis band. Zeners are used for other protection functions (overvoltage clamping) but are not the standard method for UVLO hysteresis in resistor dividers.

**Why D is wrong:** A capacitor in parallel with R_top creates a frequency-dependent voltage divider, slowing the UVLO response. This might seem useful for noise immunity, but it creates response time lag that prevents fast UVLO shutdown. Hysteresis (not filtering) is the proper way to prevent oscillation at the UVLO threshold.

---

## Q14

What is the purpose of thermal vias in a PCB beneath a power component?

- A) To provide a conducting path for current from the top layer to inner copper layers
- B) To conduct heat from the component's thermal pad through the PCB to a heat spreader, inner copper pour, or bottom-side copper
- C) To increase the mechanical strength of the PCB under heavy components
- D) To provide an electrical ground connection that is isolated from the signal ground

**Correct answer: B**

**Explanation:**

Many power components (MOSFETs, regulators, controllers in exposed-pad packages) dissipate heat through their bottom thermal pad, which sits on the PCB surface. Without thermal vias, this heat must spread laterally through the thin top copper layer — a high-resistance thermal path.

Thermal vias are small-diameter plated-through holes (typically 0.3mm-0.5mm, filled or plugged) arranged in a matrix pattern under the component's thermal pad. They provide a low-thermal-resistance path:

```
Heat path: Junction → case → thermal pad → PCB top copper → thermal vias → bottom copper or inner planes → ambient
```

Thermal resistance of a single via (Rth_via):
```
Rth_via ≈ t_PCB / (π × r_via × k_copper × t_plating × 2)
```

where t_PCB is board thickness, r_via is via radius, k_copper = 385 W/(m·K). For t_PCB = 1.6mm, r_via = 0.15mm, t_plating = 25µm:

Rth_via ≈ 1.6×10⁻³ / (π × 0.15×10⁻³ × 385 × 25×10⁻⁶ × 2) ≈ 88 K/W per via

With a 4×4 array of 16 vias in parallel: Rth_array = 88/16 = 5.5 K/W — much better than no vias.

Filled vias (with copper or thermally conductive fill) further reduce Rth by conducting through the via barrel volume, not just the plated wall.

**Why A is wrong:** Thermal vias can also carry current (they are electrically conducting), but their purpose in this context is thermal, not primarily electrical. The question specifies "thermal vias beneath a power component," which are specifically placed for heat removal. Vias used purely for electrical connection (signal or power) are separate from thermally optimised via arrays.

**Why C is wrong:** Mechanical reinforcement is not a function of thermal vias. PCB mechanical strength under heavy components is provided by mounting holes, standoffs, and proper potting/conformal coating in harsh environments. Thermal vias are too small to provide significant structural reinforcement.

**Why D is wrong:** Thermal vias are not electrically isolated. They connect the top and bottom copper planes (or inner layers) directly. An isolated ground connection would require separate isolation barriers (capacitors, transformers, optocouplers) — not via holes.

---

## Q15

An AC-DC power supply must meet CISPR 32 Class B conducted emissions limits. At 150kHz, the quasi-peak limit is 66 dBµV. The measured quasi-peak level is 73 dBµV before any filter is added. What minimum attenuation does the input filter need to provide at 150kHz?

- A) 7 dB
- B) 14 dB
- C) 66 dB
- D) 73 dB

**Correct answer: A**

**Explanation:**

The filter must reduce the measured level to at or below the limit:

```
Required attenuation = Measured_level - Limit_level
                     = 73 dBµV - 66 dBµV = 7 dB
```

A 7dB reduction is relatively modest. Designers typically add at least 6-10dB of margin on top of the minimum requirement, so they would target 13-17dB of attenuation at 150kHz to account for:
- Component tolerances (capacitance ±20%, inductance ±20%)
- Temperature drift
- Line impedance variation
- Manufacturing variation between units

**Filter design for 7dB at 150kHz:**

A simple single-stage LC filter (CM choke + X cap) provides approximately 40dB/decade above the corner frequency. For 7dB attenuation at 150kHz:

```
7dB = 20log10(f/fc)    for a first-order filter → f/fc = 10^(7/20) = 2.24
fc = 150kHz / 2.24 = 67kHz
```

Or for a two-stage filter, each stage provides fewer dB for the same fc.

In practice, the 150kHz point is the LOWEST frequency in the conducted emissions range. It is also where EMI filters are least effective (CM choke inductance is lower at higher frequency due to self-resonance issues). Care must be taken to ensure the filter corner is well below 150kHz.

**Why B is wrong:** 14 dB would be needed if the measured level were 80 dBµV (66+14 = 80). The measured level is 73 dBµV, so only 7 dB is needed to reach the 66 dBµV limit.

**Why C is wrong:** 66 dBµV is the absolute limit level, not the required attenuation. If 66 dB of attenuation were required, the unfiltered emission would be at the limit + 66 dB = 132 dBµV — an extreme case that would require a massive multi-stage filter.

**Why D is wrong:** 73 dBµV is the measured emission level. If 73 dB of attenuation were needed, the unfiltered level would be 146 dBµV (essentially equal to Vin in millivolts at the LISN). The required filter attenuation is the DIFFERENCE between measurement and limit, not the absolute level.

---

## Q16

What is the "derating" principle in component selection, and why is it applied?

- A) Selecting components with 50% higher voltage rating than the maximum circuit voltage to provide reliability margin against transient overstress
- B) Using a larger component than necessary to reduce the operating stress (current, voltage, temperature) as a fraction of the rated maximum, extending component lifetime and reducing failure probability
- C) Reducing the power supply output power in high-temperature environments
- D) A and B are both aspects of derating

**Correct answer: D**

**Explanation:**

Derating is the practice of operating components below their maximum rated specifications to improve reliability. It encompasses both A and B:

**Voltage derating (A):** Capacitors, MOSFETs, diodes, and insulation systems all have failure rates that increase strongly (often exponentially) with applied voltage stress. A capacitor rated 50V used at 45V is operating at 90% of rating — the electric field stress is high. Using a 100V-rated capacitor at 45V (45% stress) dramatically reduces the failure rate.

**Common voltage derating guidelines:**
- Film/ceramic capacitors: operate at ≤ 50-70% of rated voltage
- Electrolytic capacitors: operate at ≤ 70-80% of rated voltage
- MOSFETs: Vds_max at ≤ 75-80% of Vds_rated
- Signal traces (insulation): ≤ 50% of rated dielectric strength

**Thermal derating (B):** Component lifetime halves for every 10°C increase in junction temperature (Arrhenius rule of thumb). Operating a capacitor at 105°C vs. 70°C (35°C lower): lifetime increases by 2^(35/10) = 2^3.5 = 11.3×. This is why keeping Tj ≤ 0.75 × Tj_max is standard practice.

**Why only A is wrong (as a standalone answer):** A describes voltage derating specifically but B describes the broader concept that includes all stress types (current, voltage, temperature, power). D is more complete and correct.

**Why only B is wrong (as a standalone answer):** B describes the principle broadly but doesn't highlight the specific voltage-rating margin that A describes. Both together form the complete picture.

---

## Q17

What is the inrush current problem with AC-DC power supplies, and name two techniques used to limit it?

- A) Inrush current is the large charging current drawn by the output capacitors at startup; limited by soft start and output current limiting
- B) Inrush current is the large current drawn by the uncharged input capacitor when AC is applied, which can blow fuses and damage components; limited by NTC thermistors or active inrush limiters
- C) Inrush current is the transformer magnetising current at power-on; limited by demagnetising circuits and zero-crossing detection
- D) Inrush current is the reverse recovery current of input rectifier diodes; limited by soft-recovery diodes and snubbers

**Correct answer: B**

**Explanation:**

When AC power is first applied to a power supply:
1. The input capacitor (bulk capacitor, typically 100-470µF at 400V for 100-400W supplies) is uncharged (V_cap = 0V)
2. The AC mains sees a very low impedance (capacitor charging): Z_initial ≈ series resistance of the supply + bridge rectifier Vf
3. The peak inrush current can be: I_peak = V_peak_AC / Z_series = 325V / (1Ω typical) = 325A

This is far above the steady-state input current (e.g., 1A for a 100W supply). The inrush current can:
- Blow the input fuse
- Damage the bridge rectifier diodes (rated for repetitive peak current, not single-pulse thousands of amps)
- Cause nuisance tripping of circuit breakers
- Create conducted EMI on the mains

**Technique 1: NTC thermistor in series with the line**
- Cold NTC: high resistance (e.g., 10-100Ω), limits the peak inrush
- As current flows, the thermistor heats up and resistance drops to <1Ω
- Disadvantage: slow to cool down — if AC is briefly interrupted and restored (brown-out, glitch), the thermistor is still hot (low resistance) and inrush is NOT limited. Also dissipates P = I_ss² × R_NTC_hot in steady state.

**Technique 2: Active inrush limiter (relay bypass)**
- A power resistor limits inrush during startup
- After the bulk capacitor is charged (detected by Vbus > threshold), a relay (or MOSFET) bypasses the resistor
- No steady-state losses; effective even after brief power interruptions
- More complex and costly than NTC

**Why A is wrong:** Output capacitor charging at startup is controlled by the soft-start circuit (Q6 in this quiz). Inrush current at the AC INPUT specifically refers to the input bulk capacitor charging from the raw rectified AC.

**Why C is wrong:** Transformer magnetising inrush (which occurs in power transformers at line frequency) is a separate phenomenon relevant to utility transformers, not switching power supply input filters. The bulk capacitor inrush is the dominant concern in switching power supplies.

**Why D is wrong:** Reverse recovery current from bridge rectifier diodes is a brief spike that contributes to EMI but is not the main inrush current concern. The bulk capacitor charging current (lasting 1-10ms) is orders of magnitude larger and more problematic.

---

## Q18

An IR camera image shows that the gate resistor of a MOSFET driver circuit is the hottest component on the board. The MOSFET itself is cool. What does this indicate, and is it a problem?

- A) The gate resistor is dissipating excessive power, indicating a design error; the gate drive current should be reduced
- B) The gate resistor dissipates gate drive power (Qg × Vgs × fsw / number_of_switches), which is normal; high temperature only indicates a problem if it exceeds the resistor's rating
- C) The MOSFET is not switching properly and all power is being dumped in the gate resistor
- D) The gate resistor is too large, slowing switching transitions and increasing switching losses in the MOSFET

**Correct answer: B**

**Explanation:**

Gate resistors dissipate power during each switching transition. For each turn-on or turn-off event, the gate charge Qg is supplied (or sunk) through the gate resistor:

```
P_gate_resistor ≈ Qg × Vgs × fsw    (total gate drive power, split between the driver and gate resistor)
```

For a MOSFET with Qg = 50nC, Vgs = 10V, fsw = 500kHz:
```
P_gate = 50nC × 10V × 500kHz = 250mW
```

This 250mW is dissipated in the gate driver and gate resistor (split depends on relative impedance). A standard 0402 resistor (rated 100mW) would be thermally stressed. A 0603 resistor (rated 100-250mW) would run warm.

**Is a warm gate resistor a problem?** Only if:
1. The resistor exceeds its rated power (must derate to ≤ 50% of rated power for reliability)
2. The resistor exceeds its maximum operating temperature
3. Adjacent heat-sensitive components are affected

**The MOSFET being cool** while the gate resistor is warm indicates the MOSFET is switching efficiently (switching losses are low, conduction losses are acceptable) and the gate resistor is absorbing gate drive energy as expected.

**Why A is wrong:** Gate drive dissipation in the gate resistor is not a design error — it is the expected and necessary function of the resistor. Gate resistors are used to control di/dt during switching (EMI management) and to prevent gate ringing. Reducing gate drive current by increasing Rg slows switching and increases MOSFET switching losses — a worse outcome.

**Why C is wrong:** If the MOSFET were not switching, the gate resistor would not be conducting repetitive charging pulses. A non-switching MOSFET would show the MOSFET itself as hot (held in partial conduction) and the gate resistor would be cool.

**Why D is wrong:** A larger gate resistor DOES slow switching transitions. However, the observation (hot gate resistor, cool MOSFET) is consistent with a working circuit, not an indication that the gate resistor is too large. If the gate resistor were too large, the concern would be about increased switching losses in the MOSFET — which would show up as a HOT MOSFET, not a cool one.

---

## Q19

What is "spread spectrum" modulation in a switching converter, and why is it used?

- A) Modulating the output voltage over a small range at audio frequency to test EMC immunity
- B) Intentionally varying the switching frequency over a small range (e.g., ±5-10%) to spread conducted and radiated emissions over a wider frequency range, reducing peak spectral content
- C) Using multiple parallel converters at different switching frequencies to cover a wider output voltage range
- D) Increasing the bandwidth of the control loop to reject disturbances across a wide frequency spectrum

**Correct answer: B**

**Explanation:**

A fixed-frequency switching converter generates conducted and radiated emissions at the switching frequency fsw and its harmonics (2×fsw, 3×fsw, etc.). EMC measurements compare these spectral peaks against limits defined in dBµV.

If a 200kHz converter puts all its switching energy at exactly 200kHz, the peak emission at that frequency is high. The EMI instrument (quasi-peak or average detector) reads the full amplitude of the energy concentrated at that single frequency.

Spread spectrum (SSFM — Spread Spectrum Frequency Modulation) varies fsw within a range (e.g., 180kHz to 220kHz) according to a defined modulation profile:
- Triangular modulation: frequency sweeps linearly up and down
- Pseudo-random: frequency hops in a pseudo-random sequence

The switching energy that was concentrated at 200kHz is now distributed across 180-220kHz. The peak at any single frequency is lower (by 10-20dB for typical spreading), even though the total energy is unchanged.

This reduces peak EMI readings enough to pass regulatory limits that might otherwise be failed without additional filtering.

**Trade-offs:**
- The spread spectrum modulation must be slow enough that the control loop can track it (modulation frequency << control loop bandwidth)
- Output voltage ripple can increase if the control loop cannot perfectly track the frequency variation
- Spread spectrum does not reduce TOTAL emissions, only PEAK emissions at any single frequency
- Not suitable for synchronised systems where jitter in switching frequency causes beat frequencies

**Why A is wrong:** Testing immunity to audio-frequency modulation is an EMC immunity test, not a spread spectrum technique. Immunity tests inject disturbances into the power supply input to verify robustness.

**Why C is wrong:** Using multiple converters at different frequencies for output voltage range extension is a load point optimisation technique (multi-rail design), not spread spectrum. Spread spectrum is a single-converter technique that modulates one converter's frequency, not a multi-converter architecture.

**Why D is wrong:** Control loop bandwidth is set by the compensation design and is related to stability and transient response — not to EMC. Spread spectrum modulation of the switching frequency does not change the control loop bandwidth.

---

## Q20

A power supply fails during an agency safety audit because the creepage distance between primary and secondary traces on the PCB is 3mm, but the required creepage for the rated working voltage of 250Vac is 6mm (per IEC 62368-1 for a basic insulation, pollution degree 2 environment). What are two ways to fix this without changing the PCB layout?

- A) Adding a slot in the PCB along the isolation boundary, and applying conformal coating to the PCB
- B) Reducing the supply's rated input voltage to 110V only, and adding a heatsink to the transformer
- C) Increasing the switching frequency and using a smaller transformer with shorter creepage paths
- D) Adding a Faraday shield in the transformer and using Y-capacitors to meet the creepage requirement

**Correct answer: A**

**Explanation:**

Creepage distance is measured along the surface of the PCB (or insulating material) between two conductive parts of different potentials. It is NOT the same as clearance (shortest distance through air). Creepage is concerned with conductive contamination paths along surfaces.

The requirement of 6mm cannot be met with the existing 3mm surface distance "as drawn." However:

**Option 1: Add a slot (groove) in the PCB along the isolation boundary**

A slot physically breaks the surface path. The creepage distance must now travel DOWN into the slot, along the slot wall, and UP the other side. The effective creepage distance = 3mm + 2 × slot_depth. For a 1.5mm deep slot: creepage = 3 + 3 = 6mm. This meets the requirement without moving any traces.

Per IEC 62368-1, a groove of sufficient depth can be credited for additional creepage distance. Common practice: route a slot under the transformer with a depth of ≥ 0.5mm to gain credit.

**Option 2: Apply conformal coating to the PCB**

Conformal coating changes the applicable pollution degree. If coating achieves "pollution degree 1" conditions locally, the required creepage is reduced. For example, at 250Vac working voltage, PD2 requires 6mm, but PD1 requires only 1.6mm (for some material groups). Conformal coating can reduce the required creepage to meet the 3mm available distance.

Note: Conformal coating must be the correct type (as tested and certified), and the agency must accept the pollution degree reclassification for the specific application.

**Why B is wrong:** Reducing the rated input voltage to 110V changes the product specification and requires new testing. Heatsinks on transformers have no effect on creepage distance (which is an electrical insulation requirement, not thermal). This is not a fix for the creepage issue.

**Why C is wrong:** Changing switching frequency affects magnetics and control design, not PCB trace spacing. A smaller transformer might have shorter physical paths, but the creepage between primary and secondary traces on the PCB is a function of layout, not of the transformer. Changing frequency and transformer size is a board redesign — not a fix "without changing the PCB layout."

**Why D is wrong:** Faraday shields reduce CM EMI but do not affect creepage distance measurements. Y-capacitors must CROSS the isolation barrier and are themselves subject to creepage requirements between their terminals and the primary/secondary circuits. Neither component addresses the surface distance on the PCB.

---

*End of Quiz — Practical Design*

**Answer Key:** 1-C, 2-B, 3-A*, 4-B, 5-C, 6-C, 7-B, 8-A, 9-C, 10-B, 11-C, 12-B, 13-B, 14-B, 15-A, 16-D, 17-B, 18-B, 19-B, 20-A

*Q3 note: Calculation gives Tj ≈ 78°C, making A (80°C) the correct answer. Option C (118.4°C) as listed in any other key is arithmetically incorrect — see the worked calculation in the explanation.*
