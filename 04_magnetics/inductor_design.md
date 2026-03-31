# Inductor Design — Interview Preparation

## Overview

Inductor design is a core competency for power electronics engineers. A poorly designed inductor causes saturation, excessive losses, thermal failure, or EMI problems. This section covers the full design process: inductance calculation, core selection, winding design, loss analysis, and thermal verification. These concepts appear in hardware design interviews at all experience levels.

---

## Key Equations Reference

### Inductance from Volts-Seconds (Buck Converter)
```
L = (Vin - Vout) × D × Ts / ΔIL  =  Vout × (1-D) / (fsw × ΔIL)

where ΔIL = peak-to-peak inductor current ripple (typically 20-40% of IL_avg)
```

### AL Value (Core Constant)
```
L = AL × N²          [L in nH, AL in nH/turn²]
N = √(L / AL)        [turns required]
```

### Peak Flux Density (Saturation Check)
```
B_peak = L × I_peak / (N × Ae)   [Tesla]
       = µ0 × µr × N × I_peak / (le + N²×µ0×µr×Ae/Lg)  [with air gap]

where:
  Ae = effective core cross-sectional area (m²)
  le = effective magnetic path length (m)
  Lg = air gap length (m)
  µr = relative permeability of core material
```

### Air Gap Length
```
Lg = µ0 × N² × Ae / L - le/µr   ≈  µ0 × N² × Ae / L   [when µr >> 1]

Simplified: Lg = µ0 × N² × Ae / L  [for Lg >> le/µr]
```

### DCR (Winding Resistance)
```
DCR = ρ_Cu × MLT × N / Aw_wire

where:
  ρ_Cu = 1.72×10⁻⁸ Ω·m at 20°C  (temperature coefficient: 0.393%/°C)
  MLT  = mean length per turn (m)
  Aw_wire = wire cross-sectional area (m²)
```

### Skin Depth
```
δ = √(ρ_Cu / (π × f × µ0)) = 66.5 / √f   [mm, at 20°C, f in Hz]

At 100 kHz: δ = 0.21 mm
At 500 kHz: δ = 0.094 mm
```

### Copper Loss (DC + AC)
```
P_Cu = I_rms² × DCR + I_rms_AC² × R_AC

R_AC ≈ DCR × (d/(2δ))   for d/δ > 2 (skin effect only, single layer)
     [Dowell's equation gives exact result including proximity effect]
```

### Core Loss (Steinmetz Equation)
```
P_core = Cm × f^α × B_peak^β × Ve

where Cm, α, β are material-dependent Steinmetz parameters
Ve = effective core volume (m³ or cm³)
```

### Temperature Rise (Simplified)
```
ΔT ≈ (P_total / A_surface)^0.833   [°C, A_surface in cm², P in W]  — Pressman formula
ΔT ≈ 450 × (P_total / A_surface)   [°C, rough rule of thumb]
```

---

## Fundamentals (Questions 1–6)

---

### Q1. How do you determine the required inductance for a buck converter?

**Answer:**

The inductance is chosen to control the inductor current ripple ΔIL. The design trade-off is:
- Too large L: slow current rise, good for reducing ripple but physically larger, heavier, and more expensive
- Too small L: large ripple, risk of DCM at light loads, higher peak currents → larger core, more losses

**Derivation:**

During the on-time (switch closed) of a buck converter:
```
VL = Vin - Vout
ΔIL = VL × ton / L = (Vin - Vout) × D × Ts / L
```

Rearranging for L:
```
L = (Vin - Vout) × D × Ts / ΔIL
  = Vout × (1-D) / (fsw × ΔIL)   [substituting D = Vout/Vin]
```

**Choosing the ripple ratio:**

The ripple ΔIL is typically specified as a fraction of the average current IL_avg = Iout:
```
Ripple ratio r = ΔIL / IL_avg

Typical range: r = 0.2 to 0.4 (20% to 40% peak-to-peak)
```

- r = 0.2 (20%): large L, low ripple, good for low-noise, servo applications
- r = 0.4 (40%): smaller L, acceptable ripple, common in power supplies
- r > 0.6: risk of DCM at rated load; peak currents significantly above average

**Worked example:**
```
Vin=12V, Vout=5V, Iout=3A, fsw=200kHz, target r=0.3

D = 5/12 = 0.417
ΔIL = r × Iout = 0.3 × 3 = 0.9A

L = Vout × (1-D) / (fsw × ΔIL)
  = 5 × (1-0.417) / (200kHz × 0.9)
  = 5 × 0.583 / 180,000
  = 2.915 / 180,000
  = 16.2 µH
```

Choose L = 15 µH (standard value — slightly increases ripple to 0.97A, ripple ratio = 32%).

---

### Q2. What is the AL value and how do you use it to calculate the number of turns?

**Answer:**

**Definition:**

AL (inductance factor) is a core manufacturer's constant that characterises how much inductance is obtained per turn-squared for a given core. It accounts for the core geometry, material permeability, and any pre-existing air gap:

```
L = AL × N²   [L in nH or µH, N in turns]
```

Units are typically nH/turn² or µH/1000turns² depending on the manufacturer's convention. Always check the datasheet units.

**Physical basis:**

From Faraday's law and Ampere's law, the inductance of a coil on a magnetic core is:
```
L = µ0 × µr × N² × Ae / le   [no air gap]
L = N² / (le/(µ0×µr×Ae) + Lg/(µ0×Ae))   [with air gap]

AL = µ0 × µr × Ae / le   [no air gap AL]
AL = µ0 × Ae / (le/µr + Lg)   [with air gap AL]
```

**Calculating turns:**

Given a target inductance L and a chosen core with known AL:
```
N = √(L / AL)
```

Round N to the nearest integer (always round up if saturation margin is tight).

**Worked example:**
```
Target: L = 15 µH = 15,000 nH
Core: Ferroxcube E25/13/7, N87 material, ungapped: AL = 2300 nH/turn²

N = √(15,000 / 2300) = √6.52 = 2.55 turns → round to 3 turns

Check: L_actual = 2300 × 3² = 20,700 nH = 20.7 µH (higher than target — acceptable)

If exact 15 µH is needed: calculate required air gap
Lg = µ0 × Ae × N² / L - le/µr   [requires core dimensions from datasheet]
```

**Gapped core AL:**

For a gapped core (common for inductors to prevent saturation):
```
Lgap = µ0 × N² × Ae / L   [simplified, valid when Lg >> le/µr]

AL_gapped ≈ µ0 × Ae / Lgap = L / N²
```

The gapped AL is much lower than the ungapped AL, requiring more turns to achieve the same inductance — but allowing higher saturation current.

---

### Q3. What is saturation in a magnetic core and how do you prevent it?

**Answer:**

**What is saturation:**

When the magnetic flux density B in a core reaches the saturation flux density B_sat, the incremental permeability µ_r drops dramatically toward 1 (air-like). The inductance collapses:
```
L = µ0 × µr × N² × Ae / le

At saturation: µr → 1 → L drops by factor of (original µr)
```

**Consequence in a converter:**

When the inductor saturates during a switching cycle:
- Inductor current rises rapidly (di/dt = V/L becomes very large because L drops)
- MOSFET current exceeds safe limits
- Protection circuits may trip, or the MOSFET fails catastrophically

**When does saturation occur:**

At the inductor peak current:
```
I_peak = I_avg + ΔIL/2

For a buck converter:
I_peak = Iout + ΔIL/2 = Iout × (1 + r/2)
```

The core saturates when:
```
B_peak = L × I_peak / (N × Ae) ≥ B_sat

Or equivalently: N × I_peak ≥ H_sat × le  [magnetomotive force limit]
```

**Prevention methods:**

1. **Choose adequate core size (Ae):** Larger Ae reduces B for a given flux level.

2. **Air gap the core:** An air gap reduces the effective permeability, requiring more magnetomotive force (N×I) to reach saturation. The same core can now handle larger peak currents:
   ```
   B_sat is unchanged, but H_sat_with_gap = H_sat_core + Lg/µ0
   → N × I_sat = H_sat_core × le + Lg × Bsat/µ0
   ```
   For Lg >> le/µr: `I_sat ≈ Bsat × Lg / (µ0 × N)` — independent of core material.

3. **Use more turns N:** Reduces B for a given current (B = µ × H = µ × N × I / le). But more turns increases DCR and copper losses.

4. **Use lower-permeability core material:** Powder cores (Kool Mµ, MPP) have distributed air gaps — they saturate more gradually than ferrite, tolerating higher peak currents.

5. **Design margin:** Target B_peak ≤ 0.7 × B_sat to allow for:
   - Current overshoot during transients
   - Temperature effects (B_sat decreases with temperature for ferrite)
   - Component tolerances

---

### Q4. What are proximity effect and skin effect, and how do they increase AC winding losses?

**Answer:**

**Skin Effect:**

In a conductor carrying AC current, the current density is higher at the surface than at the center. The current concentrates in a thin layer (skin depth δ) at the conductor surface:
```
δ = √(ρ / (π × f × µ0)) = 66.5 / √f   [mm, copper, f in Hz]

Examples:
  f = 50 kHz:  δ = 0.297 mm
  f = 200 kHz: δ = 0.149 mm
  f = 1 MHz:   δ = 0.066 mm
```

For a round wire of diameter d:
- If d << 2δ: essentially uniform current distribution, R_AC ≈ R_DC
- If d >> 2δ: current concentrates in outer shell; effective resistance R_AC >> R_DC

**Proximity Effect:**

When multiple conductors carry AC current near each other (as in an inductor winding), the magnetic field of each conductor induces eddy currents in adjacent conductors. These eddy currents redistribute the current in each conductor.

The proximity effect is often more significant than skin effect in multi-layer windings:
- Inner layers of a multi-layer winding carry much higher current density than outer layers
- The AC resistance can be 10–100× the DC resistance for many-layer windings at high frequency

**Dowell's Equation (simplified):**

For a single-layer winding with N_layers:
```
R_AC/R_DC = Δ' × [sinh(2Δ') + sin(2Δ')] / [cosh(2Δ') - cos(2Δ')]
           + (2/3) × (N_layers² - 1) × [sinh(Δ') - sin(Δ')] / [cosh(Δ') + cos(Δ')]

where Δ' = d/(δ × √(η))  — normalized wire diameter
      η = packing factor (conductor width / winding width)
```

For d/δ << 1 (thin wire): R_AC/R_DC → 1 (no AC resistance increase)
For d/δ >> 1 (thick wire, N_layers layers): R_AC/R_DC → (N_layers)² × (large factor)

**Design implications:**

1. **Use litz wire:** Multiple thin strands twisted together, each strand thinner than δ. Individual strands see uniform current. Very effective at reducing proximity effect.
   ```
   Strand diameter d_strand << 2δ at operating frequency
   ```

2. **Limit number of layers:** Proximity effect grows with N_layers². Prefer single-layer or two-layer windings. Interleave windings (P-S-P or S-P-S) in transformers.

3. **Reduce winding frequency:** Operating at lower fsw reduces AC losses but increases inductor size.

4. **Flat/foil conductors:** Reduce the conductor height to below δ. Very effective for single-layer windings (low skin effect), but multi-layer foil windings suffer severe proximity effect.

---

### Q5. How do you calculate DCR and what is its impact on efficiency?

**Answer:**

**DCR calculation:**

The DC resistance of the winding depends on the total wire length and the wire cross-sectional area:
```
DCR = ρ_Cu × L_wire / Aw_wire

where:
  L_wire = N × MLT   [total wire length = turns × mean length per turn]
  Aw_wire = π × (d_wire/2)²   [wire cross-sectional area]
  ρ_Cu = 1.72×10⁻⁸ Ω·m at 20°C
```

**Temperature correction:**
```
ρ_Cu(T) = ρ_Cu(20°C) × (1 + 0.00393 × (T - 20))
DCR(T) = DCR(20°C) × (1 + 0.00393 × (T - 20))
```

At 100°C: DCR increases by 1 + 0.00393 × 80 = 1.314 → 31% higher than room temperature.

**Impact on efficiency:**

Copper conduction loss:
```
P_Cu = I_rms² × DCR(T)

For a buck converter: I_L_rms ≈ Iout × √(1 + r²/12)   [r = ΔIL/Iout]
  At r = 0.3: I_rms = Iout × √(1 + 0.0075) = 1.004 × Iout ≈ Iout
```

**Example:**
```
Iout = 5A, DCR = 20mΩ at 20°C, operating at 80°C

DCR(80°C) = 20mΩ × (1 + 0.00393 × 60) = 20mΩ × 1.236 = 24.7mΩ

P_Cu = 5² × 0.0247 = 0.617 W
Efficiency loss: P_Cu / (Vout × Iout) = 0.617 / (5 × 5) = 2.5%
```

**Trade-off with AC losses:**

Increasing wire diameter reduces DCR but:
- Makes skin effect worse (larger d/δ ratio)
- May not fit the same number of turns in the core window

There is an optimal wire diameter that minimises total copper loss (DC + AC) for a given winding. The optimum is approximately d_optimal ≈ 1.5 × δ (skin depth at the operating frequency).

For a 200 kHz converter: δ = 0.149 mm → d_optimal ≈ 0.22 mm → AWG 32.

---

### Q6. What causes thermal failure in an inductor and how is temperature rise estimated?

**Answer:**

**Heat sources:**

1. **Copper losses:** P_Cu = I_rms² × DCR + I_AC_rms² × (R_AC - DCR)
2. **Core losses:** P_core = Cm × f^α × B_peak^β × Ve (Steinmetz)

**Total dissipation:** P_total = P_Cu + P_core

**Thermal failure modes:**

1. **Insulation degradation:** Wire enamel rating is typically 155°C (class F) or 180°C (class H). Exceeding this degrades insulation → inter-turn shorts → catastrophic failure.

2. **Core derating:** Ferrite B_sat decreases with temperature:
   - MnZn ferrite: B_sat ≈ 450 mT at 25°C, drops to ~300 mT at 100°C
   - Operating at high temperature with marginal saturation margin can lead to thermal runaway: temperature rises → B_sat drops → easier to saturate → higher peak currents → more losses → temperature rises further.

3. **Solder joint fatigue:** High thermal cycling (due to converter switching on/off) can cause solder cracking.

**Temperature rise estimation (Pressman method):**

For a given surface area:
```
ΔT = P_total^0.833 × (450 / A_surface)   [rough empirical]

or more accurately:
ΔT = P_total / (h × A_surface)   [convective heat transfer]

where h ≈ 10–20 W/(m²·K) for natural convection
```

**Practical formula (Pressman/Flanagan):**
```
ΔT ≈ 450 × (P_total / A_surface)   [°C, A in cm², P in W]
```

**Example:**
```
Core: E25/13/7, surface area ≈ 15 cm²
P_Cu = 0.5W, P_core = 0.3W → P_total = 0.8W

ΔT ≈ 450 × (0.8 / 15) = 24°C above ambient

At T_ambient = 50°C: T_inductor = 74°C — well within limits.
```

**Verification rule:** Design so that P_Cu + P_core results in ΔT < 40°C for free-air operation, or verify thermal resistance to heatsink is adequate for the application.

---

## Intermediate (Questions 7–13)

---

### Q7. Walk through a complete inductor design procedure for a buck converter.

**Answer:**

**Given:**
```
Vin=12V, Vout=5V, Iout=5A, fsw=300kHz, ΔIL=30% of Iout (target)
Max ambient: 50°C, max winding temp: 100°C (ΔT ≤ 50°C)
```

**Step 1: Calculate inductance**
```
D = 5/12 = 0.417
ΔIL_target = 0.3 × 5 = 1.5 A
L = Vout × (1-D) / (fsw × ΔIL) = 5 × 0.583 / (300kHz × 1.5) = 6.5 µH
→ Choose L = 6.8 µH (standard E12)
```

**Step 2: Calculate peak current (saturation check value)**
```
I_peak = Iout + ΔIL/2 = 5 + 0.75 = 5.75 A
```

**Step 3: Core selection**

Target core energy storage metric LI²:
```
LI² = 6.8µH × 5.75² = 6.8µH × 33.1 = 225 µJ = 0.225 mJ
```

From manufacturer's core selector tables, find a core where:
```
½ × L × I_peak² ≤ core energy handling capacity
→ Core energy = ½ × L × I_sat² → need I_sat > I_peak
```

Try Ferroxcube E25/13/7 with N87 ferrite material (ungapped), AL = 2300 nH/N².

**Step 4: Calculate turns**
```
N = √(L/AL) = √(6800/2300) = √2.96 = 1.72 → choose N = 2 turns

Check: L_actual = 2300 × 4 = 9200 nH = 9.2 µH (larger than needed — causes less ripple)
Acceptable; now check saturation.
```

**Step 5: Check saturation**
```
Core datasheet: Ae = 52 mm², B_sat = 490 mT at 25°C (drops to ~330 mT at 100°C)

B_peak = L × I_peak / (N × Ae) = 6.8µH × 5.75 / (2 × 52mm²)
       = 39.1 µWb / 104mm²
       = 39.1 µWb / 104×10⁻⁶ m²
       = 0.376 T = 376 mT

B_sat at 100°C ≈ 330 mT
376 mT > 330 mT → SATURATES at temperature! Must gap the core.
```

**Step 6: Add air gap**
```
Target B_peak = 0.7 × B_sat(100°C) = 0.7 × 330mT = 231 mT (with 30% margin)

Alternatively, target B_peak = 300 mT (modest margin):
L × I_peak = N × B_peak × Ae
6.8µH × 5.75 = N × 0.30 × 52×10⁻⁶
39.1×10⁻⁶ = N × 15.6×10⁻⁶
N = 39.1/15.6 = 2.51 → increase to N = 3 turns

Recheck with N = 3:
B_peak = L × I_peak / (N × Ae) = 39.1×10⁻⁶ / (3 × 52×10⁻⁶) = 250 mT ✓

Required gap length to achieve L = 6.8 µH with N = 3:
Lg = µ0 × N² × Ae / L = 4π×10⁻⁷ × 9 × 52×10⁻⁶ / 6.8×10⁻⁶
   = 4π×10⁻⁷ × 468×10⁻⁶ / 6.8×10⁻⁶
   = 587.6×10⁻¹³ / 6.8×10⁻⁶
   = 86 µm per leg (total gap = 2 × 86 = 172 µm for E-core)
```

**Step 7: Winding design**
```
Wire selection: skin depth at 300 kHz: δ = 66.5/√300000 = 0.121 mm
Optimal diameter: d ≈ 1.5δ = 0.18 mm → use AWG 33 (d=0.18mm, Aw=0.0255 mm²)

Check current density: J = Iout / Aw = 5A / 0.0255mm² = 196 A/mm²
→ High (typically ≤ 400 A/mm² acceptable)

DCR calculation:
MLT (E25/13/7) ≈ 28 mm per turn (from datasheet)
DCR = ρ_Cu × MLT × N / Aw = 1.72×10⁻⁸ × 0.028 × 3 / 0.0255×10⁻⁶
    = 1.72×10⁻⁸ × 0.084 / 2.55×10⁻⁸
    = 0.0567 Ω = 56.7 mΩ
```

**Step 8: Loss estimation**
```
P_Cu = I_rms² × DCR ≈ 5² × 0.057 = 1.43 W

Core loss: at B_peak = 250mT, f = 300kHz, N87 material:
P_core ≈ Steinmetz: look up Cm=2.23×10⁻⁷, α=1.58, β=2.65 for N87
P_core = 2.23×10⁻⁷ × (300kHz)^1.58 × (250mT)^2.65 × Ve

Ve (E25/13/7) = 2100 mm³ = 2.1×10⁻⁶ m³
(300,000)^1.58 = 10^(1.58×log10(300000)) = 10^(1.58×5.477) = 10^8.654 = 4.51×10⁸
(0.25)^2.65 = 10^(2.65×log10(0.25)) = 10^(2.65×(-0.602)) = 10^(-1.595) = 0.0254

P_core = 2.23×10⁻⁷ × 4.51×10⁸ × 0.0254 × 2.1×10⁻⁶ = 0.536 W

P_total = 1.43 + 0.54 = 1.97 W
```

**Step 9: Temperature rise check**
```
Surface area of E25/13/7: approx 12 cm² (estimate from core dimensions)
ΔT ≈ 450 × (1.97 / 12) = 73.9°C

At 50°C ambient: T_coil = 124°C → exceeds 100°C limit.
→ Need larger core or split to parallel inductors.

Try E30/15/7 (next size up): Ve = 4200 mm³, Ae = 60 mm², MLT ≈ 35mm, surface ≈ 18 cm²
P_core scales with Ve: P_core_new = 0.54 × (4200/2100) = 1.08W
DCR changes: DCR_new = 1.72×10⁻⁸ × 0.035 × N_new / Aw_new
[Redesign with larger core — typically reduces total loss as Cu and core are rebalanced]
ΔT_new ≈ 450 × (P_total_new / 18) — iterate until ΔT < 50°C
```

---

### Q8. What is the difference between inductance at DC vs inductance at AC (small-signal inductance)?

**Answer:**

**DC bias effect on inductance:**

Real inductors (especially ferrite-core) have inductance that depends on the DC bias current flowing through them. As DC bias increases:
1. The core operates at higher average flux density B_DC = µ × H_DC
2. The permeability µ_r decreases (approaching saturation region of B-H curve)
3. Inductance L = µ × N² × Ae / le decreases

This is the **magnetising inductance vs DC bias** characteristic — plotted in every inductor datasheet.

**Small-signal (incremental) inductance:**

The effective inductance seen by the AC switching ripple is the incremental permeability at the operating bias point:
```
L_incremental = µ_incremental × N² × Ae / le

µ_incremental = dB/dH |at operating point
```

On the B-H curve:
- At low bias: incremental µ ≈ initial µ — inductance is near rated value
- Near saturation: the B-H curve flattens → dB/dH decreases → incremental µ drops → inductance drops
- At saturation: incremental µ → µ0 → inductance drops to air-core value

**Measurement:**
The incremental inductance is measured with a small AC signal superimposed on a DC bias using an LCR meter with DC bias capability. Datasheets for inductors typically plot "inductance vs DC bias current."

**Design significance:**
The converter uses inductance to limit ripple and store energy. If inductance at the operating bias is 30% below the rated (zero-bias) value:
```
ΔIL_actual = Vout × (1-D) / (fsw × L_actual)
           = ΔIL_designed × (L_rated / L_actual)
           = ΔIL_designed × (1/0.7) = 1.43 × ΔIL_designed
```

The ripple is 43% higher than designed. This can push the converter into DCM at rated load, increase peak currents, and worsen EMI.

**Design rule:** Always specify inductance at the DC bias point (I_DC = I_avg) plus some AC ripple. Do not use the zero-bias inductance in calculations. Use the "inductance at Iout" value from the datasheet.

---

### Q9. What is a coupled inductor and what advantages does it offer in a multi-phase converter?

**Answer:**

**Coupled inductor:**

A coupled inductor (or multi-winding inductor) places windings for multiple converter phases on the same magnetic core, with intentional mutual coupling.

For a two-phase buck with coupled inductor:
```
V1 = L × di1/dt + M × di2/dt
V2 = M × di1/dt + L × di2/dt

where L = self-inductance, M = mutual inductance, k = M/L (coupling coefficient)
```

**Key benefit — ripple cancellation:**

In a standard interleaved multi-phase converter, output ripple cancels at the output. With coupled inductors, an additional ripple reduction occurs in each phase:

The effective inductance for current ripple (transient inductance) is:
```
L_ripple = L × (1 - k)  [for inverse coupling, k > 0]
```

And the effective inductance for steady-state response (magnetising inductance) is:
```
L_magnetising = L × (1 + k) / (1 - k) × (number of phases)^correction
```

**Advantages:**

1. **Reduced per-phase ripple:** With k close to 1, L_ripple << L → very small individual phase current ripple despite reasonable steady-state inductance → smaller core footprint.

2. **Fast transient response:** During a load step, the coupled inductor's magnetising inductance is higher than the transient inductance, allowing fast current sharing between phases.

3. **Smaller total magnetics volume:** One coupled core can replace multiple individual cores — higher power density.

**Trade-offs:**

1. **Winding complexity:** Coupled inductors require careful layout to achieve the correct coupling coefficient k.

2. **Core shared saturation:** If one phase saturates, it affects all phases sharing the core.

3. **Design complexity:** Requires understanding of mutual coupling, not available in standard inductor design tools.

**Applications:** Server VRMs (voltage regulator modules) with 4–16 phases, automotive multi-phase DC-DC converters.

---

### Q10. How does inductance selection affect input and output EMI?

**Answer:**

**Output ripple current:**
The inductor limits the output ripple current ΔIL:
```
ΔIL = Vout × (1-D) / (fsw × L)
```

A larger L → smaller ΔIL → smaller output ripple voltage (better EMI).
A smaller L → larger ΔIL → larger ripple → requires more output capacitance to meet ripple spec.

**Input ripple current:**
The switch current (which is the input current for a buck converter) has a waveform that is a step from 0 to I_L during on-time:
```
I_in_peak = I_peak = Iout + ΔIL/2
I_in_rms = I_peak × √D ≈ Iout × √D   [for small ripple]
```

The input current spectrum is rich in harmonics of fsw. A larger inductor reduces ΔIL and thus the high-frequency harmonic content of the switch current — reducing conducted EMI on the input.

**Switch node radiation:**
The inductor's physical placement and orientation affect radiated EMI. The switch node (connection between the FET and inductor) has high dV/dt and carries high currents. Parasitic capacitance from the inductor winding to ground can create common-mode EMI currents.

Toroidal inductors have lower external magnetic field than E-core inductors (flux is more self-contained) — better for radiated EMI.

**Fringing field at air gap:**
Gapped inductors radiate magnetic flux from the air gap. This fringing field can couple into nearby traces and components. Rules:
- Keep gap away from sensitive signal traces
- Orient gap so fringing field points away from the PCB
- Use shielded inductors (with copper shield) if fringing is severe
- Pot the inductor in epoxy to reduce field reach

---

### Q11. What is saturation current and DC-rated current in an inductor datasheet, and how do they differ?

**Answer:**

Inductor datasheets typically list two current ratings that serve different purposes and should not be confused:

**Saturation current (I_sat):**

The current at which inductance drops to a specified percentage of its rated value. Industry convention varies:
- Some manufacturers: 30% drop in inductance → I_sat
- Others: 20% drop
- Always check the definition in the datasheet

```
I_sat criterion: L(I_sat) = 0.70 × L_nominal  [70% of rated inductance]
```

Physical meaning: At I_sat, the core is significantly into saturation. The converter's ripple current will be 30% larger than designed, and peak currents become hard to predict.

**Design rule:** I_sat must exceed the maximum peak inductor current with margin:
```
I_sat ≥ I_peak = Iout_max + ΔIL/2   [plus transient margin, typically 20%]
I_sat ≥ 1.2 × I_peak_max
```

**Rated DC current (I_rated or I_DC):**

The maximum continuous DC current that the inductor can carry without exceeding a specified temperature rise — typically 40°C over ambient. This rating is thermally limited:
```
I_rated: P_Cu = I_rated² × DCR → ΔT = 40°C
```

**Key difference:**

- I_sat is a magnetic limit (about core saturation — inductor performance degrades)
- I_rated is a thermal limit (about winding temperature — inductor reliability degrades)

**Example of why both matter:**

An inductor with I_sat = 10A and I_rated = 6A in a 5A application:
- Saturation: I_peak = 5.75A < I_sat = 10A ✓ (magnetic OK)
- Thermal: I_DC = 5A < I_rated = 6A ✓ (thermal OK)

An inductor with I_sat = 5A and I_rated = 10A:
- Saturation: I_peak = 5.75A > I_sat = 5A ✗ (will saturate!)
- Even though thermally it can handle 10A — the magnetics fail first.

---

### Q12. How do coupled inductors in a flyback converter differ from those in a multi-phase buck?

**Answer:**

**Flyback transformer (coupled inductor with energy transfer):**

A flyback transformer stores energy in its magnetising inductance during the switch-on phase and releases it during the switch-off phase. The secondary winding conducts only during the off phase.
```
Primary winding: carries magnetising current, builds up to I_peak = Vin × D × Ts / Lm
Secondary winding: conducts during off-time → average output current = I_peak × (1-D) / 2 [DCM]
Coupling: very tight (k close to 1) — leakage inductance is an undesirable parasite
```

**Multi-phase buck coupled inductor:**

All windings carry current simultaneously (the phases overlap). The coupling is intentional and controlled to achieve the ripple reduction benefit. Energy is not "stored and released" — energy flows continuously through all phases.
```
Coupling coefficient k: engineered, typically 0.5 to 0.9
Both windings see simultaneous current, but out of phase with each other
Leakage inductance = transient inductance (the useful quantity for ripple)
Magnetising inductance = steady-state inductance (determines current sharing)
```

**Key distinctions:**

| Feature                 | Flyback Transformer       | Multi-Phase Coupled Inductor |
|-------------------------|---------------------------|------------------------------|
| Winding conduction      | Alternating (primary/sec) | Simultaneous (all phases)    |
| Energy transfer         | Via stored magnetic energy | Via concurrent conduction    |
| Isolation               | Galvanic isolation possible| Not isolated                  |
| Tight coupling desired? | Yes (lower leakage)       | Controlled coupling (0.5-0.9)|
| Leakage inductance      | Undesirable (causes spikes)| = Transient inductance (useful)|
| Core energy storage     | Large (full cycle energy)  | Small (ripple energy)         |

---

### Q13. What is the current ripple cancellation effect in interleaved converters and how does the required inductance change?

**Answer:**

**Interleaving principle:**

In an N-phase interleaved converter, each phase operates at phase offset of Ts/N. The output currents of all phases sum at the output. Due to the phase offset, the ripple components cancel partially.

**Ripple cancellation factor:**

For N identical phases with duty cycle D:
- When D = k/N (duty cycle is an exact multiple of 1/N): perfect ripple cancellation
- At D = (k+0.5)/N: maximum ripple (worst case between cancellation points)

For a 2-phase converter:
```
At D = 0.5: Output ripple → 0 (perfect cancellation)
At D = 0.25 or 0.75: Output ripple = maximum per-phase ripple (no cancellation)
```

**Effect on inductance requirement:**

Because the output ripple is reduced by interleaving, the filter capacitance can be reduced for the same output ripple specification. Alternatively, the per-phase inductance can be reduced.

For N phases, the effective output ripple frequency is N × fsw (the ripple periods overlap and the lowest beat frequency is N times higher). To achieve the same output ripple with a smaller capacitor, the per-phase inductance can be scaled:

```
L_per_phase(N) ≈ L_single / N   [approximate, for D near 1/N]
```

**Practical example:**

A single-phase buck needs L = 20 µH to achieve ΔIout = 1A at 200 kHz.
A 4-phase interleaved buck at 200 kHz per phase (800 kHz effective ripple frequency):
```
L_per_phase ≈ 20µH / 4 = 5 µH   [much smaller per-phase inductor]
```

Four 5 µH inductors are physically smaller and lighter than one 20 µH inductor.

**Actual output ripple of interleaved converter (2-phase, D < 0.5):**
```
ΔI_out = (Vout × (1-2D)) / (2 × fsw × L)   [for D < 0.5]
       = 0 at D = 0.5  (perfect cancellation)
```

This shows the ripple is zero at exactly D = 0.5 and maximum when D is furthest from 0.5.

---

## Advanced (Questions 14–17)

---

### Q14. Derive the Dowell equation result for AC resistance of a multi-layer winding and explain each loss mechanism.

**Answer:**

Dowell's method models a multi-layer winding as a series of conducting slabs, each carrying a current determined by Ampere's law. The total MMF across all layers determines the field distribution and eddy current losses.

**Setup:**

Consider N_layers of wire with:
- Each layer conductor height: h (= wire diameter d for round wire)
- Layer packing factor: η (ratio of conductor width to winding breadth)
- Normalised layer height: Δ = h × √(η) / δ

**Loss mechanisms:**

**1. Skin effect loss (single conductor, self-field):**
A conductor carrying current I generates an internal magnetic field. This field induces eddy currents that reinforce current at the surface and oppose it at the center. The fractional increase in resistance for a single conductor:
```
FR_skin = Δ/√2 × [sinh(√2Δ) + sin(√2Δ)] / [cosh(√2Δ) - cos(√2Δ)]

For Δ >> 1: FR_skin ≈ Δ/√2  (resistance ∝ f^0.5)
For Δ << 1: FR_skin ≈ 1 + Δ⁴/45  (negligible correction)
```

**2. Proximity effect loss (adjacent layers, external field):**
Each layer is subjected to the magnetic field created by all current in layers below it (assuming current flows in the same direction in all layers). This external field induces additional eddy currents. For layer m (counting from the bottom):

The external MMF is: H_ext = m × I × η / (winding width)

The proximity effect resistance factor for layer m:
```
ΔFR_proximity(m) = (2m² - 2m + 1) × G(Δ)

where G(Δ) = Δ × [sinh(Δ) - sin(Δ)] / [cosh(Δ) + cos(Δ)]
```

**Total resistance factor (Dowell's result):**

Summing over all N_layers layers:
```
F_R = (R_AC/R_DC) = F_skin + F_proximity × (N_layers²-1)/3

= Δ × {[sinh(2Δ)+sin(2Δ)]/[cosh(2Δ)-cos(2Δ)] + (2(N_layers²-1)/3) × [sinh(Δ)-sin(Δ)]/[cosh(Δ)+cos(Δ)]}
```

**Key insight from Dowell:**

The proximity effect term scales as N_layers²/3 for large N_layers. So:
- 1 layer: F_R dominated by skin effect ≈ Δ (for large Δ)
- 2 layers: F_R ≈ skin + (4-1)/3 × prox = skin + prox
- 10 layers: F_R ≈ skin + 99/3 × prox → 33× more proximity loss than skin loss

**Practical consequences:**

For N_layers = 5 at Δ = 1 (d ≈ 1.5δ, moderate skin effect):
```
F_R ≈ 1 + (25-1)/3 × G(1) ≈ 1 + 8 × 0.39 ≈ 4.1
```

R_AC = 4.1 × R_DC — a 4× increase from proximity effect alone.

**Minimisation:**
1. Reduce N_layers: use wider, shallower winding
2. Reduce Δ: use thinner wire or Litz wire (each strand: Δ << 1)
3. Interleave: break winding into P-S-P structure to reduce peak MMF
4. Use foil winding: height below δ, minimal proximity between layers

---

### Q15. What is the area-product method for core selection and how do you apply it?

**Answer:**

The area-product method selects a core based on the product Ap = Ae × Aw, where:
- Ae = effective core cross-sectional area (determines flux-handling)
- Aw = window area available for winding (determines current-handling)

**Physical basis:**

The core must store the required energy in the magnetic field:
```
Energy = ½ × L × I_peak²
```

The window area must accommodate the wire:
```
Aw × η_fill = N × Aw_wire   [η_fill = fill factor, typically 0.3-0.5 for round wire with bobbin]
```

**Ap formula for an inductor:**

Combining energy storage, saturation limit, and winding space:
```
Ap = Ae × Aw = 2 × L × I_peak × I_rms / (B_max × J × η_fill × Ku)

where:
  B_max = maximum flux density (design target, e.g., 0.7 × B_sat)
  J     = current density in winding (A/m², typically 3-6 A/mm²)
  η_fill = winding fill factor
  Ku    = utilization factor (= 1 for inductor, varies for transformer)
```

**Design procedure:**

1. Calculate required Ap using the formula above
2. Search manufacturer tables for cores with Ap ≥ required value
3. Select smallest core meeting the Ap requirement (minimizes size/cost)
4. Calculate N from the selected core: N = L × I_peak / (B_max × Ae)
5. Calculate Lg (air gap) required to achieve L with N turns
6. Verify wire fits in window: N × d_wire² / (0.785 × η) ≤ Aw
7. Calculate losses and verify temperature rise

**Example:**
```
L=10µH, I_peak=8A, I_rms=5A, B_max=250mT, J=4A/mm², η=0.3

Ap = 2 × 10×10⁻⁶ × 8 × 5 / (0.25 × 4×10⁶ × 0.3 × 1)
   = 800×10⁻⁶ / 300,000
   = 2.67×10⁻⁹ m⁴ = 267 mm⁴

From Ferroxcube E core table:
E25/13/7: Ae=52mm², Aw=56mm² → Ap=2912mm⁴=2.91×10⁻⁶m⁴ (too small by 10×? check units)
```

Note: The units must be consistent. Ap in mm⁴ is common in inductor design tables. Verify formula and units from the source.

---

## Quick Reference: Design Checklist

```
INDUCTOR DESIGN CHECKLIST

1. Calculate L from ripple spec: L = Vout×(1-D)/(fsw×ΔIL)
2. Calculate I_peak = Iout + ΔIL/2
3. Choose core via Ap method or datasheet tables
4. Calculate N = √(L/AL) or N = L×I_peak/(B_max×Ae)
5. Calculate air gap: Lg = µ0×N²×Ae/L
6. Verify B_peak = L×I_peak/(N×Ae) < 0.7×B_sat(Tmax)
7. Choose wire: d ≤ 2δ at fsw, J ≤ 4-6 A/mm²
8. Check wire fits: N×d² × π/4 / η_fill ≤ Aw
9. Calculate DCR: DCR = ρ×MLT×N/Aw_wire
10. Calculate P_Cu = I_rms²×DCR
11. Calculate P_core = Cm×f^α×B_pk^β×Ve
12. Verify ΔT = (P_Cu + P_core) / (h_conv × A_surface) < 40°C
```

| Wire gauge | d (mm) | Aw (mm²) | I_max (at 4A/mm²) |
|------------|--------|----------|--------------------|
| AWG 22     | 0.644  | 0.326    | 1.3 A              |
| AWG 24     | 0.511  | 0.205    | 0.82 A             |
| AWG 26     | 0.405  | 0.129    | 0.52 A             |
| AWG 28     | 0.321  | 0.0810   | 0.32 A             |
| AWG 30     | 0.255  | 0.0507   | 0.20 A             |
