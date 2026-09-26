# Worked Problem 02: Transformer Leakage Inductance Analysis

## Problem Statement

A flyback converter transformer has been wound and is showing worse performance than expected. Analyse the leakage inductance: measure it, predict its effect on converter performance, and evaluate mitigation techniques.

**Converter and transformer specifications:**
```
Vin_max  = 375 V DC (rectified from 265 Vac)
Vin_min  = 100 V DC (rectified from 85 Vac)
Vout     = 15 V, Iout_max = 2 A (Pout = 30W)
fsw      = 100 kHz, Ts = 10 µs
Turns:   Np = 50 turns, Ns = 10 turns → n = 5
Core:    EE25/13/7, N87 ferrite
Winding: Non-interleaved (primary bottom, secondary top, separated by 3 layers of 0.1mm tape)
```

**Measured values:**
```
Primary inductance (Ls open):    L_primary = 52 mH → magnetising inductance Lm = 52 mH
Primary leakage share:           Ll_primary = 18 µH   (half of the 36 µH read with Ls shorted, LCR at 100 kHz — equal-split assumption)
Secondary leakage share:         Ll_secondary = 0.72 µH  (= 18 µH / n²)
Total leakage referred to primary: Ll_total = 18 + 25 × 0.72 = 18 + 18 = 36 µH
```

---

## Part A: Measuring Leakage Inductance

**Measurement procedure:**

**Step 1 — Measure magnetising inductance:**
```
Short the secondary winding.
Measure inductance of primary with LCR meter at operating frequency (100 kHz).
→ L_measured = Ll_primary + (Lm || n²×Ll_secondary)   [Lm is shunted by the reflected secondary leakage]
For n²×Ll_secondary << Lm:  L_measured ≈ Ll_primary + n²×Ll_secondary = Ll_total
→ Short-circuit primary inductance = total leakage referred to primary
→ Ll_total = 36 µH  (measured with secondary shorted)
```

**Step 2 — Open secondary, measure primary inductance:**
```
Open the secondary winding.
Measure inductance of primary.
→ L_primary_open = Lm + Ll_primary ≈ Lm = 52 mH  (since Ll << Lm)
```

**Step 3 — Calculate leakage fraction:**
```
k² = 1 - (Ll_total / Lm)  [coupling coefficient squared]
k = √(1 - 36µH / 52mH) = √(1 - 0.000692) = √(0.99931) = 0.99965

Leakage fraction: σ = Ll_total / Lm = 36µH / 52mH = 0.069%

Coupling coefficient k = 1 - σ/2 ≈ 99.97%  [high coupling, but leakage still significant for spikes]
```

---

## Part B: Predicting the Voltage Spike at Turn-Off

**Switch-off analysis:**

When the MOSFET turns off at peak current I_peak, the leakage inductance energy cannot be immediately transferred to the secondary (the magnetising inductance current transfers via the turns ratio, but the leakage flux has no secondary path).

**Peak current calculation (DCM flyback):**
```
Pout = ½ × Lm × I_peak² × fsw / η
30W = ½ × 52mH × I_peak² × 100kHz / 0.85

I_peak² = 30 × 2 × 0.85 / (52×10⁻³ × 100×10³)
        = 51 / 5200 = 0.00981
I_peak = 0.099 A   [this seems low — check]

Actually: DCM flyback at 85Vac minimum:
Vin_min = 100V (after rectification), D_max = 0.45

I_peak = Vin_min × D_max × Ts / Lm  [magnetising current rise]
       = 100 × 0.45 × 10µs / 52mH
       = 450×10⁻⁶ / 52×10⁻³
       = 8.65 mA

This is the magnetising current only. For a DCM flyback at Pout=30W:
I_peak_primary = √(2 × Pout × Ts / (Lm × η)) = √(2 × 30 × 10µs / (52mH × 0.85))
               = √(600×10⁻⁶ / 44.2×10⁻³) = √(0.01357) = 0.116 A

Hmm — still low. Let us verify:
Lm = Vin_min² × D_max² / (2 × Pout × fsw) = 100² × 0.45² / (2 × 30 × 100k)
   = 2025 / 6,000,000 = 337.5 µH

The measured Lm = 52 mH is too large for DCM at these specifications. Either the design intends CCM, or the specifications have a discrepancy.

For this worked problem, use a self-consistent calculation:
Assume CCM flyback, I_peak = 2 × Pout / (Vin_min × D_max × η × 2)
[for CCM: I_peak = I_avg + ΔIL/2, approximate I_peak ≈ 1.2A at full load, minimum Vin]
```

**Using I_peak = 1.2 A for the spike calculation:**

**Energy in leakage inductance at turn-off:**
```
E_leakage = ½ × Ll_total × I_peak²
          = ½ × 36×10⁻⁶ × 1.2²
          = ½ × 36×10⁻⁶ × 1.44
          = 25.9 µJ
```

**Voltage spike without snubber:**

The leakage energy drives a resonance between Ll_total and the MOSFET output capacitance Coss:
```
V_spike_additional = I_peak × √(Ll_total / Coss)

Typical Coss for a 500V, 2A FET at Vds = 200V: Coss ≈ 50 pF

V_spike = I_peak × √(Ll_total / Coss)
        = 1.2 × √(36µH / 50pF)
        = 1.2 × √(720,000)
        = 1.2 × 848
        = 1018 V
```

**Total drain voltage at turn-off:**
```
V_DS_total = Vin_max + n × Vout + V_spike
           = 375 + 5 × 15 + 1018
           = 375 + 75 + 1018
           = 1468 V
```

A 600V MOSFET (commonly used with 375V DC bus) will fail immediately. This design requires a snubber.

---

## Part C: Snubber Design to Limit the Spike

**RCD snubber design:**

Target: limit V_DS to 550V (safe for a 600V MOSFET with 90% derating):
```
V_clamp = 550V - Vin_max - n × Vout
        = 550 - 375 - 75 = 100V
→ The snubber clamp voltage must be 100V above the reflected secondary voltage

V_clamp_total = Vin_max + V_clamp_additional
We want total V_DS,max = V_clamp = 550V:
V_snubber = 550 - (Vin + n×Vout) = 100V above the normal reflected voltage
```

**Snubber capacitor:**

The capacitor clamps the voltage at V_clamp. The leakage energy charges it:
```
V_clamp² × C_snubber / 2 ≥ E_leakage  [C must absorb all leakage energy]
C_snubber ≥ 2 × E_leakage / V_clamp²
           = 2 × 25.9×10⁻⁶ / (100)²   [voltage across snubber cap = V_clamp - (Vin+n×Vout)]
           = 51.8×10⁻⁶ / 10,000
           = 5.18 nF → choose C_snubber = 10 nF (double for voltage margin)
```

**Snubber resistor:**

The resistor must discharge the capacitor within the off-time (max 10µs):
```
Time constant: τ = R × C ≤ off-time/3 = (1-D)×Ts/3 = 0.55 × 10µs / 3 = 1.83 µs

R_snubber = τ / C = 1.83µs / 10nF = 183 Ω → choose R = 150 Ω (standard, slightly smaller)
```

**Verify snubber dissipation:**
```
P_snubber = ½ × C × V_clamp² × fsw
          = ½ × 10nF × 100² × 100kHz
          = ½ × 10×10⁻⁹ × 10,000 × 100,000
          = 5W

This is 5W of wasted power — significant for a 30W converter (16.7% loss!)
```

**Assessment:** The 36 µH leakage inductance causes a 5W snubber loss, reducing efficiency by 16.7%. This design is unacceptable. The winding must be redesigned to reduce leakage.

---

## Part D: Reducing Leakage Through Winding Redesign

**Option 1 — Interleaved P-S-P winding:**

Split the primary into two equal halves (25 turns each) and sandwich the secondary between them:
```
Layer order: Primary (25T) → Insulation → Secondary (10T) → Insulation → Primary (25T)
```

**Leakage reduction:**

For a P-S-P interleave, leakage reduces approximately as 1/N_sections² = 1/4 = 25%:
```
Ll_interleaved ≈ Ll_non-interleaved / 4 = 36µH / 4 = 9 µH
```

More precisely, for the P-S-P structure:
```
Ll ≈ µ0 × (Np/2)² × lw × (d_ins / 3 + d_cond/(N_sections²)) / bw
   ≈ Ll_original × (1/N_sections²) × correction_factor
   ≈ 36µH × 0.25 = 9 µH  [rough estimate]
```

With Ll = 9 µH:
```
E_leakage_new = ½ × 9µH × 1.2² = 6.5 µJ
V_spike_new = 1.2 × √(9µH / 50pF) = 1.2 × 424 = 509 V

V_DS_total = 375 + 75 + 509 = 959V — still too high!
```

Even with interleaving, a snubber is needed, but now much smaller:
```
V_clamp = 600 - 375 - 75 = 150V  [for 600V MOSFET]
P_snubber = ½ × C' × V_clamp² × fsw

C' = 2 × E_leakage_new / V_clamp² = 2 × 6.5µJ / 150² = 578 pF ≈ 1 nF
P_snubber = ½ × 1nF × 150² × 100kHz = 1.125W  (77% reduction vs original 5W)
```

Remaining snubber loss = 1.125W → efficiency improvement = (5 - 1.125)/30 = 12.9%.

**Option 2 — Triple-insulated wire (TIW) with full interleave:**

Using TIW allows primary and secondary to be wound in the same physical layer:
```
Full interleave: P-S-P-S-P-S... every few turns
Leakage can be reduced by factor of 10-20× compared to non-interleaved
Ll_TIW ≈ 1–3 µH (rough estimate, depends on specific geometry)
```

With Ll = 2 µH:
```
E_leakage_TIW = ½ × 2µH × 1.2² = 1.44 µJ
V_spike_TIW = 1.2 × √(2µH / 50pF) = 1.2 × 200 = 240 V
V_DS = 375 + 75 + 240 = 690V — still above 600V MOSFET limit

Need either: 800V MOSFET, or active clamp, or further reduce leakage
```

**Option 3 — Switch to LLC topology:**

In an LLC resonant converter, the leakage inductance is used as part of the resonant network (Lr). Instead of causing spikes, Lr is a designed element:
```
Lr_LLC = Ll_measured = 36 µH (could be used directly)
Cr_LLC = 1 / (Lr × ωr²) where ωr = 2π × fsw_resonance

This converts a problem into a feature.
```

---

## Part E: Measuring Leakage with Physical Verification

**LCR meter measurement procedure:**

1. **Open-circuit inductance (Lopen):**
   - Primary connected to LCR meter
   - Secondary open
   - Measure L at fsw = 100 kHz
   - Result: Lopen = Lm + Ll_primary ≈ Lm = 52 mH

2. **Short-circuit inductance (Lshort):**
   - Primary connected to LCR meter
   - Secondary shorted with a copper clip (low resistance)
   - Measure L at fsw = 100 kHz
   - Result: Lshort ≈ Ll_total_referred_to_primary = 36 µH

3. **Secondary leakage:**
   - Secondary connected to LCR meter
   - Primary shorted
   - Measure L
   - Result: Lshort_sec = Ll_total / n² = 36 µH / 25 = 1.44 µH (total leakage referred to the secondary)
   - Verify: n² × Lshort_sec = 25 × 1.44 = 36 µH ✓ (matches the primary-side short-circuit reading)

**Coupling coefficient:**
```
k = √(1 - Lshort/Lopen) = √(1 - 36µH/52mH) = √(0.99931) = 0.99965
```

**Time-domain verification:**

Apply a voltage pulse (e.g., using a DC bench supply with a MOSFET switch) and monitor the primary current. The initial di/dt gives Ll:
```
di/dt_initial = Vin / Ll_total   [before magnetising current rises]
→ Ll_total = Vin / (di/dt_initial)

Slope measurement: if Vin = 10V, initial slope = 278 A/ms = 278,000 A/s
→ Ll_total = 10 / 278,000 = 36 µH ✓
```

---

## Summary: Leakage Inductance Effects and Mitigation

| Ll (µH) | V_spike (V) | V_DS_max (V) | Snubber loss | Action                        |
|---------|-------------|--------------|--------------|-------------------------------|
| 36      | 1018        | 1468         | 5.0 W        | Must redesign or change FET   |
| 9       | 509         | 959          | 1.1 W        | Interleave winding P-S-P      |
| 2       | 240         | 690          | 0.3 W        | TIW with full interleave      |
| 0.5     | 120         | 570          | 0.07 W       | Max interleave + TIW           |

**Key equations:**
```
E_leakage = ½ × Ll × I_peak²
V_spike = I_peak × √(Ll / Coss)
V_DS_max = Vin + n×Vout + V_spike
P_snubber = ½ × C_snubber × V_clamp² × fsw
Ll_interleaved ≈ Ll_original / (N_interleave)²
k = √(1 - Ll/Lm)   [coupling coefficient]
```
