# Worked Problem 01: Inductor Design from Scratch for a Buck Converter

## Problem Statement

Design a complete inductor from scratch for the following buck converter. Verify saturation, calculate losses, and check temperature rise.

**Converter specifications:**
```
Vin    = 24 V
Vout   = 5 V
Iout   = 8 A  (maximum load)
fsw    = 300 kHz
D      = Vout/Vin = 5/24 = 0.208
ΔIL    = 30% of Iout = 2.4 A (target peak-to-peak ripple)
Tamb   = 50°C
ΔT_max = 40°C (maximum temperature rise above ambient)
```

**Available core:** Ferroxcube E25/13/7, N87 material
```
Ae     = 52.0 mm²   (effective cross-sectional area)
Ve     = 2990 mm³   (effective core volume)
le     = 58 mm      (effective magnetic path length)
Aw     = 56.0 mm²   (winding window area)
MLT    = 28 mm      (mean length per turn — approximate for this core)
AL     = 2300 nH/N² (ungapped, from datasheet)
µi     = 2200       (initial relative permeability)
B_sat  = 490 mT at 25°C, 390 mT at 100°C (TDK N87 datasheet)
```

**Available wire:** Use AWG 24 (d = 0.511 mm, Aw_wire = 0.205 mm²)

---

## Step 1: Calculate Required Inductance

```
L = Vout × (1-D) / (fsw × ΔIL)
  = 5 × (1 - 0.208) / (300,000 × 2.4)
  = 5 × 0.792 / 720,000
  = 3.96 / 720,000
  = 5.5 µH

Choose L = 5.6 µH (nearest E12 standard value)
```

**Verify resulting ripple with L = 5.6 µH:**
```
ΔIL_actual = Vout × (1-D) / (fsw × L)
           = 5 × 0.792 / (300,000 × 5.6×10⁻⁶)
           = 3.96 / 1.68
           = 2.36 A  (ripple ratio = 2.36/8 = 29.5% — close to 30% target ✓)
```

---

## Step 2: Calculate Peak Current

```
I_peak = Iout_max + ΔIL_actual/2
       = 8 + 2.36/2
       = 8 + 1.18
       = 9.18 A
```

This is the current that must not saturate the core.

---

## Step 3: Calculate Number of Turns

**Initial calculation from AL (ungapped):**
```
N_ungapped = √(L / AL) = √(5600 nH / 2300 nH/N²) = √2.435 = 1.56 turns → round to 2
```

N = 2 is too few turns to have design flexibility. Let us verify B at N=2:
```
B_peak = L × I_peak / (N × Ae) = 5.6×10⁻⁶ × 9.18 / (2 × 52×10⁻⁶)
       = 51.4×10⁻⁶ / 104×10⁻⁶
       = 0.494 T = 494 mT

B_sat at 100°C = 390 mT → 494 mT >> 390 mT → SATURATES. Must gap and add turns.
```

**Design with more turns to reduce B:**

Target B_peak ≤ 0.7 × B_sat(100°C) = 0.7 × 390 mT = 273 mT (with 30% margin)

Required N to achieve B_peak ≤ 273 mT:
```
N ≥ L × I_peak / (B_max × Ae)
  = 5.6×10⁻⁶ × 9.18 / (273×10⁻³ × 52×10⁻⁶)
  = 51.4×10⁻⁶ / 14.2×10⁻⁶
  = 3.62 → N = 4 gives 247 mT; choose N = 5 turns for extra margin
```

**Verify B_peak with N = 5:**
```
B_peak = L × I_peak / (N × Ae)
       = 5.6×10⁻⁶ × 9.18 / (5 × 52×10⁻⁶)
       = 51.4×10⁻⁶ / 260×10⁻⁶
       = 0.198 T = 198 mT

198 mT < 273 mT ✓  (27% below limit — comfortable margin)
198 mT / 390 mT = 51% of B_sat at 100°C — good design point
```

---

## Step 4: Calculate Required Air Gap

With N = 5 turns and target L = 5.6 µH, the required AL is:
```
AL_required = L / N² = 5,600 nH / 25 = 224 nH/N²

This is much less than the ungapped AL = 2300 nH/N², so a significant gap is needed.
```

**Air gap length calculation:**

For a gapped ferrite core (gap >> le/µi):
```
Lg = µ0 × N² × Ae / L   [simplified formula]
   = 4π×10⁻⁷ × 25 × 52×10⁻⁶ / 5.6×10⁻⁶
   = 1.636×10⁻⁹ / 5.6×10⁻⁶
   = 292×10⁻⁶ m = 292 µm total gap

For an E-core with two gap sections: gap per side = 292/2 = 146 µm
```

**Verify AL with gap:**
```
AL_gapped = µ0 × Ae / Lg = 4π×10⁻⁷ × 52×10⁻⁶ / 292×10⁻⁶
          = 65.3×10⁻¹² / 292×10⁻⁶
          = 224 nH/N² ✓ (matches required value)
```

**Verify L:**
```
L = AL_gapped × N² = 224 × 25 = 5600 nH = 5.6 µH ✓
```

**Practical note on gap:** 146 µm per gap section is achievable with:
- Precision-ground gap on ferrite halves (manufacturer can grind to spec ± 10%)
- Plastic or Mylar tape spacer (Mylar tape: 50–127 µm per layer — use 2-3 layers)
- Ordering gapped cores from manufacturer (specify AL = 225 nH/N², they control the gap)

---

## Step 5: Winding Design

**Wire selection (AWG 24 as specified):**
```
d_wire = 0.511 mm
Aw_wire = 0.205 mm²
```

**Skin depth check at 300 kHz:**
```
δ = 66.5 / √(300,000) = 66.5 / 547.7 = 0.121 mm
d_wire / (2δ) = 0.511 / (2 × 0.121) = 2.11

d_wire / (2δ) = 2.11 > 1  → significant skin effect
```

For better high-frequency performance, AWG 26 (d=0.405mm) or Litz wire would be preferred. For this example, continue with AWG 24 and calculate the AC resistance factor.

**Winding window check:**

With N = 5 turns, single layer, the winding occupies:
```
Window area used = N × π × (d_wire/2)² / η_fill  (with fill factor η_fill ≈ 0.5 for single layer)

More simply: N × π × d_wire² / 4 = 5 × π × 0.511² / 4 = 5 × 0.205 = 1.025 mm²

Available window: Aw = 56 mm²
Utilisation = 1.025/56 = 1.8% — massively underutilised

This allows using much thicker wire for lower DCR.
Alternative: AWG 14 (d=1.63mm, Aw=2.08mm²), 5 turns:
Utilisation = 5 × 2.08/56 = 18.6% (still OK for single layer)
```

**Decision: Use AWG 14 for lower DCR:**

AWG 14: d = 1.628 mm, Aw_wire = 2.081 mm²
```
Skin depth check: d_wire/(2δ) = 1.628/(2×0.121) = 6.73 >> 1 → severe skin effect
→ For 5 turns single layer with thick wire, skin effect causes R_AC >> R_DC
→ Use Litz wire of equivalent cross-section for good HF performance
```

**Compromise: Use AWG 18 (d=1.024mm, Aw=0.823mm²), or bundle of AWG 26:**

For this worked example, use a bundle of 4 × AWG 26 (d=0.405mm each) in parallel:
```
Equivalent diameter: 4 strands, Aw_total = 4 × 0.129 = 0.516 mm²
d_strand = 0.405 mm → d_strand/(2δ) = 0.405/(2×0.121) = 1.67 (still some skin effect)
```

Proceed with AWG 26 × 4 in parallel for the calculation:

**DCR (one strand of AWG 26):**
```
DCR_strand = ρ × MLT × N / Aw_wire_one = 1.72×10⁻⁸ × 0.028 × 5 / 0.129×10⁻⁶
           = 2.408×10⁻⁹ / 0.129×10⁻⁶ = 18.7 mΩ

With 4 strands in parallel: DCR_total = 18.7 / 4 = 4.67 mΩ
```

---

## Step 6: Loss Calculation

**DC copper loss:**
```
I_rms ≈ Iout = 8A (for small ripple)
More precisely: I_rms = Iout × √(1 + (ΔIL/(Iout×2√3))²) = 8 × √(1 + (2.36/27.7)²) ≈ 8.03A

P_Cu_DC = I_rms² × DCR = 64.5 × 0.00467 = 0.301 W
```

**AC copper loss:**

For a 1-layer winding of AWG 26 (d=0.405mm) at 300 kHz:
```
Δ = (d_wire/δ) × √(fill_factor) = (0.405/0.121) × √0.5 = 3.35 × 0.707 = 2.37

Dowell F_R for 1 layer (N_layers = 1):
F_R = Δ × [sinh(2Δ)+sin(2Δ)] / [cosh(2Δ)-cos(2Δ)]
    = 2.37 × [sinh(4.74)+sin(4.74)] / [cosh(4.74)-cos(4.74)]
    = 2.37 × [57.21+(-1.000)] / [57.22-(0.028)]
    = 2.37 × 56.21 / 57.19
    = 2.37 × 0.983
    = 2.33
```

AC ripple current RMS value:
```
I_AC_rms = ΔIL / (2√3) = 2.36 / 3.464 = 0.681 A

P_Cu_AC = (F_R - 1) × DCR × I_AC_rms² = (2.33 - 1) × 0.00467 × 0.681²
        = 1.33 × 0.00467 × 0.464
        = 2.88 mW  (negligible compared to DC loss)
```

**Total copper loss:**
```
P_Cu = 0.301 + 0.003 = 0.304 W
```

**Core loss:**

Using iGSE for triangular waveform:
```
feq = (2 × fsw/π²) / (D × (1-D))
    = (2 × 300,000/9.87) / (0.208 × 0.792)
    = 60,791 / 0.165
    = 368,800 Hz

ΔB = 2 × B_peak_ripple = 2 × L × (ΔIL/2) / (N × Ae)
   Wait — let us use the flux swing directly:
ΔB = (Vin - Vout) × D × Ts / (N × Ae) = 19 × 0.208 × 3.33µs / (5 × 52×10⁻⁶)
   = 13.15×10⁻⁶ / 260×10⁻⁶ = 0.0506 T = 50.6 mT

N87 at 100°C: Cm = 2.23×10⁻⁷, α = 1.58, β = 2.65

The Steinmetz form uses the peak flux density B̂ = ΔB/2 = 25.3 mT (not the peak-to-peak swing):

Pv = Cm × feq^(α-1) × B̂^β × fsw
   = 2.23×10⁻⁷ × (368,800)^0.58 × (0.0253)^2.65 × 300,000

(368,800)^0.58 = 10^(0.58 × log10(368800)) = 10^(0.58 × 5.567) = 10^3.229 = 1695

(0.0253)^2.65 = 10^(2.65 × log10(0.0253)) = 10^(2.65 × (-1.597)) = 10^(-4.232) = 5.86×10⁻⁵

Pv = 2.23×10⁻⁷ × 1695 × 5.86×10⁻⁵ × 300,000
   = 2.23×10⁻⁷ × 1695 × 17.6
   = 0.0066 W/cm³ = 6.6 mW/cm³

P_core = Pv × Ve = 6.6 × 2.99 = 19.9 mW = 0.020 W   (the full iGSE integral gives 19 mW)
```

**Total power dissipation:**
```
P_total = P_Cu + P_core = 0.304 + 0.020 = 0.324 W

Loss split: Cu = 94%, Core = 6%
```

---

## Step 7: Temperature Rise Verification

**Surface area estimation for E25/13/7:**
```
Core dimensions: approximately 25mm × 13mm × 7mm
Winding surface area (exposed surfaces):
  Two end faces: 2 × (25 × 7) = 350 mm² = 3.5 cm²
  Four sides of the wound core: 4 × (25/2 × 7) ≈ 350 mm² = 3.5 cm² (rough estimate)
  Total surface area ≈ 7–12 cm² (typical for E25 wound component)

Use A_surface ≈ 9 cm²  (intermediate estimate)
```

**Temperature rise (Pressman formula):**
```
ΔT ≈ 450 × P_total / A_surface = 450 × 0.324 / 9 = 16.2°C
```

**Operating temperature:**
```
T_coil = T_ambient + ΔT = 50°C + 16.2°C = 66.2°C

66.2°C < 100°C (maximum inductor temperature) ✓
```

Verify B_sat margin at 66°C (interpolating between 25°C and 100°C values):
```
B_sat(66°C) ≈ 490 - (490-390) × (66-25)/(100-25) = 490 - 100 × 0.55 = 490 - 55 = 435 mT

B_peak = 198 mT << 435 mT ✓  (substantial margin)
```

---

## Step 8: Complete Design Summary

| Parameter       | Value             | Notes                                |
|-----------------|-------------------|--------------------------------------|
| Inductance      | 5.6 µH            | Standard E12 value                   |
| Core            | E25/13/7, N87     | Ferroxcube or equivalent             |
| Air gap (total) | 292 µm            | 146 µm per leg; specify AL=224 nH/N² |
| Turns           | 5                 | N = 5                                |
| Wire            | 4 × AWG 26 parallel| Bundle of 4 strands                 |
| DCR             | 4.67 mΩ           | At 20°C                              |
| DCR at 66°C     | 4.67 × 1.181 = 5.52 mΩ | ρ increases 18.1% from 20° to 66°C|
| B_peak          | 198 mT            | At 66°C operating point              |
| B_sat margin    | 198/435 = 46%     | 46% of B_sat — good margin           |
| P_Cu            | 304 mW            | Dominates loss                       |
| P_core          | 20 mW             | iGSE at 100°C parameters             |
| P_total         | 324 mW            |                                      |
| Temperature rise| 16.2°C            | Well below 40°C target ✓             |
| Operating temp  | 66.2°C            | Within insulation limits ✓           |

---

## Step 9: What-If Analysis

**If switching frequency were doubled to 600 kHz:**
```
L_new = Vout × (1-D) / (fsw_new × ΔIL_same) = 5 × 0.792 / (600k × 2.4) = 2.75 µH

B_peak_new = L_new × I_peak / (N × Ae) = 2.75×10⁻⁶ × 9.18 / (5 × 52×10⁻⁶) = 97 mT ✓

Core loss scales with f^α × ΔB^β:
New feq scales with fsw → doubles
New ΔB halves
P_core_new/P_core = 2^(α) × 0.5^β = 2^1.58 × 0.5^2.65
                  = 2.99 × 0.157 = 0.47  (47% of original)
P_core_new = 0.020 × 0.47 = 0.009 W

But DC copper loss is the same (same Iout) — core loss drops with frequency.
Total P_new ≈ 0.304 + 0.009 = 0.313 W  (slight improvement)

However: AC copper loss increases (higher frequency → more skin/proximity loss).
Need Litz wire or smaller strand diameter for 600 kHz operation.
```

**If inductance were undersized (L = 2.75 µH, ΔIL = 60%):**
```
I_peak = 8 + 0.6×8/2 = 10.4 A
B_peak = 2.75×10⁻⁶ × 10.4 / (5 × 52×10⁻⁶) = 110 mT (lower peak B)
I_AC_rms = 4.8/(2√3) = 1.39 A
P_Cu_AC = (2.33-1) × 0.00467 × 1.39² = 12.0 mW (still small but 4× higher)
P_core unchanged (ΔB is set by the volt-seconds and N, not by L) and P_Cu about same → similar total

But output capacitor must handle 2× more ripple current: capacitor ESR limits output voltage ripple.
Also: converter enters DCM at light loads → different operating mode.
```
