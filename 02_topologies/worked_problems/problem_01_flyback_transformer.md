# Worked Problem 01 — Flyback Transformer Design

## Problem Statement

Design the transformer for a flyback converter with the following specification:

| Parameter | Value |
|-----------|-------|
| Input voltage (Vin) | 24–36 V (battery-powered industrial) |
| Output voltage (Vout) | 12 V |
| Output power (Pout) | 30 W |
| Switching frequency (fsw) | 150 kHz |
| Efficiency target (η) | 85% |
| Operating mode | DCM |
| Maximum duty cycle | D_max = 0.45 |
| Core available | EE25 ferrite (N87 material, Ae = 52 mm², Aw = 68 mm²) |

Design the turns ratio, primary turns, secondary turns, winding wire gauges, verify saturation, and estimate transformer losses.

---

## Step 1 — Establish Turns Ratio

**Design philosophy for DCM flyback:**

Choose the turns ratio n = Np/Ns such that:
1. The MOSFET voltage stress (Vin + n×Vout) is within a safe limit.
2. The duty cycle at minimum Vin achieves regulation with margin below D_max.
3. The reflected output voltage (n×Vout) is reasonable relative to Vin.

**MOSFET voltage stress constraint:**

Target V_DS_max = 80 V (provides good margin for 100V MOSFET):
```
V_DS = Vin_max + n × Vout + V_spike
V_spike ≈ 15V (RCD snubber overhead estimate)
80 = 36 + n × 12 + 15
n × 12 = 80 - 36 - 15 = 29
n = 29 / 12 = 2.42 → use n = 2.5 (5:2 winding ratio)
```

**Verify duty cycle at minimum Vin:**

In DCM, the duty cycle for a flyback is set by energy balance. The primary peak current required:
```
Ip_peak = Vin × D / (Lm × fsw)
Energy per cycle = 0.5 × Lm × Ip_peak² = Pout / (fsw × η)
```

At Vin_min = 24V, D_max = 0.45:
```
Reflected voltage = n × Vout = 2.5 × 12 = 30V
Vin_min/reflected = 24/30 = 0.8 → converter is step-down for primary switching ratio
```

Check: at Vin=24V and D=0.45, will the secondary duty cycle allow DCM?
```
D2 = D × Vin / (n × Vout) = 0.45 × 24 / 30 = 0.36
D3 (deadtime) = 1 - D - D2 = 1 - 0.45 - 0.36 = 0.19 > 0 → DCM confirmed at Vin=24V
```

---

## Step 2 — Calculate Magnetising Inductance (Lm)

**Energy balance in DCM:**
```
Pout = 0.5 × Lm × Ip_peak² × fsw × η

Ip_peak = Vin × D / (Lm × fsw)
→ Pout = 0.5 × Lm × (Vin × D / (Lm × fsw))² × fsw × η
→ Pout = 0.5 × (Vin × D)² × η / (Lm × fsw)
→ Lm = 0.5 × (Vin × D)² × η / (Pout × fsw)
```

Use worst case: Vin_min = 24V, D_max = 0.45:
```
Lm = 0.5 × (24 × 0.45)² × 0.85 / (30 × 150e3)
   = 0.5 × (10.8)² × 0.85 / 4,500,000
   = 0.5 × 116.64 × 0.85 / 4,500,000
   = 49.57 / 4,500,000
   = 11.0 µH
```

**Peak primary current (at Vin=24V, D=0.45):**
```
Ip_peak = Vin × D / (Lm × fsw) = 24 × 0.45 / (11.0e-6 × 150e3)
        = 10.8 / 1.65
        = 6.55 A
```

**Peak secondary current:**
```
Is_peak = n × Ip_peak = 2.5 × 6.55 = 16.4 A
```

Average secondary current (equal to Iout = 30W/12V = 2.5A in steady state).

---

## Step 3 — Core Selection and Primary Turns

**Core selection — EE25 (N87 ferrite, given):**
- Ae = 52 mm² = 52 × 10⁻⁶ m²
- Aw = 68 mm² (available window area)
- Bsat ≈ 390 mT at 100°C for N87 ferrite (TDK datasheet)

**Maximum flux density (avoid saturation):**

For DCM flyback, the flux swings from 0 to B_peak each cycle (unipolar):
```
B_peak = Lm × Ip_peak / (Np × Ae)
```

Target B_peak ≤ 200 mT (leaves 190 mT margin below Bsat at temperature):
```
Np = Lm × Ip_peak / (B_peak × Ae)
   = 11.0e-6 × 6.55 / (0.200 × 52e-6)
   = 72.05e-6 / 10.4e-6
   = 6.93 → use Np = 7 turns (round up to keep B_peak below limit)
```

**Verify B_peak with Np = 7:**
```
B_peak = 11.0e-6 × 6.55 / (7 × 52e-6)
       = 72.05e-6 / 364e-6
       = 0.198 T = 198 mT   ← PASS (< 200 mT target)
```

---

## Step 4 — Calculate Air Gap

**Required air gap for Lm = 11.0 µH with Np = 7 turns:**

The inductance of a gapped core:
```
Lm = µ0 × Np² × Ae / (lg + le/µr)

where:
  µ0 = 4π × 10⁻⁷ H/m
  lg = air gap length
  le = effective magnetic path length (E25/13/7: le = 58 mm)
  µr = relative permeability of N87 ferrite (≈ 2200 at 25°C, ≈ 1800 at 100°C)
```

The gap term dominates when lg >> le/µr:
```
le/µr = 58e-3 / 1800 = 32.2 µm

Lm = µ0 × Np² × Ae / lg  [air-gap dominated]
lg = µ0 × Np² × Ae / Lm

µ0 × Np² × Ae = 4π×10⁻⁷ × 49 × 52×10⁻⁶
              = 1.2566×10⁻⁶ × 49 × 52×10⁻⁶
              = 1.2566×10⁻⁶ × 2548×10⁻⁶
              = 3201×10⁻¹² = 3.201 nH·m

lg = 3.201×10⁻⁹ / 11.0×10⁻⁶ = 0.000291 m = 0.291 mm
```

**Air gap: lg ≈ 0.30 mm** (specify this to the core manufacturer or use grinding/shimming)

Note: This is a single air gap in one leg of the EE core. Physical gap = 0.15 mm per half-core.

---

## Step 5 — Secondary Turns

**Turns ratio equation:**
```
n = Np / Ns = 2.5
Ns = Np / n = 7 / 2.5 = 2.8 → cannot use non-integer turns
```

Adjust: Use Np = 10, Ns = 4 → n = 2.5 exactly.

**Recheck B_peak with Np = 10:**
```
B_peak = 11.0e-6 × 6.55 / (10 × 52e-6) = 72.05e-6 / 520e-6 = 138 mT ← better margin
```

**Recalculate air gap for Np = 10:**
```
µ0 × Np² × Ae = 4π×10⁻⁷ × 100 × 52×10⁻⁶ = 6.534×10⁻⁹ H·m

lg = 6.534×10⁻⁹ / 11.0×10⁻⁶ = 0.594 mm  ← larger gap required
```

**Verify window utilisation (10 primary + 4 secondary turns fit in EE25):**

With a 4-layer winding structure (2 primary layers + 2 secondary layers, interleaved):
- Primary: 10 turns, each requiring approximately 0.5 mm wire diameter → 5 mm width for single layer
- EE25 bobbin width ≈ 12 mm → 10 turns per layer fits easily

**Selected winding:** Np = 10 turns, Ns = 4 turns, n = 2.5.

---

## Step 6 — Wire Size Selection

**Primary winding (DC current + AC ripple):**

Primary current is pulsed — RMS current matters for winding loss.

For DCM: primary current is a triangle wave during on-time D=0.45, zero during off-time.
```
Ip_rms = Ip_peak × √(D/3) = 6.55 × √(0.45/3) = 6.55 × √0.15 = 6.55 × 0.387 = 2.54 A
```

Current density limit (J = 4 A/mm² for this application):
```
Wire area required = 2.54 / 4 = 0.635 mm²
Wire diameter = 2 × √(0.635/π) = 2 × √(0.202) = 2 × 0.449 = 0.899 mm
```

At 150 kHz, skin depth in copper:
```
δ = 66.1 / √(150000) = 66.1 / 387 = 0.171 mm
```

Wire diameter (0.9 mm) is much larger than 2δ (0.34 mm) → significant skin effect!

**Solution:** Use Litz wire or multi-strand wire for primary at 150 kHz.
- Use 7 strands of AWG28 (0.32 mm diameter, area = 0.0804 mm² each)
- Total area = 7 × 0.0804 = 0.563 mm² → J = 2.54/0.563 = 4.5 A/mm² (acceptable)
- Each strand diameter ≈ 0.32 mm ≈ 2δ → good skin depth utilisation

**Secondary winding:**

Secondary current (RMS):
```
Is_peak = 16.4 A, D2 = 0.36
Is_rms = Is_peak × √(D2/3) = 16.4 × √(0.36/3) = 16.4 × √0.12 = 16.4 × 0.346 = 5.68 A
```

Wire area required: 5.68 / 4 = 1.42 mm²

At 150 kHz, skin depth = 0.171 mm. For 4-turn secondary with high current, use foil winding:
- Copper foil: 1.5 mm wide × 0.2 mm thick (area = 0.3 mm²) per foil — need 5 parallel foils
- Or: 1.5 mm × 1.0 mm foil strip = 1.5 mm² area — single foil per turn works
- Foil thickness should be ≤ 2δ = 0.34 mm for minimal AC loss

**Select: 1.5 mm wide × 0.3 mm thick copper foil for secondary (4 turns)**
- Area = 0.45 mm² → J = 5.68/0.45 = 12.6 A/mm² — slightly high; use 0.5 mm thick foil (area 0.75 mm², J = 7.6 A/mm²; ≈ 3δ thick, so expect some extra AC loss)

---

## Step 7 — Leakage Inductance Estimate

For an EE25 with non-interleaved winding (primary wound first, secondary over it), leakage inductance is approximately:
```
Llk ≈ µ0 × Np² × lw × hw / (3 × bw)

where:
  lw = mean turn length ≈ 35 mm (EE25)
  hw = winding height ≈ 4 mm (primary + secondary layer height)
  bw = bobbin width ≈ 12 mm
```
```
Llk ≈ 4π×10⁻⁷ × 100 × 0.035 × 0.004 / (3 × 0.012)
    = 4π×10⁻⁷ × 100 × 1.4×10⁻⁴ / 0.036
    = 4π×10⁻⁷ × 0.389
    = 490 nH
```

Leakage inductance referred to primary: ~490 nH.

This means at turn-off with Ip_peak = 6.55A:
```
E_leakage = 0.5 × 490e-9 × 6.55² = 0.5 × 490e-9 × 42.9 = 10.5 µJ
```
This energy must be absorbed by the RCD snubber each cycle:
```
P_snubber = E_leakage × fsw = 10.5e-6 × 150e3 = 1.58 W
```

Significant power loss. To reduce: use interleaved winding (primary-secondary-primary) → typically reduces Llk by 4-8×.

---

## Step 8 — Winding Loss Estimate

**Primary DCR:**
```
R_DCR_pri = ρ_Cu × (Np × l_turn) / A_wire
          = 17.2e-9 × (10 × 0.035) / 0.563e-6
          = 17.2e-9 × 0.35 / 0.563e-6
          = 6.02e-9 / 0.563e-6
          = 10.7 mΩ

P_primary_DCR = Ip_rms² × R_DCR_pri = 6.44 × 0.0107 = 68.9 mW
```

**Secondary DCR:**
```
R_DCR_sec = ρ_Cu × (Ns × l_turn) / A_wire
          = 17.2e-9 × (4 × 0.035) / 0.75e-6
          = 17.2e-9 × 0.14 / 0.75e-6
          = 3.21 mΩ

P_secondary_DCR = Is_rms² × R_DCR_sec = 32.3 × 0.00321 = 103.7 mW
```

**Total winding loss (DC only):** 69 + 104 = 173 mW

**Core loss (using Steinmetz for N87, approximate):**

The TDK N87 datasheet gives typical Pv at 100°C of 57 kW/m³ (25 kHz, 200 mT), 375 kW/m³ (100 kHz, 200 mT) and 390 kW/m³ (300 kHz, 100 mT). Fitting Pv = k × f^α × B^β through these points gives α ≈ 1.36, β ≈ 2.10.

```
B_AC = B_peak / 2 = 138 / 2 = 69 mT = 0.069 T  (in DCM, core sees unipolar triangle)
Actually: for DCM flyback, ΔB = B_peak (swings from 0 to B_peak each cycle)
Use B_AC = ΔB/2 = 0.069 T for Steinmetz

Pv = 375 kW/m³ × (150/100)^1.36 × (69/200)^2.10   [scaled from the 100 kHz, 200 mT point]
   = 375 × 1.74 × 0.107
   ≈ 70 kW/m³

Core volume E25/13/7 = 2990 mm³ = 2.99×10⁻⁶ m³ (Ferroxcube datasheet)
P_core = 70e3 × 2.99e-6 = 209 mW
```

**Total transformer losses: 173 + 209 = 382 mW ≈ 1.3% of Pout**

This is within acceptable range for an 85% efficiency target.

---

## Summary

| Parameter | Result |
|-----------|--------|
| Turns ratio (n = Np/Ns) | 2.5 (10:4) |
| Magnetising inductance (Lm) | 11.0 µH |
| Air gap (total) | 0.594 mm |
| B_peak at worst case | 138 mT |
| Primary turns / wire | 10 turns, 7× AWG28 Litz |
| Secondary turns / wire | 4 turns, 0.5mm copper foil |
| Primary Ip_peak | 6.55 A |
| Secondary Is_peak | 16.4 A |
| Leakage inductance (estimated) | ~490 nH (non-interleaved) |
| Winding losses | 173 mW |
| Core losses | 209 mW |
| Total transformer loss | 382 mW (1.3% of Pout) |

**Key design notes:**
1. Interleave windings (P-S-P) to reduce leakage inductance and snubber loss.
2. Use NTC-compensated DCR sensing if current sensing is needed.
3. Verify B_peak over full temperature range (Bsat drops at high temperature for N87).
4. RCD snubber must be sized for 1.58W dissipation — use a 2W or 5W resistor.
