# Component Selection — Interview Preparation

## Overview

Component selection determines whether a power supply meets its performance, efficiency, reliability, and cost targets. Every MOSFET, inductor, capacitor, and diode involves trade-offs that require quantitative reasoning. Top-tier interviews test whether you can size components from first principles, recognise failure modes, and justify choices against alternatives.

---

## Key Parameters Reference

### MOSFET Selection Figures of Merit

```
Rds_on  — on-state resistance (mΩ); determines conduction loss
Qg      — total gate charge (nC); determines gate drive loss
Qgd     — gate-drain (Miller) charge; determines switching speed
Qoss    — output charge; determines capacitive switching loss
FOM1    — Rds_on × Qg  (switching FOM, lower is better)
FOM2    — Rds_on × Qgd (hard-switching FOM)

Conduction loss (high-side):  Pcond_HS = Iout^2 × Rds_on × D
Conduction loss (low-side):   Pcond_LS = Iout^2 × Rds_on × (1-D)
Switching loss (high-side):   Psw = 0.5 × Vin × Iout × (tr + tf) × fsw
Gate drive loss:               Pgate = Qg × Vgs × fsw
```

### Inductor Selection

```
Inductance:   L = Vout × (1-D) / (fsw × ΔIL)
Peak current: IL_peak = Iout + ΔIL/2
DCR loss:     P_DCR = Iout^2 × DCR  (DC winding resistance)
AC loss:      P_AC = Rac × ΔIL^2/12  (skin + proximity effect)
Saturation:   IL_sat must exceed IL_peak with margin (≥20%)
Energy stored: E = 0.5 × L × Ipeak^2
```

### Capacitor Selection

```
ESR ripple:    ΔV_ESR = ΔIL × ESR
ESL ripple:    ΔV_ESL = ESL × (dI/dt)  [high-frequency spike]
Cap ripple:    ΔV_C   = ΔIL / (8 × fsw × Cout)  [triangular ripple]
RMS current:   I_rms_cout = ΔIL / (2√3)  [for triangular waveform]
Input cap:     I_rms_cin  = Iout × √(D × (1-D))  [worst case at D=0.5]
```

### Diode Selection (Asynchronous or Rectifier)

```
Average current: I_D_avg = Iout × (1-D)
Peak current:    I_D_peak = IL_peak
Reverse voltage: V_RRM ≥ 1.5 × Vin (derating margin)
Conduction loss: P_D = Vf × Iout × (1-D)
Recovery loss:   P_rr = 0.5 × Qrr × Vin × fsw  (relevant for Si, not Schottky)
```

### Thermal Derating

```
Junction temperature:  Tj = Ta + P_diss × (Rth_jc + Rth_cs + Rth_sa)
Maximum case temp:     Tc_max = Tj_max - P_diss × Rth_jc
Derating guideline:    Tj_max ≤ 125°C (commercial), ≤ 150°C (automotive with margin)
Power derating:        Reduce rated power ≥10% per 10°C above 25°C (rule of thumb)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. What is MOSFET Rds_on and why does it matter for power converter efficiency?

**Answer:**

Rds_on (drain-source on-resistance) is the resistance of the MOSFET channel when it is fully enhanced (gate-source voltage well above threshold). It is the primary source of conduction loss.

**Conduction loss calculation:**

For a synchronous buck converter:
```
P_cond_HS = IL_rms_HS^2 × Rds_on_HS ≈ Iout^2 × D × Rds_on_HS
P_cond_LS = IL_rms_LS^2 × Rds_on_LS ≈ Iout^2 × (1-D) × Rds_on_LS
```

For Vin=12V, Vout=3.3V (D=0.275), Iout=10A, Rds_on=5 mΩ (each MOSFET):
```
P_cond_HS = 100 × 0.275 × 0.005 = 0.138 W
P_cond_LS = 100 × 0.725 × 0.005 = 0.363 W
Total conduction = 0.5 W
```

**Important dependencies:**
- Rds_on increases with junction temperature: approximately doubles from 25°C to 150°C (positive temperature coefficient for N-channel MOSFETs). This means initial room-temperature values must be derated.
- Rds_on decreases with higher Vgs: always drive the gate fully (within Vgs_max limits) using the recommended gate voltage.
- Die area inversely proportional to Rds_on: lower Rds_on = larger die = more expensive and higher Qg.

**Common mistake:** Candidates use room-temperature Rds_on in loss calculations. The actual Rds_on at 100°C junction temperature is typically 1.5–2x higher, making thermal calculations circular (more loss → higher Tj → higher Rds_on → more loss).

---

### Q2. Explain gate charge (Qg) and why it matters for switching frequency selection.

**Answer:**

Gate charge is the total charge that must be supplied to the gate to transition the MOSFET from off to on. It is used to calculate gate drive current and power.

**Gate drive power:**
```
P_gate = Qg × Vgs × fsw
```

This power is dissipated in the gate driver's pull-up and pull-down resistors, not in the MOSFET itself (assuming no external Rg, all loss is in the driver output impedance). However, it is lost from the system perspective.

**Switching time:**
```
t_on ≈ Qg / I_gate_avg
```

The Miller plateau (Qgd region) is where the switching transition actually occurs — during this flat region in Vgs, the switch node transitions while Vgs is roughly constant at the Miller voltage. This is where switching losses occur.

**FOM (Figure of Merit):**
```
FOM = Rds_on × Qg
```

Lower FOM means you can achieve lower Rds_on at a given switching speed, or switch faster at a given Rds_on. For a 100V MOSFET, a good FOM is below 10 Ω·nC.

**Practical selection:**
- High-frequency designs (>1 MHz): prioritise low Qg even at the cost of higher Rds_on — switching losses dominate.
- Low-frequency, high-current designs (<200 kHz): prioritise low Rds_on — conduction losses dominate.
- Cross-over point: find fsw where P_cond = P_sw, then optimise accordingly.

---

### Q3. What is the switching Figure of Merit (FOM) and how do you use it to compare MOSFETs?

**Answer:**

The FOM relates the two competing loss mechanisms in a MOSFET:

**FOM1 = Rds_on × Qg**

This is useful for comparing devices at the same voltage class. A lower FOM means the device achieves better efficiency for a given die technology. FOM is primarily limited by the semiconductor process (cell pitch, gate oxide, drift region design) rather than die area.

**Why FOM1 = Rds_on × Qg is process-limited:**
- Increasing die area: Rds_on ∝ 1/Area, but Qg ∝ Area, so FOM stays constant.
- You only improve FOM by changing the device physics (better process node).

**FOM2 = Rds_on × Qgd** (more relevant for hard-switching topologies)

This captures the Miller charge that actually controls the switch node transition rate. Lower FOM2 means faster switching at a given Rds_on, which reduces switching loss.

**Practical example — comparing two MOSFETs for a 500 kHz synchronous buck:**

| Device | Rds_on (mΩ) | Qg (nC) | Qgd (nC) | FOM1 | FOM2 |
|--------|-------------|---------|----------|------|------|
| A      | 3.5         | 25      | 8        | 87.5 | 28   |
| B      | 6.0         | 12      | 4        | 72   | 24   |

Device B has a better FOM despite higher Rds_on. At high frequency, device B will be more efficient. At low frequency with high current, device A wins on conduction loss.

**Common interview mistake:** Choosing solely on Rds_on without considering switching loss. The optimum depends on operating point (frequency, duty cycle, load current).

---

### Q4. How do you select the inductance value for a buck converter?

**Answer:**

The inductance is selected based on the desired current ripple ratio (r = ΔIL / Iout).

**Starting point — rule of thumb:**
```
r = ΔIL / Iout = 0.2 to 0.4  (20–40% ripple ratio)
```

**Inductance calculation:**
```
L = Vout × (1-D) / (fsw × ΔIL)
  = Vout × (1-D) / (fsw × r × Iout)
```

**Example:** Vin=12V, Vout=3.3V, Iout=2A, fsw=500kHz, r=0.3:
```
D = 3.3/12 = 0.275
ΔIL = 0.3 × 2 = 0.6 A
L = 3.3 × (1-0.275) / (500e3 × 0.6) = 3.3 × 0.725 / 300e3 = 7.98 µH → select 8.2 µH
```

**Trade-offs for different inductance values:**

Higher L:
- Lower ripple → lower output voltage ripple → smaller output cap
- Slower transient response (inductor limits di/dt)
- Larger physical size and cost
- Lower DCM onset current (stays in CCM at lighter loads)

Lower L:
- Higher ripple → higher peak/RMS currents → more conduction loss and capacitor stress
- Faster transient response
- Smaller size, lower cost
- Enters DCM at higher load currents

**Key check:** The peak current IL_peak = Iout + ΔIL/2 must be below the inductor's saturation current rating with at least 20–30% margin.

---

### Q5. What is DCR in an inductor and how does it affect converter performance?

**Answer:**

DCR (DC Resistance) is the ohmic resistance of the inductor winding measured at DC. It is the dominant loss mechanism in inductors for most switching converters.

**DCR power loss:**
```
P_DCR = Iout^2 × DCR
```

This is a constant loss independent of frequency, proportional to the square of output current. For a 2A converter with DCR=50mΩ:
```
P_DCR = 4 × 0.05 = 0.2 W
```

This represents 0.2W / (3.3V × 2A) = 3% efficiency loss, which is significant.

**Why DCR varies with inductance and package:**
- More turns → higher inductance → higher DCR (more wire length)
- Smaller package → thinner wire → higher DCR
- For a given package and inductance, lower DCR means larger conductor cross-section (less turns, fewer layers, or heavier gauge wire)

**DCR sensing for current measurement:**
DCR can be exploited as a lossless current sensor by placing a parallel RC network across the inductor:
```
R_sense = R_filter, C_sense = L / DCR
```
When the time constant RC = L/DCR, the voltage across the capacitor is proportional to inductor current:
```
V_C = IL × DCR
```

This avoids the power loss of a series sense resistor. Common in high-efficiency VRMs.

**Temperature dependence:** DCR increases with temperature at ~+0.393% per °C (copper TCR). At 100°C above ambient, DCR is approximately 40% higher than at 25°C.

---

### Q6. What is inductor saturation and why is it a critical design limit?

**Answer:**

Inductor saturation occurs when the core magnetic material can no longer support additional flux density increase — the relative permeability drops sharply, reducing inductance often by 80-90%.

**Physical mechanism:**
At saturation, the magnetic domains in the core are fully aligned. Additional current produces flux in air (permeability of 1) rather than in the core (permeability of hundreds to thousands). The effective inductance collapses.

**Consequence in a power converter:**
- Inductance drops → current ripple increases dramatically (ΔI = V×D×Ts/L)
- With low inductance, the inductor current slews rapidly
- This can cause overcurrent shutdown, MOSFET failure, or capacitor failure
- Saturation is self-reinforcing: lower L → higher peak current → deeper saturation

**Two types of saturation rating:**
1. **Hard saturation (Isat):** the current at which inductance drops by 30% or 40% (manufacturer-dependent). Avoid operating at or above this.
2. **Soft saturation:** gradual inductance reduction below Isat; design margin should account for the inductance at the actual peak operating current, not just the rated value.

**Design rule:** Peak inductor current (Iout_max + ΔIL/2 + overload/fault currents) must be well below Isat. Margin of 1.3–1.5x is typical. For automotive applications, use 2x margin.

**Core geometry effects:** Gapped cores (including powder cores) have a softer saturation knee than ungapped ferrite — they tolerate more peak current before complete collapse.

---

## Intermediate (Questions 7–12)

---

### Q7. How do you select the output capacitor for a synchronous buck converter?

**Answer:**

Output capacitor selection involves three independent requirements: ripple voltage, transient response, and RMS current rating.

**Step 1 — Determine the dominant ripple source:**

At low frequency (large cap, low ESR): capacitive ripple dominates
```
ΔV_C = ΔIL / (8 × fsw × Cout)
```

At high frequency (ceramic caps): ESR dominates
```
ΔV_ESR = ΔIL × ESR
```

ESL contributes a spike at the switching edge:
```
ΔV_ESL = ESL × (dI/dt)
```

**Step 2 — Minimum capacitance for ripple:**
```
Cout_min = ΔIL / (8 × fsw × ΔV_target)
```
For ΔIL=0.6A, fsw=500kHz, ΔV=10mV:
```
Cout_min = 0.6 / (8 × 500e3 × 0.01) = 15 µF
```

**Step 3 — Transient response:**

For a load step ΔIload, the capacitor must supply current during the time it takes the inductor to respond (approximately L/Vin × ΔIload for voltage-mode control):
```
ΔV_transient ≈ ΔIload × ESR + ΔIload × t_response / (2 × Cout)
```

**Step 4 — RMS current rating:**
```
I_rms = ΔIL / (2√3) ≈ 0.289 × ΔIL
```
The capacitor's ripple current rating must exceed this. For ceramics, this is rarely the limiting factor. For electrolytics, it often is.

**Capacitor type comparison:**

| Type       | ESR        | ESL   | Capacitance density | Temp stability | Cost  |
|------------|------------|-------|---------------------|----------------|-------|
| Ceramic    | <5 mΩ      | ~0.5nH| Good (X5R/X7R)      | Moderate       | Low   |
| Electrolytic| 20-100mΩ  | ~5nH  | Excellent           | Poor at cold   | Low   |
| Polymer    | 5-20 mΩ    | ~2nH  | Good                | Good           | Medium|
| MLCC (0402)| <1 mΩ      | ~0.2nH| Poor per part       | Moderate       | Low   |

**Modern practice:** Place 2–4 ceramic MLCCs (10–47 µF, X5R/X7R) in parallel directly at the output for low ESR/ESL, supplemented by a bulk electrolytic or polymer cap if transient performance requires more capacitance.

---

### Q8. How do you select the input capacitor and why is its RMS current rating critical?

**Answer:**

The input capacitor of a buck converter is stressed by pulsating current. Unlike the output, where the inductor smooths the current, the input cap must handle the full switching current transitions.

**Input current waveform:**
During the high-side switch on-time (D × Ts), the converter draws Iout from the input. During the off-time, it draws zero from the input. This creates a square-wave current of amplitude Iout and duty cycle D.

**RMS current in the input capacitor:**
```
I_rms_cin = Iout × √(D × (1-D))
```
Maximum at D = 0.5 (worst case): I_rms = Iout/2

For Iout=2A, D=0.275:
```
I_rms_cin = 2 × √(0.275 × 0.725) = 2 × 0.446 = 0.89 A
```

This is a large ripple current that will heat and degrade electrolytics. Ceramic caps handle this easily due to their low ESR and high current ratings.

**Voltage rating:**
Select capacitors rated for ≥1.5 × Vin (to account for derating, voltage coefficient of ceramics, and transients). X7R ceramics lose 50-80% of rated capacitance at rated voltage — always check the capacitance vs voltage curve.

**Inductance of input capacitor path:**
Any parasitic inductance between the input cap and switching node creates a voltage spike at turn-on:
```
V_spike = L_parasitic × dI/dt
```
This spike must not exceed MOSFET Vds_max. Place the input ceramic cap physically as close as possible to the switching node (HS drain to LS source) with short, wide traces.

---

### Q9. How do you select a freewheeling diode for an asynchronous buck converter?

**Answer:**

The freewheeling diode must handle the inductor current during the off-time and survive the reverse voltage equal to Vin during the on-time.

**Electrical requirements:**

1. **Average forward current:** I_avg = Iout × (1-D)
   For Iout=2A, D=0.275: I_avg = 2 × 0.725 = 1.45A
   Select a diode rated ≥2× this value (derating and pulsed current peaks)

2. **Peak current:** IL_peak = Iout + ΔIL/2 (same as inductor peak)

3. **Reverse voltage:** V_RRM ≥ 1.5 × Vin
   For Vin=12V: V_RRM ≥ 18V → select 30V or 40V rated Schottky

4. **Reverse recovery (trr):** Use Schottky diodes for all low-voltage (<200V) synchronous or asynchronous applications. Schottky diodes have no minority carrier storage and negligible trr (~5-10 ns). Silicon PN diodes have trr of 50-500 ns and significant reverse recovery current — each recovery event dissipates energy:
   ```
   P_rr = 0.5 × Qrr × Vin × fsw
   ```

**Power dissipation calculation:**
```
P_diode = Vf × Iout × (1-D) + 0.5 × Qrr × Vin × fsw
```
For a Schottky with Vf=0.4V at 1.45A:
```
P_diode = 0.4 × 2 × 0.725 = 0.58 W (negligible Qrr)
```

**Thermal check:**
```
Tj = Ta + P_diode × (Rth_jc + Rth_cs + Rth_sa)
```

**SiC Schottky for high-voltage applications (>200V):** SiC Schottky diodes have no minority carrier storage even at high voltages (standard Si Schottky is limited to ~150V). Used in PFC boost converters and high-voltage bus converters.

---

### Q10. Explain thermal derating of power components. Why can you not use a component at its full rated specifications?

**Answer:**

Thermal derating is the practice of reducing the allowable stress on a component (current, voltage, power) below its datasheet maximum when operating at elevated temperatures. It exists because:

1. **Tj limits are absolute maximums, not operating conditions:** Datasheet absolute maximum ratings apply for short-duration pulses, not continuous operation. Continuous Tj at 150°C reduces semiconductor lifetime exponentially (Arrhenius acceleration factor ≈ 2× per 10°C above 70°C for electromigration-limited mechanisms).

2. **Thermal resistance is not perfectly controllable:** PCB Rth_cs depends on solder quality, board copper area, airflow conditions — all have tolerance. Derating provides safety margin against a worse-than-expected thermal resistance.

3. **End-of-life degradation:** Solder joints, die attach, and bond wires degrade under thermal cycling. Keeping Tj lower extends operational life dramatically.

**Standard derating rules:**

| Component     | Parameter        | Derating guideline              |
|---------------|------------------|---------------------------------|
| MOSFET        | Id (current)     | 70–80% of rated at 25°C        |
| MOSFET        | Vds (voltage)    | 80% of V_DSS                   |
| Capacitor     | Voltage          | 50–80% of rated                 |
| Inductor      | Current (thermal)| 80% of rated at 25°C           |
| Inductor      | Current (sat)    | 70% of Isat                    |
| Diode         | IF (forward)     | 75% of rated                    |
| Resistor      | Power            | 50% of rated                    |
| Electrolytic  | Voltage          | 50–60% of rated (life-critical) |

**Ceramic capacitor voltage coefficient (important):** A 100µF, 10V X5R ceramic may only provide 30µF at 8V bias. Always check the capacitance vs voltage derating curve and select the capacitor based on its effective capacitance at the operating voltage.

**Practical implication:** A 100V MOSFET in a 48V input converter (operating Vds_max ≈ 53V with spikes) provides adequate voltage margin at 80% derating (80V rated), but insufficient at 60V rated. Size for worst case: use 100V rating minimum.

---

### Q11. What is the role of gate resistance in MOSFET switching and how do you select it?

**Answer:**

Gate resistance (Rg, external + internal) controls the rate of charge delivery to the gate capacitance during switching transitions, which directly sets switching speed and determines switching losses and EMI.

**Switching time:**
```
t_rise ≈ (Rg_ext + Rg_int) × (Qgs2 + Qgd) / (Vdrv - Vmiller)
```
where Vmiller is the plateau voltage. Faster switching (lower Rg) reduces switching loss but increases dV/dt and dI/dt, which:
- Increases EMI (radiated and conducted)
- Creates voltage spikes from parasitic inductance (V = L_par × dI/dt)
- Can cause shoot-through if dead time is inadequate

**Switching energy (per transition):**
```
E_sw = 0.5 × Vin × Iout × (tr + tf)
P_sw  = E_sw × fsw
```

**Gate drive loss:**
```
P_gate = Qg × Vdrv × fsw
```
This is dissipated in Rg (split between external and internal). Using a large Rg increases dissipation in the external resistor, not the gate driver.

**Separate turn-on and turn-off resistors:**
Common practice is to use separate resistors (via a diode network) for turn-on and turn-off:
- Turn-off Rg lower → fast turn-off reduces shoot-through risk
- Turn-on Rg higher → controlled dV/dt to manage EMI and voltage overshoot

**Typical values:** 2–10 Ω for high-frequency (>300 kHz) designs. Lower as frequency increases and gate charge decreases.

---

### Q12. How do you calculate and manage MOSFET body diode conduction during dead time?

**Answer:**

In a synchronous buck converter, there is a brief dead time between high-side turn-off and low-side turn-on (and vice versa) to prevent shoot-through (both MOSFETs conducting simultaneously).

**During dead time:**
The inductor current must continue flowing. With both MOSFETs off, the only available path is the low-side MOSFET body diode (or Schottky clamp if placed in parallel).

**Body diode loss:**
```
P_body = Vf_body × Iout × 2 × t_dead × fsw
```
For Vf_body ≈ 0.7V, Iout=2A, t_dead=20ns, fsw=500kHz:
```
P_body = 0.7 × 2 × 2 × 20e-9 × 500e3 = 0.028 W
```
Small but not negligible at very high frequency.

**Body diode reverse recovery:**
Standard MOSFET body diodes are PN diodes with significant reverse recovery charge (Qrr). When the high-side MOSFET turns on, it must first reverse-recover the low-side body diode. This creates a current spike:
```
I_rr_peak = Qrr / t_rr
```
This spike dissipates energy: `E_rr = 0.5 × Qrr × Vin`.

**Mitigation:**
- Use GaN FETs or MOSFETs with low Qrr body diodes
- Place a Schottky diode in parallel with the low-side MOSFET (lowers Vf, clamps body diode)
- Optimise dead time (shorter reduces body diode conduction but risks shoot-through)

**Dead time control:** Many modern gate drivers and controllers offer adaptive dead time that measures when the body diode starts/stops conducting and adjusts dead time accordingly.

---

## Advanced (Questions 13–18)

---

### Q13. Derive the conduction and switching loss breakdown for a synchronous buck converter at a given operating point. How do you optimise MOSFET selection given this?

**Answer:**

**Given:** Vin=12V, Vout=3.3V, Iout=10A, fsw=300kHz, D=0.275.

**Conduction losses (both MOSFETs, assuming equal Rds_on = 5 mΩ at Tj=100°C):**
```
P_cond_HS = Iout^2 × Rds_on × D = 100 × 0.005 × 0.275 = 0.138 W
P_cond_LS = Iout^2 × Rds_on × (1-D) = 100 × 0.005 × 0.725 = 0.363 W
P_cond_total = 0.50 W
```

**Switching losses (high-side only, LS switches at near-zero voltage in synchronous):**
```
tr = tf = Qgd / (I_gate)  [assume I_gate = (Vdrv - Vmiller)/Rg_total = (10-3)/3 = 2.3A]
Qgd_HS = 8 nC  →  tr = 8e-9/2.3 = 3.5 ns
P_sw = Vin × Iout × (tr + tf) × fsw = 12 × 10 × 7e-9 × 300e3 = 0.252 W
```

**Gate drive losses (both MOSFETs):**
```
Qg_HS = 25 nC, Qg_LS = 30 nC, Vdrv = 10V
P_gate = (25 + 30) × 10^-9 × 10 × 300e3 = 0.165 W
```

**Dead time / body diode (t_dead = 25 ns each transition):**
```
P_body = 0.7 × 10 × 2 × 25e-9 × 300e3 = 0.105 W
```

**Total MOSFET loss ≈ 0.50 + 0.252 + 0.165 + 0.105 = 1.02 W**

**Optimisation approach:**
- Conduction losses scale with Rds_on — use lower Rds_on device if thermal budget allows.
- Switching losses scale with Qgd and fsw — use device with lower Qgd for HS.
- LS MOSFET: primarily conduction loss (ZVS turn-on) → optimise for Rds_on.
- HS MOSFET: both conduction and switching → optimise for FOM2 (Rds_on × Qgd).
- Reducing Rg (faster switching) reduces Psw but increases EMI and body diode Qrr effects.

---

### Q14. How do you select inductors for high-frequency converters where AC losses become significant?

**Answer:**

At frequencies above 500 kHz, AC winding losses (skin and proximity effect) can exceed DCR losses. Core losses also become significant.

**Skin effect:**
Current density is concentrated near the conductor surface. The skin depth at frequency f:
```
δ = √(ρ / (π × µ0 × f))
```
For copper at 500 kHz: δ = 94 µm. Wires thicker than 2δ carry current inefficiently.

**Proximity effect:**
Fields from adjacent conductors drive current to one side of each wire. This is often more severe than skin effect in multi-layer windings.

**AC resistance factor:**
```
Rac/Rdc ratio increases with (wire diameter / skin depth)^4 approximately
```
For a 4-layer inductor at 1 MHz, Rac can be 5–10× Rdc.

**AC loss (winding):**
```
P_AC_winding = Rac × (ΔIL_rms)^2 = Rac × (ΔIL / 2√3)^2
```

**Core loss (Steinmetz equation):**
```
Pv = k × f^α × B_AC^β  [W/m³]
B_AC = Vout × (1-D) × D / (L × fsw × 2)  [simplification, ΔB = µ0 × µr × ΔH]
```
Typical for Mn-Zn ferrite (TDK PC95): k=2×10^-4, α=1.56, β=2.53

**Mitigation strategies:**
1. **Litz wire:** Multiple individually insulated strands twisted together, each strand ≤ 2δ diameter. Dramatically reduces AC resistance but expensive and difficult to wind.
2. **Foil windings:** Single-layer foil conductors with thickness equal to skin depth. Low proximity effect but limited to single-layer designs.
3. **Lower turn count (fewer layers):** Use gapped ferrite or powder core to allow larger air gap (lower µr_eff) → need more turns for same inductance → trade-off.
4. **Higher-permeability core material:** Fewer turns needed → lower AC loss.

**Practical selection at 1–5 MHz:** Use shielded power inductors with low-loss core material (Würth WE-MAPI, Coilcraft XGL series), which use optimised winding geometry for high-frequency performance. Compare using the AC loss data in the datasheet (often given as Q factor or impedance vs frequency).

---

### Q15. What is the voltage coefficient of ceramic capacitors and why does it create a hidden design error?

**Answer:**

X5R and X7R ceramic capacitors exhibit a strong reduction in capacitance as DC bias voltage increases. This is an intrinsic property of the barium titanate dielectric used in Class II ceramics.

**Typical derating:**
A 100 µF, 10V X5R MLCC may only provide:
- 100 µF at 0V
- 70 µF at 5V
- 30 µF at 8V
- 15 µF at 10V (rated voltage)

**Why this causes errors:**
Designers often place ceramic caps rated at the supply voltage. In a 3.3V output circuit, a 3.3V-rated cap provides almost no capacitance at operating voltage. The actual capacitance at 3.3V on a 4V rated cap could be 20% of the nominal value.

**Example — the dangerous mistake:**
Design requires Cout = 100 µF for 3.3V output. Engineer selects a 100 µF, 4V X5R MLCC. At 3.3V (83% of rated), the actual capacitance is approximately 25 µF — a 4x error in capacitance, causing the output ripple and transient response to be 4x worse than calculated.

**Correct approach:**
1. Check the manufacturer's capacitance vs. voltage derating curve (available in the datasheet or TDK/Murata online tools).
2. Select a capacitor rated at 2x the operating voltage for X5R/X7R ceramics.
3. Or use C0G (NP0) ceramics — no voltage coefficient, but available only to ~10 µF.
4. Or use polymer capacitors (near-zero voltage coefficient) for bulk capacitance.

**Temperature coefficient:**
X5R is rated ±15% over -55°C to +85°C. X7R is rated ±15% over -55°C to +125°C. Neither accounts for DC bias derating, which is additional.

---

### Q16. How do you perform a complete thermal analysis for a power MOSFET to verify it will survive in a given application?

**Answer:**

**Thermal circuit model:**
```
Tj = Ta + P_total × (Rth_jc + Rth_cs + Rth_sa)
```

Where:
- Rth_jc: junction-to-case thermal resistance (datasheet, fixed)
- Rth_cs: case-to-heatsink (depends on TIM quality; 0.1–0.5 °C/W typical)
- Rth_sa: heatsink-to-ambient (depends on heatsink selection and airflow)
- P_total: all power dissipated in the die

**Step 1 — Calculate total power dissipation:**
```
P_total = P_cond + P_sw + P_gate + P_body_diode
```
(Calculated in Q13 above)

**Step 2 — Find Rth_jc from datasheet:**
Example: MOSFET in D2PAK package → Rth_jc = 5 °C/W

**Step 3 — Estimate board thermal resistance (no heatsink, PCB mounting):**
For D2PAK on 2oz copper, 4-layer board with thermal vias: Rth_board ≈ 15–30 °C/W
(Use thermal simulation or manufacturer application notes)

**Step 4 — Calculate Tj:**
```
P_total = 0.5 W per MOSFET (from Q13, simplified)
Tj = 25 + 0.5 × (5 + 20) = 25 + 12.5 = 37.5°C   ← comfortable
```

**Step 5 — Verify Rds_on at actual Tj:**
Most MOSFETs show Rds_on increase with temperature. At 37.5°C (near room temp), no significant correction needed. If Tj were 100°C, Rds_on might be 1.7× higher, requiring iteration.

**Iteration for thermally stressed designs:**
1. Assume Tj (start with Tj = Tc_max - 20°C)
2. Calculate Rds_on at assumed Tj
3. Calculate P_total using this Rds_on
4. Calculate actual Tj from P_total
5. If actual Tj ≠ assumed Tj, iterate until converged

**Common mistake:** Using room-temperature Rds_on for thermal calculations. The resulting Tj will be underestimated, leading to unexpected thermal shutdown or reduced MTBF.

---

### Q17. Compare MOSFET technologies (Si planar, Si trench, SiC, GaN) for power supply applications.

**Answer:**

| Parameter          | Si Planar      | Si Trench (Super-junction) | SiC MOSFET    | GaN HEMT      |
|--------------------|----------------|---------------------------|---------------|---------------|
| Breakdown field    | 0.3 MV/cm      | ~0.3 MV/cm (SJ enhanced)  | 3.0 MV/cm     | 3.3 MV/cm     |
| Vds_max (practical)| 30–200V        | 400–900V                  | 600–1700V     | 40–650V       |
| Rds_on × Area FOM  | Baseline       | 5–10× better than planar  | 100× better   | 1000× better  |
| Gate charge Qg     | Moderate       | Low-moderate              | Higher        | Very low      |
| Body diode Qrr     | High           | High (SJ has high Qrr)    | Low-moderate  | No body diode (Qoss)|
| Switching speed    | Moderate       | Moderate                  | Fast          | Extremely fast|
| Gate drive voltage | 10–15V         | 10–15V                    | 15–20V        | 5V (GS_max ±7V)|
| Shoot-through risk | Low            | Moderate                  | Low           | High (dV/dt induced)|
| Cost (relative)    | 1×             | 2–3×                      | 5–10×         | 4–8×          |
| Frequency range    | <500kHz        | <1MHz                     | 1–5MHz        | 1–10MHz+      |

**GaN HEMT specifics:**
- Lateral device with 2DEG channel (no body diode; Qoss instead)
- Qoss recovery is much faster than body diode Qrr → enables very fast dead time
- Negative Vgs_th (some are depletion mode — normally on → requires special gate driver)
- dV/dt-induced turn-on risk: fast dV/dt at drain can charge Cgd and momentarily turn on the device; requires careful gate resistor selection and PCB layout

**SiC MOSFET specifics:**
- Higher gate threshold voltage (2–3V) improves noise immunity
- Wide bandgap enables operation at high junction temperatures (Tj up to 175°C)
- Body diode has no minority carrier storage but still has some forward voltage drop (~2.7V) → parallel SiC Schottky diode beneficial in high-frequency designs

**Selection guidelines:**
- <100V, <1MHz: Si trench MOSFET (cost-effective)
- <100V, 1–10MHz: GaN HEMT (highest efficiency)
- 200–600V, <500kHz: Si super-junction MOSFET
- 400–1700V, >100kHz: SiC MOSFET

---

### Q18. How do you select and validate input filter components for conducted EMI compliance?

**Answer:**

The input filter must attenuate the high-frequency switching current noise (at fsw and harmonics) to levels that meet CISPR 32 / EN 55032 or MIL-STD-461 limits before it reaches the power source.

**Noise source model:**
The switching converter generates a differential-mode (DM) noise current at fsw harmonics. The magnitude approximates:
```
I_noise ≈ ΔI_sw / (2π × f × Cin)  [fundamental]
```
The input capacitor already attenuates this; the external filter provides additional attenuation.

**LC filter attenuation:**
```
Attenuation(dB) = -40 × log10(f / f_resonant)  [for f >> f_resonant]
f_resonant = 1 / (2π × √(L_filter × C_filter))
```

**Design procedure:**
1. Determine the switching noise amplitude at the measurement frequency (typically 150 kHz for CISPR)
2. Compare against the limit (e.g., CISPR 32 Class B quasi-peak limit ≈ 66 dBµV at 150 kHz)
3. Calculate required attenuation = noise level - limit + margin (6 dB minimum)
4. Select Lf and Cf to achieve required attenuation with f_res well below fsw

**Filter resonance damping:**
The LC filter resonance can destabilise the converter input impedance (negative impedance interaction). Damp the resonance by placing a resistor in series with Cf (Rd ≈ 0.5 × √(Lf/Cf)) or use a parallel RC damping network.

**Negative impedance interaction:** The converter presents negative incremental input impedance (−Vin²/Pin) at its input. If the filter output impedance exceeds this magnitude, the system can become unstable (Middlebrook criterion). The filter impedance must be much lower than the converter's input impedance at all frequencies.

**Common mistake:** Designing the input filter to meet EMI limits without checking the Middlebrook stability criterion, resulting in oscillation at the filter resonance frequency.
