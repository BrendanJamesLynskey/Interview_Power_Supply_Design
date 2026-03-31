# Worked Problem 01 — Complete Buck Converter Design

## Problem Statement

Design a synchronous buck converter meeting the following specification:

| Parameter | Value |
|-----------|-------|
| Input voltage (Vin) | 12 V nominal (10.8–13.2 V range) |
| Output voltage (Vout) | 3.3 V |
| Maximum output current (Iout_max) | 2 A |
| Switching frequency (fsw) | 500 kHz |
| Output voltage ripple (ΔVout) | ≤ 20 mV pk-pk |
| Ambient temperature (Ta) | 50°C |

Tasks: calculate duty cycle, select inductor, select output and input capacitors, select MOSFETs, and estimate full-load efficiency.

---

## Step 1 — Duty Cycle

**Ideal duty cycle (CCM, ignoring drops):**
```
D = Vout / Vin = 3.3 / 12 = 0.275  (27.5%)
```

**Verification with worst-case Vin:**
```
D_max = 3.3 / 10.8 = 0.306  (minimum input voltage → maximum duty cycle)
D_min = 3.3 / 13.2 = 0.250  (maximum input voltage → minimum duty cycle)
```

The controller must accommodate D_max = 0.306 without hitting maximum duty-cycle clamp limits.

**Corrected duty cycle accounting for conduction drops:**

In steady state, volt-second balance applies including MOSFET drops. With Rds_HS = Rds_LS = 15 mΩ (estimate) and inductor DCR = 50 mΩ:
```
Vout = D × Vin - Iout × [D × Rds_HS + (1-D) × Rds_LS + DCR]
3.3  = D × 12 - 2 × [D × 0.015 + 0.725 × 0.015 + 0.05]
```
Solving: D_actual ≈ 0.282

The feedback loop corrects for this automatically. Use D = 0.275 for component sizing at nominal Vin.

**CCM/DCM boundary current:**
```
Iboundary = Vout × (1-D) / (2 × L × fsw)
```
This will be checked after inductor selection.

---

## Step 2 — Inductor Selection

**Ripple ratio selection:**

Choose r = ΔIL / Iout = 0.30 (30%). This balances inductor size against ripple current stress on the output capacitor. Higher r → smaller L but larger capacitor required.

```
ΔIL = r × Iout_max = 0.30 × 2 = 0.6 A
```

**Required inductance:**
```
L = Vout × (1-D) / (fsw × ΔIL)
  = 3.3 × (1 - 0.275) / (500×10³ × 0.6)
  = 3.3 × 0.725 / 300,000
  = 2.3925 / 300,000
  = 7.975 µH
```

Select standard value: **8.2 µH** (E12 series, next value above 7.975 µH).

**Verify ripple with selected inductance:**
```
ΔIL_actual = 3.3 × 0.725 / (500e3 × 8.2e-6)
           = 2.3925 / 4.1
           = 0.583 A   (slightly below target — acceptable)
```

**Peak inductor current:**
```
IL_peak = Iout_max + ΔIL/2 = 2 + 0.583/2 = 2.29 A
```

**Saturation current requirement (with 30% derating margin):**
```
Isat_required = IL_peak / 0.70 = 2.29 / 0.70 = 3.27 A minimum
```

**RMS current through inductor:**
```
IL_rms ≈ √(Iout² + (ΔIL)²/12) = √(4 + 0.028) ≈ 2.01 A
```

**Component selection — Würth Elektronik WE-TPC 744 771 4082:**

| Parameter | Requirement | Selected Part | Result |
|-----------|-------------|---------------|--------|
| Inductance | 8.2 µH | 8.2 µH | PASS |
| Isat | ≥ 3.27 A | 3.8 A | PASS (1.66× margin) |
| Irms rated | ≥ 2.01 A | 2.2 A | PASS |
| DCR | ≤ 100 mΩ | 48 mΩ typical | PASS |
| Package | — | 5.0×5.0×2.5 mm | — |

**DCR loss at full load:**
```
P_DCR = Iout² × DCR = 4 × 0.048 = 0.192 W
```

**CCM/DCM boundary current with selected L:**
```
I_boundary = Vout × (1-D) / (2 × L × fsw)
           = 3.3 × 0.725 / (2 × 8.2e-6 × 500e3)
           = 2.3925 / 8.2
           = 0.292 A
```

The converter enters DCM below Iout = 0.292 A (14.6% of full load). This is acceptable — the controller must support DCM or the design must include a minimum load specification.

---

## Step 3 — Output Capacitor Selection

**Ripple budget allocation (total ≤ 20 mV):**

| Contribution | Budget |
|-------------|--------|
| Capacitive ripple (ΔV_C) | 10 mV |
| ESR ripple (ΔV_ESR) | 7 mV |
| ESL spike (ΔV_ESL) | 3 mV |

**Minimum capacitance:**
```
Cout_min = ΔIL / (8 × fsw × ΔV_C)
         = 0.583 / (8 × 500e3 × 0.010)
         = 0.583 / 4000
         = 14.6 µF
```

**Maximum ESR:**
```
ESR_max = ΔV_ESR / ΔIL = 0.007 / 0.583 = 12.0 mΩ
```

**Capacitor type justification:**

At 500 kHz, ceramic MLCCs are the clear choice:
- ESR < 5 mΩ per capacitor (well below 12 mΩ)
- No significant ESL effect below 10 MHz for 1210 package
- No ageing or temperature degradation concerns vs. electrolytic

**Voltage derating consideration:**

For a 3.3V output, select capacitors rated ≥ 6.3V (ideally 10V) to account for X5R/X7R voltage coefficient. A 22 µF 4V X5R has approximately 30% capacitance at 3.3V (≈ 6.6 µF) — this is the common design trap.

A 22 µF, 10V X7R in 1210 package retains approximately 75% capacitance at 3.3V → 16.5 µF effective.

**Selection: 3× 22 µF, 10V X7R, 1210 package:**
```
Cout_effective = 3 × 16.5 µF = 49.5 µF (at 3.3V bias)
ESR_parallel  = 3 mΩ / 3 = 1 mΩ
```

**Verification:**
```
ΔV_C   = 0.583 / (8 × 500e3 × 49.5e-6) = 0.583 / 198 = 2.9 mV   ← PASS
ΔV_ESR = 0.583 × 0.001                               = 0.6 mV   ← PASS
Total ripple ≈ 3.5 mV + ESL noise   ≪ 20 mV target             ← PASS
```

**Output capacitor RMS current:**
```
Icap_rms = ΔIL / (2√3) = 0.583 / 3.46 = 0.169 A
```
Ceramic capacitors handle this easily.

---

## Step 4 — Input Capacitor Selection

**Input capacitor RMS current (worst case at D = 0.5, actual at D = 0.275):**
```
I_rms_cin = Iout × √(D × (1-D)) = 2 × √(0.275 × 0.725) = 2 × 0.446 = 0.893 A
```

**Voltage rating:** Vin_max + 20% spike margin:
```
Vcap_rated ≥ 13.2 × 1.5 = 19.8 V  → select 25V rated capacitors
```

**Minimum capacitance for input ripple ≤ 50 mV:**
```
Cin_min = Iout × D × (1-D) / (fsw × ΔVin)
        = 2 × 0.275 × 0.725 / (500e3 × 0.050)
        = 0.399 / 25000
        = 15.9 µF
```

**Selection:**

High-frequency decoupling (ceramic, directly at switching node):
- 2× 22 µF, 25V X7R, 1210 package → effective ≈ 14 µF each at 12V → 28 µF total

Bulk capacitance (polymer or low-ESR electrolytic):
- 1× 100 µF, 25V polymer electrolytic (Panasonic EEVFK1E101P): ESR ≈ 18 mΩ, handles 1A+ ripple current

**Verify input ripple with 28 µF ceramic:**
```
ΔVin = 2 × 0.275 × 0.725 / (500e3 × 28e-6) = 0.399 / 14 = 28.5 mV  ← PASS
```

**Critical placement note:** The input ceramics must be placed with the shortest possible path between the HS MOSFET drain pad and the LS MOSFET source pad. Any parasitic inductance in this path creates a voltage spike at turn-on:
```
V_spike = L_parasitic × dI/dt
```
For L_par = 3 nH and dI/dt = 2A / 5 ns = 400 A/µs: V_spike = 1.2 V (acceptable for 30V MOSFET).

---

## Step 5 — MOSFET Selection

### High-Side MOSFET

**Requirements:**
- Vds_max ≥ Vin_max × 1.5 = 13.2 × 1.5 = 19.8 V → use 30V device
- Id ≥ IL_peak = 2.29 A
- Optimise for FOM2 = Rds_on × Qgd (hard-switching losses dominate HS)

### Low-Side MOSFET

**Requirements:**
- Same Vds rating (must block Vin during HS on-time)
- Id ≥ IL_peak = 2.29 A
- Optimise for Rds_on (zero-voltage switching → minimal switching loss)

**Selected: Vishay SiR626ADP — dual N-channel, PowerPAK 1212-8 package:**

| Parameter | HS spec | LS spec | Part value |
|-----------|---------|---------|------------|
| Vds_max | 30V | 30V | 30V |
| Rds_on (10V Vgs) | min | min | HS: 5.6 mΩ, LS: 3.2 mΩ |
| Qg | low | moderate | HS: 12 nC, LS: 16 nC |
| Qgd | low | n/a | HS: 3 nC |
| Rth_jc | — | — | 50 °C/W per switch |

**Loss calculations (Tj estimated at 70°C; Rds_on × 1.3 temperature factor):**

Conduction losses:
```
Rds_HS_hot = 5.6 mΩ × 1.3 = 7.3 mΩ
Rds_LS_hot = 3.2 mΩ × 1.3 = 4.2 mΩ

P_cond_HS = Iout² × Rds_HS_hot × D    = 4 × 0.0073 × 0.275  = 8.0 mW
P_cond_LS = Iout² × Rds_LS_hot × (1-D) = 4 × 0.0042 × 0.725 = 12.2 mW
P_cond_total = 20.2 mW
```

Switching losses (HS, hard switching):
```
Gate current: I_gate = (Vdrv - Vmiller) / Rg_total = (10 - 3) / 2 = 3.5 A
              [Rg_ext = 1 Ω, Rg_int = 1 Ω, Vmiller ≈ 3V]
Transition time: tr = tf = Qgd / I_gate = 3e-9 / 3.5 = 0.86 ns
P_sw = Vin × IL_peak × (tr + tf) × fsw
     = 12 × 2.29 × 1.72e-9 × 500e3
     = 23.6 mW
```

Gate drive losses (both MOSFETs):
```
P_gate = (Qg_HS + Qg_LS) × Vdrv × fsw
       = (12 + 16) × 10⁻⁹ × 10 × 500e3
       = 140 mW
```

Dead-time body diode (t_dead = 20 ns, 2 transitions per cycle):
```
P_body = Vf_body × Iout × 2 × t_dead × fsw
       = 0.7 × 2 × 2 × 20e-9 × 500e3
       = 28 mW
```

**Thermal check (combined losses on dual package):**
```
P_device_total = 20 + 24 + 140 + 28 = 212 mW
Tj = Ta + P_total × Rth_ja(PCB)  [no heatsink; use PCB copper spreading]
   = 50 + 0.212 × 60°C/W  [estimated PCB thermal resistance]
   = 50 + 12.7
   = 62.7°C   ← well within 150°C limit
```

Assumption of 70°C Tj for Rds_on correction was conservative — actual Tj lower. No iteration required.

---

## Step 6 — Efficiency Estimate

**Complete loss summary at Vin = 12V, Vout = 3.3V, Iout = 2A:**

| Loss source | Power (mW) | % of Pout |
|-------------|------------|-----------|
| Inductor DCR (Iout² × DCR) | 192 | 2.91% |
| HS MOSFET conduction | 8.0 | 0.12% |
| LS MOSFET conduction | 12.2 | 0.18% |
| HS MOSFET switching | 23.6 | 0.36% |
| Gate drive (both) | 140 | 2.12% |
| Body diode conduction | 28 | 0.42% |
| Inductor core + AC winding | ~20 | 0.30% |
| Controller quiescent (Iq) | ~15 | 0.23% |
| **Total estimated losses** | **439** | **6.65%** |

**Output power:** Pout = 3.3 × 2 = **6.6 W**

**Input power:** Pin = 6.6 + 0.439 = **7.039 W**

**Estimated full-load efficiency:**
```
η = Pout / Pin = 6.6 / 7.039 = 93.7%
```

**Dominant loss: gate drive at 140 mW (32% of all losses)**

This is high relative to the output power because:
- 500 kHz is a high switching frequency for this power level
- Total Qg = 28 nC across both MOSFETs

Options to recover 1–1.5% efficiency:
1. Use lower-Qg MOSFETs (Qg ≈ 8 nC each → P_gate = 80 mW, saving 60 mW)
2. Use gate drive voltage of 5V instead of 10V (halves P_gate but increases Rds_on ~20%)
3. Reduce switching frequency to 300 kHz (allows larger L but proportionally less gate loss)

---

## Summary

| Component | Selection | Verified |
|-----------|-----------|---------|
| L | 8.2 µH, Isat=3.8A, DCR=48mΩ | Peak=2.29A < 3.8A, P_loss=192mW |
| Cout | 3× 22µF 10V X7R MLCC (49.5µF eff.) | Ripple=3.5mV ≪ 20mV |
| Cin | 2× 22µF 25V X7R + 100µF polymer | I_rms handled, ΔVin=28.5mV |
| HS MOSFET | 30V, Rds=5.6mΩ, Qg=12nC | Tj=63°C |
| LS MOSFET | 30V, Rds=3.2mΩ, Qg=16nC | Tj=63°C |
| Efficiency | 93.7% full load | Gate drive is dominant loss |

---

## Key Interview Follow-Ups

**Q: Why is gate drive loss so large relative to switching loss at this operating point?**

A: For a 500 kHz, 2A converter, Psw scales with Vin × Iout × (tr+tf) × fsw which is small because both Iout and (tr+tf) are small. P_gate = Qg × Vdrv × fsw scales directly with frequency and total gate charge — at 500 kHz with 28 nC total Qg, this dominates. At higher current levels (20–50A), switching loss grows faster (∝ Iout) while gate loss stays constant, so switching loss eventually dominates.

**Q: How do you choose between 500 kHz and 1 MHz switching frequency?**

A: Higher frequency allows smaller L and Cout (both roughly halved), but doubles gate drive loss, doubles switching loss, increases inductor AC losses, and demands better PCB layout. Choose higher frequency only when size is the primary constraint. For efficiency-first designs, operate at the lowest frequency consistent with the required inductor/capacitor size.

**Q: What happens to efficiency at 10% load (0.2A)?**

A: Fixed losses (gate drive = 140 mW, controller quiescent = 15 mW) remain constant while output power drops to 0.66 W. These losses alone represent 155/816 = 19% of input power — efficiency falls below 82%. Most real converters use pulse skipping or burst mode at light load to reduce these fixed losses.
