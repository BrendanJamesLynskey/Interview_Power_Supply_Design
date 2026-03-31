# Thermal Management — Interview Preparation

## Overview

Thermal management determines the long-term reliability of power supplies. Junction temperatures directly control semiconductor failure rates (every 10°C rise approximately doubles failure rate for some mechanisms), inductor and transformer insulation lifetime, and capacitor lifetime. This topic is tested at every level in power electronics interviews.

---

## Key Equations Reference

### Thermal Resistance Network
```
P_device → Rth_jc → Tc (case) → Rth_cs → Ts (sink) → Rth_sa → Ta (ambient)

T_junction = Ta + P × (Rth_jc + Rth_cs + Rth_sa)

where:
  Rth_jc = junction-to-case thermal resistance [°C/W] — from datasheet
  Rth_cs = case-to-sink thermal resistance [°C/W] — depends on TIM (thermal interface material)
  Rth_sa = sink-to-ambient thermal resistance [°C/W] — heatsink specification
```

### Maximum Power Dissipation
```
P_max = (Tj_max - Ta) / (Rth_jc + Rth_cs + Rth_sa)
```

### MOSFET Power Dissipation (Switch)
```
P_cond = Iout² × Rds_on × D        [conduction loss, top switch]
P_sw   = ½ × Vin × IL × (tr + tf) × fsw + Qoss × Vin × fsw   [switching loss]
P_gate = Qg × Vgs × fsw            [gate charge loss]
P_total = P_cond + P_sw + P_gate   [total per switch]
```

### Capacitor Lifetime (Electrolytic)
```
L = L0 × 2^((T0 - T) / 10)   [Arrhenius doubling rule]

L0 = rated lifetime at T0 (typically 2000 h at 105°C)
T = actual operating temperature
```

### Heatsink Thermal Resistance
```
Natural convection: Rth_sa ≈ 50°C/W for 1cm² fin, ≈ 5°C/W for 100cm²
Forced air: Rth_sa reduces by 3-10× with airflow

For a standard extrusion heatsink:
Rth_sa ≈ 45 / A_heatsink^0.9   [rough estimate, A in cm², Rth in °C/W]
```

### PCB Thermal Resistance (No Heatsink)
```
Rth_pcb = 1 / (h_conv × A_pkg)   [natural convection, h ≈ 10 W/(m²·K) for natural conv]
For an SO-8 package (A ≈ 50 mm²): Rth_pcb ≈ 1/(10 × 50×10⁻⁶) = 2000°C/W — need external heatsinking
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Explain the thermal resistance network from junction to ambient and how each element is determined.

**Answer:**

Heat flows from the semiconductor junction to the ambient environment through a series of thermal resistances — the thermal resistance network is analogous to an electrical resistor circuit with power (P) as current and temperature difference (ΔT) as voltage.

**Complete network:**
```
P_dissipated → [Rth_jc] → Case → [Rth_cs] → Heatsink → [Rth_sa] → Ambient
                           (Tc)              (Ts)                  (Ta)
```

**Rth_jc (junction-to-case):**

This is a property of the semiconductor package and is always provided in the datasheet. It is determined by:
- The silicon die area (larger die → lower Rth_jc)
- The package material (copper leadframe, direct-bonded copper DBC, moulding compound)
- The die attach method (solder, silver-filled epoxy, sintering)
- The package design (number of thermal vias within the package, exposed paddle)

For a TO-220 FET: Rth_jc ≈ 1–3 °C/W
For a D2PAK: Rth_jc ≈ 1–2 °C/W
For a small SO-8: Rth_jc ≈ 10–50 °C/W

**Rth_cs (case-to-sink):**

This is determined by the thermal interface between the device package and the heatsink:
- Direct metal-to-metal contact (no TIM): Rth_cs ≈ 0.5–1 °C/W (only for smooth, flat surfaces)
- Thermal grease (silicone-based): Rth_cs ≈ 0.1–0.5 °C/W
- Thermal pad (phase-change or graphite): Rth_cs ≈ 0.2–1 °C/W
- No heatsink (package on PCB): Rth_cs is replaced by Rth_jb (junction-to-board) — typically 3–20 °C/W

**Rth_sa (sink-to-ambient):**

This is a property of the heatsink, determined by fin geometry, surface treatment, and airflow:
- No forced air (natural convection): Rth_sa = 3–50 °C/W depending on heatsink size
- Forced air (1 m/s): Rth_sa typically reduces 2–3×
- Liquid cooling: Rth_sa < 0.1 °C/W possible

**Total junction temperature:**
```
Tj = Ta + P × (Rth_jc + Rth_cs + Rth_sa)

Design target: Tj ≤ 0.75 × Tj_max_rated   [25% derating for reliability]
```

---

### Q2. What is Rth_jb (junction-to-board) and when is it the correct thermal resistance to use?

**Answer:**

**Rth_jb definition:**

Rth_jb is the thermal resistance from the semiconductor junction to the PCB (bottom of the package). It describes heat transfer from the die, through the package, into the printed circuit board.

**When to use Rth_jb:**

Rth_jb applies when the PCB is the primary heatsink — no separate metallic heatsink attached. This is the case for:
- SMD packages without exposed paddle (SOIC, SOT-23, TSSOP)
- Packages on PCB without a heatsink where the package case is not accessible for direct contact
- Low-power devices where PCB spreading is sufficient

**Rth_jb vs Rth_jc:**

| Path         | Meaning                              | When applicable                         |
|--------------|--------------------------------------|-----------------------------------------|
| Rth_jc       | Junction → exposed thermal pad/case  | When a heatsink is attached to the case |
| Rth_jb       | Junction → PCB bottom               | When PCB is the heatsink                |
| Rth_ja       | Junction → ambient (combined path)  | Quick estimate, no heatsink             |

**Rth_ja (junction-to-ambient):**

The total from junction to ambient with no heatsink, tested on a specified JEDEC board (1S2P or 2S2P board). This combines all thermal paths:
```
Rth_ja = Rth_jc + Rth_pcb_to_ambient
or
Rth_ja = Rth_jb + Rth_board_to_ambient
```

**Important warning:** Rth_ja from the datasheet is measured on a specific JEDEC test board. Actual Rth_ja on a real PCB depends on copper area, board layers, and airflow. The datasheet Rth_ja is typically only valid for that specific test condition — use it only for rough initial estimates, then verify with thermal simulation or measurement.

---

### Q3. How do you calculate MOSFET power dissipation in a synchronous buck converter?

**Answer:**

A synchronous buck has two MOSFETs: high-side (switches with duty cycle D) and low-side (switches with duty cycle 1-D). Their loss mechanisms differ.

**High-side MOSFET:**

**Conduction loss:**
```
P_cond_HS = I_rms_HS² × Rds_on(Tj)

I_rms_HS = IL_avg × √(D × (1 + r²/12))  where r = ΔIL/IL_avg
         ≈ IL_avg × √D   [for small ripple]

P_cond_HS ≈ Iout² × D × Rds_on(Tj)

Important: Rds_on increases with junction temperature. From datasheet:
Rds_on(Tj) = Rds_on(25°C) × (Tj/300)^2.3   [typical silicon, normalized to 300K]
Rds_on doubles from 25°C to ~100°C for typical power MOSFETs
```

**Switching loss (high-side):**
```
P_sw = ½ × Vin × IL_peak × (tr + tf) × fsw   [hard switching]
     + Qoss × Vin × fsw   [output capacitor charge/discharge]

where tr, tf = rise and fall times from datasheet (at given test conditions)
```

**Gate charge loss:**
```
P_gate = Qg × Vdrv × fsw   [driver charges/discharges gate capacitance]
```

**Low-side MOSFET (synchronous):**

**Conduction loss:**
```
P_cond_LS = Iout² × (1-D) × Rds_on_LS(Tj)
```

**Body diode conduction (during dead time):**
```
P_diode = Vf × IL × (t_dead_on + t_dead_off) × fsw
```

**No turn-on switching loss:** The low-side FET turns on at near-zero voltage (ZVS) because the inductor current has already commutated the body diode before the gate turns on. Only turn-off loss applies:
```
P_sw_LS ≈ ½ × Vin × IL × tf × fsw   [turn-off loss only]
```

**Example calculation:**
```
Buck: Vin=12V, Vout=3.3V, Iout=10A, fsw=300kHz, D=0.275
FET: Rds_on(25°C)=5mΩ, Rds_on(100°C)≈9mΩ, Qg=30nC, Vdrv=10V, tr=tf=10ns

High-side:
P_cond_HS = 10² × 0.275 × 9mΩ = 0.248 W
P_sw_HS = ½ × 12 × 10 × 20ns × 300kHz = ½ × 12 × 10 × 6×10⁻³ = 0.360 W
P_gate_HS = 30nC × 10V × 300kHz = 90 mW

Total HS loss: 0.248 + 0.360 + 0.090 = 0.698 W

Low-side:
P_cond_LS = 100 × 0.725 × 9mΩ = 0.653 W
P_diode ≈ 0.7V × 10A × 2×10ns × 300kHz = 42 mW

Total LS loss: 0.653 + 0.042 = 0.695 W

Total both FETs: 1.39 W
```

---

### Q4. What is the purpose of thermal interface material (TIM) and how do you select one?

**Answer:**

**Purpose:**

Even "flat" surfaces have microscopic roughness (Ra = 0.5–5 µm for machined metal). When two surfaces contact, only the high spots touch — the valleys are filled with air, which has very low thermal conductivity (k_air = 0.025 W/m·K). This creates high contact resistance.

TIM fills the air gaps with a material of much higher thermal conductivity, dramatically reducing interface thermal resistance.

**TIM types and characteristics:**

**1. Thermal grease (silicone-based):**
- Thermal conductivity: 1–10 W/m·K (e.g., Shin-Etsu X-23, Dowsil TC-5026)
- Rth_cs typical: 0.05–0.3 °C/W (very low)
- Requires application (pump-out risk over time, messy)
- Not good for field replacement (service)
- Bond line thickness: 25–100 µm

**2. Phase-change material (PCM):**
- Thermal conductivity: 2–5 W/m·K
- Rth_cs typical: 0.1–0.5 °C/W
- Solid at room temperature, liquefies at 45–55°C → self-fills gaps
- Preferred for production (no mess, repeatable)
- Examples: Bergquist GP3000, Henkel PCM45F

**3. Thermal pad (silicone with filler):**
- Thermal conductivity: 1–15 W/m·K (common: 5–8 W/m·K)
- Rth_cs typical: 0.2–1 °C/W
- Good for field service (peel-and-stick)
- Thicker than grease (0.5–2 mm typical) → higher absolute thermal resistance
- Examples: Bergquist GP3000, Laird Tflex 600

**4. Graphite foil:**
- Thermal conductivity: 150–700 W/m·K (in-plane), 5–15 W/m·K (through-plane)
- Very low Rth_cs: 0.01–0.1 °C/W
- Rigid, requires flat surfaces
- Examples: Panasonic PGS graphite sheet

**5. Liquid metal (gallium-based):**
- Thermal conductivity: 10–35 W/m·K
- Extremely low Rth_cs: 0.005–0.05 °C/W
- Electrically conductive — must not contact leads
- Used in high-performance applications (laptop CPU cooling)

**Selection criteria:**
1. Thermal conductivity (higher = better)
2. Bond line thickness achievability
3. Production process (manual application, screen print, pad)
4. Electrical insulation required? (phase-change pads have dielectric layers)
5. Service requirements (field replaceable = pads; lab prototype = grease)
6. Cost

---

### Q5. How do you calculate heatsink requirements for a power transistor?

**Answer:**

**Step 1: Determine total device power dissipation**
```
P_device = [calculated from losses: P_cond + P_sw + P_gate]
Example: P_device = 3W
```

**Step 2: Determine maximum junction temperature**
```
Tj_max from datasheet: typically 150°C (silicon) or 175°C (silicon carbide)
Design target: Tj_operating ≤ 0.75 × Tj_max = 0.75 × 150 = 112°C (25% derating)
```

**Step 3: Determine maximum allowable total thermal resistance**
```
Rth_total_max = (Tj_operating - Ta) / P_device

For Ta = 50°C, Tj_operating = 110°C, P = 3W:
Rth_total_max = (110 - 50) / 3 = 20 °C/W
```

**Step 4: Subtract known thermal resistances**
```
From datasheet: Rth_jc = 1.5 °C/W (e.g., TO-252 package)
TIM (thermal pad): Rth_cs ≈ 0.5 °C/W

Rth_sa_max = Rth_total_max - Rth_jc - Rth_cs
           = 20 - 1.5 - 0.5 = 18 °C/W

→ Need a heatsink with Rth_sa ≤ 18 °C/W
```

**Step 5: Select heatsink**
```
Rth_sa = 18 °C/W is achievable with a relatively small heatsink in natural convection.
From heatsink manufacturer data: a 50 cm² fin heatsink typically achieves Rth_sa ≈ 10–15 °C/W natural convection.

Check: some controllers/modules have an exposed paddle directly soldered to the PCB. In that case:
Rth_jb ≈ 5–15 °C/W (from datasheet) replaces Rth_jc + Rth_cs if no heatsink.
Then Rth_total = Rth_jb + Rth_board (PCB spreading resistance to ambient).
```

**Step 6: Verify with thermal model or measurement**

Use finite element analysis (ANSYS Icepak, FloTHERM) or measure with a thermocouple on the case and calculate:
```
Tj_estimated = Tc_measured + P_device × Rth_jc
```

---

### Q6. What is electrolytic capacitor lifetime and how does operating temperature affect it?

**Answer:**

**Capacitor lifetime mechanism:**

Electrolytic capacitors have liquid electrolyte that evaporates slowly over time, particularly at high temperature. As electrolyte depletes:
- Capacitance decreases (below spec)
- ESR increases (above spec)
- The capacitor eventually fails (open circuit)

**Arrhenius doubling rule:**

Electrolyte evaporation rate approximately doubles for every 10°C increase in temperature:
```
L = L0 × 2^((T0 - T) / 10)

where:
  L  = expected lifetime at operating temperature T
  L0 = rated lifetime at rated temperature T0 (from datasheet)
  T  = actual operating temperature
```

**Example:**
```
Capacitor: rated lifetime L0 = 5000 h at T0 = 105°C

Operating at T = 85°C:
L = 5000 × 2^((105 - 85)/10) = 5000 × 2^2 = 5000 × 4 = 20,000 h

Operating at T = 65°C:
L = 5000 × 2^((105-65)/10) = 5000 × 2^4 = 5000 × 16 = 80,000 h ≈ 9 years

Operating at T = 105°C (at rated temperature):
L = 5000 × 2^0 = 5000 h ≈ 208 days (operating continuously — insufficient for most products)
```

**Design implication:**

For a 10-year product (87,600 hours at 100% duty): the capacitor must last 87,600 hours.
```
Required: L ≥ 87,600 h
87,600 = L0 × 2^((T0 - T)/10)

If L0 = 10,000 h at T0 = 105°C:
87,600/10,000 = 8.76 = 2^((105-T)/10)
log2(8.76) = (105-T)/10 → 3.13 = (105-T)/10 → T = 105 - 31.3 = 73.7°C

→ Capacitor must operate below 74°C for a 10-year continuous life.
```

**Current ripple derating:**

The datasheet also specifies maximum ripple current. Exceeding this causes internal heating:
```
ΔT_internal = I_ripple² × ESR × Rth_cap_to_ambient
```
The capacitor's operating temperature = ambient temperature + case-to-ambient + internal heating.
For accurate lifetime calculation, use the total temperature including internal heating.

**Practical rule:** Keep electrolytic capacitors as cool as possible. Every 10°C reduction doubles the lifetime. Do not place electrolytics directly above hot inductors or FETs.

---

## Intermediate (Questions 7–13)

---

### Q7. How do you perform a complete thermal analysis for a 50W buck converter?

**Answer:**

**Given:**
```
Buck converter: Vin=24V, Vout=5V, Iout=10A, Pout=50W
Switching frequency: 200 kHz
Efficiency: 90% → total losses = 50/0.9 - 50 = 5.56 W
Ambient temperature: 40°C
```

**Step 1: Loss breakdown**

Component-by-component losses (from design calculations):
```
High-side FET (IRF7843):  1.2 W
Low-side FET (IRF7843):   1.8 W
Inductor (DCR + core):    0.8 W
Input capacitor (ESR):    0.2 W
Output capacitor (ESR):   0.3 W
Gate driver + other:      0.25 W + (controller IC 0.2W)
Controller IC:            0.2W
Total:                    4.75 W  [note: differs from 5.56W — some losses missing or efficiency was estimated]
```

**Step 2: FET thermal analysis**

High-side FET (IRF7843, D2PAK, Rth_jc = 1.0 °C/W, Rds_on(25°C) = 3.3 mΩ):
```
P_HS = 1.2W

For mounting without heatsink (on PCB thermal pad):
Rth_jb (D2PAK) = 20 °C/W (from JEDEC test board)
On actual PCB with copper pour (3 × 3 cm = 9 cm² of 2 oz copper):
Rth_board_to_ambient ≈ 1/(10 W/m²K × 9×10⁻⁴) = 111 °C/W (natural convection only — poor!)
```

D2PAK needs a proper PCB copper pour heatsink or external clip-on heatsink.

**With 9 cm² copper pour + 2 thermal vias to inner plane:**
```
Rth_effective ≈ 40 °C/W (rough estimate for 9cm² copper area, natural convection)
Tj = 40 + 1.2 × (1.0 + 0 + 40) = 40 + 49.2 = 89.2°C ✓ (< 112°C limit)
```

Low-side FET (same package, P_LS = 1.8W):
```
Tj = 40 + 1.8 × (1.0 + 40) = 40 + 73.8 = 113.8°C → exceeds 112°C limit!
Need larger copper pour or heatsink for low-side FET.
```

**Step 3: Inductor thermal analysis**
```
P_inductor = 0.8W
Surface area (E25 core): ≈ 9 cm²
ΔT ≈ 450 × 0.8 / 9 = 40°C
T_inductor = 40 + 40 = 80°C ✓

Check B_sat at 80°C: B_sat(80°C) ≈ 380 mT (N87)
Design B_peak ≈ 150 mT → 150/380 = 39% of B_sat ✓
```

**Step 4: Capacitor thermal analysis**
```
P_output_cap = 0.3W on 3× 47µF polymer caps (estimated total ESR 5mΩ)
These caps are surface-mount, small, near the circuit board heat.
Cap case temperature ≈ ambient + small rise from self-heating ≈ 50°C
→ Lifetime: L = rated_lifetime × 2^((T0-T)/10) — need rated lifetime from datasheet.
```

**Step 5: System thermal conclusion**
- High-side FET: OK with 9 cm² copper pour
- Low-side FET: Needs larger copper pour (12+ cm²) or forced airflow
- Inductor: OK
- Capacitors: OK

---

### Q8. What is derating and why is it important for reliability?

**Answer:**

**Derating definition:**

Derating is operating a component below its specified maximum rating to reduce stress and improve reliability. It is a key design practice, particularly in military, aerospace, and industrial applications.

**Why derating works:**

Most semiconductor failure mechanisms have strong non-linear dependence on stress parameters (voltage, current, temperature). For example:
- **Dielectric breakdown (oxide failures):** Failure rate ∝ V^n where n = 3–5 for silicon oxide
  → Operating at 70% of rated voltage reduces failure rate by 3.7–7× (for n=4: 0.7^4 = 0.24, or 4.2× reduction)
- **Electromigration (metal migration in ICs):** Failure rate ∝ J^2 × e^(Ea/kT)
  → Reducing temperature by 10°C or current density by 30% dramatically extends life

**Common derating guidelines (from MIL-HDBK-217, NASA PD-AP-1307):**

| Component        | Stress Parameter   | Typical Derating    |
|------------------|--------------------|---------------------|
| MOSFET Vds       | Voltage            | 0.75 × Vds_max      |
| MOSFET Tj        | Temperature        | 0.80 × Tj_max       |
| Resistor P       | Power              | 0.50 × P_rated      |
| Electrolytic V   | Voltage            | 0.80 × V_rated      |
| Capacitor I_ripple| Current           | 0.80 × I_rated      |
| Inductor I_sat   | Current            | 0.80 × I_sat        |
| Wire current     | Current density    | 0.75 × rated        |

**Example — MOSFET voltage derating:**

A flyback converter has a maximum drain voltage of Vds_max = 2 × Vin + n × Vout + V_spike = 600V. A MOSFET rated for 650V has 8% margin. With 75% derating:
```
Required Vds_rating ≥ 600V / 0.75 = 800V → use 800V MOSFET
```

The 800V MOSFET has more margin and the higher-voltage semiconductor process typically provides more robust oxide at lower voltage stress.

**Military vs commercial derating:**

Military (MIL-HDBK-217F): 50% derating on most parameters. Very conservative.
Commercial/industrial: 75–90% derating is typical. Balances reliability with cost and size.
Consumer electronics: Often no formal derating — cost dominates, lifetime is 3–5 years.

---

### Q9. How does forced airflow change the thermal design and what parameters are key?

**Answer:**

**Natural vs forced convection:**

In natural convection, hot air rises and is replaced by cooler ambient air. The heat transfer coefficient h is determined by the temperature difference itself:
```
h_natural ≈ 5–10 W/(m²·K)   [for vertical surfaces, laminar flow]
```

Forced convection uses a fan or blower to move air over the heatsink. Heat transfer coefficient:
```
h_forced ≈ 30–100 W/(m²·K)   [depending on velocity and surface geometry]
```

This typically reduces Rth_sa by 3–10×.

**Airflow velocity effect:**

For a heatsink in crossflow (fan blowing across fins):
```
Rth_sa ≈ Rth_sa_natural / (1 + (v/v0)^0.5)   [simplified, v = airflow velocity, v0 = reference]

More commonly: Rth_sa is plotted vs. airflow (in LFM or m/s) in heatsink datasheets
```

**Fan specification parameters:**

1. **Airflow (CFM or m³/h):** Volume of air moved per unit time
2. **Static pressure (Pa or in H₂O):** Pressure the fan can generate against resistance. Low-restriction (open board): use high CFM fans. High-restriction (fins, card cage): use high static pressure fans.
3. **Fan curve:** Plot of static pressure vs. airflow — operating point is where fan curve intersects system resistance curve.
4. **MTBF:** Fan bearings fail. Ball-bearing fans: 50,000–100,000 h MTBF at 40°C. Sleeve bearing: 20,000–30,000 h. Specify fan MTBF as a system reliability constraint.

**Fan failure impact:**

If the fan fails in a server power supply:
- Without fan: Rth_sa increases 3–10×
- Junction temperature rises rapidly
- Thermal protection must shut down within minutes before damage

**Redundant fan design:** Use two smaller fans. If one fails, the other maintains 70–80% of total airflow, sufficient to keep the system running at reduced load until the fan is replaced.

**Fan acoustic noise:**

Radiated noise = 20 × log(v_tip / v_ref) × number_of_blades_correction. Higher speed → more noise. Design for minimum speed that meets thermal requirements, with speed control (PWM fan or tachometer feedback) to reduce noise at light load.

---

### Q10. What is thermal resistance of a PCB trace and how does it limit maximum current?

**Answer:**

**Trace heating mechanism:**

A PCB copper trace carrying current I has resistance R = ρ × L / (w × t) where:
- ρ = resistivity of copper = 1.72×10⁻⁸ Ω·m
- L = trace length, w = trace width, t = trace thickness

Power dissipated: P = I² × R

Heat is removed by conduction through the PCB laminate and convection from the trace surface.

**IPC-2221 current capacity rule:**

The industry standard formula for trace current carrying capacity:
```
I = (0.048 × ΔT^0.44 × A^0.725)   [external trace, 1 oz copper]

where:
  ΔT = temperature rise above ambient [°C]
  A  = trace cross-section area [mils²] = width(mils) × thickness(mils)
  I  = current in Amps

For ΔT = 10°C, 1 oz copper (1.4 mil thick), 50 mil (1.27mm) wide trace:
A = 50 × 1.4 = 70 mils²
I = 0.048 × 10^0.44 × 70^0.725 = 0.048 × 2.754 × 18.54 = 2.45 A
```

**Simplified rule of thumb:**
```
1 oz copper: 600 mA/mm width for ΔT = 10°C external trace
2 oz copper: 1200 mA/mm width for ΔT = 10°C
```

**Internal traces:** Reduce by 50% (less convection cooling, surrounded by laminate with low thermal conductivity k ≈ 0.3 W/m·K).

**Thermal limitation vs resistance limitation:**

At low currents, the resistance limitation is dominant (ohmic heating is the concern). At very high currents (> 30A), the mechanical reliability of copper also becomes a concern (thermal cycling stress, solder joint fatigue).

For high-current applications (> 20A):
- Use multiple parallel traces
- Use thick copper (3 oz or 4 oz)
- Use bus bars or copper bars bonded to the PCB
- Route traces to internal layers AND external for parallel current paths

---

## Advanced (Questions 11–15)

---

### Q11. Explain thermal runaway and identify three power supply topologies or components where it is a risk.

**Answer:**

**Thermal runaway mechanism:**

Thermal runaway occurs when an increase in component temperature causes an increase in power dissipation, which further increases temperature — a positive feedback loop that can cause catastrophic failure.

**Mathematical condition:**

Thermal runaway occurs when:
```
dP/dT × Rth > 1

where:
  dP/dT = rate of change of power dissipation with temperature
  Rth = total thermal resistance from component to ambient
```

If this condition is met, small temperature perturbations grow without bound.

**Three power supply risks:**

**1. Ferrite core near loss minimum:**

Ferrite (e.g., MnZn) core loss has a minimum at ~100°C. Above this temperature, loss increases with temperature. If the converter operates above 100°C:
```
dP_core/dT > 0 at T > T_loss_min

If Rth_core × dP_core/dT > 1 → thermal runaway
```

Mitigation: Design operating temperature below the core loss minimum, or use heatsinking to ensure Rth_core × dP_core/dT < 1 even at peak temperature.

**2. Electrolytic capacitors with increasing ESR:**

As electrolytes age or dry out, ESR increases. Higher ESR → more heating from ripple current → faster electrolyte evaporation → higher ESR. This can be a slow runaway (days/months) or rapid (hours if current is very high):
```
P_cap = I_ripple² × ESR
dP_cap/dESR × (dESR/dT) × Rth > 1 → accelerated failure
```

Mitigation: Ensure ripple current is below rated value at the operating temperature, with margin for ESR increase.

**3. NTC thermistor in parallel with a power resistor:**

An NTC (Negative Temperature Coefficient) thermistor has resistance that decreases with temperature. If used incorrectly in a current-limiting circuit, increased current → heating → lower NTC resistance → less current limiting → more current → more heating:
```
dP_NTC/dT = I² × dR_NTC/dT  [as T rises, R_NTC drops, P = I²R drops... actually this is self-limiting]
```

Actually, the correct runaway case with NTC: if the NTC is used in parallel with a load resistor as a thermally-controlled circuit, and the NTC current path has poor thermal coupling to the reference thermal mass, a runaway can occur. More commonly, PTC (Positive Temperature Coefficient) thermistors are used for overcurrent protection precisely because they do NOT exhibit runaway.

**3 (revised) — MOSFET avalanche without adequate heat sinking:**

Repetitive avalanche events (when Vds exceeds BVDSS briefly during clamped inductive switching) dissipate energy in the FET:
```
E_avalanche = ½ × L × I_a² / (V_clamp - Vin)

Each event raises Tj by ΔTj = E_av × Rth_jc
```

If the thermal resistance is too high and events are frequent, Tj rises → Rds_on increases (positive TC) → more current → more loss → thermal runaway. However, for MOSFETs with positive Rds_on temperature coefficient (typical), this is actually self-limiting in conduction. Avalanche runaway is more relevant for FETs operating near BVDSS with capacitive load discharges.

---

### Q12. How do you use thermal impedance (transient) data to analyse pulse power dissipation?

**Answer:**

**Thermal impedance vs time (Zth_jc(t)):**

Datasheet provides Zth_jc(t) — the transient thermal impedance from junction to case as a function of time. For a single power pulse of magnitude P and duration t:
```
ΔTj = P × Zth_jc(t)
```

For t → ∞: Zth_jc → Rth_jc (steady-state)
For short t: Zth_jc << Rth_jc (junction can dissipate more power transiently)

**Foster thermal model:**

The transient Zth is modelled as a ladder network of R-C elements:
```
Zth_jc(t) = Σ Ri × (1 - e^(-t/τi))   [sum of R-C stages]
```

Typically 4–6 stages (R1C1, R2C2, ... each with different time constants covering die, die attach, package, header).

**Pulse power analysis:**

For a rectangular pulse of power P occurring at duty cycle δ = t_on/T_period:
```
T_j_peak = Ta + P_peak × Z_th_transient(t_on) × (correction for duty cycle)
         + P_avg × (Rth_jc - Z_th_transient(T_period))

Simplified for duty cycle δ:
ΔT_j_peak = P_peak × [Zth_jc(t_on) - δ × Zth_jc(T_period)] + P_avg × Rth_jc
```

**Example:**

A converter has 100A load pulses of 100 µs duration at 10 Hz (δ = 0.001). MOSFET dissipates 10W during the pulse.

From datasheet: Zth_jc(100µs) = 0.05 °C/W, Rth_jc = 1.0 °C/W

```
P_avg = 10W × 0.001 = 10 mW

ΔTj_peak ≈ 10 × 0.05 - 0.001 × 1.0 + 0.01 × 1.0
         = 0.5 - 0.001 + 0.01 = 0.509°C

This is very small — the 100µs pulse barely heats the junction.
For a 10ms pulse: Zth_jc(10ms) ≈ 0.3 °C/W
ΔTj_peak ≈ 10 × 0.3 + 0.1 × 1.0 = 3.1°C → still small.
For a 1s pulse: Zth_jc → Rth_jc → ΔTj ≈ 10W × 1.0 = 10°C + ambient
```

---

## Quick Reference: Thermal Design Checklist

```
THERMAL DESIGN PROCESS:
1. Calculate loss breakdown: P_device for each component
2. Determine Tj_max (operating) = 0.75 × Tj_max_rated
3. Calculate required Rth_total = (Tj_max_op - Ta) / P_device
4. Select Rth_jc from datasheet
5. Estimate Rth_cs from TIM and interface area
6. Calculate required Rth_sa = Rth_total - Rth_jc - Rth_cs
7. Select heatsink with Rth_sa ≤ required value
8. Verify capacitor temperatures (use Arrhenius for lifetime)
9. Verify inductor temperature (B_sat at operating T)
10. Test with thermocouple or thermal camera at full load

THERMAL RESISTANCE TYPICAL VALUES:
Rth_jc (FET):     0.5–3 °C/W (TO-220), 1–2 °C/W (D2PAK), 10–30 °C/W (SO-8)
Rth_cs (grease):  0.05–0.2 °C/W
Rth_cs (pad):     0.2–0.5 °C/W
Rth_sa (natural): 3–50 °C/W depending on size and orientation
Rth_sa (forced):  0.5–15 °C/W
Rth_jb (PCB):     3–20 °C/W (depends on copper area)

RELIABILITY RULES:
Tj operating ≤ 0.75 × Tj_max_rated
Tj_max_rated ≤ 150°C (Si), ≤ 175°C (SiC), ≤ 200°C (GaN, some)
Electrolytic: reduce Tcap by 10°C → 2× lifetime
Every 10°C increase in Tj → ~2× failure rate (Arrhenius)
```
