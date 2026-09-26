# Worked Problem 02 — LLC Gain Curve Analysis and Design Point Selection

## Problem Statement

An LLC resonant converter is to be designed with the following specification:

| Parameter | Value |
|-----------|-------|
| Input voltage (Vin) | 380–420 V (PFC-regulated bus) |
| Output voltage (Vout) | 54 V (telecom standard) |
| Output power (Pout) | 500 W |
| Target switching frequency at full load, nominal Vin | 120 kHz |
| Target efficiency at full load | ≥ 95% |

Tasks:
1. Select the turns ratio
2. Select k = Lm/Lr and calculate the operating Q range
3. Draw the gain curve (tabulated) and identify the operating region
4. Calculate Lr, Cr, Lm
5. Verify ZVS at minimum load (10%)
6. Identify the frequency range of operation

---

## Step 1 — Turns Ratio Selection

**Design constraint:**

At the series resonant frequency (fr), the LLC gain is always exactly 1 regardless of Q:
```
M = 1 at fn = fsw/fr = 1
```

For unity gain: `Vout = Vin / (2 × n)` (the factor of 2 comes from the half-bridge primary)

At nominal Vin = 400V, target operating near fr at full load:
```
n = Vin / (2 × Vout) = 400 / (2 × 54) = 400 / 108 = 3.70 → use n = 4 (8:2 winding)
```

With n = 4: gain at fn = 1 gives:
```
Vout_ideal = Vin / (2 × n) = 400 / 8 = 50 V
```

50V vs. target 54V → the operating point must be slightly below fr (gain > 1) at full load to achieve 54V:
```
Required gain M = 54 / 50 = 1.08
```

The converter must operate at a frequency where M = 1.08, which is below fr. This is acceptable (fn < 1 region, gain > 1), but ensure ZVS is maintained.

Alternative: use n = 3.5 (7:2 winding):
```
Vout_at_unity_gain = 400 / (2 × 3.5) = 400/7 = 57.1V
Required gain M = 54/57.1 = 0.945 → operate slightly above fr (gain < 1, fn > 1)
```

This is more conventional (operating above fr for regulation). Use **n = 3.5** (7:2 winding).

---

## Step 2 — Select k and Calculate Q Range

**Inductance ratio k = Lm/Lr:**

Typical range: k = 3 to 10.
- Small k (3–5): wider gain range, but more circulating magnetising current (more conduction loss)
- Large k (7–10): less circulating current (better efficiency), narrower gain range

**Select k = 7** (good balance for narrow Vin range of ±5%)

**Load resistance (primary-referred):**

Secondary load resistance at full load:
```
Rload_sec = Vout² / Pout = 54² / 500 = 2916 / 500 = 5.83 Ω
```

Primary-referred equivalent resistance:
```
Rac = (8/π²) × n² × Rload_sec = 0.8106 × 12.25 × 5.83 = 57.9 Ω
```

**Q at full load (Q_max):**

From Q = √(Lr/Cr) / Rac = Lr × ωr / Rac

We need to choose fr first. Design for fr = 120 kHz (operating at full load ≈ at fr):

If operating at fn = 1 at full load, required gain M = 0.945 (from Step 1 with n=3.5). At fn=1, M=1 always → mismatch.

This means at fn = 1 we get Vout = 57.1V which is 5.7% too high. We need fn > 1 for M < 1.

Let us define: target M = 0.945 at full load. Using FHA formula and solving for fn at M=0.945:
```
At fn slightly above 1 (e.g., fn = 1.05):
M ≈ 1/(1 + 1/k) × 1/fn² × ... [complex, use FHA formula]
```

For simplicity, let us re-target: operate nominally at fr with n = 3.7 (adjusted):
```
n = 3.7: Vout_at_unity = 400/(2×3.7) = 54.05 V ≈ 54V ✓
```

Use n = 3.7 → practical winding: use 37:10 turns ratio (Np=37, Ns=10) or 7.5:2 if feasible.

More practically, accept n = 4 with slight boost above unity gain. Proceed with n = 4, acknowledging the operating point is at fn < 1 (below fr for M > 1).

**Proceed with n = 4, fr = 120 kHz, target M = 1.08 at full load nominal (Step 5 checks whether the tank can deliver it):**

Q at full load (n=4):
```
Rac = 0.8106 × 16 × 5.83 = 75.6 Ω

Q_max = Lr × 2π × fr / Rac
      → to be calculated in Step 3 after choosing Lr
```

For now, choose Q_max = 0.5 as a starting point (Step 5 shows it leaves too little gain with k = 7).

---

## Step 3 — Gain Curve Tabulation

**FHA voltage gain formula** (fn = fsw/fr, Q = √(Lr/Cr)/Rac, k = Lm/Lr):

```
M(fn, Q, k) = k × fn² / √[((k+1) × fn² − 1)² + k² × fn² × (fn² − 1)² × Q²]
```

With k = 7, computing M for various fn and Q values:

**At Q = 0.5 (≈ full load in this design — see Step 4):**

| fn = fsw/fr | M (gain) | Note |
|------------|---------|------|
| 0.60 | 1.09 | Below the gain peak (fn ≈ 0.63) — ZVS lost |
| 0.70 | 1.08 | Near the gain peak |
| 0.80 | 1.06 | Below resonance |
| 0.85 | 1.04 | Below resonance |
| 0.90 | 1.03 | Approaching fr |
| 0.95 | 1.01 | Just below fr |
| 1.00 | 1.00 | At resonance (always, for any Q) |
| 1.05 | 0.99 | Above resonance |
| 1.10 | 0.97 | Above resonance |
| 1.20 | 0.94 | Well above resonance |
| 1.50 | 0.86 | High frequency |

**At Q = 0.8 (heavier load, ≈ 780 W):**

| fn | M (gain) | Note |
|----|---------|------|
| 0.80 | 1.01 | Below resonance |
| 0.85 | 1.02 | Below resonance (near peak) |
| 0.90 | 1.02 | Approaching fr |
| 0.95 | 1.01 | Just below fr |
| 1.00 | 1.00 | At resonance |
| 1.05 | 0.98 | Above resonance |
| 1.10 | 0.97 | Above resonance |
| 1.20 | 0.92 | Well above resonance |

**At Q = 2.0 (overload or very heavy load):**

| fn | M (gain) |
|----|---------|
| 0.90 | 0.95 |
| 1.00 | 1.00 |
| 1.10 | 0.91 |

**Key observations from gain table:**
1. At fn = 1.0: M = 1.00 for ALL Q values — the curves all pass through this point.
2. Below fn = 1: gain increases, but gains flatten with higher Q (heavy load limits peak gain).
3. Above fn = 1: gain decreases monotonically; curves diverge (more Q-dependent).
4. For tight Vin range (±5%) the gain needs to vary only ±5% → frequency range is narrow.

---

## Step 4 — Calculate Tank Components (Lr, Cr, Lm)

**Design target:**
- fr = 120 kHz
- Q at full load ≈ 0.5 (starting point — see Step 5)
- k = 7

**Calculate Lr from Q and fr:**
```
Q = Lr × ωr / Rac
Lr = Q × Rac / ωr = Q × Rac / (2π × fr)
   = 0.5 × 75.6 / (2π × 120e3)
   = 37.8 / 753,982
   = 50.1 µH
```

Select: **Lr = 50 µH**

**Calculate Cr:**
```
fr = 1 / (2π × √(Lr × Cr))
fr² = 1 / (4π² × Lr × Cr)
Cr = 1 / (4π² × fr² × Lr)
   = 1 / (4π² × (120e3)² × 50e-6)
   = 1 / (4 × 9.87 × 1.44e10 × 50e-6)
   = 1 / (4 × 9.87 × 720,000)
   = 1 / (28,469,280)
   = 35.1 nF
```

Select: **Cr = 33 nF** (standard film capacitor value, slightly adjusts fr to:)
```
fr_actual = 1/(2π×√(50e-6×33e-9)) = 1/(2π×√(1.65e-12)) = 1/(2π×1.285e-6) = 123.8 kHz ≈ 124 kHz
```

**Calculate Lm:**
```
Lm = k × Lr = 7 × 50 = 350 µH
```

**Verify fm (parallel resonant frequency):**
```
fm = 1/(2π×√((Lr+Lm)×Cr)) = 1/(2π×√(400e-6×33e-9))
   = 1/(2π×√(1.32e-11)) = 1/(2π×3.633e-6) = 43.8 kHz
```

Operating range must stay above 43.8 kHz to maintain ZVS (avoid below-fm operation).

**Verify Q at full load with selected components:**
```
Q_check = Lr × 2π × fr / Rac = 50e-6 × 2π × 123.8e3 / 75.6
        = 50e-6 × 777,760 / 75.6
        = 38.89 / 75.6
        = 0.514  ← consistent with design target
```

---

## Step 5 — Determine Operating Frequency Range

**Required gain range:**

Vin range: 380–420V. With n = 4:
```
M at Vin=380V: Vout_needed = 54V, Vin_primary_AC = 380V/2 = 190V
M = 2 × n × Vout / Vin = 2 × 4 × 54 / 380 = 432/380 = 1.137
```

Wait — let me use the standard formula:
```
M = Vout × n × 2 / Vin   [for half-bridge LLC]
```

At Vin = 380V: M = 54 × 4 × 2 / 380 = 432/380 = 1.137
At Vin = 400V: M = 432/400 = 1.080
At Vin = 420V: M = 432/420 = 1.029

**Finding frequencies for these gains at Q=0.514 (full load):**

Solving the FHA formula (k = 7, Q = 0.514):

The full-load gain curve peaks at only **M ≈ 1.08, at fn ≈ 0.63**. Below that peak the tank input turns capacitive and ZVS is lost.

At M = 1.029 (full load, Vin_max = 420V):
fn ≈ 0.90 → fsw = 0.90 × 124 = 111 kHz

At M = 1.080 (full load, nominal Vin):
reached only at the gain peak, fn ≈ 0.63–0.66 → fsw ≈ 80 kHz — no margin at all

At M = 1.137 (full load, Vin_min = 380V):
**unreachable** — the peak gain (1.08) is below the required 1.137

**Design consequence:** k = 7 with Q = 0.514 cannot regulate 54 V at full load below about 400 V. A lower k or Q is needed. For example, k = 4 at the same Q peaks at M ≈ 1.29 (fn ≈ 0.57), and k = 7 needs Q ≤ 0.4 for a peak of ≈ 1.21.

**Light load frequency (10% load, Q_light ≈ 0.05):**

At light load (Q → 0) the gain below resonance is:
```
M ≈ k × fn² / ((k+1) × fn² − 1)
```
For M = 1.08: fn² = 1.08 / (1.08 × 8 − 7) = 0.659 → fn ≈ 0.81 → fsw ≈ 101 kHz

**Summary of frequency range (k = 7, Q = 0.514 as designed):**

| Condition | fn | fsw (kHz) |
|-----------|-----|----------|
| Vin=380V, full load (Q=0.51) | — | unreachable (peak gain 1.08) |
| Vin=400V, full load (Q=0.51) | ≈0.65 | ≈80 (at the gain peak) |
| Vin=420V, full load (Q=0.51) | 0.90 | 111 |
| Vin=400V, 10% load (Q≈0.05) | 0.81 | 101 |
| Vin=400V, burst threshold | 1.50 | 186 |

With these tank values the gain range needed from the PFC bus (1.03–1.14) is not available at full load — the tank must be redesigned with more gain headroom before the frequency range can be fixed.

---

## Step 6 — ZVS Verification at Minimum Load

**ZVS requires magnetising current to swing switch node voltage:**
```
0.5 × Lm × Imag_peak² > Coss_eff × Vin²   [energy condition]
```

**Magnetising current peak:**
```
The magnetising inductance sees ±Vin/2 (half-bridge), so:
Imag_peak = (Vin/2) / (4 × fsw × Lm) = Vin / (8 × fsw × Lm)
          = 400 / (8 × 101e3 × 350e-6)   [10% load, fsw ≈ 101 kHz]
          = 400 / 282.8 = 1.41 A
```

**Switch capacitance requirement:**

For a 600V GaN FET with Coss = 120 pF (effective), two Coss must be swung through Vin:
```
E_Coss = 2 × ½ × Coss × (Vin)² = 120e-12 × (400)²
       = 120e-12 × 160,000
       = 19.2 µJ
```

**ZVS energy from Lm:**
```
E_Lm = 0.5 × Lm × Imag_peak² = 0.5 × 350e-6 × 1.41²
     = 0.5 × 350e-6 × 1.99
     = 348 µJ
```

**ZVS check:**
```
E_Lm = 348 µJ >> E_Coss = 19.2 µJ   → ZVS achieved with large margin at minimum load
```

This confirms that the LLC maintains ZVS at all load levels, including very light load — because Imag is load-independent (it depends only on Lm and fsw).

**Dead time requirement:**
```
t_dead_min = 2 × Coss × Vin / Imag_peak = 2 × 120e-12 × 400 / 1.41
           = 96e-9 / 1.41 = 68 ns
```

Select dead time = 100 ns (provides margin and is practical for gate driver timing).

---

## Final Design Summary

| Parameter | Value |
|-----------|-------|
| Turns ratio n | 4 (8:2 winding) |
| k = Lm/Lr | 7 |
| Series resonant frequency fr | 124 kHz |
| Parallel resonant frequency fm | 43.8 kHz |
| Resonant inductor Lr | 50 µH |
| Resonant capacitor Cr | 33 nF (film) |
| Magnetising inductance Lm | 350 µH |
| Full-load Q | 0.514 |
| Operating gain range | 1.03–1.14 |
| Operating frequency range | not achievable at full load, Vin < 400 V — tank needs more gain (see Step 5) |
| Burst mode threshold | fn ≈ 1.5 (186 kHz) |
| ZVS margin at min load | 18× (excellent) |
| Dead time | 100 ns |

**Key design cross-checks:**
1. fm = 43.8 kHz is well below the operating frequencies, but at full load the gain peak (fn ≈ 0.63, ≈ 78 kHz) is the real ZVS boundary — and it sits where nominal-Vin operation would need to be.
2. The peak gain at full load (1.08) is below the 1.14 needed at Vin = 380 V — reduce k or Q (Step 5).
3. Large ZVS margin allows GaN FETs without concern about losing ZVS.
4. Q < 1 at full load provides reasonable gain curve shape and regulation range.
