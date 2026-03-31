# Worked Problem 03 — Output Capacitor Ripple Analysis

## Problem Statement

A buck converter has the following characteristics:

| Parameter | Value |
|-----------|-------|
| Vout | 1.8 V |
| Iout_max | 10 A |
| fsw | 300 kHz |
| Inductor L | 2.2 µH |
| Inductor ripple current ΔIL | 3.0 A pk-pk |
| Output ripple target | ≤ 15 mV pk-pk |
| Operating temperature range | -20°C to 85°C |

Tasks:
1. Calculate ripple contributions from ESR, ESL, and capacitance
2. Design a ceramic capacitor bank to meet the 15 mV target
3. Design an electrolytic capacitor alternative and compare
4. Design a mixed ceramic + polymer bank and compare all three
5. Identify which capacitor source dominates at different frequencies

---

## Step 1 — Ripple Sources and Equations

Output voltage ripple in a buck converter has three additive contributions:

**Capacitive ripple (triangular wave charge/discharge):**
```
ΔV_C = ΔIL / (8 × fsw × Cout)
```
This is a triangular waveform at fsw, in phase with the inductor current integral.

**ESR ripple (resistive voltage drop across ESR):**
```
ΔV_ESR = ΔIL × ESR
```
This is a quasi-square wave at fsw, proportional to the inductor current waveform. Peaks at inductor current peaks.

**ESL ripple (inductive voltage spike at switching transitions):**
```
ΔV_ESL = ESL × (dIL/dt)
```
This appears as a narrow spike at each switching transition. The current slew rate:
```
dIL/dt_on  = (Vin - Vout) / L
dIL/dt_off = Vout / L
```

For our design with L = 2.2 µH, Vin = 5V (assume), Vout = 1.8V:
```
dIL/dt_on  = (5 - 1.8) / 2.2e-6 = 1.45 A/µs
dIL/dt_off = 1.8 / 2.2e-6       = 0.82 A/µs
```

**Total ripple (approximate, worst case sum):**
```
ΔVout_total ≈ ΔV_C + ΔV_ESR + ΔV_ESL
```
In practice, ΔV_C and ΔV_ESR do not add directly because they are not in phase. The actual worst-case combination depends on the ESR zero frequency relative to fsw.

**ESR zero frequency:**
```
f_ESR_zero = 1 / (2π × ESR × Cout)
```
When f_ESR_zero >> fsw: ESR ripple dominates.
When f_ESR_zero << fsw: capacitive ripple dominates.

---

## Step 2 — Option A: All-Ceramic Capacitor Bank

Ceramic X5R or X7R MLCCs offer very low ESR (1–5 mΩ per capacitor) and low ESL (0.5–1 nH for 1210 package). ESR ripple is negligible; capacitive ripple dominates.

**Design for capacitive ripple only (since ESR ≈ 0):**
```
Cout_min = ΔIL / (8 × fsw × ΔV_target)
```

Allocate ΔV_target = 12 mV to capacitive ripple, 3 mV margin for ESR/ESL:
```
Cout_min = 3.0 / (8 × 300e3 × 0.012)
         = 3.0 / 2,880,000
         = 1.04 µF × 1000 = 1042 nF → 1.04 µF
```

Wait — this seems too small. Let us recheck:
```
Cout_min = 3.0 / (8 × 300,000 × 0.012) = 3.0 / 2,880,000 = 1.04 µF
```

That is correct. 1.04 µF would achieve 12 mV ripple from capacitance alone at 300 kHz with 3A ripple. However, we need adequate capacitance for transient response as well.

**Transient response requirement:**

For a load step of ΔIload = 5A (50% load step), assume the converter takes t_response = 5 µs to respond (limited by inductor slew rate L/Vin × ΔIload):
```
t_response = L × ΔIload / (Vin - Vout) = 2.2e-6 × 5 / (5 - 1.8) = 11e-6 / 3.2 = 3.44 µs
```

During this time, the capacitor supplies the load current difference:
```
ΔVout_transient = ΔIload × t_response / (2 × Cout)
```
For ΔVout_transient ≤ 50 mV (typical transient spec):
```
Cout_transient ≥ 5 × 3.44e-6 / (2 × 0.050) = 17.2e-6 / 0.1 = 172 µF
```

The transient requirement (172 µF) dominates over the steady-state ripple requirement (1.04 µF).

**Capacitor selection:**

For 1.8V output, use 2.5V or 4V rated X5R ceramics:

Voltage coefficient check: A 47 µF, 4V X5R 1210 retains approximately 60% at 1.8V → 28 µF effective.

Required: 172 µF effective → number of 47µF caps:
```
N = 172 / 28 = 6.14 → use 7 caps
```

Select: **7× 47 µF, 4V X5R, 1210 package (Murata GRM32ER60G476ME20)**

Effective capacitance at 1.8V:
```
Cout_eff = 7 × 28 = 196 µF
```

ESR of bank: 3 mΩ per cap / 7 = 0.43 mΩ in parallel

**Verify ripple:**
```
ΔV_C   = 3.0 / (8 × 300e3 × 196e-6) = 3.0 / 470.4 = 6.4 mV
ΔV_ESR = 3.0 × 0.00043             = 1.3 mV
ΔV_ESL ≈ 0.7 nH × 1.45 A/µs       = 1.0 mV spike  [ESL = 0.7nH for 7 caps in parallel]
Total ≈ 6.4 + 1.3 + 1.0            = 8.7 mV pk-pk  ← PASS (< 15 mV)
```

**Transient verification:**
```
ΔVout_transient = 5 × 3.44e-6 / (2 × 196e-6) = 17.2e-6 / 392e-6 = 43.9 mV ← PASS (< 50mV)
```

**Cost/size:** 7 caps × $0.15 each = $1.05, PCB area: 7 × 1210 footprints

---

## Step 3 — Option B: Electrolytic Capacitor Bank

Standard aluminum electrolytic capacitors offer high capacitance density but high ESR and ESL. At 300 kHz, ESR ripple will dominate.

**Typical electrolytic parameters (105°C rated, radial):**

For 100 µF, 6.3V:
- ESR at 100 kHz: ~80 mΩ typical (increases at lower frequencies)
- ESL: ~10 nH
- Temperature coefficient: capacitance drops 20% at -20°C

**ESR ripple with one 100 µF electrolytic:**
```
ΔV_ESR = ΔIL × ESR = 3.0 × 0.080 = 240 mV   ← FAR exceeds 15 mV target
```

A single electrolytic completely fails the ripple requirement at this switching frequency. To meet 15 mV from ESR alone:
```
ESR_required = 15e-3 / 3.0 = 5 mΩ
N_caps = 80 mΩ / 5 mΩ = 16 capacitors
```

16× 100 µF electrolytics in parallel:
- Capacitance: 1600 µF (far more than needed)
- ESR: 5 mΩ (just meets requirement)
- ESL: ~10 nH / 16 = 0.625 nH in parallel (still significant at 300 kHz)
- PCB area: 16 large through-hole caps — impractical

**ESL spike with 16 electrolytics:**
```
ΔV_ESL = 0.625e-9 × 1.45e6 = 906 mV   ← enormous — ESL dominates at 300 kHz
```

This demonstrates a fundamental issue: at 300 kHz, the parasitic inductance (ESL) of electrolytic capacitors produces voltage spikes that easily exceed the ESR ripple. Electrolytic capacitors are not suitable as the primary output filter at frequencies above approximately 100 kHz without ceramic bypass capacitors.

**Conclusion for Option B:** Standard electrolytics fail at 300 kHz due to ESR and ESL. Not recommended as sole output filter.

Low-ESR electrolytic (Nichicon UHW or Panasonic FM series) have ESR ≈ 15–30 mΩ per 100µF cap — still requires many parallel parts and ceramic bypass. Not a viable single-type solution.

---

## Step 4 — Option C: Mixed Ceramic + Polymer Bank

The optimum practical solution combines bulk polymer capacitors for capacitance with ceramic MLCCs for low-ESR/ESL high-frequency bypassing.

**Polymer capacitor characteristics:**
- ESR: 5–20 mΩ (much lower than electrolytic)
- ESL: 2–5 nH (lower than electrolytic)
- Voltage coefficient: near zero (unlike ceramics)
- Temperature stability: good (-55°C to +105°C)
- No electrolyte degradation
- Typical parts: Panasonic EEVFK, KEMET T520/T521 series, Würth WE-SUPD

**Design approach:**
- Polymer capacitors handle bulk capacitance (transient and low-frequency ripple)
- Ceramics handle high-frequency bypassing (reduce ESL-induced spikes)

**Polymer bulk capacitor selection:**

Use 2× 100 µF, 4V polymer (e.g., Panasonic EEVFK0G101P):
- ESR: 15 mΩ each → 7.5 mΩ in parallel
- ESL: 4 nH each → 2 nH in parallel
- Effective capacitance: 200 µF (stable vs. voltage and temperature)

**Ceramic bypass capacitors (placed physically close to switching node):**

Use 3× 22 µF, 4V X5R, 0805:
- ESR: 2 mΩ each → 0.67 mΩ in parallel
- ESL: 0.5 nH each → 0.17 nH in parallel
- Effective at 1.8V: 3 × 18 µF = 54 µF

**Combined bank analysis:**

Total Cout = 200 + 54 = 254 µF
Combined ESR: polymer path (7.5 mΩ / 200 µF) in parallel with ceramic path (0.67 mΩ / 54 µF)

For high-frequency ripple (at fsw = 300 kHz), ceramics dominate due to much lower impedance:
```
Z_ceramic = √(ESR² + (1/(2π × f × C))²) at 300kHz
          = √(0.00067² + (1/(2π×300e3×54e-6))²)
          = √(4.5e-7 + 9.7e-9)
          ≈ 0.00067 Ω  (ESR-dominated)

Z_polymer  = √(0.0075² + (1/(2π×300e3×200e-6))²)
           = √(5.6e-5 + 7.0e-10)
           ≈ 0.0075 Ω  (also ESR-dominated)
```

Ceramic impedance is 11× lower → ceramics carry ~11× more ripple current than polymer at 300 kHz.

**Effective ESR at 300 kHz:**
```
ESR_eff = Z_ceramic || Z_polymer ≈ Z_ceramic / 12 × Z_polymer (parallel)
        ≈ 0.00067 × 0.0075 / (0.00067 + 0.0075)
        ≈ 5.0e-6 / 0.00817
        ≈ 0.61 mΩ
```

**Verify ripple (Option C):**
```
ΔV_C   = 3.0 / (8 × 300e3 × 254e-6)   = 3.0 / 609.6  = 4.9 mV
ΔV_ESR = 3.0 × 0.00061                  = 1.8 mV
ΔV_ESL = 0.17e-9 × 1.45e6               = 0.25 mV   [ceramic path dominates]
Total  ≈ 4.9 + 1.8 + 0.25               = 6.95 mV   ← PASS
```

**Transient check (Option C):**
```
ΔVout_transient = 5 × 3.44e-6 / (2 × 254e-6) = 17.2e-6 / 508e-6 = 33.9 mV ← PASS
```

---

## Step 5 — Comparison and Frequency Domain Analysis

**Summary comparison:**

| Parameter | Option A (Ceramic only) | Option B (Electrolytic only) | Option C (Polymer + Ceramic) |
|-----------|------------------------|------------------------------|-------------------------------|
| Capacitors | 7× 47µF X5R 1210 | 16× 100µF electrolytic | 2× 100µF polymer + 3× 22µF MLCC |
| Total Cout | 196 µF | 1600 µF | 254 µF |
| ESR at 300kHz | 0.43 mΩ | 5 mΩ | 0.61 mΩ |
| ESL at 300kHz | 0.7 nH | 0.625 nH | 0.17 nH |
| ΔVout steady-state | 8.7 mV | Fails (>100mV) | 6.95 mV |
| Transient ΔV (5A step) | 43.9 mV | Requires simulation | 33.9 mV |
| Temperature stability | Moderate (X5R) | Poor at cold | Excellent |
| Cost (approx) | $1.05 | $3.20 + layout issues | $2.40 |
| PCB area | Moderate | Very large | Moderate |
| Reliability | Excellent (ceramic) | Limited (electrolyte) | Excellent |

**Frequency domain perspective:**

The impedance of each capacitor type vs. frequency:
```
1. At low frequency (<10 kHz):
   - Polymer/electrolytic dominates (high capacitance, low impedance)
   - Ceramics have high impedance (small absolute capacitance)

2. At switching frequency (300 kHz):
   - Ceramics dominate (lowest ESR)
   - Polymer still useful (lower ESR than electrolytic)
   - Electrolytic has too high ESL-related impedance

3. At high frequency (>10 MHz, from MOSFET switching edges):
   - Only small ceramics (0402/0603) have low enough ESL
   - Place these directly at switch node and output pin
```

**Practical recommendation:** Option C (Polymer + Ceramic) provides the best balance of performance, reliability, and cost. The polymer caps handle bulk capacitance without voltage coefficient issues; ceramics suppress high-frequency noise.

---

## Step 6 — ESL Spike Deep-Dive

**Why ESL matters at 300 kHz:**

The voltage spike from ESL occurs at the edges of the switching transitions, not at the switching frequency itself. The spike duration is approximately equal to the MOSFET rise/fall time (1–10 ns typically), making it effectively a MHz-frequency event even in a 300 kHz converter.

**Spike magnitude:**
```
ΔV_spike = ESL × (dI/dt)

For dI/dt = 1.45 A/µs = 1.45×10⁶ A/s:
With ESL = 10 nH (single electrolytic): ΔV_spike = 10e-9 × 1.45e6 = 14.5 mV
With ESL = 2 nH (polymer):             ΔV_spike = 2e-9 × 1.45e6  = 2.9 mV
With ESL = 0.17 nH (parallel ceramics): ΔV_spike = 0.17e-9 × 1.45e6 = 0.25 mV
```

The ceramics reduce ESL spikes by nearly 60× compared to a single electrolytic — this is why ceramic bypass capacitors are always placed alongside bulk capacitors in modern power supply designs.

**Physical layout impact:**

ESL is not purely a component property — it includes the parasitic inductance of PCB traces between the capacitor and the load. Each centimeter of trace adds approximately 1–3 nH. Therefore:
- Place output capacitors as close as possible to the switching node (inductor output → capacitor → GND via → plane)
- Use multiple short vias to connect capacitor ground pins to the ground plane
- Minimise trace length between inductor output and capacitor pad

---

## Final Recommendation

**Select Option C** with the following BOM:

| Qty | Component | Value | Package | Note |
|-----|-----------|-------|---------|------|
| 2 | Polymer electrolytic | 100 µF, 4V | D-case SMD | Bulk capacitance, stable vs. T and V |
| 3 | MLCC X5R | 22 µF, 4V | 0805 | High-frequency bypass, low ESL |
| 1 | MLCC X7R | 100 nF, 10V | 0402 | Very high-frequency decoupling |

Place the 0402 ceramic directly at the MOSFET switch node. Place 0805 ceramics next to the inductor output. Place polymer caps near the load connector.

This layered capacitor placement strategy — with decreasing ESL from power stage to load — provides effective ripple filtering across all frequency decades from DC to >100 MHz.
