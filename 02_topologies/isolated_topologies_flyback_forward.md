# Isolated Topologies: Flyback and Forward Converters — Interview Preparation

## Overview

Isolated topologies use a transformer to provide galvanic isolation between input and output, enabling safety isolation, voltage step-up/down ratios beyond what non-isolated topologies allow, and multiple independent output rails. The flyback and forward converters are the workhorse topologies for low-to-medium power (1–300W) isolated applications.

---

## Key Equations Reference

### Flyback Converter

**Turns ratio (DCM/CCM):**
```
n = Np / Ns = Vout_reflected / Vin
n = (Vin × D) / (Vout × (1-D))    [CCM approximation, ignoring diode drop]
```

**Magnetising inductance (CCM boundary):**
```
Lm_crit = n² × Vout × (1-D)² / (2 × fsw × Pout / Vin)
```

**Peak magnetising current:**
```
IL_peak = (Vin × D) / (Lm × fsw) + Im_initial
```

**Output voltage ripple:**
```
ΔVout = Iout × (1-D) / (fsw × Cout)   [capacitive, DCM approximation]
```

**MOSFET voltage stress (flyback):**
```
V_DS_max = Vin + n × Vout + V_spike   [clamp or snubber limits V_spike]
```

### Forward Converter

**Turns ratio:**
```
n = Np / Ns
D = n × Vout / Vin  (same as buck in terms of output volt-second balance)
```

**Maximum duty cycle (reset constraint):**
```
D_max ≤ N_reset / (N_reset + Np)   [for RCD or third-winding reset]
D_max = 0.5                          [for active clamp or two-switch forward]
```

**Reset voltage (third winding, equal turns):**
```
V_reset = Vin × (D / (1-D))   [must not exceed MOSFET Vds_max]
```

---

## Fundamentals (Questions 1–6)

---

### Q1. How does a flyback converter work? Explain the energy transfer mechanism.

**Answer:**

The flyback converter stores energy in the transformer's magnetising inductance during the switch on-time and transfers it to the output during the off-time. Despite using a "transformer," it is conceptually a boost converter with galvanic isolation — energy is never transferred directly (primary to secondary simultaneously).

**Switch ON phase:**
- Primary switch (MOSFET) closes, applying Vin across the primary winding.
- Current in the primary ramps up linearly: `dIm/dt = Vin / Lm`
- The secondary diode is reverse biased (the secondary winding polarity opposes output).
- Energy is stored in the magnetising inductance (physically in the air gap of the gapped core).

**Switch OFF phase:**
- MOSFET opens. The magnetising current must continue to flow (Lenz's law).
- The transformer flips its polarity. Secondary winding now forward biases the diode.
- Energy transfers to the output capacitor and load: `V_secondary = n × Vin` (reflected voltage)
- Wait — more precisely: `V_secondary = n × (Vin + V_clamp)` is not right; the secondary voltage during discharge is `Vout + Vf` which appears on the primary as `n × (Vout + Vf)`.

**Key difference from a forward converter:**
- Forward converter: energy transferred simultaneously (like a transformer)
- Flyback converter: energy stored then transferred (like a two-switch buck-boost)

**Coupled inductor model:**
The "transformer" in a flyback is properly called a coupled inductor. It has intentional air gap to store energy (unlike a real power transformer, which should have no air gap).

---

### Q2. Derive the voltage conversion ratio for a flyback converter in CCM and DCM.

**Answer:**

**CCM (Continuous Conduction Mode):**

Volt-second balance applies to the magnetising inductance. During on-time: `VLm = Vin`. During off-time: `VLm = -n × Vout` (secondary voltage reflected to primary).

```
Vin × D × Ts = n × Vout × (1-D) × Ts
Vout = Vin × D / (n × (1-D))
```

This is analogous to the boost converter with a turns ratio scaling factor. Note that Vout/Vin can be greater than 1 or less than 1 depending on n and D.

**DCM (Discontinuous Conduction Mode):**

In DCM, the magnetising current returns to zero before the end of each switching period. An additional parameter — the load current — enters the conversion ratio.

The DCM voltage conversion ratio (M = Vout × n / Vin):
```
M = (1 + √(1 + 4D² / K)) / 2
where K = 2 × Lm × fsw / (n² × Rload)
```

For K << 1 (deep DCM):
```
Vout ≈ Vin × D / (n × √(2 × Lm × fsw / Rload))
```

This means Vout depends on load in DCM — lighter load → higher Vout for the same duty cycle. The control loop must adjust D to regulate against load changes.

**Why DCM is common in flybacks:**

Many flyback converters are designed to operate in DCM because:
1. No right-half-plane (RHP) zero in DCM — simpler loop compensation
2. Transformer resets every cycle — no transformer saturation risk from duty cycle imbalance
3. Output diode turns off at zero current — no reverse recovery loss

---

### Q3. What is the MOSFET voltage stress in a flyback converter and how do you clamp it?

**Answer:**

**Voltage stress analysis:**

When the primary MOSFET turns off, the drain voltage rises to:
```
V_DS = Vin + V_reflected + V_spike
```
where:
- Vin: input voltage
- V_reflected = n × Vout: reflected output voltage (n = Np/Ns)
- V_spike: overvoltage from leakage inductance energy release

**Leakage inductance spike:**

The primary leakage inductance (Llk) is not coupled to the secondary. When the MOSFET turns off, energy stored in Llk must be dissipated or clamped:
```
E_leakage = 0.5 × Llk × Iprimary_peak²
```
This energy releases as a high-voltage spike with fast rise time. Without a snubber, V_spike can exceed 100V even with Vin = 24V.

**Clamp circuits:**

1. **RCD snubber (dissipative):**
   - Resistor R, capacitor C, and diode D form a clamp.
   - Clamp voltage: `V_clamp = R × I_leakage_peak² × fsw × Llk / (2 × (V_clamp - V_reflected))`
   - Simple, low cost, but leakage energy is wasted as heat in R.
   - Typically clamps to V_reflected + 20–30% overhead.

2. **TVS diode clamp (dissipative):**
   - Transient voltage suppressor diode clamps the drain voltage.
   - Instantaneous response, but must be sized for the peak power.
   - Leakage energy wasted as heat.

3. **Active clamp (regenerative):**
   - An additional MOSFET and capacitor return leakage energy to the input.
   - Nearly zero additional loss.
   - Enables zero-voltage switching of the main switch.
   - More complex gate drive circuitry required.

**MOSFET selection:**
```
V_MOSFET_rated ≥ 1.5 × (Vin_max + n × Vout + V_clamp)
```
For Vin=24V, n=0.4, Vout=12V, V_clamp=15V: rated ≥ 1.5 × (24 + 4.8 + 15) = 65.7V → use 100V MOSFET.

---

### Q4. What is the forward converter and how does it differ from a flyback?

**Answer:**

The forward converter transfers energy from primary to secondary simultaneously during the switch on-time, like a conventional transformer — hence the name "forward." It includes an output LC filter (like a buck converter) on the secondary.

**Key structural differences:**

| Feature | Flyback | Forward |
|---------|---------|---------|
| Energy transfer | Stored then transferred | Simultaneously transferred |
| Output filter | Capacitor only | LC filter (like buck) |
| Core utilisation | Unidirectional, gapped | Unidirectional, reset required |
| Output ripple | Higher (capacitor only) | Lower (LC filter) |
| Transformer | Coupled inductor (gapped) | True transformer (small gap) |
| Multiple outputs | Diode + cap each | Inductor + diode + cap each |
| Duty cycle limit | <0.5 (reset constraint) | <0.5 typically |
| Power range | 1–200W typical | 50–500W typical |

**Forward converter operation:**
- ON time: MOSFET closes, primary current flows, secondary diode D1 conducts, energy transfers to output.
- OFF time: MOSFET opens, core must reset (demagnetise). Reset winding or clamp circuit brings core flux back to zero.
- Output inductor provides continuous secondary current (like a buck converter) — freewheeling diode D2 conducts during off-time.

**Core reset requirement:**
The volt-second applied to the core during on-time must be reversed during off-time to prevent transformer saturation. This limits duty cycle to ≤ 50% for a single-switch forward with equal turns reset winding.

---

### Q5. Describe the four main transformer reset methods for a single-switch forward converter.

**Answer:**

**1. Third winding (demagnetisation winding) reset:**

A third winding with the same number of turns as the primary is wound bifilar with the primary. A diode connects this winding to Vin.

During off-time, the core flux drives current through the reset winding back into Vin. The reset voltage equals Vin (for equal turns), so reset time equals on-time. This limits D_max ≤ 0.5.

- Pros: Simple, energy returned to input (efficient).
- Cons: D_max ≤ 0.5, extra winding adds transformer size.

**2. RCD snubber (clamp) reset:**

A resistor, capacitor, and diode clamp the drain voltage. The capacitor absorbs the reset energy; the resistor dissipates it.

- Pros: Simple, no extra winding, D_max can approach 1.0.
- Cons: Reset energy wasted in R (especially at high duty cycle). More lossy than winding reset.

**3. Active clamp reset:**

An additional MOSFET and capacitor (active clamp) are placed across the primary switch or across the primary winding. The clamp MOSFET turns on during the off-time, allowing the magnetising current to charge/discharge the clamp capacitor.

- Pros: Zero-voltage switching possible, energy is recycled, D_max > 0.5 achievable.
- Cons: Requires complementary gate driver, careful dead-time management.

**4. Two-switch forward converter:**

Two MOSFETs (high-side and low-side) switch simultaneously. Two diodes connected from each MOSFET drain to the rails clamp the voltage. During off-time, the diodes conduct the magnetising current back to the input.

- Pros: Each MOSFET only sees Vin (not 2×Vin), so lower voltage rating needed.
- Cons: High-side gate driver needed, more components, D_max = 0.5.
- Pros: No separate reset winding needed, more reliable transformer design.

---

### Q6. What is the key trade-off between operating a flyback in CCM versus DCM?

**Answer:**

**DCM advantages:**
1. **No RHP zero:** The right-half-plane zero that appears in CCM flyback (similar to boost) makes the loop difficult to compensate. DCM eliminates the RHP zero, allowing simpler Type II compensation with wider bandwidth.
2. **Natural transformer reset:** In DCM, the secondary current reaches zero each cycle — the transformer core fully demagnetises without requiring special reset circuits.
3. **No reverse recovery:** Output diode turns off at zero current, eliminating reverse recovery losses (important at high switching frequency).

**DCM disadvantages:**
1. **Higher peak currents:** For the same average output power, DCM requires larger peak primary currents (1.5–2× higher than CCM). This increases MOSFET and diode current ratings.
2. **Higher RMS currents:** More copper loss in transformer windings.
3. **Load-dependent voltage:** Vout varies with load in DCM. The control loop must compensate.
4. **EMI:** Larger current swings create more conducted and radiated EMI.

**CCM advantages:**
1. **Lower peak and RMS currents** for the same output power.
2. **Better EMI** characteristics (smaller current ripple).
3. **Wider regulation range** — duty cycle less sensitive to load.

**CCM disadvantages:**
1. **RHP zero:** The RHP zero at `f_RHP = (1-D)² × Rload / (2π × Lm)` limits the achievable bandwidth.
2. **Transformer reset:** Requires active clamp or two-switch topology for D > 0.5.
3. **Diode reverse recovery:** Output diode turns off with forward current — reverse recovery can cause ringing and loss.

**Practical design choice:**
- Low power (<50W), cost-sensitive: DCM flyback (simple compensation, tolerates transformer reset variation).
- Medium power (50–300W), efficiency-critical: CCM flyback with active clamp or resonant transition.
- High power (>300W): Consider LLC or phase-shifted full bridge instead.

---

## Intermediate (Questions 7–12)

---

### Q7. How do you design an RCD snubber for a flyback converter?

**Answer:**

The RCD snubber absorbs energy from the primary leakage inductance when the main switch turns off.

**Design procedure:**

**Step 1 — Measure or estimate leakage inductance:**

Measure Llk by shorting the secondary winding and measuring primary inductance:
```
Llk ≈ Lprimary(secondary_shorted) / 10 to 100  [typically 1-5% of Lm]
```

**Step 2 — Determine peak primary current:**
```
Ip_peak = Vin × D / (Lm × fsw)  [DCM] or Iout × (n + ripple)/n  [CCM]
```

**Step 3 — Calculate snubber capacitor (Cs):**

The clamp capacitor must be large enough to limit the voltage rise to the desired clamp level V_clamp:
```
Cs ≥ Llk × Ip_peak² / (2 × (V_clamp - Vin_max - n×Vout)²)
```

where V_clamp is the target clamp voltage (typically Vin + 30–50% headroom above reflected voltage).

For Vin=100V, n=0.5, Vout=12V, reflected=24V, target V_clamp=150V:
```
V_clamp - Vin - V_reflected = 150 - 100 - 24 = 26V
Cs = 5µH × 3² / (2 × 26²) = 45µJ / 1352 = 33 nF
```

**Step 4 — Select snubber resistor (Rs):**

Rs must discharge Cs to near V_reflected before the next switching cycle:
```
V_clamp(t) = (V_clamp - V_reflected) × e^(-t/τ) + V_reflected
τ = Rs × Cs should be ≤ (1-D) × Ts / 5  [at least 5τ discharge]

Rs ≤ (1-D) × Ts / (5 × Cs)
```

For D=0.4, fsw=100kHz, Ts=10µs, Cs=33nF:
```
Rs ≤ 0.6 × 10e-6 / (5 × 33e-9) = 6e-6 / 165e-9 = 36.4 kΩ
```

**Step 5 — Verify power dissipation:**
```
P_snubber = 0.5 × Cs × V_clamp² × fsw + Llk × Ip_peak² × fsw / 2
          ≈ Llk × Ip_peak² × fsw / 2   [dominant term for tight clamp]
```

**Common mistakes:**
- Making Cs too small → voltage clamp too high → MOSFET stress
- Making Rs too large → Cs doesn't fully discharge → clamp voltage drifts up over time
- Not derate Rs for temperature — snubber resistors can get very hot

---

### Q8. How does the flyback transformer differ from a conventional power transformer in design?

**Answer:**

**Flyback transformer (coupled inductor):**

A flyback "transformer" must store significant magnetic energy during the primary on-time. This requires a controlled air gap in the core:
```
Energy stored = 0.5 × Lm × Ipeak²
```

The air gap provides:
1. Defined magnetising inductance (Lm ∝ 1/gap_length)
2. Energy storage capability (gapped cores have higher energy density)
3. Soft saturation characteristic (some inductors prefer powder cores)

**Core characteristics needed for flyback:**
- High energy storage (Ae × le / µ0 × µr must be adequate)
- Low core loss at operating frequency (ferrite preferred above 50 kHz)
- Consistent air gap (gapped E-cores or toroidal powder cores)

**True power transformer (forward, bridge topologies):**

A power transformer transfers energy instantaneously — it should not store significant magnetising energy. Design objectives:
- Minimum magnetising inductance variation (tight gap tolerance)
- Small or no air gap (just to prevent DC saturation from imbalance)
- High coupling coefficient (low leakage inductance)
- Low core loss
- Sufficient thermal handling

**Winding structure comparison:**

Flyback:
- Primary and secondary wound separately (not interleaved)
- Separate winding layers reduce leakage inductance — but for flyback, some leakage is managed by snubbers
- Creepage/clearance distance critical for safety isolation

Forward/bridge:
- Interleaved primary and secondary winding reduces leakage inductance dramatically
- Low leakage inductance improves output voltage regulation and reduces diode stress

**Key interviewer trap:** If asked "how does a flyback transformer work," do not say it works "like a transformer." Correct answer: it works like a coupled inductor — energy is stored in the magnetic field during one phase, then released during another.

---

### Q9. What is the right-half-plane (RHP) zero in a CCM flyback and why does it limit bandwidth?

**Answer:**

**What causes the RHP zero:**

In a CCM flyback (or boost), increasing the duty cycle initially causes the output current to decrease before it increases. This counterintuitive behaviour arises because:
1. Increased D → longer on-time → primary current builds to higher peak
2. But immediately after the duty cycle step, the off-time shortens
3. Shorter off-time → less energy delivered to output per cycle (energy = area under secondary current pulse)
4. Short-term output voltage drops even though long-term it will rise

This phase reversal in the feedback path appears as a zero in the right half of the s-plane — a non-minimum phase characteristic.

**RHP zero location (CCM flyback):**
```
f_RHP = (1-D)² × n² × Vout / (2π × Lm × Iout)
      = (1-D)² × Rload_secondary / (2π × Lm/n²)
```

Note: Lm/n² is the magnetising inductance referred to the secondary side.

**Why it limits bandwidth:**

The RHP zero adds +20 dB/decade gain slope but -90° phase shift (opposite to a normal zero which adds +90°). The phase shift starts occurring at approximately f_RHP/10, so the loop must be closed with a bandwidth at least 3–5× below f_RHP:
```
f_crossover ≤ f_RHP / 5
```

For a flyback at D=0.45, n=0.5, Rload=10Ω, Lm=500µH:
```
f_RHP = (0.55)² × 10 / (2π × 500e-6/0.25)
      = 0.3025 × 10 / (2π × 2e-3)
      = 3.025 / 0.01257
      = 240 Hz
f_crossover_max = 240 / 5 = 48 Hz
```

A 48 Hz bandwidth gives very sluggish transient response. This is why CCM flybacks are either:
1. Operated in DCM (no RHP zero)
2. Use active clamp with current-mode control (partially mitigates RHP zero)
3. Limited to slower output voltage regulation applications

---

### Q10. Describe the optocoupler and TL431 feedback network used in isolated flyback converters.

**Answer:**

In an isolated converter, the output voltage is on the secondary side while the PWM controller is on the primary side. Feedback must cross the isolation barrier.

**Common solution: TL431 + optocoupler:**

**Secondary side (error amplifier — TL431):**
```
TL431 operates as a programmable precision reference/shunt regulator.
Vout is divided by R1/R2 and compared to the TL431's internal 2.5V reference.
If Vout > setpoint: TL431 conducts → optocoupler LED current increases
If Vout < setpoint: TL431 less conduction → LED current decreases
```

**Component configuration:**
```
Vout ──┬──[R1]──┬── to TL431 Ref pin (sets Vout = 2.5V × (1 + R1/R2))
       │        │
      [R2]    [R3]──── optocoupler LED anode
       │        │
      GND    TL431 K (cathode, connected to LED cathode)
              TL431 A (anode) to output GND
```

**Primary side (optocoupler output):**
The optocoupler transistor modulates the feedback voltage to the PWM controller's COMP or FB pin:
```
As LED current increases → optocoupler transistor harder on → FB pin voltage decreases → duty cycle decreases
```

**Bandwidth limitations of optocoupler:**
- Optocoupler pole at: `f_opto = 1 / (2π × CTR × Rcomp × Copto)`
- Typical optocoupler bandwidth: 5–30 kHz (PC817 class devices)
- High-speed optocouplers (HCPL-817A) extend to ~100 kHz
- CTR (current transfer ratio) varies 2:1 over temperature — compensator gain must be designed for worst case

**Type II compensation with TL431:**

The TL431 cathode resistor (R3) and capacitor to its ref pin form a pole-zero pair:
```
Zero at: fz = 1 / (2π × R1_parallel_R2 × Czero)
Pole at: fp = 1 / (2π × R3 × Cp)
```
This provides the Type II (one pole, one zero) compensation needed for most flyback designs.

---

### Q11. How is a two-switch forward converter different from a single-switch forward, and when would you use each?

**Answer:**

**Single-switch forward:**

Circuit: one primary MOSFET, one reset mechanism (winding, RCD, or active clamp), output LC filter.

Key constraints:
- D_max ≤ 0.5 (demagnetisation in remaining half cycle)
- Primary switch voltage = Vin + reset voltage = 2 × Vin (for equal-turn reset winding)
- Requires high-voltage MOSFET: e.g., for 400V rectified AC input, need 900V MOSFET

**Two-switch forward:**

Circuit: two MOSFETs (one high-side, one low-side), two body diodes serve as freewheeling/reset clamp diodes.

Operation:
- Both MOSFETs switch simultaneously (same gate drive signal)
- During off-time, body diodes (or external diodes) clamp drain voltage at Vin
- Magnetising current freewheels through these diodes back to the input (core resets)
- Each MOSFET only ever sees Vin (no reset overvoltage)

**Voltage stress comparison:**

| Parameter | Single-switch | Two-switch |
|-----------|--------------|------------|
| Each MOSFET Vds | 2 × Vin | 1 × Vin |
| Total switch "Volt × Amp" | 2 × Vin × I | 2 × Vin × I (same) |
| Transformer windings | 3 (+ reset winding) | 2 (primary + secondary) |
| Gate drive complexity | Simple | High-side driver needed |

**Selection guidelines:**

Use single-switch forward when:
- Input voltage < 200V DC (manageable with 400–600V MOSFET)
- Cost and simplicity are primary
- Power < 100W

Use two-switch forward when:
- Input voltage 300–400V DC (rectified 230V AC) — avoids needing 900V+ MOSFET
- Higher power (100–500W)
- Better transformer utilisation (no reset winding wasted copper)

---

### Q12. What is volt-second balance in a transformer and why does it matter for avoiding saturation?

**Answer:**

**Volt-second balance:** In steady state, the flux in a transformer core must return to its starting value each switching cycle. Since flux is proportional to the integral of voltage across the winding (V = L × dI/dt → flux ∝ V × t), the volt-seconds applied during the on-time must equal the volt-seconds removed during the off-time (reset).

**Mathematical statement:**
```
∫ V_primary dt = 0  (over one complete cycle)
Vin × D × Ts - V_reset × (1-D) × Ts = 0
```

**Why imbalance causes saturation:**

If the volt-second balance is violated (e.g., different positive and negative time integrals), the DC flux in the core accumulates cycle by cycle:
```
ΔΦ per cycle = (Vin × t_on - V_reset × t_off) / Np ≠ 0
```

The flux walks toward saturation. Once the core saturates, the transformer impedance collapses, current increases rapidly, and the primary switch will typically fail from overcurrent.

**Sources of volt-second imbalance:**
1. Asymmetric reset (different reset voltage or timing)
2. Transformer DC resistance creating voltage drops
3. PWM controller asymmetry (in push-pull topologies)
4. Different forward voltage drops across switching devices

**Prevention techniques:**
1. **Current-mode control:** The inner current loop naturally corrects for volt-second imbalance by adjusting duty cycle based on actual primary current.
2. **AC coupling:** Use a series capacitor to block DC component.
3. **Flux walking detection:** Monitor magnetising current peak each cycle.
4. **Active reset:** Precisely control both forward and reverse volt-seconds.

---

## Advanced (Questions 13–18)

---

### Q13. Derive the peak primary current and transformer turns ratio for a flyback operating in DCM with a given power level.

**Answer:**

**Given:** Vin=48V, Vout=12V, Pout=50W, fsw=100kHz, D=0.35, DCM operation.

**Step 1 — Determine turns ratio n = Np/Ns:**

In DCM, the voltage conversion ratio is:
```
Vout = Vin × D × n / (1 + n × D / (1-D_off))  [complex expression]
```

For initial design, use the approximation that the reflected secondary voltage equals the primary clamp voltage (for a clamp-based flyback). Target reflected voltage = Vin × D / (1-D):
```
n × Vout = Vin × D / (1-D) × adjustment_factor
```

Simpler approach: choose n based on the desired MOSFET voltage stress:
```
V_DS_max = Vin + n × Vout (ignoring leakage spike)
For V_DS_max = 100V: n × Vout = 100 - 48 = 52V
n = 52 / 12 = 4.33  → use n = 4 (standard turns ratio)
```

**Step 2 — Verify voltage conversion:**

In DCM, the output voltage is set by D and the circuit constants. For n=4:
```
Reflected voltage = n × Vout = 4 × 12 = 48V (= Vin for D = 0.5)
```

At D=0.35 and DCM (secondary current decays fully), the duty cycle of the secondary conduction (D2) is:
```
D2 = D × Vin / (n × Vout) = 0.35 × 48 / 48 = 0.35
```

Deadtime: D3 = 1 - D - D2 = 1 - 0.35 - 0.35 = 0.30 (30% dead time — DCM confirmed if D3 > 0)

**Step 3 — Calculate peak primary current:**

In DCM, all energy is transferred each cycle:
```
Energy per cycle = Pout / fsw = 50 / 100e3 = 500 µJ

Energy stored = 0.5 × Lm × Ip_peak²
Ip_peak = √(2 × Pout / (fsw × Lm))

Also: Ip_peak = Vin × D × Ts / Lm = Vin × D / (fsw × Lm)

From energy balance: Lm = (Vin × D)² / (2 × fsw × Pout)
                       = (48 × 0.35)² / (2 × 100e3 × 50)
                       = (16.8)² / 10e6
                       = 282.24 / 10e6
                       = 28.2 µH
```

Peak primary current:
```
Ip_peak = Vin × D / (fsw × Lm) = 48 × 0.35 / (100e3 × 28.2e-6)
        = 16.8 / 2.82
        = 5.96 A
```

**Step 4 — Verify output power:**
```
Pout = 0.5 × Lm × Ip_peak² × fsw = 0.5 × 28.2e-6 × 35.5 × 100e3
     = 0.5 × 28.2e-6 × 3.55e6
     = 50 W  ✓
```

---

### Q14. What is the difference between CCM and DCM flyback control loop design? What compensator types are needed for each?

**Answer:**

**CCM flyback loop characteristics:**

Power stage transfer function (control to output, Gvd):
```
Gvd(s) = G0 × (1 - s/ω_RHP) × (1 + s/ω_ESR) / [(1 + s/ω_p1) × (1 + s/ω_p2)]
```

Features:
- RHP zero at ω_RHP = (1-D)²×n²×Rload / Lm — limits bandwidth severely
- Output pole from LC filter: ω_LC = 1/√(Lm_sec × Cout) — double pole, 180° phase shift
- ESR zero from output capacitor: ω_ESR = 1/(ESR × Cout)

**Required compensator for CCM:** Type III or Type II with gain roll-off well before the RHP zero. The crossover frequency is typically limited to f_RHP/5 or f_RHP/10.

**DCM flyback loop characteristics:**

Power stage transfer function (DCM, simplified):
```
Gvd(s) = G0_DCM / (1 + s/ω_p_DCM)
```

Features:
- Single dominant output pole from RC load: ω_p_DCM = 2/(Rload × Cout) × (1 + M)
  where M = Vout × n / Vin
- **No RHP zero** (energy is fully transferred each cycle — no stored energy to "reverse")
- No double pole from LC (the inductor is the coupled inductor, effectively absent in DCM)
- ESR zero still present

**Required compensator for DCM:** Type II compensator typically sufficient. Much wider bandwidth possible (up to fsw/10 or higher).

**Comparison table:**

| Feature | CCM Flyback | DCM Flyback |
|---------|------------|------------|
| RHP zero | Yes — limits BW | No |
| Double pole | Yes (Lm + Cout) | No |
| Compensator | Type III typical | Type II sufficient |
| Achievable bandwidth | Low (f_RHP/5) | Higher (up to fsw/5) |
| Transient response | Slow | Faster |
| Peak primary current | Lower | Higher |

---

### Q15. How do you design a forward converter transformer to handle flux reset and avoid saturation?

**Answer:**

**Core reset voltage requirement:**

For a single-switch forward with N_reset = Np (equal turns):
```
V_reset = Vin (applied in reverse during off-time)
Reset time = Vin / V_reset × D × Ts = D × Ts
```

This mandates D + D_reset = 1, so D_max = 0.5.

**Transformer design procedure:**

**Step 1 — Select core geometry:**
- ETD or EE cores for wound transformers (easy to gap, good window area)
- Size based on Ap = Ae × Aw ≥ Pout / (k × Bmax × fsw × Jmax)

**Step 2 — Select Bmax:**
- For CCM forward: use Bmax ≤ 0.25 T (allows for positive and negative flux excursion)
- The core only uses one quadrant of the B-H curve (unipolar excitation)
- Reset brings B back to origin each cycle

**Step 3 — Calculate primary turns:**
```
Np = Vin × D_max × Ts / (Bmax × Ae)
   = Vin × D_max / (Bmax × Ae × fsw)
```

For Vin=48V, D_max=0.45, Bmax=0.2T, Ae=78mm²=78e-6m², fsw=200kHz:
```
Np = 48 × 0.45 / (0.2 × 78e-6 × 200e3)
   = 21.6 / 3.12
   = 6.92 → use 7 turns
```

**Step 4 — Secondary turns:**
```
Ns = Np × Vout / (Vin × D_max) = 7 × 5 / (48 × 0.45) = 35 / 21.6 = 1.62 → 2 turns
```

Adjust: actual Vout = Vin × D × Ns/Np = 48 × D × 2/7. For Vout=5V: D = 5×7/(48×2) = 0.365 (within range).

**Step 5 — Reset winding turns:**
Equal to Np = 7 turns (for Vin reset voltage, D_max = 0.5 constraint).

**Step 6 — Verify no saturation:**
Check that the maximum flux density (at maximum Vin, maximum D, including any asymmetry) does not exceed Bsat:
```
B_max = Vin_max × D_max / (Np × Ae × fsw)
      = 52 × 0.45 / (7 × 78e-6 × 200e3)
      = 23.4 / 109.2
      = 0.214 T  < 0.3 T (typical Bsat for MnZn ferrite at 100°C)  ✓
```

---

### Q16. Compare flyback and forward converters for a 24V input, 5V/10A output isolated power supply. Which would you choose and why?

**Answer:**

**Specification:**
- Vin = 24V, Vout = 5V, Pout = 50W
- Target efficiency > 88%
- Isolated output

**Flyback analysis:**

Turns ratio for reasonable MOSFET stress (V_DS_max = 80V target):
```
n = (V_DS_max - Vin) / Vout = (80 - 24) / 5 = 11.2 → n = 10 practical
```

Primary peak current (DCM, 50W, 200kHz):
```
Lm = (Vin × D)² / (2 × fsw × Pout) = (24 × 0.4)² / (2 × 200e3 × 50)
   = 92.16 / 20e6 = 4.6 µH
Ip_peak = Vin × D / (fsw × Lm) = 24 × 0.4 / (200e3 × 4.6e-6) = 9.6 / 0.92 = 10.4 A
```

Secondary peak current: n × Ip_peak = 10 × 10.4 = 104A! This is impractical for a 10A output. DCM at n=10 produces enormous secondary current.

Reduce n to 2 (reflected voltage = 10V, V_DS_max = 34V — usable):
```
Secondary peak = 2 × 10.4 = 20.8A (high but feasible)
```

Output ripple from capacitor only → large output cap needed at 10A output.

**Forward converter analysis:**

Turns ratio:
```
n = Vin × D_max / Vout = 24 × 0.45 / 5 = 2.16 → use n = 2, D_target = 5×2/24 = 0.417
```

Secondary current = Iout = 10A (continuous in output inductor — same as buck)
Secondary inductor carries 10A average, significantly lower peak current than flyback.
Output LC filter provides low ripple naturally.

**Comparison table for this design:**

| Parameter | Flyback | Forward |
|-----------|---------|---------|
| Peak primary current | 10.4 A | ~12.5 A (average primary I = Iout/n × D ≈ 2.08A, peak 2× higher in CCM) |
| Peak secondary current | 20.8 A | 10A × ripple factor |
| Output capacitor | Large (no filter inductor) | Moderate (LC filter) |
| Output ripple | Higher | Lower |
| Transformer complexity | Simpler (no reset winding needed if active clamp) | Reset winding or two-switch |
| Efficiency estimate | 85–87% | 88–91% |

**Recommendation: Forward converter** for this application.

Reasons:
1. 50W at 10A output is at the upper end of practical flyback power — secondary peak currents become excessive.
2. The forward converter's LC output filter provides inherently lower output ripple, important for a 5V logic supply.
3. At 50W, transformer design for forward converter is straightforward; the efficiency advantage justifies the added reset winding or two-switch configuration.
4. Flyback would be preferred if the design were <20W, needed multiple outputs, or had very tight space constraints.
