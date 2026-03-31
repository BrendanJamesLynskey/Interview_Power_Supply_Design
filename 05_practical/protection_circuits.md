# Protection Circuits — Interview Preparation

## Overview

Protection circuits prevent power supply failure from damaging downstream loads, causing fires, or creating safety hazards. Every commercial power supply includes at minimum overcurrent, overvoltage, and undervoltage protection. Understanding the design and trade-offs of each protection mechanism is expected in any hardware power electronics interview.

---

## Key Equations Reference

### Overcurrent Protection (OCP) Threshold
```
I_OCP = V_sense / R_sense   [sense resistor method]
I_OCP = I_limit × (R1 + R2) / R2   [resistor divider on CS pin]

Cycle-by-cycle: comparator trips every cycle where I_peak > I_OCP
Hiccup: fault → off → restart after delay
```

### Overvoltage Protection (OVP) Threshold
```
V_OVP = Vref × (1 + R_top/R_bot)   [voltage divider to comparator]
         or set by dedicated OVP pin with reference
```

### Undervoltage Lockout (UVLO)
```
V_start = Vref × (1 + R1/R2)   [turn-on threshold]
V_stop  = Vref × (1 + R1_parallel/R2)  [turn-off threshold — R1 in parallel after start]
Hysteresis = V_start - V_stop = Vref × R1 × R_hyst / (R2 × (R1 + R_hyst))
```

### Soft Start Ramp Time
```
t_softstart = C_ss × V_ss_peak / I_ss   [for constant-current soft-start]
or: t_softstart = C_ss × V_ss_peak / (I_ref / R_feedback)  [controller-specific]
```

### Inrush Current (NTC Thermistor)
```
I_inrush_limited = V_supply / (R_NTC_cold + R_load)
R_NTC_cold = room temperature resistance (before thermistor warms up)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. What are the three main overcurrent protection modes and what are their trade-offs?

**Answer:**

**Mode 1 — Cycle-by-Cycle (CBC) current limiting:**

The peak inductor current is measured each switching cycle. If it exceeds the threshold, the switch is immediately turned off for the remainder of that cycle, and turns on again at the start of the next cycle.

```
Operation: measure I_peak each cycle → if I > I_limit: turn off switch → wait for next cycle
```

**Characteristics:**
- Fast response: limits current within the same cycle (microseconds)
- Does not interrupt power delivery: the output voltage remains regulated (at reduced level) as long as the demand is below the current limit × duty cycle
- Dissipative: during a fault, the converter continues switching at reduced duty cycle, dissipating power in the switch and inductor

**Use case:** Suitable for mild overloads and short-circuit protection where the converter should maintain partial output (e.g., welding power supplies, motor drives where the load can be inductive).

**Mode 2 — Hiccup mode (intermittent):**

After detecting an overcurrent condition (sustained for N cycles), the converter shuts down completely. After a blanking delay (hiccup time), it attempts to restart with soft start. If the fault persists, it goes into hiccup again.

```
Operation: OCP triggered for > N cycles → output off for t_hiccup → soft-start attempt → repeat if fault
```

**Characteristics:**
- Low average dissipation during fault: the converter is off most of the time
- Output voltage is absent during hiccup → loads with capacitive holdup may tolerate this
- Takes longer to establish output voltage after fault clears (must wait for hiccup timeout)

**Use case:** Standard for most consumer and industrial power supplies. Prevents thermal stress on the converter during sustained faults.

**Mode 3 — Foldback current limiting:**

As the output voltage drops (due to increasing load or short circuit), the current limit threshold is also reduced. The converter follows a "foldback characteristic" where both V_out and I_limit decrease together.

```
I_limit(V_out) = I_max × V_out / V_out_nominal   [linear foldback]
or: I_limit = I_max - k × (V_out_nominal - V_out)  [linear reduction]
```

**Characteristics:**
- Power during short circuit = V_out × I_limit ≈ 0 × small_I → very low fault dissipation
- Risk of "foldback latch-up": if a load requires more start-up current than the reduced current limit allows, the converter can get stuck in the foldback region and never start the load (the load stays at partial power, which keeps V_out low, which keeps I_limit low)

**Use case:** High-voltage supplies where MOSFET voltage stress during fault (full Vin, full current) is a concern. Requires careful characterisation of load start-up behaviour.

---

### Q2. How is overvoltage protection (OVP) implemented and what triggers it?

**Answer:**

**OVP purpose:**

Protects downstream loads from overvoltage damage. Causes of overvoltage:
1. Feedback loop failure (broken resistor in feedback divider → output rises uncontrolled)
2. Control IC failure
3. Transient load removal (inductor energy dumps into capacitor briefly)
4. Optocoupler failure (isolated converter)

**Implementation methods:**

**1. Independent comparator OVP:**

A resistor divider on the output monitors V_out. When V_out exceeds V_OVP, a comparator triggers:
- Disables the PWM signal to the switch
- Latches off (requires power cycle to reset) or self-clears (auto-recovery)

```
V_OVP = Vref × (R_top + R_bot) / R_bot

For V_out_nom = 5V, V_OVP = 5.5V, Vref = 1.25V:
Set R_top/R_bot = (5.5/1.25 - 1) = 3.4 → R_top = 340kΩ, R_bot = 100kΩ
```

**2. Controller IC OVP pin:**

Many PWM controller ICs have a dedicated OVP pin. Connecting a resistor divider to this pin enables built-in OVP with hysteresis. When OVP trips, the IC typically shuts off PWM and may latch until VCC is cycled.

**3. Crowbar (SCR-based or MOSFET):**

A thyristor (SCR) is connected across the output. When V_out rises above V_OVP:
1. The SCR fires and short-circuits the output
2. Fuse or current limiter upstream opens (blows)
3. The output is shorted until the fuse is replaced or the SCR is reset

Advantage: Works even if the PWM controller fails. Disadvantage: Fuse must blow → hard shutdown. Used where catastrophic load protection is required (e.g., SCR crowbar on a 5V supply to protect expensive ICs).

**OVP threshold setting:**

```
V_OVP = V_out_nominal × (1 + margin)

Typical margin: 10–20% above nominal
For 3.3V output: V_OVP = 3.3V × 1.15 = 3.8V

Too tight: nuisance trips from transients
Too loose: load damaged before protection fires
```

**Common mistake:**

OVP set too close to V_out_nominal with hysteresis too small → OVP oscillates (trips, output drops, clears, output rises, trips again). This can damage loads that see repeated voltage cycling. Add sufficient hysteresis (50–100 mV) and ensure OVP holds off long enough for loads to power down gracefully.

---

### Q3. What is UVLO (undervoltage lockout) and how is it designed with hysteresis?

**Answer:**

**UVLO purpose:**

Ensures the converter only operates when the supply voltage is above the minimum required for proper operation. Below this threshold:
- Gate drivers may not provide adequate gate swing
- Control ICs may malfunction
- MOSFETs may operate in linear region with high power dissipation

UVLO prevents operation in an indeterminate state.

**Hysteresis principle:**

Without hysteresis: if Vin is near the UVLO threshold (from noise, ripple, or load variation), the converter oscillates on and off rapidly — damaging to components and producing EMI.

With hysteresis: once the converter starts (Vin > V_start), it continues to operate until Vin drops significantly below V_start (to V_stop). This provides a "don't care" zone of Vin between V_stop and V_start.

**Typical design values:**
```
V_start = minimum Vin for guaranteed controller operation + margin
V_stop  = minimum Vin for safe operation with some additional margin
Hysteresis = V_start - V_stop = typically 0.5–2V for a 12V supply, proportionally
```

**Circuit implementation (external resistor network):**

```
Vin ─── R1 ─┬─── R2 ─── GND
             │
             └─── UVLO_pin (of controller IC, Vref_UVLO = 1.25V)

V_start = Vref × (R1 + R2) / R2
V_stop  = same formula but with the hysteresis current added by the IC pin (controller pulls additional current into UVLO pin when running)
```

Many IC controllers have a built-in hysteresis current (I_hys) that flows from the UVLO pin when the IC is enabled:
```
V_start = Vref × (R1 + R2) / R2
V_stop  = Vref × (R1 + R2) / R2 - I_hys × R1

Hysteresis = I_hys × R1

Example: I_hys = 50 µA, R1 = 100kΩ → Hysteresis = 5V (too large — adjust R1 to 20kΩ → 1V hysteresis)
```

**UVLO for isolated converters:**

In isolated converters (flyback), the controller IC often has its own VCC supply (bias winding). UVLO monitors VCC, not Vin directly. The VCC rises during startup as the IC draws power from the startup resistor or bias winding. The UVLO ensures the IC waits until its own supply is fully established before beginning switching.

---

### Q4. What is soft start and how does it prevent inrush current on startup?

**Answer:**

**The inrush problem:**

At startup, the output capacitor is uncharged. The converter immediately tries to regulate V_out to nominal. Without soft start:
- The error is large (nominal - 0 = full error signal)
- The duty cycle jumps to maximum
- The inductor current rises rapidly (limited by L, but if L is small, very fast)
- The input capacitor and other upstream components see a large current pulse

For a 100W converter at 12V input, without soft start:
```
Initial surge current ≈ Vin / (L × ?) — limited by input impedance and inductance
Input capacitor and source see this inrush, which can trip overcurrent protection
Downstream loads may see voltage overshoot if the output capacitor charges faster than the feedback responds
```

**Soft start implementation:**

The reference or current limit is gradually increased from 0 to its operating value over a programmable time (typically 1–10 ms).

**Method 1 — Ramp the reference:**
A capacitor on the soft-start pin charges from a current source inside the IC:
```
V_ss(t) = I_ss × t / C_ss   [linear ramp]
t_ss = C_ss × V_ss_final / I_ss
```
The error amplifier uses the lower of V_ref and V_ss as its reference. Initially V_ss < V_ref, so the converter ramps toward V_ss (the ramp). Once V_ss ≥ V_ref, normal regulation takes over.

**Method 2 — Ramp the current limit:**
Start with a low current limit and gradually increase it. Prevents overcurrent trips during startup.

**Method 3 — Digital ramp (in digital controllers):**
The digital reference register is incremented each cycle until it reaches the target value. Very clean and programmable.

**Typical timing:**
```
C_ss = 100 nF, I_ss = 5 µA (from IC):
t_ss = 100×10⁻⁹ × 1.25 / 5×10⁻⁶ = 25 ms  [for Vref = 1.25V]
```

25 ms soft start allows the output to ramp up smoothly, preventing current spikes, overshoot, and false OCP trips.

**Multi-rail sequencing:**

In multi-output systems, soft start enables power sequencing:
- Rail 1 (3.3V) soft-starts first (0–5 ms)
- Rail 2 (1.8V) starts after Rail 1 is established (5–10 ms)
- Rail 3 (1.2V) starts last (10–15 ms)

This ensures the microprocessor core voltage comes up in the correct sequence as required by the device datasheet.

---

### Q5. How does inrush current limiting work for mains-connected power supplies?

**Answer:**

**The inrush problem at mains:**

When a mains power supply is first switched on, the large input electrolytic capacitor (typically 100–470 µF for a 100W supply) is uncharged. The mains supply instantly applies up to 325 V peak (230 Vac × √2). The capacitor charges through the series impedance:
```
I_peak ≈ V_mains / (R_series + Z_source)

With no current limiting: Z_source ≈ 0.1–0.5 Ω (transformer and cable)
I_peak = 325 / 0.2 = 1625 A — massive inrush current
```

This inrush:
- Trips circuit breakers
- Welds relay contacts
- Destroys input bridge rectifier diodes (exceeds I_FM rating)
- Creates EMI from the large current pulse

**Solution 1 — NTC thermistor:**

A negative temperature coefficient thermistor placed in series with the AC input:
- Cold (startup): resistance = R_cold (e.g., 15 Ω) → limits inrush to V_mains/R_cold = 325/15 = 22 A
- Hot (steady state): resistance drops to < 0.1 Ω → negligible power loss

```
I_inrush_limited = V_peak / (R_NTC_cold + R_line)
P_thermistor = I_steady² × R_NTC_hot   [small, typically < 0.5W]
```

**Problem:** If the power supply is turned off briefly and back on, the NTC is still warm → lower cold resistance → higher inrush than rated. Solution: allow 30–60 seconds for NTC to cool before re-energisation.

**Solution 2 — Relay bypass (NTC + relay):**

NTC limits inrush at startup. After 100–500 ms, a relay closes to short the NTC:
- Inrush limited by NTC during startup
- Steady-state efficiency maintained (relay has very low resistance)
- Hot re-start problem eliminated (NTC is bypassed, inrush on re-start depends on capacitor charge state, not NTC temperature)

```
Relay timing: delay = RC charging time of bootstrap capacitor = R_charge × C_delay
Typical: 200 ms delay before relay closes
```

**Solution 3 — Active inrush limiter:**

A MOSFET with controlled turn-on rate limits di/dt:
```
Control: V_gate ramps up slowly → FET in linear region → controlled inrush
Ramp time: 10–100 ms, set by gate RC
```
Used in high-reliability systems and for hot-swap applications. More complex but provides precise control and works for unlimited re-start cycles.

---

### Q6. What is reverse polarity protection and how is it implemented efficiently?

**Answer:**

**The problem:**

If DC power is connected with reversed polarity (accidental battery reversal, incorrect connector orientation), current flows backwards through the circuit. In the best case, protection diodes blow; in the worst case, electrolytic capacitors are destroyed (polarity reversal), MOSFETs fail (body diode conducts backwards with high current), or the load is destroyed.

**Solution 1 — Series diode:**

Cheapest and simplest. A diode in series with the positive supply blocks reverse current:
```
Normal: Vin(+) → D1 (forward biased) → Circuit → Vin(-) [OK]
Reversed: Vin(-) → D1 (reverse biased, blocks) → no current [protected]
```

**Problem:** Diode has forward voltage drop (0.3–0.7V for Schottky). At high current, this causes significant power loss:
```
P_diode = I_load × Vf   [at 10A, Vf=0.3V: P = 3W — significant]
```

**Solution 2 — P-channel MOSFET:**

A P-channel MOSFET with source at Vin, drain at the circuit, and gate connected to GND through a resistor (or directly to GND):
```
Normal polarity: Vin(+) > GND → gate lower than source → FET turns ON → low Rds_on → low loss
Reversed polarity: Vin(-) < GND → gate higher than source → FET stays OFF → no conduction
```

Power loss in normal operation:
```
P_FET = I_load² × Rds_on(T)
For Rds_on = 10 mΩ at 10A: P = 100 × 0.010 = 1W — much better than series diode
```

**Gate drive for P-channel:**

The gate can be driven to a voltage equal to Vin (gate at Vin → |Vgs| = 0 → FET off, wrong). The correct connection:
- Gate connected to GND through R_gate (e.g., 100 kΩ)
- Optional: small N-channel FET to actively clamp gate to GND when Vin is present
- Zener diode from gate to source to limit Vgs (protect gate oxide if Vin is high)

**Solution 3 — N-channel MOSFET (ideal diode):**

An N-channel FET with source connected toward the supply negative, drain toward the load, and a control circuit (ideal diode controller IC like LTC4359) that:
- Turns the FET ON when Vin > 0 (forward polarity)
- Turns the FET OFF when Vin < 0 (reversed polarity)
- Achieves very low voltage drop (Rds_on of N-channel is 4× lower than equivalent P-channel)

For large systems (>10A), N-channel with ideal diode controller is preferred for minimum power loss.

---

## Intermediate (Questions 7–12)

---

### Q7. How does current sensing work for overcurrent protection, and what are the accuracy trade-offs between different sensing methods?

**Answer:**

**Method 1 — Shunt resistor (R_sense):**

A precision low-value resistor in series with the current path. V = I × R_sense. A comparator or ADC measures V:

Advantages:
- Accurate (resistor tolerance ±1% or better)
- Works at DC and AC
- Simple, no saturation risk

Disadvantages:
- Power loss: P = I² × R_sense (for 10A, R_sense=10mΩ: P=1W)
- Voltage drop (1W at 1V drop = 1% of a 100V supply, but significant at low voltages)
- Ground-referenced (high-side sensing requires differential amplifier — more cost)

**Method 2 — DCR sensing (inductor winding resistance):**

Measure the voltage across an RC network (in parallel with the inductor) where the RC time constant equals L/DCR. The voltage across C equals I × DCR:

```
RC network: R_dcr || C_dcr in parallel with inductor L with series DCR

If R_dcr × C_dcr = L / DCR:
V_C = I_L × DCR   [lossless current sensing]
```

Advantages:
- Essentially lossless (only DCR dissipation, which exists regardless)
- No additional sensing resistor needed

Disadvantages:
- DCR varies with temperature (0.39%/°C) → accuracy degrades with temperature
- L and DCR must be matched to the RC network (component tolerances → ±20% accuracy without calibration)
- Only works for inductor current (not input current or switch current)

**Method 3 — Current transformer (CT):**

A high-frequency CT with primary (one turn, the power conductor) and secondary (many turns) provides isolation and current scaling. V_secondary = I_primary / N × R_burden:

```
I_measure = I_primary / N × (V_secondary / R_burden)
```

Advantages:
- Galvanic isolation
- Very accurate at high frequency
- Low power loss (transformer core loss + winding loss only)

Disadvantages:
- Saturates at DC → only for AC or high-frequency switching current
- Adds core, bobbin, winding → cost and volume
- Needs reset circuit to demagnetise core each cycle

**Accuracy summary:**

| Method     | Accuracy   | Power loss | Isolation | Cost   |
|------------|------------|------------|-----------|--------|
| R_sense    | ±1–2%      | High (I²R) | No        | Low    |
| DCR sense  | ±15–25%    | Negligible | No        | Medium |
| CT         | ±1–3%      | Very low   | Yes       | High   |

---

### Q8. What is a "latch" in OVP or OCP and when should it be used vs auto-restart?

**Answer:**

**Latching protection:**

When a fault condition (OCP, OVP, thermal shutdown) is detected, the converter shuts off and stays off until:
- Power is cycled (VCC drops below UVLO threshold, then reapplied)
- An external reset signal is applied
- A dedicated reset pin is pulsed

**Auto-restart (hiccup) protection:**

After a fault, the converter waits for a blanking time (t_hiccup), then attempts to restart. If the fault persists, it faults and waits again. Average current during hiccup:
```
I_avg_hiccup = I_peak × (t_ss / (t_ss + t_hiccup))
```
For t_ss = 2 ms and t_hiccup = 50 ms: duty cycle = 2/52 = 3.8% → average current = 3.8% of short-circuit current.

**When to use latching:**

1. **Safety-critical applications:** Medical equipment, lab power supplies — if a fault occurs, it must be investigated by a human before restarting. Auto-restart could restart a fault condition that is dangerous (e.g., a shorted motor that is drawing fault current and overheating).

2. **Catastrophic fault scenarios:** If OVP fires due to a hardware failure (broken feedback resistor), auto-restart just makes the converter keep damaging the load at high voltage. A latch prevents repeated overvoltage until fixed.

3. **Irreversible load damage:** If the load (e.g., a processor) can be damaged by one overvoltage event, a latch provides one-shot protection — prevents repeated exposure.

**When to use auto-restart:**

1. **Industrial equipment with transient faults:** A motor stall, cable short, or temporary overload that clears itself (e.g., motor starts under load, draws 5× rated current briefly, then settles). The converter trips and restarts — normal operation without user intervention.

2. **Consumer products:** Users do not understand "latch" — they turn the product on and off (power cycling) which resets a latch anyway. Auto-restart is simpler (one fewer failure mode).

3. **Battery-powered systems:** A battery can be nearly dead when connected → input current high, UVLO/OCP trips → hiccup while battery charges. Eventually clears. Latch would prevent the charger from ever starting.

**Hybrid approach:**

Some controllers count restarts. If N restarts fail (fault persists), the controller latches after N attempts. This gives the transient fault a chance to clear (auto-restart behaviour) while still latching for persistent faults.

---

### Q9. How is ESD protection implemented on power supply outputs and inputs?

**Answer:**

**ESD (Electrostatic Discharge) threat:**

ESD events occur when a person (charged to 1–30 kV from walking on carpet) touches the connector of a power supply. The discharge current pulse is very fast (subnanosecond rise time) and very energetic.

**Standards:**
- IEC 61000-4-2 (ESD immunity test): ±2 kV contact discharge, ±4 kV air discharge (Level 2 — typical commercial)
- Higher levels for industrial/automotive: ±6 kV, ±8 kV

**Output ESD protection:**

The power supply output must survive ESD without damage AND must not conduct the ESD pulse into the downstream circuitry (which may be more sensitive).

**TVS (Transient Voltage Suppressor) diode:**
- Bidirectional TVS or unidirectional TVS from output to GND
- Clamps voltage to V_clamp = V_br × 1.1 (1.1× reverse breakdown voltage)
- Very fast response (picoseconds to nanoseconds)
- Selection: V_clamp must be below the protected circuit's damage level, but above normal V_out (to avoid conduction during normal operation)

```
For 5V output:
Normal: Vout = 5.0V → TVS must not conduct at 5.0V → V_br > 5.5V
ESD: Vpulse = 1000V → TVS clamps to V_clamp = 6.0V → protects load
Choose: TVS with V_br = 5.6V, V_clamp = 7.0V at I_peak
```

**Selecting TVS power rating:**

IEC 61000-4-2 Level 2 ESD: peak current ≈ 30A for contact, pulse width ≈ 100 ns.
Peak power = 30A × 7V (clamp) = 210 W.
TVS must survive this pulse. Choose TVS with P_PP (peak pulse power) ≥ 210 W (at the specified pulse width).

**Input ESD and transient protection:**

For DC inputs, additional protection is needed for:
- ESD from connector contact
- Load dump transients (in automotive: up to 40V during alternator load dump)
- Lightning coupling transients

Add a bulk TVS or MOV (metal oxide varistor) rated for the expected transient voltage.

---

### Q10. How does thermal shutdown work and what triggers it?

**Answer:**

**Thermal shutdown function:**

When the junction temperature of the control IC or power device exceeds a safe limit, the thermal shutdown circuit disables the PWM output. The purpose:
- Prevent catastrophic thermal failure of the IC
- Provide a second line of defence after OCP (if OCP misses a fault that causes heating)
- Alert the system to a thermal problem (persistent shutdown = hardware issue)

**Implementation in a controller IC:**

Inside the IC, a temperature-sensing circuit (CTAT voltage reference, silicon bandgap, or threshold detector) monitors die temperature:

```
At T < T_shutdown (e.g., 150°C): normal operation
At T = T_shutdown: thermal flag set → PWM disabled → output off
At T = T_shutdown - T_hysteresis (e.g., 130°C): thermal flag cleared → restart
```

**Hysteresis prevents rapid on-off cycling:**

Without hysteresis: the IC temperature might oscillate around T_shutdown if the heat source is not removed. Hysteresis ensures the IC cools 20°C before restarting, giving time for heat to dissipate.

**Thermal shutdown in power FETs:**

Discrete power FETs do not have built-in thermal shutdown (the FET is silicon, no logic). The external controller must monitor FET temperature (via NTC thermistor on the heatsink or by measuring Rds_on from Vds/Id). If a thermal protection FET is used (e.g., Infineon OptiMOS with integrated current sensor), the controller can infer temperature from Rds_on.

**Thermal design implication:**

If the IC thermal shutdown trips frequently in normal operation, the thermal design is inadequate (too much heat, not enough heatsinking). If it trips only under fault conditions (short circuit, fan failure), the design is correct — thermal shutdown is acting as an emergency backup.

**Correct use of thermal shutdown:**

Thermal shutdown should NOT be the primary protection mechanism — OCP, OVP, and UVLO should prevent most fault conditions. Thermal shutdown is the last line of defence. A product that routinely relies on thermal shutdown to limit junction temperature is under-designed thermally.

---

### Q11. What is the difference between short-circuit protection and overcurrent protection?

**Answer:**

**Overcurrent protection (OCP):**

Activates when the current exceeds a specified limit above the nominal operating range, typically at 110–150% of rated current. The output voltage may still be regulated.

**Short-circuit protection (SCP):**

Activates when the output is directly shorted to ground (V_out = 0). In this case:
- The control loop demands maximum duty cycle (trying to bring voltage up)
- The current is limited only by the inductor, FET, and sense resistor
- Without specific SCP, this can lead to very high currents and component damage

**Why SCP is different from OCP:**

During a short circuit, the output voltage drops to 0. If the converter is in voltage mode:
- Error = V_ref - V_out = V_ref (large)
- Duty cycle ramps to maximum
- Inductor current rises: I = Vin × D_max × Ts / L → can be 3–10× rated current

A simple OCP threshold set at 150% of rated current may trip correctly. But the fault current can rise faster than the OCP can respond (limited by cycle-by-cycle detection latency). During the one or two cycles before OCP acts, the current can be very high.

**Short-circuit protection techniques:**

1. **Current-sense filter:** Ensure the current sense signal reaches the comparator quickly (keep sense filter bandwidth above 10× fsw).

2. **Latch-off on SCP:** After a short circuit is detected, latch off immediately. No soft start retry until power is cycled.

3. **Foldback current limiting:** Described in Q1. Reduces current limit as V_out drops.

4. **Hiccup mode:** Standard hiccup provides very low average short-circuit power — safe for most designs.

**Short-circuit current path through input capacitor:**

During the brief interval before OCP acts, the input capacitor provides the short-circuit current. The input capacitor must handle the associated current surge without voltage collapse. The ESL of the input capacitor path sets the impedance for this transient.

---

### Q12. How do you implement a crowbar protection circuit?

**Answer:**

**Crowbar principle:**

A crowbar (named for placing an iron crowbar across a circuit to short it) is a circuit that deliberately short-circuits the output when the voltage rises above a set threshold. This rapidly brings the output to near 0V, protecting the load.

**SCR-based crowbar:**

```
Vout (+) ──── R_sense ──── SCR (anode to Vout+) ──── Fuse ──── Vout (+)
                             │
                         Gate drive circuit (comparator → gate)
                             │
                          Zener (sets V_trip)
                             │
Vout (-) ─────────────── GND
```

**Operation:**

1. V_out rises above V_trip (set by Zener in gate circuit)
2. Comparator triggers → SCR gate receives current
3. SCR fires (latches ON) → short circuit across output
4. Current limited only by series impedance (fuse)
5. Fuse blows → circuit opens
6. SCR is now in "off" state (current removed when fuse opens)
7. System requires fuse replacement + fault investigation

**SCR crowbar design:**

- SCR selection: must handle I_SC for the time it takes the fuse to blow (I²t rating of SCR > I²t of fuse)
- SCR I²t > fuse I²t (the SCR must survive longer than the fuse)
- Typical crowbar SCR: 400V, 10–50A (depending on supply rating)
- Gate trigger current: 10–100 mA (provide from comparator output with driver transistor)

**Crowbar trigger voltage:**

```
V_trip = V_out_nom × (1 + OVP_margin)

For 5V output, 10% margin: V_trip = 5.5V
Set Zener to 5.1V (in series with R to adjust fine threshold)
```

**Modern alternative — fast OVP + MOSFET:**

Instead of an SCR crowbar, use a P-channel MOSFET that shunts the output to ground when OVP fires. The MOSFET is then turned off after a delay (no blown fuse). More controllable, but requires the converter to also shut off (to prevent the shunt FET from conducting continuously).

---

## Advanced (Questions 13–16)

---

### Q13. Design an OCP circuit for a 10A, 12V buck converter using cycle-by-cycle limiting.

**Answer:**

**Specifications:**
```
I_OCP target = 12A (120% of rated 10A)
Sensing method: shunt resistor
Converter: peak current mode control (CS signal already available)
```

**Step 1: Select sense resistor**
```
R_sense = V_cs / I_OCP = ?

Controller IC (e.g., UCC28C4x): CS pin comparator trips at 1.0V (typical)
R_sense = 1.0V / 12A = 83.3 mΩ → choose R_sense = 82 mΩ (standard value)

Verify power dissipation:
P_sense = I_rms² × R_sense ≈ 10² × 0.082 = 8.2W — too high!

Better: use lower resistance and add gain:
R_sense = 10 mΩ → V_sense = 10A × 0.010 = 100mV
Amplify × 10 to get 1V at CS pin → use op-amp or instrumentation amplifier

Or: use a controller with 100mV CS threshold (many modern synchronous buck controllers)
→ R_sense = 100mV / 12A = 8.3 mΩ → choose 8 mΩ
P_sense = 10² × 0.008 = 0.8W — acceptable for a power resistor
```

**Step 2: Filter to prevent leading-edge spike triggering**

When the MOSFET turns on, an initial current spike (from output capacitor discharge through inductor) can exceed I_OCP threshold falsely. Add RC filter:
```
R_filter = 100–500Ω in series with CS trace
C_filter = 100–470pF from CS to GND

Time constant τ = R × C = 300Ω × 200pF = 60ns

This filters the leading edge spike without significantly delaying the actual overcurrent detection (OCP events rise over 1–10µs, much slower than the 60ns filter).
```

**Step 3: Propagation delay consideration**

The comparator detects the OCP, but the PWM switch does not turn off until the next clock cycle (in voltage-mode) or immediately in current-mode. For CBC protection:
```
Turn-off time = propagation delay of comparator + gate drive pull-down time
             ≈ 50ns + 20ns = 70ns

During 70ns with Vin=12V, L=10µH:
ΔI = Vin × t / L = 12 × 70ns / 10µH = 84mA extra current above OCP threshold
→ Peak current = 12A + 0.084A ≈ 12.1A (negligible error, acceptable)
```

**Step 4: Implement with resistor divider**

For a controller with fixed 1V threshold and 8 mΩ sense resistor:
```
R_sense = 8 mΩ → V_sense at 12A = 96mV

Need to scale: add a 10× gain stage (op-amp or simple resistor divider if CS pin impedance is high)
→ V_CS = 960mV ≈ 1V (with adjustment for exact threshold matching)
```

---

## Quick Reference: Protection Checklist

```
PROTECTION DESIGN CHECKLIST:

OCP (Overcurrent):
[ ] Calculate OCP threshold: I_OCP = 1.1 to 1.5 × I_rated
[ ] Select sensing method: R_sense (accurate), DCR (lossless), CT (isolated)
[ ] Add leading-edge blanking filter (RC, τ = 50-100ns)
[ ] Choose protection mode: CBC, hiccup, or foldback
[ ] Verify average fault power is within component ratings

OVP (Overvoltage):
[ ] Set V_OVP = 1.1 to 1.2 × V_rated
[ ] Choose: auto-recover or latch (safety-critical = latch)
[ ] Response time < 1 µs for fast OVP
[ ] Verify load cannot be damaged during OVP trip time

UVLO:
[ ] Set V_start = minimum Vin for guaranteed operation + 5% margin
[ ] Add hysteresis: V_start - V_stop ≥ 0.5V (prevent chatter)
[ ] Verify converter operation at all corners: low Vin, high load

SOFT START:
[ ] t_ss ≥ 1 ms (10 ms typical for most applications)
[ ] Match t_ss to load capacitance (avoid current spike)
[ ] For multi-rail: implement sequencing with staggered soft start

INRUSH LIMITING:
[ ] NTC thermistor: size for V_peak / R_NTC_cold ≤ 10 × I_rated
[ ] Allow 30 s between hot plug cycles (NTC thermal time constant)
[ ] For < 5Ω NTC after warm-up: add relay bypass

THERMAL:
[ ] Thermal shutdown T_trip ≤ 0.8 × T_j_max_abs
[ ] Hysteresis: 15-25°C (prevents rapid toggling)
[ ] Verify thermal protection never trips in normal operation
```
