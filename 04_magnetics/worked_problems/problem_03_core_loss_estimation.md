# Worked Problem 03: Core Loss Estimation Using the Steinmetz Equation

## Problem Statement

Estimate the core losses for an inductor in a boost PFC converter using the Steinmetz equation. Compare two core material candidates, analyse thermal implications, and determine which material is preferable.

**Converter specifications:**
```
Topology:  Boost PFC
Vin_range: 85-265 Vac (rectified: 120-375 V DC)
Vout:      400 V DC
Pout:      300 W
fsw:       65 kHz
η:         93% (estimated)
D:         Variable (0.05 at Vin=375V to 0.7 at Vin=120V, see below)
```

**Inductor specifications:**
```
L = 500 µH (chosen for 20% ripple at full power, minimum Vin)
I_avg_pk = 4.0 A (peak average current at minimum Vin, maximum power)
I_pk = 4.0 × 1.1 = 4.4 A  (including 20% ripple, half-ripple added)
ΔIL_max = 0.8 A  (20% of 4.0A, peak-to-peak)
```

**Two candidate cores (toroid):**

**Core A: Kool Mµ 60µ powder toroid (illustrative 77 mm OD size — not a specific catalogue part)**
```
OD = 77.0 mm, ID = 48.0 mm, HT = 29.0 mm
Ve = 78.1 cm³, Ae = 5.22 cm²
AL = 157 nH/N² (ungapped, permeability = 60µ)
µi = 60
B_sat = 1.0 T (soft saturation)
```

**Core B: Sendust (Kool Mµ-like) equivalent — use same core geometry but N87-grade ferrite (for comparison):**

Actually, for fair comparison, use:

**Core B: Fair-Rite #77 MnZn ferrite toroid (equivalent outer dimensions)**
```
Ve = 78 cm³, Ae = 5.0 cm²
AL = 900 nH/N²
µi = 2000
B_sat = 0.49 T at 25°C, 0.35 T at 100°C
```

**Steinmetz parameters:**

Kool Mµ (60µ) at room temperature:
```
Cm_KM  = 3.49×10⁻³  (W·s^α·T^(-β)/cm³)
α_KM   = 1.46
β_KM   = 2.00
```

N87 MnZn ferrite at 100°C (loss minimum):
```
Cm_N87 = 1.90×10⁻⁴   (W·s^α·T^(-β)/cm³)
α_N87  = 1.58
β_N87  = 2.65
```

(Parameters are illustrative, consistent with published data trends.)

---

## Part A: Determine Operating Flux Density

**Inductor current ripple (worst case: minimum Vin = 120 V DC):**

At minimum DC input voltage (peak of 85Vac): Vin_DC_min ≈ 120V (after rectification and drop)
```
D_max = 1 - Vin_DC_min/Vout = 1 - 120/400 = 0.70

ΔIL = Vin_DC_min × D_max / (fsw × L) = 120 × 0.70 / (65,000 × 500×10⁻⁶)
    = 84 / 32.5 = 2.585 A

This exceeds the target of 0.8 A — our L = 500µH assumption gives more ripple at low Vin.
```

Let us use the correct ΔIL at the nominal operating point for loss calculation:
```
At Vin = 230 Vac (nominal):
  Vin_DC = 230 × √2 × 0.9 = 293 V (after bridge and filter)
  D = 1 - 293/400 = 0.268

ΔIL = 293 × 0.268 / (65,000 × 500×10⁻⁶) = 78.5 / 32.5 = 2.415 A

Still larger than 0.8A — the 500µH gives 2.4A ripple at nominal conditions.
(For 0.8A ripple at Vin=293V: L_required = 293×0.268/(65k×0.8) = 1.51mH)

For this problem, use L=500µH and work with the actual ΔIL.
```

**AC flux density swing (peak-to-peak):**

**For Kool Mµ (Core A), determine N first:**

Target: I_pk × N ≤ H_sat × le [keep B < B_sat with margin]
With µi = 60 and B_sat = 1.0T:
H_sat = B_sat / (µ0 × µi) = 1.0 / (4π×10⁻⁷ × 60) = 13,263 A/m

Effective path length for this toroid: le ≈ 2π × (OD+ID)/4 = 2π × (77+48)/8mm = 2π × 15.625mm × 10⁻³ = 0.0982m
Actually for a toroid: le = π × (OD + ID) / 2 = π × (77+48)/2 × 10⁻³ = π × 62.5mm = 196.3mm

Actually: le_effective for round cross-section toroid = 2π × r_c where r_c = mean radius
r_c = (OD + ID) / 4 = (77 + 48) / 4 mm = 31.25 mm
le = 2π × 31.25mm = 196.4 mm ≈ 196 mm

H_sat × le = 13,263 × 0.196 = 2599 A·turns
N × I_pk ≤ 2599 → N ≤ 2599/4.4 = 591 turns

Required N from inductance:
N = √(L / AL) = √(500,000 / 157) = √3185 = 56.4 → N = 57 turns

N × I_pk = 57 × 4.4 = 250.8 A·turns << 2599 A·turns limit ✓

Peak flux density:
B_pk = µ0 × µi × N × I_pk / le = 4π×10⁻⁷ × 60 × 57 × 4.4 / 0.196
     = 4π×10⁻⁷ × 60 × 250.8 / 0.196
     = 4π×10⁻⁷ × 15,048 / 0.196  (wait — let me redo cleanly)

Alternatively: B_pk = L × I_pk / (N × Ae)
= 500×10⁻⁶ × 4.4 / (57 × 5.22×10⁻⁴)
= 2200×10⁻⁶ / (29.75×10⁻³)
= 73.9 mT  [DC bias B_pk]

Peak-to-peak AC flux:
ΔB = L × ΔIL / (N × Ae)
   = 500×10⁻⁶ × 2.415 / (57 × 5.22×10⁻⁴)
   = 1207.5×10⁻⁶ / 29.75×10⁻³
   = 40.6 mT  peak-to-peak

B_half_swing = ΔB/2 = 20.3 mT  (this is what goes into Steinmetz as B_peak for the AC ripple)
```

---

## Part B: Core Loss Calculation — Kool Mµ (Core A)

**Using classical Steinmetz for sinusoidal approximation:**
```
Pv = Cm_KM × f^α × B_half^β
   = 3.49×10⁻³ × (65,000)^1.46 × (0.0203)^2.00
   [B_half = 20.3mT = 0.0203 T]

(65,000)^1.46 = e^(1.46 × ln(65000)) = e^(1.46 × 11.08) = e^16.18 = 1.07×10⁷

(0.0203)^2.00 = 4.12×10⁻⁴

Pv = 3.49×10⁻³ × 1.07×10⁷ × 4.12×10⁻⁴
   = 3.49×10⁻³ × 4408
   = 15.4 mW/cm³
```

**Apply iGSE correction for triangular waveform:**

For a buck/boost converter with D = 0.268 at nominal operating point:
```
feq = (2 × fsw/π²) / (D × (1-D))
    = (2 × 65,000/9.87) / (0.268 × 0.732)
    = 13,172 / 0.196
    = 67,200 Hz

Pv_iGSE = Cm_KM × feq^(α-1) × B_half^β × fsw
         = 3.49×10⁻³ × (67,200)^0.46 × (0.0203)^2.00 × 65,000

(67,200)^0.46 = e^(0.46 × ln(67200)) = e^(0.46 × 11.115) = e^5.113 = 165.9

(0.0203)^2.00 = 4.12×10⁻⁴

Pv_iGSE = 3.49×10⁻³ × 165.9 × 4.12×10⁻⁴ × 65,000
         = 3.49×10⁻³ × 165.9 × 26.8
         = 3.49×10⁻³ × 4,444
         = 15.5 mW/cm³

Note: this equivalent-frequency form, like classical Steinmetz, takes the half-swing B_half = ΔB/2 (the Steinmetz parameters are fitted to sinusoidal peak flux).
At D = 0.268, feq (67.2 kHz) is close to fsw, so the waveform correction is small here — the result matches the sinusoidal estimate.
```

**Total core loss — Core A (Kool Mµ):**
```
P_core_A = Pv × Ve = 15.5 mW/cm³ × 78.1 cm³ = 1.21 W
```

---

## Part C: Core Loss Calculation — N87 Ferrite (Core B)

**Determine N for N87 ferrite:**

N87 has AL = 900 nH/N² (for this core size approximation):

But wait — N87 has B_sat = 350 mT at 100°C. Peak DC bias B_pk = 73.9 mT is well within limits. The ferrite can accommodate this field.

However, N87 in a toroid form does not have a practical air gap. Let us assume the ferrite toroid is available as a gapped toroid (or that we wind it as an ungapped inductor — but then AL = 900 nH/N² applies):

```
N = √(L/AL) = √(500,000/900) = √556 = 23.6 → N = 24 turns

B_pk = L × I_pk / (N × Ae)
     = 500×10⁻⁶ × 4.4 / (24 × 5.0×10⁻⁴)
     = 2200×10⁻⁶ / 12×10⁻³
     = 183 mT   [DC bias]

183 mT > 350 mT at 100°C? No: 183 mT < 350 mT ✓ (only 52% of B_sat — OK)

ΔB = L × ΔIL / (N × Ae) = 500×10⁻⁶ × 2.415 / (24 × 5.0×10⁻⁴)
   = 1207.5×10⁻⁶ / 12×10⁻³ = 100.6 mT  peak-to-peak

B_half_swing = 50.3 mT
```

**Note:** High DC bias for N87 ferrite. At 183 mT DC + 50 mT AC = 233 mT peak. This is within B_sat but is getting high. The DC bias effect on N87 loss needs to be considered (Steinmetz does not account for it).

**Core loss — N87 at 100°C:**
```
feq = 67,200 Hz  (same as before, same D)

Pv_iGSE = Cm_N87 × feq^(α-1) × B_half^β × fsw
         = 1.90×10⁻⁴ × (67,200)^0.58 × (0.0503)^2.65 × 65,000

(67,200)^0.58 = e^(0.58 × 11.115) = e^6.447 = 630.9

(0.0503)^2.65 = e^(2.65 × ln(0.0503)) = e^(2.65 × (-2.990)) = e^(-7.923) = 3.62×10⁻⁴

Pv_iGSE = 1.90×10⁻⁴ × 630.9 × 3.62×10⁻⁴ × 65,000
         = 1.90×10⁻⁴ × 630.9 × 23.5
         = 1.90×10⁻⁴ × 14,860
         = 2.8 mW/cm³

P_core_B = 2.8 × 78 = 220 mW = 0.22 W
```

---

## Part D: Comparison and Material Selection

**Summary of core losses:**

| Metric              | Core A (Kool Mµ) | Core B (N87 Ferrite) |
|---------------------|-----------------|----------------------|
| N (turns)           | 57              | 24                   |
| B_DC_bias           | 73.9 mT         | 183 mT               |
| ΔB (peak-to-peak)   | 40.6 mT         | 100.6 mT             |
| Pv (iGSE)           | 15.5 mW/cm³     | 2.8 mW/cm³           |
| P_core              | 1.21 W          | 0.22 W               |
| P_core / Pout       | 0.40%           | 0.07%                |

**Why N87 ferrite wins on core loss:**
- N87 has much lower Steinmetz Cm and steeper β → losses drop faster with lower ΔB
- N87's ΔB is larger in absolute terms (100.6 vs 40.6 mT) because fewer turns
- But N87 has lower Cm, so the loss is still lower

**Copper loss comparison:**

N87 has fewer turns (24 vs 57) → less wire → lower DCR.
```
Assume same MLT (toroid of similar dimensions):
MLT ≈ 2 × (OD - ID) / π + 2 × HT ≈ 2 × (77-48)/π + 2×29 ≈ 18.5 + 58 = 76.5 mm (approximate)

DCR_A (Kool Mµ, N=57): DCR = ρ × MLT × N / Aw
  Use AWG 18 (d=1.024mm, Aw=0.823mm²):
  DCR = 1.72×10⁻⁸ × 0.0765 × 57 / 0.823×10⁻⁶ = 74.95×10⁻⁹ / 0.823×10⁻⁶ = 91.1 mΩ
  P_Cu_A = I_avg² × DCR = 4.0² × 0.0911 = 1.46 W

DCR_B (N87, N=24): Use AWG 14 (d=1.628mm, Aw=2.081mm²):
  DCR = 1.72×10⁻⁸ × 0.0765 × 24 / 2.081×10⁻⁶ = 31.57×10⁻⁹ / 2.081×10⁻⁶ = 15.2 mΩ
  P_Cu_B = 4.0² × 0.0152 = 0.243 W
```

**Total losses:**

| Metric          | Core A (Kool Mµ) | Core B (N87 Ferrite) |
|-----------------|-----------------|----------------------|
| P_core          | 1.21 W          | 0.22 W               |
| P_Cu            | 1.46 W          | 0.24 W               |
| P_total         | 2.67 W          | 0.46 W               |
| η_impact        | 0.89%           | 0.15%                |

**N87 ferrite is clearly superior** in total losses at 65 kHz for this application. However:

**Reasons why Kool Mµ is still commonly used:**
1. **B_sat with DC bias:** At 183 mT DC bias, N87 inductance starts to decline (approaching saturation zone of the B-H curve). At 233 mT peak, N87 is working hard. At high line transient, I_peak could exceed 5–6A → B_peak could reach 280 mT → getting close to 350 mT B_sat at 100°C.
2. **Saturation protection:** A PFC converter can experience high currents on startup, at voltage dip, or during fault conditions. Kool Mµ's soft saturation allows some overcurrent tolerance. N87 would hard-saturate.
3. **Temperature margin:** N87 B_sat drops more sharply with temperature than Kool Mµ material.
4. **Gap in ferrite:** Gapping a ferrite toroid requires cutting it (difficult, brittle) or ordering a gapped core. Powder cores have integrated distributed gap.

**Practical recommendation:**
For a production design targeting highest efficiency with robust saturation behaviour: use Kool Mµ with careful design for the operating current range. For a lab demonstration or high-efficiency target at fixed load: N87 ferrite would achieve better efficiency numbers.

---

## Part E: Thermal Implications

**Core A (Kool Mµ) at P_total = 2.67 W:**
```
Surface area of toroid ≈ π × OD × HT + π/4 × (OD² - ID²) × 2
≈ π × 77×29 + π/4 × (77²-48²) × 2 mm²
≈ 7021 + π/4 × (5929-2304) × 2 mm²
≈ 7021 + 5695 mm² = 12,716 mm² = 127.2 cm²

ΔT ≈ 450 × P_total / A_surface = 450 × 2.67 / 127.2 = 9.4°C

At 50°C ambient: T_core = 59.4°C — acceptable (within Kool Mµ capabilities)
```

**Core B (N87 ferrite) at P_total = 0.46 W:**
```
ΔT ≈ 450 × 0.46 / 127.2 = 1.6°C

T_core = 51.6°C — excellent. Well within N87's optimal temperature range (80-100°C for minimum loss).

At only 51.6°C, the N87 is actually running somewhat cool — its core loss is slightly higher than at 100°C (N87 has a loss minimum around 80-100°C). But the overall losses are still much lower than Kool Mµ.
```

---

## Part F: Waveform Complexity — Accounting for Ripple at Different Vin

The PFC inductor current ripple varies with instantaneous Vin (which follows the rectified sinusoid):
```
ΔIL(t) = Vin(t) × (1 - Vin(t)/Vout) / (fsw × L)

This varies from near-zero (at zero crossing of Vin) to maximum (near Vin_peak for this formula's peak)
```

The core loss should be integrated over one AC cycle for accurate total loss:
```
P_core_avg = (1/π) × ∫₀^π Pv(ΔB(θ)) × Ve × dθ

where ΔB(θ) = L × ΔIL(θ) / (N × Ae)
and ΔIL(θ) = Vm × |sin(θ)| × (1 - Vm×|sin(θ)|/Vout) / (fsw × L)
```

This integration is typically done numerically. The result is that the average core loss is lower than the peak ripple value (because at zero crossings, ΔB → 0 and core loss → 0).

A simplified estimate: multiply the peak-ΔB loss by approximately 0.5–0.6 to get the average loss over the AC cycle.

```
P_core_A_avg ≈ 1.21 × 0.55 = 0.67 W
P_core_B_avg ≈ 0.22 × 0.55 = 0.12 W
```

This makes the ferrite option even more attractive in practice.

---

## Key Equations Summary

```
STEINMETZ (sinusoidal): Pv = Cm × f^α × B_peak^β  [W/cm³]

iGSE (switching waveform):
  feq = (2 × fsw/π²) / (D × (1-D))   [for triangular wave]
  Pv = Cm × feq^(α-1) × (ΔB/2)^β × fsw   [ΔB = peak-to-peak flux density; use the half-swing]

TOTAL CORE LOSS: P_core = Pv × Ve

FLUX DENSITY: B_pk = L × I_peak / (N × Ae)
              ΔB   = L × ΔIL / (N × Ae)

TEMPERATURE RISE: ΔT ≈ 450 × P_total / A_surface  [°C, P in W, A in cm²]
```
