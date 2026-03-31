# Worked Problem 02 — Inductor Selection and Validation

## Problem Statement

A synchronous buck converter operates at:

| Parameter | Value |
|-----------|-------|
| Vin | 12 V |
| Vout | 5 V |
| Iout_max | 5 A |
| fsw | 400 kHz |
| Target ripple ratio r | 0.25 (25%) |
| Max case temperature | 85°C ambient |

Perform the following:
1. Calculate required inductance
2. Determine saturation current requirement
3. Estimate core size from the area product (Ap) method
4. Calculate DCR losses and temperature rise
5. Verify saturation margin under fault conditions
6. Compare two candidate inductors and justify final selection

---

## Step 1 — Calculate Required Inductance

**Duty cycle:**
```
D = Vout / Vin = 5 / 12 = 0.4167
```

**Target ripple current:**
```
ΔIL = r × Iout_max = 0.25 × 5 = 1.25 A
```

**Inductance equation (from volt-second balance):**
```
L = Vout × (1 - D) / (fsw × ΔIL)
  = 5 × (1 - 0.4167) / (400×10³ × 1.25)
  = 5 × 0.5833 / 500,000
  = 2.9165 / 500,000
  = 5.833 µH
```

**Select standard value: 6.8 µH** (next E12 value above 5.833 µH)

**Verify actual ripple with selected value:**
```
ΔIL_actual = Vout × (1-D) / (fsw × L)
           = 5 × 0.5833 / (400e3 × 6.8e-6)
           = 2.9165 / 2.72
           = 1.072 A   (r_actual = 1.072/5 = 21.4%)
```

The ripple is slightly below the 25% target (because we chose the next larger standard value). This is acceptable — lower ripple reduces capacitor stress.

**Check at worst-case D (minimum Vin = 10.8V):**
```
D_max = 5 / 10.8 = 0.463
ΔIL_max = 5 × (1-0.463) / (400e3 × 6.8e-6)
        = 5 × 0.537 / 2.72
        = 2.685 / 2.72
        = 0.987 A   (lower ripple at low Vin)
```

Maximum ripple occurs at the minimum duty cycle (maximum Vin):
```
D_min = 5 / 13.2 = 0.379
ΔIL_Vinmax = 5 × (1-0.379) / (400e3 × 6.8e-6)
           = 5 × 0.621 / 2.72
           = 3.105 / 2.72
           = 1.141 A
```

Peak current at worst case (max Vin, full load):
```
IL_peak_max = Iout_max + ΔIL_Vinmax/2 = 5 + 0.571 = 5.57 A
```

---

## Step 2 — Saturation Current Requirement

**Operating peak current (nominal Vin):**
```
IL_peak_nom = Iout_max + ΔIL_actual/2 = 5 + 0.536 = 5.54 A
```

**Derating margin for safety:**

The inductor's saturation current (Isat) must exceed the peak operating current with sufficient margin to account for:
- Component tolerances (inductance ±20% → ripple varies ±25%)
- Overload conditions before protection activates
- Temperature effects on core permeability (saturation reduces at high temperature)

**Recommended margin: 30–40% above operating peak:**
```
Isat_required = IL_peak_max / 0.70 = 5.57 / 0.70 = 7.96 A minimum
```

Use **Isat ≥ 8 A** as the selection criterion.

**Fault condition analysis:**

During output short circuit or startup (when Vout = 0), the inductor current can ramp at the maximum rate:
```
dIL/dt_max = Vin / L = 12 / 6.8e-6 = 1.76 A/µs
```

If the overcurrent protection (OCP) threshold is set at 8A and triggers within 500 ns (typical for valley current mode control), the peak fault current is:
```
IL_fault_peak = I_OCP + Vin/L × t_response = 8 + 1.76 × 0.5 = 8.88 A
```

This suggests either:
- Select Isat ≥ 9 A, or
- Verify the OCP response time and reduce it

For a robust design, select Isat ≥ 10 A.

---

## Step 3 — Core Size Estimation (Area Product Method)

The area product (Ap = Ae × Aw) is a figure of merit that relates the inductor's energy storage capability to its core size.

**Energy stored at peak current:**
```
E = 0.5 × L × IL_peak² = 0.5 × 6.8e-6 × 5.54² = 0.5 × 6.8e-6 × 30.7 = 104 µJ
```

**Area product estimation:**
```
Ap = Ae × Aw = (2 × E) / (Bmax × Jmax × ku)

where:
  Bmax = maximum flux density (typically 0.25–0.35 T for ferrite to stay below saturation)
  Jmax = current density in winding (typically 3–5 A/mm² for air-cooled inductors)
  ku   = window utilisation factor (typically 0.4 for single layer, 0.3 for multilayer)
```

Using Bmax = 0.30 T, Jmax = 4 A/mm², ku = 0.35:
```
Ap = (2 × 104e-6) / (0.30 × 4e6 × 0.35)
   = 208e-6 / 420,000
   = 4.95 × 10⁻¹⁰ m⁴
   = 0.495 cm⁴
```

This corresponds to approximately an EE25 or ETD29 core for a wound inductor, or a 5mm × 5mm SMD power inductor footprint.

**Practical implication:**

For standard SMD power inductors in power supply designs, use manufacturer selection tables:
- 5A class inductors typically fit in 5050 (5mm × 5mm) to 6045 (6mm × 6mm) packages
- 8-10A inductors need 8080 or larger, or purpose-designed high-current inductors

---

## Step 4 — DCR Loss and Temperature Rise

**Candidate inductors (6.8 µH, target Isat ≥ 10A):**

| Parameter | Inductor A | Inductor B |
|-----------|-----------|-----------|
| Inductance | 6.8 µH | 6.8 µH |
| Isat (40% drop) | 10.5 A | 9.8 A |
| Irms rated | 5.5 A | 6.2 A |
| DCR (typical) | 22 mΩ | 38 mΩ |
| DCR (max) | 28 mΩ | 47 mΩ |
| Package | 8.0 × 8.0 × 4.5 mm | 6.6 × 6.6 × 3.1 mm |
| Self-resonant freq | 18 MHz | 25 MHz |

**DCR loss calculation (using max DCR for worst case):**

Inductor A:
```
P_DCR_A = Iout_max² × DCR_max = 25 × 0.028 = 0.70 W
```

Inductor B:
```
P_DCR_B = Iout_max² × DCR_max = 25 × 0.047 = 1.175 W
```

**Temperature rise estimation:**

The surface temperature rise of an SMD inductor can be estimated from:
```
ΔT ≈ P_total / (θ_sa)
```
where θ_sa is the surface-to-ambient thermal resistance (from datasheet or empirical).

For Inductor A (8080 package, typical θ_sa ≈ 30°C/W with natural convection):
```
ΔT_A = 0.70 / (1/30) = 0.70 × 30 = 21°C
T_surface = 85 + 21 = 106°C  ← acceptable (rated to 125°C)
```

For Inductor B (6666 package, typical θ_sa ≈ 50°C/W):
```
ΔT_B = 1.175 × 50 = 58.75°C
T_surface = 85 + 58.75 = 143.75°C  ← too hot (exceeds 125°C rating)
```

Inductor B's smaller size leads to excessive temperature at full load.

**Temperature correction for DCR:**

DCR increases with temperature (copper TCR ≈ 0.393% per °C):
```
DCR_hot_A = DCR_25°C × (1 + 0.00393 × (Tj - 25))
           = 28 × (1 + 0.00393 × (106 - 25))
           = 28 × (1 + 0.318)
           = 28 × 1.318
           = 36.9 mΩ
```

Recalculate with hot DCR:
```
P_DCR_A_hot = 25 × 0.0369 = 0.923 W
ΔT_A_hot    = 0.923 × 30 = 27.7°C
T_surface   = 85 + 27.7 = 112.7°C  ← within 125°C rating (PASS with margin)
```

The iteration converges — no further correction required for Inductor A.

---

## Step 5 — Saturation Margin Verification

**Inductor A saturation check across operating conditions:**

| Condition | IL_peak | Isat_A | Margin |
|-----------|---------|--------|--------|
| Nominal (12V, 5A) | 5.54 A | 10.5 A | 1.90× |
| Worst ripple (13.2V, 5A) | 5.57 A | 10.5 A | 1.89× |
| Overload (OCP at 8A) | 8.44 A | 10.5 A | 1.24× |
| Short circuit (OCP + delay) | 8.88 A | 10.5 A | 1.18× |

The 1.18× margin during fault response is acceptable if OCP is reliably active — confirm with simulation. For automotive applications requiring AECQ-200, a minimum 1.3× saturation margin during fault is expected by most OEMs.

**Temperature effect on Isat:**

Ferrite core saturation flux density (Bsat) decreases with temperature:
- At 25°C: Bsat ≈ 0.45 T (typical MnZn ferrite)
- At 100°C: Bsat ≈ 0.35 T (approximately 22% reduction)

This means the effective Isat of a gapped ferrite inductor decreases at high temperature:
```
Isat_hot ≈ Isat_25°C × (Bsat_hot / Bsat_25°C)
         = 10.5 × (0.35 / 0.45)
         = 10.5 × 0.778
         = 8.17 A
```

Revised short-circuit margin: 8.17 / 8.88 = 0.92 — this means the inductor **could saturate** during a fault at high temperature.

**Corrective action:** Either:
1. Select an inductor with Isat_25°C ≥ 12 A (to ensure Isat_hot > 9.3 A), or
2. Reduce OCP threshold to 6A and verify fault current stay below 8.17 A at temperature, or
3. Use a powder core inductor (softer saturation, lower temperature sensitivity)

**Powder core advantage:** MPP, High Flux, or Kool Mµ powder cores have a much gentler saturation characteristic. Inductance decreases gradually with current rather than collapsing suddenly. This provides soft-saturation protection — the inductor naturally limits peak current rise rate.

---

## Step 6 — Final Component Selection and Comparison

**Decision matrix:**

| Criterion | Weight | Inductor A (8080) | Inductor B (6666) |
|-----------|--------|------------------|------------------|
| Isat margin (nominal) | 25% | 1.90× — Excellent | 1.76× — Good |
| Isat at temperature | 25% | 8.17A vs 8.88A — Marginal | Similar — Marginal |
| DCR loss | 20% | 0.70–0.92 W — Good | 1.18–1.55 W — Poor |
| Thermal (surface temp) | 20% | 112°C — OK | 144°C — FAIL |
| Size | 10% | 8080 — Larger | 6666 — Smaller |

**Verdict: Select Inductor A (8080 package)**

Inductor B fails the thermal check at full load in an 85°C ambient environment. The higher DCR caused by its smaller size results in excessive self-heating.

**Final design note:**

If board space is severely constrained, consider the following alternatives:
1. Increase switching frequency to 600 kHz → L can be reduced to 4.7 µH → smaller physical inductor
2. Use a custom wound inductor on an ETD29 core with litz wire for lower AC loss
3. Split into two-phase multiphase converter (each phase handles 2.5A → much smaller inductors)

---

## Summary of Results

| Parameter | Calculated Value | Selected Design |
|-----------|-----------------|-----------------|
| Required L | 5.83 µH | 6.8 µH |
| Actual ripple current | — | 1.07 A (21.4% ratio) |
| Peak inductor current | — | 5.57 A (worst case) |
| Required Isat | 8.0 A (min), 10 A (preferred) | 10.5 A (Inductor A) |
| DCR loss (hot) | — | 0.92 W |
| Surface temperature | — | 112°C at 85°C ambient |
| Isat at temperature | — | 8.17 A (marginal for faults) |
| Final recommendation | — | Inductor A; review OCP threshold |
