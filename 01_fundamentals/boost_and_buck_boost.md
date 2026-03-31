# Boost and Buck-Boost Converters — Interview Preparation

## Overview

Boost converters step voltage up; buck-boost converters invert or step voltage both up and down. Both topologies introduce the right-half-plane (RHP) zero, which fundamentally limits control bandwidth. Understanding the CCM/DCM boundary, gain characteristics, and the RHP zero is a frequent interview topic.

---

## Key Equations Reference

### Boost Converter — CCM Duty Cycle
```
Vout = Vin / (1 - D)
D = 1 - Vin/Vout
```

### Boost — Inductor Current Ripple
```
ΔIL = Vin × D / (fsw × L) = Vin × D × Ts / L
```

### Boost — Average Input Current (= average inductor current)
```
IL_avg = Iout / (1 - D)  [from power balance: Vin × IL_avg = Vout × Iout]
```

### Boost — RHP Zero Frequency
```
fRHP = Rload × (1-D)² / (2π × L)
```

### Inverting Buck-Boost — CCM Duty Cycle
```
|Vout| = Vin × D / (1-D)
D = |Vout| / (Vin + |Vout|)
```

### Four-Switch Buck-Boost
Operates as buck (D1 modulated, Q3 on) or boost (Q1 on, D2 modulated) or transition between.

### SEPIC — CCM Voltage Gain
```
Vout = Vin × D / (1-D)   [non-inverting, same magnitude as buck-boost]
```

### Cuk — CCM Voltage Gain
```
|Vout| = Vin × D / (1-D)  [inverting, continuous input and output current]
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Describe the operation of a boost converter through one switching cycle in CCM.

**Answer:**

A boost converter consists of an inductor L (connected from Vin), a switch Q1 (NMOS, connected from the L-D junction to GND), a diode D1 (from the L-D junction to Vout), and output capacitor Cout.

**Phase 1 — Switch ON (duration: D × Ts):**
- Q1 closes. The inductor is connected between Vin and GND.
- Voltage across inductor: `VL = Vin` (positive).
- Inductor current ramps up: `dIL/dt = Vin/L`.
- Diode D1 is reverse biased by Vout — energy is stored in the inductor.
- Output capacitor Cout supplies the load current alone during this phase.

**Phase 2 — Switch OFF (duration: (1-D) × Ts):**
- Q1 opens. Inductor current must continue flowing (Lenz's law).
- The inductor voltage reverses and drives the switch node above Vout until D1 conducts.
- When D1 conducts: switch node voltage ≈ Vout - Vf.
- Voltage across inductor: `VL = Vin - Vout` (negative, since Vout > Vin).
- Inductor current ramps down.
- Both inductor and Vin deliver energy to the load and Cout.

**Volt-second balance (steady state):**
```
Vin × D × Ts + (Vin - Vout) × (1-D) × Ts = 0
Vin × D + Vin × (1-D) - Vout × (1-D) = 0
Vin = Vout × (1-D)
Vout = Vin / (1-D)    [D = 1 - Vin/Vout]
```

**Key observation**: Unlike the buck, where the input current is pulsating and the output current is smooth, the boost has a **smooth input current** (inductor is on the input) and a **pulsating output current** (diode is in series with the output). The output capacitor must filter this pulsating current.

---

### Q2. Derive the duty cycle equation for a boost converter in CCM and compare it to the buck.

**Answer:**

**Volt-second balance on the inductor:**

During switch ON:
```
VL_on = Vin
```
During switch OFF:
```
VL_off = Vin - Vout   (negative, since Vout > Vin)
```

Setting the average VL to zero:
```
Vin × D + (Vin - Vout) × (1-D) = 0
Vin × D + Vin - Vin×D - Vout + Vout×D = 0
Vin - Vout + Vout×D = 0
Vout = Vin / (1-D)
```

Solving for D:
```
D = 1 - Vin/Vout
```

**Comparison with buck:**

| Parameter        | Buck                    | Boost                        |
|------------------|-------------------------|------------------------------|
| Vout vs Vin      | Vout < Vin always       | Vout > Vin always            |
| Duty cycle       | D = Vout/Vin            | D = 1 - Vin/Vout             |
| D range          | 0 to 1 (0% = 0 V out)  | 0 to 1 (1 = infinite Vout)  |
| Input current    | Pulsating               | Continuous (smooth triangle) |
| Output current   | Continuous (smooth)     | Pulsating                    |
| RHP zero         | No                      | Yes (in CCM)                 |

**Numerical example**: Boost from 5 V to 12 V:
```
D = 1 - 5/12 = 1 - 0.417 = 0.583 (58.3%)
```
The switch is on for 58.3% of each cycle.

**Ideal gain at D = 1**: Vout/Vin → infinity. In practice, losses (switch resistance, inductor DCR) limit the maximum achievable voltage gain to roughly 5–10×, depending on efficiency requirements.

---

### Q3. Explain the right-half-plane (RHP) zero in a boost converter. Why does it limit control bandwidth?

**Answer:**

**What is an RHP zero?**

A zero in the right half of the s-plane (s = +jω plane). In a Bode plot:
- **Magnitude**: increases at +20 dB/decade above the zero frequency (same as left-half-plane zero)
- **Phase**: decreases by 90° above the zero frequency (opposite to LHP zero — this is the pathological behaviour)

A LHP zero adds phase (phase lead — good for stability). An RHP zero subtracts phase (phase lag — damaging for stability).

**Physical origin in boost converter:**

When duty cycle D increases suddenly (controller commands more power to the output):
1. Switch ON time increases.
2. Diode is blocked for longer.
3. Instantaneous output current (through the diode) **decreases** momentarily.
4. Output voltage initially dips — exactly the wrong direction.
5. Only after the switch turns off and the inductor (now storing more energy) delivers it through the diode does Vout increase.

The initial wrong-direction response is the RHP zero. The control-to-output transfer function has a zero at:
```
fRHP = Rload × (1-D)² / (2π × L)
```

**Impact on control loop design:**

The phase contributed by the RHP zero is:
```
φ_RHP = -arctan(f / fRHP)  [phase lag]
```

At f = fRHP: -45° of extra phase lag.
At f = 10 × fRHP: -84° of extra phase lag.

**Design rule**: The loop crossover frequency must be kept well below fRHP, typically:
```
fcross < fRHP / 3  to  fRHP / 5
```

This limits how fast the control loop can respond, which limits transient response bandwidth.

**Numerical example**: Boost: Vin = 5 V, Vout = 12 V, Iout = 1 A, L = 100 µH:
```
D = 1 - 5/12 = 0.583
Rload = 12/1 = 12 Ω
fRHP = 12 × (1-0.583)² / (2π × 100×10⁻⁶)
     = 12 × 0.174 / (628×10⁻⁶)
     = 2.09 / 0.000628
     = 3.33 kHz
```

Maximum loop crossover: ≈ 1 kHz — a significant constraint. Increasing L moves fRHP lower (worse). Decreasing L moves fRHP higher (better but increases ripple).

---

### Q4. Describe the CCM/DCM boundary in a boost converter and the behaviour in each mode.

**Answer:**

**CCM/DCM boundary:**

The inductor current touches zero at the boundary condition. The boundary inductance is:
```
Lcrit = Vout × D × (1-D)² / (2 × fsw × Iout)
```

For `L > Lcrit`: CCM. For `L < Lcrit`: DCM.

**CCM characteristics:**
- Duty cycle: `D = 1 - Vin/Vout` (independent of load)
- Voltage gain is determined only by D — predictable, load-independent
- RHP zero is present — limits control bandwidth
- Inductor current never reaches zero — continuous

**DCM characteristics:**
- Duty cycle becomes load-dependent: lower load → smaller D for same Vout/Vin
- The "missing" energy is compensated by higher peak current during the on-time
- Three phases per cycle: switch on, diode on, idle (both off, inductor current = 0)
- Switch node rings during idle phase (LC resonance with switch node capacitance)
- RHP zero is absent in DCM — the control-to-output transfer function becomes first-order
- Voltage gain in DCM: `Vout/Vin = (1 + √(1 + 4D²/(K))) / 2` where K = 2L×fsw/Rload

**DCM advantages:**
- RHP zero eliminated → can achieve higher control bandwidth
- Diode (or body diode) current returns to zero each cycle → eliminates reverse recovery issues
- Natural ZCS for the diode

**DCM disadvantages:**
- Higher peak currents (for the same average current, peak is higher in DCM)
- Higher output voltage ripple (less energy delivery per cycle is harder to filter)
- Voltage gain becomes load-dependent → control loop must compensate

**Transition load current** (CCM→DCM boundary):
```
Iout_crit = Vout × D × (1-D)² / (2 × fsw × L)
```
Below this current, the converter operates in DCM.

---

### Q5. Describe the inverting buck-boost converter topology. Compare it to the boost.

**Answer:**

**Inverting buck-boost (single-switch):**

The switch Q1 is connected from Vin to the switch node. The inductor connects from the switch node to GND. The diode D1 connects from the switch node to the negative terminal of the output capacitor (which then connects to GND). The output voltage is negative with respect to GND.

**Phase 1 — Switch ON:**
- Q1 closes: Vin appears across the inductor. VL = Vin.
- Inductor current ramps up. Diode blocked.
- Output capacitor supplies the load.

**Phase 2 — Switch OFF:**
- Inductor current forces the switch node below GND.
- Diode conducts: the switch node sits at -Vout - Vf.
- Inductor delivers energy to the load (which is at negative potential).

**Volt-second balance:**
```
Vin × D + (-|Vout|) × (1-D) = 0
|Vout| = Vin × D / (1-D)
D = |Vout| / (Vin + |Vout|)
```

**Comparison with boost:**

| Property            | Boost                       | Inverting Buck-Boost        |
|---------------------|-----------------------------|-----------------------------|
| Output polarity     | Same as input (+)           | Inverted (-)                |
| Voltage range       | Vout > Vin always           | Any |Vout| vs Vin           |
| Input current       | Continuous (smooth)         | Pulsating (switch in series)|
| Output current      | Pulsating                   | Pulsating                   |
| RHP zero            | Yes (CCM)                   | Yes (CCM)                   |
| Switch count        | 1                           | 1                           |
| Switch voltage stress| Vout                       | Vin + |Vout|                |

The inverting buck-boost can produce an output voltage smaller or larger in magnitude than Vin, making it more versatile but always inverting.

---

### Q6. What is the SEPIC converter? When would you use it instead of a boost or buck?

**Answer:**

**SEPIC (Single-Ended Primary Inductance Converter):**

The SEPIC uses two inductors (L1, L2), a coupling capacitor Cc, a switch Q1, a diode D1, and output capacitor Cout.

```
[Vin] → [L1] → [Q1 to GND]
                  |
              [Cc] — [L2] → [D1] → [Vout]
                              |
                             [GND]
```

The coupling capacitor Cc allows the DC operating points of the two inductors to be set independently.

**Voltage gain (CCM):**
```
Vout = Vin × D / (1-D)    [same magnitude relationship as buck-boost]
D = Vout / (Vin + Vout)
```

**Key properties:**
- Output polarity: **same as input** (non-inverting) — key advantage over inverting buck-boost
- Can step up AND step down Vout relative to Vin, with no polarity inversion
- Input current is continuous (L1 is in series with Vin) — lower EMI than buck-boost
- Output current is pulsating (like boost and buck-boost) — requires output capacitor
- RHP zero is present in CCM
- Two inductors can be wound on the same core (coupled SEPIC inductor) to save space

**When to use SEPIC:**
- Need non-inverting output from a supply that ranges above and below Vout
- Example: 2.7–4.2 V Li-ion battery → 3.3 V output (sometimes above, sometimes below Vin)
- Buck alone fails when Vin < Vout; boost alone fails when Vin > Vout; SEPIC handles both seamlessly
- When negative output voltage is not acceptable (e.g., SEPIC vs buck-boost for load with single-ended supply rail)

**Disadvantage**: More complex than buck or boost; two inductors; coupling capacitor must handle full load current at the switching frequency.

---

## Intermediate (Questions 7–13)

---

### Q7. Explain the four-switch buck-boost topology and its advantages over the inverting buck-boost.

**Answer:**

**Four-switch (4-switch) buck-boost:**

Uses four MOSFETs: Q1 and Q3 form a buck stage; Q2 and Q4 form a boost stage. A common inductor L connects the midpoints.

```
[Vin] → [Q1]---[SW1]---[L]---[SW2]---[Q3] → [Vout]
              [Q2 to GND]        [Q4 to GND]
```

**Operating modes:**

1. **Buck mode (Vin > Vout + margin)**: Q3 is always on (synchronous side of boost). Q4 is always off. Q1 and Q2 switch normally as a buck stage. Boost stage is bypassed.

2. **Boost mode (Vin < Vout - margin)**: Q1 is always on (synchronous side of buck). Q2 is always off. Q3 and Q4 switch normally as a boost stage. Buck stage is bypassed.

3. **Transition (Vin ≈ Vout)**: Both stages switch simultaneously or in interleaved fashion. Multiple modulation strategies exist (e.g., single-inductor 4-switch PWM, inverted operation).

**Advantages over single-switch inverting buck-boost:**

| Property                | Single-switch Buck-Boost | 4-Switch Buck-Boost       |
|-------------------------|--------------------------|---------------------------|
| Output polarity         | Inverted                 | Non-inverting             |
| Efficiency              | Lower (all power cycles through the inductor in both phases) | Higher in buck or boost mode |
| Switch voltage stress   | Vin + |Vout|             | Max(Vin, Vout) only       |
| Inductor current        | High (= Iin + Iout)      | Lower in pure buck/boost mode |
| Complexity              | Simple                   | More complex; needs transition control |

**Common application**: Laptop battery chargers, where Vin (adapter) can be above or below the battery voltage; USB-C power delivery systems where Vin varies 5–20 V and Vout must be regulated.

---

### Q8. Compare the Cuk converter to the SEPIC. What are the advantages of the Cuk?

**Answer:**

**Cuk converter topology:**

```
[Vin] → [L1] → [D1] → [Cout'] → [L2] → [Vout(-)]
                    |                  |
                  [Q1]               [GND]
                    |
                  [GND]
```

More precisely: Q1 connects the coupling capacitor junction to GND; L1 is from Vin to the Q1/D1 junction; D1 is from that junction to the Vout end; L2 is from the D1 cathode to the negative output.

**Voltage gain (CCM):**
```
|Vout| = Vin × D / (1-D)   [same as SEPIC in magnitude, but inverted]
```

**Cuk advantages over SEPIC:**

1. **Continuous input AND output current**: Both L1 and L2 are in series with input and output respectively. Current in L1 (input) is a smooth triangle wave; current in L2 (output) is also a smooth triangle wave. This is unique — both buck and boost have one pulsating current side, SEPIC has a pulsating output. Cuk has no pulsating node at input or output, making EMI filtering easier and simpler.

2. **Coupled inductor possible**: L1 and L2 can be wound on the same core with the right coupling to achieve zero ripple at input or output (ripple steering effect). This theoretically allows infinite effective inductance with a finite-size coupled component.

3. **Same switch voltage stress as SEPIC**: Vq_max = Vin + |Vout|.

**Cuk disadvantages:**
- Output polarity is inverted (like single-switch buck-boost, unlike SEPIC)
- Negative output complicates single-supply system integration
- Coupling capacitor stress: carries the full volt-second product

**When to choose Cuk:**
- Negative supply rail required with very low EMI input and output ripple
- When coupled inductors can be used to minimise total magnetic size

---

### Q9. Calculate the RHP zero frequency for a CCM boost converter with the following parameters: Vin = 3.6 V, Vout = 5 V, Iout = 500 mA, L = 4.7 µH, fsw = 1 MHz.

**Answer:**

**Step 1: Duty cycle**
```
D = 1 - Vin/Vout = 1 - 3.6/5 = 1 - 0.72 = 0.28
```

**Step 2: Load resistance**
```
Rload = Vout / Iout = 5 / 0.5 = 10 Ω
```

**Step 3: RHP zero frequency**
```
fRHP = Rload × (1-D)² / (2π × L)
     = 10 × (1-0.28)² / (2π × 4.7×10⁻⁶)
     = 10 × (0.72)² / (29.53×10⁻⁶)
     = 10 × 0.5184 / (29.53×10⁻⁶)
     = 5.184 / 0.00002953
     = 175.5 kHz
```

**Step 4: Maximum loop crossover frequency**
```
fcross_max ≈ fRHP / 5 = 175.5 / 5 = 35.1 kHz
```
Or conservatively: `fRHP / 3 = 58.5 kHz`.

**Step 5: Verify against switching frequency**
```
fsw = 1 MHz
fcross_max / fsw = 35 kHz / 1 MHz = 3.5%  — extremely low
```

This is a 1 MHz boost converter limited to < 35 kHz crossover — 3.5% of fsw. This is the challenge of CCM boost control at low duty cycle with small inductors.

**Effect of increasing L to 47 µH:**
```
fRHP = 10 × 0.5184 / (2π × 47×10⁻⁶) = 5.184 / 295.3×10⁻⁶ = 17.5 kHz
```
Larger inductor makes the RHP zero worse (lower frequency)! This counter-intuitive result means a larger inductor does not help stability — it hurts it.

**Correct approach to improve RHP zero**: Operate in DCM (RHP zero disappears in DCM), or reduce Vout/Vin ratio (reduces D, improves (1-D)²).

---

### Q10. How does the input and output current behaviour of a boost converter differ from a buck? Why does this matter for capacitor selection?

**Answer:**

**Buck converter:**
- Input current: pulsating (high amplitude switch current during Q1 on-time, zero during off-time)
- Output current: continuous (smooth triangular inductor current)

**Boost converter:**
- Input current: continuous (smooth triangular inductor current)
- Output current: pulsating (diode current during Q1 off-time, zero during on-time)

**Quantitative comparison:**

For a boost with Vin = 5 V, Vout = 12 V, Iout = 1 A:
```
IL_avg = Iout / (1-D) = 1 / (1-0.583) = 2.4 A
D = 0.583
IL_peak = IL_avg + ΔIL/2
```

Input capacitor RMS ripple current (boost):
```
Iin_ripple_rms ≈ ΔIL / (2√3) = (ΔIL is small for boost with good L sizing)
```
The input capacitor of a boost sees only the small triangular ripple — much lower stress than a buck's input cap.

Output capacitor RMS ripple current (boost):
```
Iout_cap_rms = Iout × √(D/(1-D))
```
At D = 0.583: `Iout_cap_rms = 1 × √(0.583/0.417) = 1 × 1.18 = 1.18 A_rms`

The output capacitor of a boost handles a large pulsating current — must be rated for high ripple current.

**Capacitor selection implications:**

For **boost input capacitor**: Low capacitance, low ESL are sufficient. Focus on handling the switching frequency noise and small triangular ripple.

For **boost output capacitor**: Must be rated for high RMS ripple current. Low ESR critical to minimise ESR-induced ripple from the large pulsating current. Electrolytic or polymer capacitors are common for boost output stages due to their ripple current rating.

**Common design error**: Undersizing the boost output capacitor's ripple current rating. The pulsating output current causes significant heating in a capacitor with high ESR, reducing lifetime.

---

### Q11. Explain burst mode and PFM operation in a boost converter at light load.

**Answer:**

At light load in CCM, the converter continues switching at fsw but delivers less energy per cycle. The dominant losses (gate drive, switching, quiescent current) remain roughly constant, while Pout decreases — efficiency drops sharply.

**PFM (Pulse Frequency Modulation):**

In PFM mode, the converter operates with a fixed peak inductor current (not a fixed duty cycle or frequency). Each time Vout falls below the reference, a pulse of fixed energy is delivered. The frequency of these pulses varies with load:
```
fpfm ≈ Pout / (0.5 × L × Ipeak²)   [energy per pulse = 0.5 × L × Ipeak²]
```

At full load: fpfm = fsw (transitions to PWM mode)
At light load: fpfm is much lower than fsw → much less gate drive and switching loss per unit time

**Burst mode:**

Burst mode is a more aggressive form of PFM where the converter delivers a burst of switching pulses (several cycles at fsw), then idles completely for a much longer period. The duty cycle of bursts is proportional to load.

**Advantages:**
- Very high efficiency at light load (quiescent current dominates briefly, then off)
- Simple implementation

**Disadvantages:**
- Output voltage ripple increases (more charge removed between bursts)
- Burst frequency can fall into audio range (20 Hz–20 kHz) → audible noise from inductor/caps
- Output ripple frequency is irregular → EMI filter design becomes complex

**Minimum burst frequency to avoid audible noise:**
```
fburst > 25 kHz  (above upper limit of human hearing)
```
Some ICs guarantee this; others allow the engineer to set the burst frequency threshold.

**Design practice**: In battery-powered applications (phone chargers, portable devices), burst mode is almost universally used. The efficiency gain at 1–10% load can be 20–30 percentage points compared to fixed-frequency PWM.

---

### Q12. For a battery-powered system with Vin = 2.5 V to 4.2 V and Vout = 3.3 V, which topology would you choose and why?

**Answer:**

**Analysis of operating conditions:**

The input range 2.5–4.2 V straddles the output 3.3 V:
- At full charge (4.2 V): Vin > Vout → buck operation needed
- At partial discharge (3.3 V): Vin = Vout → neither buck nor boost; must transition through
- At low discharge (2.5 V): Vin < Vout → boost operation needed

**Topology options:**

**Option 1 — Buck only**: Works only when Vin > 3.3 V + dropout (~3.4 V minimum). Fails below 3.4 V, abandoning ~25% of battery capacity.

**Option 2 — Boost only**: Must always step up to 3.3 V. Fails when Vin ≈ 3.3 V (duty cycle D ≈ 0, extreme sensitivity). Works but is inefficient at full charge (boosts 4.2 V to 3.3 V — must buck and boost, which an ideal boost cannot do efficiently).

**Option 3 — SEPIC**: Non-inverting. Vout = Vin × D/(1-D). At Vin = 4.2 V, Vout = 3.3 V: D = 3.3/(4.2+3.3) = 0.44. At Vin = 2.5 V, Vout = 3.3 V: D = 3.3/(2.5+3.3) = 0.569. Operates continuously across the full range. Good choice but two inductors.

**Option 4 — Four-switch buck-boost**: High efficiency in buck and boost modes separately. Smooth transition through Vin = Vout. Single inductor. Most efficient overall.

**Option 5 — Inverting buck-boost**: Produces -3.3 V — not useful here.

**Recommended choice**: **Four-switch buck-boost** for highest efficiency across the full battery range. SEPIC is a valid alternative if the simpler control of a 3-switch or 4-switch design is not available.

**Design note**: The transition region (Vin ≈ Vout ± 500 mV) requires careful control design. In pure buck mode: D rises as Vin falls. In pure boost mode: D rises as Vin falls further. At the transition, the controller must seamlessly switch modes without a glitch on Vout — often implemented with a small overlap region where both stages switch.

---

### Q13. How does the RHP zero frequency change with load and duty cycle in a boost converter? What are the implications for compensation design?

**Answer:**

**RHP zero frequency:**
```
fRHP = Rload × (1-D)² / (2π × L)
```

**Dependence on load (Rload = Vout/Iout):**

```
fRHP ∝ Rload = Vout/Iout
```

At **light load** (high Rload): fRHP moves higher → less phase lag at the crossover frequency → easier to control.

At **heavy load** (low Rload): fRHP moves lower → more phase lag at crossover → harder to control.

**Worst-case for compensation**: Maximum load (minimum Rload). Design the compensator for maximum rated Iout.

**Dependence on duty cycle D (which varies with Vin):**

```
fRHP ∝ (1-D)²
```

Higher D (lower Vin, higher Vout/Vin ratio) → (1-D)² decreases → fRHP decreases → harder to control.

**Worst-case for duty cycle**: Maximum D, which occurs at minimum Vin.

**Combined worst case**: Maximum load AND minimum Vin simultaneously. Design the compensation for this corner.

**Implication for compensation design:**

1. Loop bandwidth must track with fRHP. If fRHP varies 10× across operating conditions, but the crossover frequency is fixed at fRHP_min/5, then at light load the crossover is at fRHP_max/50 — extremely conservative and slow transient response.

2. Some control ICs implement gain scheduling or use current mode control (which transforms the plant and reduces the RHP zero problem somewhat) to maintain consistent loop behaviour across operating conditions.

3. Adaptive bandwidth compensation: the crossover frequency is set to fRHP(current operating point)/N, adjusted by monitoring Vout, Iout, and Vin.

4. Practical rule: For a single compensator operating over all conditions, target fcross = fRHP_worstcase / 5, and accept that at light load the converter will be over-compensated (sluggish but stable).

---

## Advanced (Questions 14–18)

---

### Q14. Derive the control-to-output transfer function for a CCM boost converter in voltage mode control and identify the RHP zero.

**Answer:**

**Small-signal model of CCM boost:**

Using average-value modelling, the switch/diode pair is replaced by small-signal dependent sources. For a duty cycle perturbation d̂:

The key averaged equations are:
```
v̂L = Vin - (1-D)×v̂out - Vout×d̂    [inductor voltage]
î_diode = (1-D)×iL - IL×d̂           [diode (output) current]
```

**Control-to-output Gvd(s):**

Solving the small-signal model (with Cout and Rload at the output):

```
Gvd(s) = v̂out / d̂|_{v̂in=0}
```

After solving the state equations:

```
Gvd(s) = [Vout/(1-D)] × (1 - s×L/(Rload×(1-D)²)) / (1 + s×L/(Rload×(1-D)²) + s²×L×Cout)
```

Simplified with explicit pole/zero notation:

```
Gvd(s) = Vout/(1-D) × (1 - s/ωRHP) / (1 + s/(Q×ω0) + (s/ω0)²)

where:
  ω0 = (1-D)/√(L×Cout)        [double pole; note (1-D) factor — lower than LC resonance]
  ωRHP = Rload×(1-D)²/L       [RHP zero — positive zero, right-half plane]
  Q = Rload×(1-D)×√(Cout/L)   [quality factor]
```

**Key differences from buck:**
- DC gain: `Vout/(1-D)` = Vin/(1-D)² — higher than Vin, the gain is boosted
- Double pole: at `(1-D)/√(LC)` — NOT at `1/√(LC)` — D shifts the resonant frequency down
- RHP zero: at `Rload×(1-D)²/L` — in the right half plane, causing phase lag instead of phase lead
- No ESR zero shown (add `(1 + s×resr×Cout)` in numerator for real capacitor)

**Phase plot characteristics:**
- 0° at DC
- -90° at ω0 (double pole contributes -90° at resonance in a well-damped system)
- Approaches -180° above ω0
- RHP zero subtracts additional phase: approaches -270° at very high frequency
- This severe phase lag makes boosting the control bandwidth above ωRHP impossible without going unstable

---

### Q15. Compare current mode control vs voltage mode control for a boost converter. How does current mode affect the RHP zero?

**Answer:**

**Voltage mode control (VMC) for boost:**

The single outer voltage loop controls duty cycle directly. The control-to-output transfer function (as derived above) includes:
- Double LC pole at (1-D)ω0
- RHP zero at ωRHP
- Compensation must handle -40 dB/decade slope, -180° phase shift at LC resonance, plus RHP zero phase lag

**Peak current mode control (PCMC) for boost:**

The inner current loop regulates the peak inductor current. The outer voltage loop controls the current reference.

**How current mode modifies the boost transfer function:**

The inner current loop effectively eliminates the inductor from the outer (voltage) loop's perspective. The current-controlled inductor appears as a current source to the output. The resulting outer-loop plant is approximately first-order:

```
Gvd_cmc(s) ≈ [Vout/(1-D)] × (1 - s/ωRHP) / (1 + s/ωp)

where ωp = (1-D)² / (Rload × Cout × (some factor))
```

**Impact on RHP zero:**

The RHP zero **does NOT disappear** with current mode control. It is still present at the same frequency:
```
fRHP = Rload × (1-D)² / (2π × L)
```

However, current mode control provides one benefit: the double LC pole is replaced by a single output pole. This means there is less phase lag near the LC resonance, making the compensator design simpler. But the fundamental RHP zero limitation remains the binding constraint on bandwidth.

**Practical benefit of current mode for boost:**
- No double pole to manage (only single output pole)
- Inherent current limiting (peak current reference can be clamped)
- Better transient response for sudden load changes
- Slope compensation still required for D > 0.5

**Conclusion**: Current mode control does not eliminate the RHP zero — it is an inherent property of the boost topology, not the control method. The maximum achievable bandwidth is still limited by fRHP.

---

### Q16. A boost converter operates at D = 0.8 (Vin = 5 V, Vout = 25 V). Analyse the design challenges at this extreme duty cycle.

**Answer:**

At D = 0.8, (1-D) = 0.2. This extreme duty cycle introduces several challenges:

**Challenge 1: Very low fRHP**
```
fRHP = Rload × (1-D)² / (2π × L)
     = 25 × 0.04 / (2π × L)
     = 1 / (2π × L)   [for Iout = 1 A, Rload = 25 Ω]
```
With L = 100 µH: fRHP = 25,330 Hz. But at full load (Iout = 5 A, Rload = 5 Ω):
```
fRHP = 5 × 0.04 / (2π × 100×10⁻⁶) = 318 Hz
```
Maximum crossover: ~63 Hz — extremely sluggish loop response.

**Challenge 2: High inductor current**
```
IL_avg = Iout / (1-D) = 5 / 0.2 = 25 A
```
The inductor must be rated for 25 A average. Peak current adds to this. Inductor cost and size scale dramatically.

**Challenge 3: Very short off-time**
```
toff = (1-D) × Ts = 0.2 × Ts
```
At fsw = 100 kHz: `toff = 2 µs`. The diode/MOSFET must fully turn on and turn off in this window, plus dead time. This requires very fast gate drivers and low total gate charge.

**Challenge 4: Voltage stress**
Switch Q1 sees Vout = 25 V when off. Must select a MOSFET rated for at least 1.5–2× Vout = 37.5–50 V.

**Challenge 5: Efficiency**
```
I_rms_switch = Iout × √D/(1-D) = 5 × √(0.8/0.2) = 5 × 2 = 10 A_rms
```
Conduction loss at 10 mΩ: `P = 10² × 0.01 = 1 W` per switch.

**Mitigation strategies:**
- Reduce D by accepting Vin/Vout ratio nearer to 1 (not always possible)
- Use DCM at high D to eliminate RHP zero (may require large peak current)
- Use coupled inductors or resonant boost to achieve ZVS and improve efficiency
- Consider a cascade (two-stage) topology: first stage boosts 5 V → 12 V (D = 0.58), second stage 12 V → 25 V (D = 0.52) — two manageable duty cycles instead of one extreme one

---

### Q17. Describe the effect of parasitic components on boost converter performance, particularly the switch-node inductance and diode reverse recovery.

**Answer:**

**Switch-node parasitic inductance (Lpar):**

The PCB trace, component lead inductance in the loop formed by the switch Q1, diode D1, and output capacitor Cout.

**Effect:**
When Q1 turns off, the current through Lpar must commutate to the diode. The parasitic inductance creates a voltage spike on the switch node:
```
ΔV_spike = Lpar × (dI/dt)_commutation
```

At 10 nH parasitic and 1 A/ns commutation: `ΔV_spike = 10 V` added to Vout. The MOSFET must withstand Vout + ΔV_spike.

**Mitigation:**
- Minimise loop area: place D1 and Cout as close as possible to Q1
- Use a snubber (RC or RCD) across Q1 or D1 to clamp the spike
- Reduce dI/dt by slowing gate drive (trade: higher switching loss)

**Diode reverse recovery (in non-synchronous boost):**

When Q1 turns on (switch ON phase begins), the diode D1 must block. For a standard PN junction diode, reverse recovery occurs: the diode conducts briefly in reverse (for time trr) while stored minority charge is removed. During trr, both Q1 and D1 conduct simultaneously — a current spike.

```
Erec = 0.5 × Irr × Vout × trr
Prec = Erec × fsw
```

For D1 with Irr = 2 A, Vout = 25 V, trr = 100 ns, fsw = 200 kHz:
```
Prec = 0.5 × 2 × 25 × 100×10⁻⁹ × 200,000 = 500 mW
```

**Mitigation:**
- Use Schottky diode (majority carrier — no reverse recovery, trr < 1 ns)
- In synchronous boost, the diode is replaced by a MOSFET (body diode used during dead time only)
- Schottky has higher capacitance than fast recovery diode — trade-off at very high frequency

**Combined effect at high switching frequency:**
At 1 MHz, both the parasitic inductance ringing (low-Q resonance with Coss) and any remaining reverse recovery become significant contributors to EMI. Synchronous rectification with careful layout becomes essential.

---

### Q18. What is the minimum-load constraint in a boost converter, and how do you handle it?

**Answer:**

**Minimum load constraint in CCM:**

A boost converter operating in CCM requires a minimum load current to maintain CCM operation. If the load is removed (no load), the converter enters DCM and then — if operated with fixed frequency — can exhibit significant output voltage overshoot.

**Why overshoot at no load in CCM-designed boost:**

In CCM, the compensation is designed around the CCM transfer function (with the double pole at ω0). In DCM (no load), the transfer function changes drastically — the gain increases and the double pole disappears. If the CCM compensation over-drives the boost in DCM, the output voltage can overshoot significantly.

**Quantitative example:**

CCM boost: Vin = 5 V, Vout = 12 V, D = 0.583. If load is suddenly removed:
- Inductor had been carrying 2.4 A average.
- At the moment of load removal, the inductor current continues pumping energy into Cout.
- Vout rises until the converter reacts.
- If the control loop has slow transient response (limited by RHP zero compensation), Vout can overshoot by 10–30%.

**Solutions:**

1. **Minimum load resistor**: Connect a bleed resistor across Vout to ensure minimum current. Simple but wastes power continuously.

2. **DCM-capable control**: Use a control IC that seamlessly transitions between CCM and DCM modes, including PFM at light load. The control law changes between modes.

3. **Load detection and burst mode**: Monitor output current; at zero load, stop switching entirely and allow Vout to discharge slowly through the bleed resistor, then issue a single burst to restore Vout.

4. **Overvoltage protection (OVP)**: Implement OVP to shut down the converter if Vout rises above Vout_max. This protects the load but the converter will cycle (hiccup) without a minimum load.

5. **Pre-charge the compensation**: In burst mode or no-load PFM, the integrator in the compensator must be pre-charged to prevent windup from driving D to maximum when load returns.

**Practical rule**: Always specify a minimum load current in the boost converter datasheet or design specification. For LED drivers (where the load is always present), this is not a concern. For general-purpose regulators, minimum load handling must be explicitly designed.
