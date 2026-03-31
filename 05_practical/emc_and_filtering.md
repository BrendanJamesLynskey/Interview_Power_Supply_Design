# EMC and Filtering — Interview Preparation

## Overview

EMC (electromagnetic compatibility) is a mandatory compliance requirement for virtually all power supply products sold commercially. Failing EMC tests delays product launch and can require expensive redesign. Understanding differential-mode and common-mode noise, input filter design, EMI standards, and measurement techniques is essential for any power electronics engineer working on commercial products.

---

## Key Equations Reference

### Differential-Mode (DM) Filter (LC)
```
Attenuation at frequency f:
  A_DM(f) = 20 × log(1 / |1 - (f/f_LC)²|)   [dB, above resonance]
  f_LC = 1 / (2π × √(L_DM × C_DM))

Above f_LC: -40 dB/decade attenuation
```

### Common-Mode (CM) Filter (choke + Y-caps)
```
CM choke impedance: Z_CM = 2 × jω × L_CM

Attenuation with Y-caps:
  A_CM = 20 × log(|1 + 2 × jω × L_CM × Cy × ω|)  [simplified]
  f_CM = 1 / (2π × √(L_CM × Cy/2))   [resonant frequency of CM filter]
```

### Conducted EMI Limits (CISPR 32 / EN55032 Class B)
```
Frequency range: 150 kHz to 30 MHz
Quasi-peak limit: 66–56 dBµV (150k to 500 kHz), 56 dBµV (500k to 30 MHz)
Average limit:    60–50 dBµV (150k to 500 kHz), 50 dBµV (500k to 30 MHz)
```

### Mains Impedance (LISN — Line Impedance Stabilisation Network)
```
LISN impedance: 50 Ω (for measuring conducted emissions)
EMI signal at LISN: V_LISN = I_DM × 25 Ω + I_CM × 50 Ω   [both conductors loaded]
```

### Snubber for Switch-Node Ringing
```
Snubber capacitor: Cs = IL_peak / (2π × f_ring × V_ring)   [rough approximation]
Snubber resistor:  Rs = 1 / (2π × f_ring × Cs) = √(Ls / Cs)  [critical damping]
Power in snubber:  P_s = ½ × Cs × V_sw² × fsw   [approx]
```

---

## Fundamentals (Questions 1–6)

---

### Q1. What is the difference between differential-mode (DM) and common-mode (CM) conducted EMI?

**Answer:**

**Differential-mode (DM) noise:**

DM noise flows in opposite directions on the two power conductors (L and N in AC, or + and - in DC). When measured with a LISN, DM noise produces equal and opposite voltages on the two lines.

**Physical source:** The switching current ripple (ΔI at fsw and harmonics) that flows in the input capacitor → MOSFET → inductor → output loop. This current is "differential" — it flows into one terminal and out of the other.

**Magnitude estimate:**
```
DM noise voltage at LISN (single conductor):
  V_DM ≈ I_DM_peak × Z_LISN/2 = ΔI_switch × 25 Ω   [LISN is 50Ω per conductor in parallel = 25Ω DM]
```

For ΔI = 100 mA (small, well-filtered): V_DM = 2.5 V = +128 dBµV — well above the 56 dBµV limit!
This shows why an input filter is absolutely essential.

**Common-mode (CM) noise:**

CM noise flows in the same direction on both conductors — it returns through the earth/chassis ground. When measured at the LISN, CM noise produces equal voltages on both lines simultaneously.

**Physical source:** The switch node (SW) has high dV/dt (1–10 V/ns). The stray capacitance from the switch node to the chassis/earth ground (through the PCB, heatsink, Y-capacitors, transformer inter-winding capacitance) creates a common-mode current:
```
I_CM = C_stray × dV/dt

For C_stray = 10 pF and dV/dt = 5 V/ns:
I_CM = 10×10⁻¹² × 5×10⁹ = 50 mA peak
```
This 50 mA common-mode current flows on both conductors simultaneously and can easily exceed EMI limits.

**Measurement distinction:**
```
DM EMI: voltages on L and N are equal and opposite
CM EMI: voltages on L and N are equal in magnitude and same phase
```

A LISN measurement shows the total noise (DM + CM) on each line. To separate: use current probes with common-mode and differential-mode rejection, or use a CM/DM separator network.

---

### Q2. How does a common-mode choke work and how do you select one?

**Answer:**

**Principle:**

A common-mode choke has two windings wound on the same core in the same direction (bifilar or close winding). For DM current (opposing on the two windings), the magnetic fluxes cancel — the choke has negligible DM impedance. For CM current (same direction on both windings), the fluxes add — the choke presents high impedance.

```
CM inductance:  L_CM = L_winding + M ≈ 2L   [M ≈ L for tight coupling]
DM inductance:  L_DM = L_winding - M ≈ 0    [leakage inductance only]
```

**Impedance spectrum:**

A good CM choke presents high impedance from 150 kHz to 30 MHz (conducted EMI range). The impedance peaks at the self-resonant frequency (SRF) of the winding and then decreases at higher frequencies (capacitive). The SRF should be above 30 MHz.

**Selection criteria:**

1. **CM inductance (L_CM):** Determines attenuation with Y-capacitors:
   ```
   f_CM_resonance = 1 / (2π × √(L_CM × Cy_each))
   ```
   Higher L_CM → lower resonant frequency → broader attenuation range.
   Typical: L_CM = 1–100 mH for SMPS.

2. **Current rating:** The DC (or AC) current through the choke must not saturate the core. For CM chokes, the DM flux cancels, so the core does not saturate from load current — only magnetising current from asymmetries. Specify the rated current ≥ I_max.

3. **Leakage inductance (DM inductance):** Some CM chokes are designed with small intentional leakage (DM inductance = a few µH) to also provide some DM attenuation. Useful when DM filter requirements are moderate.

4. **Self-resonant frequency:** Must be above the highest frequency of concern. Check the impedance vs frequency plot in the datasheet.

5. **Insulation / rated voltage:** For mains-connected CM chokes, the winding-to-winding voltage is the full mains voltage. Specify appropriate voltage rating and creepage/clearance.

---

### Q3. What are CISPR 32 conducted EMI limits and what frequency range do they apply to?

**Answer:**

CISPR 32 (published by IEC) specifies limits for conducted and radiated emissions from multimedia equipment (computers, monitors, audio/video equipment). It replaced CISPR 22 for IT and multimedia equipment. For power supplies, it is the most commonly referenced standard.

**Conducted EMI limits (mains port, Class B — residential/commercial):**

| Frequency Range      | Quasi-Peak Limit  | Average Limit   |
|----------------------|-------------------|-----------------|
| 150 kHz – 500 kHz   | 66 dBµV (decreasing to 56 dBµV)  | 56 dBµV (to 46 dBµV) |
| 500 kHz – 5 MHz     | 56 dBµV           | 46 dBµV         |
| 5 MHz – 30 MHz      | 60 dBµV           | 50 dBµV         |

**Measurement standard (CISPR 16):**
- LISN (Line Impedance Stabilisation Network): 50 µH || 50 Ω per conductor
- Measurement bandwidth: quasi-peak detector (CISPR bandwidth = 9 kHz for < 30 MHz)
- Measurement distance: conducted = at mains terminals

**Class A limits:** 10 dB relaxed (66 dBµV from 150k to 30 MHz), for industrial environments.

**FCC Part 15 (US equivalent):**

| Frequency        | Class A (µV/m at 10m) | Class B (µV/m at 3m) |
|------------------|-----------------------|----------------------|
| 30–88 MHz        | 90 dBµV/m             | 100 dBµV/m           |
| 88–216 MHz       | 150                   | 150                  |
| 216–960 MHz      | 210                   | 200                  |

FCC Part 15B applies to unintentional radiators (power supplies are unintentional radiators — the switching is not their intended function from an EMC standpoint). Class B is for household use; Class A for commercial/industrial.

**Practical working rule:**

When designing, aim for 6–10 dB margin below the limits. EMI pre-compliance measurements have ±6 dB uncertainty; margin avoids rework.

---

### Q4. What is an LC input filter and how do you size it for a switching converter?

**Answer:**

**LC filter function:**

The input filter attenuates the switching current ripple drawn from the supply (or from the mains) before it reaches the LISN or source. The filter must attenuate the fundamental (fsw) and harmonics to below the EMI limit.

**Required attenuation at fsw:**

Calculate the unfiltered noise level at fsw and compare to the EMI limit:
```
Unfiltered V_noise ≈ I_switch × Z_LISN = I_switch × 25 Ω   [DM noise at LISN]

Unfiltered level (dBµV) = 20 × log(V_noise / 1µV)

Required attenuation = Unfiltered level - EMI limit - Margin
```

**Example:**
```
I_switch = 500 mA (typical for a 50W, 12V output buck at 100% duty step)
V_noise = 0.5A × 25Ω = 12.5 V = 141.9 dBµV
EMI limit at fsw = 200 kHz: 66 dBµV (quasi-peak, CISPR 32 Class B)
Required attenuation: 141.9 - 66 - 6 (margin) = 69.9 dB

At 200 kHz with a 2nd-order LC filter (-40 dB/decade above f_LC):
From f_LC to fsw: must achieve 70 dB
70 dB at -40 dB/dec: 70/40 = 1.75 decades above f_LC
f_LC × 10^1.75 = fsw = 200 kHz
f_LC = 200 kHz / 56.2 = 3.56 kHz

So: LC filter with corner frequency ≈ 3.5 kHz provides ~70 dB at 200 kHz.
```

**Component calculation:**
```
f_LC = 1 / (2π × √(L × C))
Choose C = 10 µF (electrolytic + ceramic):
L = 1 / (C × (2π × f_LC)²) = 1 / (10×10⁻⁶ × (2π × 3560)²)
  = 1 / (10×10⁻⁶ × 5.0×10⁸)
  = 200 µH → choose L = 220 µH standard value
```

**Critical consideration — input filter stability (Middlebrook criterion):**

The input filter can interact with the converter's negative input impedance and cause oscillation. For stability:
```
|Zout_filter(jω)| < |Zin_converter(jω)|   at all frequencies

Zin_converter ≈ -Vin² / Pin   [negative input impedance of regulated converter]

To prevent interaction: add damping to the filter. Common approach: add R-C damping branch:
  R_damp in series with C_damp, in parallel with L_filter
  R_damp ≈ √(L/C) / Q_desired   [set Q = 0.5 for critically damped]
```

Without filter damping, the filter resonance at f_LC can sustain oscillation (the converter's negative input impedance provides positive feedback).

---

### Q5. What is a snubber circuit and how does it reduce EMI?

**Answer:**

**Snubber purpose:**

When a switching transistor (MOSFET or diode) turns off, parasitic inductances and capacitances cause voltage ringing at the switch node. This ringing:
1. Creates high-frequency EMI (radiated and conducted)
2. Stresses the device (overvoltage risk)
3. Causes false triggering if coupled to control signals

**RC snubber across the switch node:**

A series RC (R_s, C_s) is placed across the MOSFET or diode to damp the ringing:

```
                    C_s
Drain ─────────────|─────────────┐
                   R_s           │
Switch ────────────┤             │
Node                             ├── Drain (same node)
Source ──────────────────────────┘
```

**How it damps ringing:**

The parasitic ringing is an LC resonance between the stray inductance L_stray (PCB trace, inductor leakage) and the device output capacitance C_oss. The RC snubber adds dissipative impedance across the resonant tank.

**Snubber component calculation (critical damping):**

1. Measure or estimate the ringing frequency f_ring and the ringing voltage amplitude V_ring.
2. Estimate the stray inductance: L_stray = Z_ring / (2π × f_ring) where Z_ring = V_ring / I_pk
3. Choose C_s = 2–3 × C_oss of the switch (rules of thumb)
4. Choose R_s = √(L_stray / C_s) (characteristic impedance for critical damping)

**EMI impact:**

Without snubber: The switch node rings at f_ring (typically 20–100 MHz). The energy in this ringing radiates and conducts as broadband noise at very high frequency (above CISPR 32 conducted limits at 30 MHz but within radiated limits).

With snubber: Ringing amplitude reduced by >20 dB. The switch node settles to its steady-state value within 1–2 cycles rather than 5–10 cycles. The energy removed from the ringing is dissipated in R_s (increases converter loss slightly).

**Power in snubber:**
```
P_snubber ≈ ½ × C_s × V_ring² × fsw
```
For C_s = 100 pF, V_ring = 50 V, fsw = 200 kHz:
P_snubber = ½ × 100×10⁻¹² × 2500 × 200,000 = 25 mW — negligible.

---

### Q6. What is the role of Y-capacitors and what limits their value?

**Answer:**

**Y-capacitors (Class Y, safety-rated):**

Y-capacitors connect between the power conductors and the chassis/earth ground. They:
1. Provide a low-impedance path for CM noise currents to return to the source without flowing through long cables
2. Attenuate the CM EMI at the mains terminals

**Physical connections:**
```
L (live) ──┬──── Converter
           Cy1
           GND/Earth ──── Chassis
           Cy2
N (neutral) ──┬──── Converter
```

**Why "Y" classification:**
The "Y" designation indicates capacitors that connect between a conductor and earth. They must withstand:
- Full mains voltage (when one conductor fails to earth)
- Severe transient overvoltages (lightning, switching surges)
- Long-term reliability (20+ year lifetime)

Classes:
- Y1: ≥ 500V AC rating, ≥ 8 kV pulse, used in primary-to-earth (IEC 60950)
- Y2: ≥ 300V AC rating, ≥ 5 kV pulse, most common in single-insulation applications

**Value limits — safety leakage current:**

The Y capacitor passes a continuous leakage current:
```
I_leak = Vmains × 2π × f_mains × Cy = 230V × 314 × Cy

Maximum leakage current limits:
  IEC 62368-1: ≤ 3.5 mA for IT equipment (Class I, earthed)
  IEC 60990: ≤ 0.75 mA for handheld, ≤ 3.5 mA for fixed
  Medical (IEC 60601): ≤ 0.5 mA (patient contact), ≤ 5 mA (chassis current)
```

**Maximum Y-cap value:**
```
Cy_max = I_leak_max / (2π × f_mains × Vmains_max)
       = 3.5×10⁻³ / (2π × 60 × 264)   [US 60Hz, 264V max]
       = 3.5×10⁻³ / 99,483
       = 35.2 nF per capacitor

IEC 61000-3-2 also considers harmonic content.

Typical Y-cap values in SMPS: 2.2 nF to 10 nF (keeping leakage well within limits)
For medical equipment: typically ≤ 2.2 nF to comply with 0.5 mA limit
```

**CM filter effectiveness with Y-caps:**

The CM filter cut-off frequency:
```
f_CM = 1 / (2π × √(L_CM × Cy/2))   [Cy in series (two caps, one each side)]

For L_CM = 10 mH, Cy = 4.7 nF:
f_CM = 1 / (2π × √(10×10⁻³ × 2.35×10⁻⁹)) = 1/(2π × 4.85µs) = 32.8 kHz
```

All CM noise above 33 kHz is attenuated by this filter. The switching frequency (200 kHz) is well above this frequency → significant CM attenuation.

---

## Intermediate (Questions 7–14)

---

### Q7. How do you pre-comply with EMC standards before a formal test?

**Answer:**

Formal EMC testing at an accredited laboratory is expensive (≥$5,000 per test) and requires full production hardware. Pre-compliance testing during development saves time and money.

**Pre-compliance equipment:**

1. **Spectrum analyser:** Covers 9 kHz to 3 GHz. Used to observe the EMI signature without a LISN. Example: Rigol DSA815 (low cost), R&S FPC (mid-range).

2. **LISN (Line Impedance Stabilisation Network):**
   - Stabilises the impedance seen by the EUT (Equipment Under Test) to 50 µH || 50 Ω
   - Provides a 50 Ω port to connect the spectrum analyser
   - Separates the conducted noise from the mains
   - Options: commercial LISN (Fischer F50-4, Beehive BL-1), DIY (50 µH + 50 Ω network)

3. **Near-field probe set:**
   - Small loop probes (for B-field / magnetic) and monopole/E-field probes
   - Used to identify location of EMI sources on the PCB
   - Connected to spectrum analyser
   - Not a substitute for formal measurement but invaluable for finding hot spots

4. **Current probe:**
   - Clamp-on current transformer measuring the conducted noise current
   - Useful for separating DM from CM currents

**Pre-compliance procedure:**

1. Set up EUT with LISN on both supply conductors
2. Connect LISN output to spectrum analyser (use quasi-peak detector, 9 kHz RBW, 120 kHz VBW)
3. Run the converter at worst-case conditions (full load, nominal input) and at each voltage
4. Measure from 150 kHz to 30 MHz
5. Compare to CISPR Class B limits with 6 dB margin target
6. If a frequency exceeds the limit: use near-field probe to identify source
7. Try filter modifications: additional X-caps (DM), additional Y-caps (CM), CM choke
8. Iterate until 6 dB below limits across the full frequency range

**Common findings:**

- Conducted emission peaks at fsw and harmonics (DM noise)
- Broadband floor elevated (often CM noise from switch node coupling)
- Resonant peaks in the filter (underdamped LC filter resonance)

---

### Q8. What is spread spectrum and how much EMI reduction does it provide?

**Answer:**

**Spread spectrum (SSM) principle:**

Instead of a fixed switching frequency fsw, the frequency is modulated (dithered) ±Δf around the nominal. The energy in the EMI spectrum is spread over a band 2Δf wide rather than concentrated at a single frequency.

**Modulation waveforms:**

1. **Triangular (sawtooth) modulation:** Frequency ramps linearly from fsw - Δf to fsw + Δf and back. Power spectral density is flat across the band.
2. **Pseudorandom:** More natural-sounding if audio frequencies are involved; better peak EMI reduction.

**EMI reduction calculation:**

The EMI measurement uses a quasi-peak (QP) detector with 9 kHz bandwidth (CISPR 16) for the 150 kHz – 30 MHz range. The QP reading depends on how many "hits" the EMI peak makes within the detector charge/discharge time constant.

For a triangular SSM with Δf = ±5% around fsw = 200 kHz (spread from 190 kHz to 210 kHz = 20 kHz band):
```
Time for one frequency sweep (assume 1 ms modulation period):
  The signal spends (9 kHz / 20 kHz) × 0.5 ms = 225 µs at any given 9 kHz wide slot

Quasi-peak detector charge time: ~1 ms at 9 kHz
The detector only sees "hits" about 22.5% of the time → QP reading is reduced

QP reduction ≈ 20 × log(9 kHz / 20 kHz) = -6.9 dB (rough)
More detailed analysis: typically 10–15 dB QP reduction at the fundamental

Average detector reduction: 20 × log(bandwidth / 9kHz) = 20 × log(20/9) = 6.9 dB  [additional vs QP]
```

**Practical results:**

Typical measured EMI reduction with SSM: 6–12 dB at the fundamental frequency. Harmonics see less reduction (because at higher harmonics, the spread is wider relative to the fundamental spread, but the QP detector integrates differently).

**Limitations:**
- Harmonics may not benefit as much as the fundamental
- The total energy is conserved — SSM does not eliminate the EMI, only spreads it
- Some applications prohibit SSM (e.g., PLC-over-powerline communication, where certain frequencies must be avoided)

---

### Q9. How does the PCB layout directly affect conducted EMI?

**Answer:**

PCB layout is the primary determinant of conducted EMI before any filtering is applied. The switching current loop geometry sets the noise source impedance that the filter must attenuate.

**Mechanism — switch node parasitic capacitance:**

The switch node (SW) changes from 0 to Vin (or 0 to Vin + reverse recovery) at dV/dt = 5–50 V/ns. Any capacitance from SW to earth ground creates CM current:
```
I_CM = C_stray × dV_SW/dt
```

Sources of C_stray:
- PCB copper pour on SW node (over a ground plane): C ≈ ε₀εr × A/d = 370 pF/cm² at 0.1mm FR4
- MOSFET drain to heatsink (if heatsink is grounded): C_pad = 50–500 pF
- Inductor winding to core (if core grounded): C_winding ≈ 10–50 pF

**Minimising switch node copper area:**

```
Bad layout: SW copper pour extends 5 cm × 5 cm = 25 cm²
→ C_stray = 25 × 370 pF = 9250 pF
→ At dV/dt = 10 V/ns: I_CM = 9250×10⁻¹² × 10⁹ = 92.5 mA → severe CM noise

Good layout: SW copper pour minimized to 1 cm × 1 cm = 1 cm²
→ C_stray = 370 pF
→ I_CM = 370×10⁻¹² × 10⁹ = 3.7 mA → much more manageable
```

**Differential-mode noise from input loop:**

The switching current pulse flows in the input hot loop. The high-frequency spectral content of this pulse (steep rise/fall times) creates DM noise:
```
Spectrum of trapezoidal pulse:
Below 1/(π×t_rise):  -20 dB/decade envelope
Above 1/(π×t_rise): -40 dB/decade envelope

For t_rise = 10 ns: corner at 31.8 MHz → below 31.8 MHz, -20 dB/decade
```

The amplitude at 200 kHz is primarily determined by the duty cycle and magnitude, filtered by the input capacitor. But the high-frequency content (at FSW harmonics up to 30 MHz) depends on the current waveform shape — and that is partly set by layout (stray inductance slowing the edges, capacitance adding ringing).

**Layout actions for EMI reduction (before filter):**

1. Minimise switch node copper area → reduce CM capacitance
2. Minimise hot loop area → reduce stray inductance → cleaner current edges
3. Orient inductor gap away from PCB/chassis
4. Avoid running power traces near chassis ground connections
5. Use slower gate drive (higher R_gate) to reduce dI/dt and dV/dt → lower EMI but higher switching losses

---

### Q10. What is X-capacitor and how does it differ from Y-capacitor?

**Answer:**

**X-capacitors (Class X, safety-rated):**

X-capacitors are connected directly across the supply conductors (L to N). They filter differential-mode noise. If they fail, the only result is loss of filtering (they short L to N — acceptable for DM filter, as the circuit is already connected L-to-N). Therefore, their failure mode must be OPEN (fail-open or no sustained arcing).

Classes:
- X1: ≥ 4 kV peak (for heavy electrical environment)
- X2: ≥ 2.5 kV peak (most common in consumer equipment, mains supplies)
- X3: ≥ 1.2 kV peak (for local distribution mains)

**Y-capacitors (Class Y):**

Y-capacitors connect from L or N to earth ground. If they fail short, the chassis becomes live — dangerous. Must be extremely reliable, fail-safe, and rated for continuous mains-to-earth voltage. Never replace with a standard capacitor.

**Summary table:**

| Parameter          | X-capacitor (X2)         | Y-capacitor (Y2)           |
|--------------------|--------------------------|----------------------------|
| Connection         | L to N (across mains)    | L or N to Earth            |
| Failure mode       | Must fail open           | Must NOT fail short        |
| Peak voltage rating| ≥ 2.5 kV (X2)            | ≥ 5 kV pulse (Y2)          |
| AC voltage rating  | 275V AC (X2)             | 300V AC (Y2)               |
| Noise filtered     | Differential-mode        | Common-mode                |
| Typical value      | 0.1–2.2 µF               | 2.2–10 nF                  |
| Safety concern     | Low (L-N short is normal)| High (L-Earth short = shock)|
| Standards          | IEC 60384-14, UL 1414     | IEC 60384-14, UL 1414       |
| Max leakage current| N/A                      | ≤ 3.5 mA (IEC 62368)       |

**DM filter use of X-caps:**

Multiple X-caps in the input filter provide the capacitance in the LC DM filter. Values:
- C_X1 (before CM choke): 100 nF to 1 µF (bulk X2)
- C_X2 (after CM choke): 47 nF to 470 nF (finer filtering)

The X-caps handle the large ripple current from the DM noise current. They must be rated for the RMS ripple current that will flow through them.

---

### Q11. Describe a complete two-stage EMI filter design for a 100W isolated flyback converter.

**Answer:**

**Converter characteristics (noise source):**
```
Vin = 85–264 Vac, Vout = 12V, Pout = 100W, fsw = 65 kHz
Primary current peak ≈ 3A, dI/dt ≈ 3A/200ns = 15 A/µs
dV/dt at switch node ≈ 380V/200ns = 1.9 V/ns
```

**Stage 1 (near power cord entry):**

Provides bulk filtering; must handle full mains current.

```
Components:
  L1 (CM choke, stage 1): 10 mH, 2A rating, toroid ferrite
  C_Y1, C_Y2: 2.2 nF each (L/N to earth), Y2-rated
  C_X1: 470 nF, X2-rated (L to N)
  MOV1: varistor for surge protection (385V, 40J rating)
  F1: fuse (2A slow-blow for 100W at 85Vac input)
```

**Stage 2 (between CM choke and rectifier):**

Finer DM filtering:
```
Components:
  L2 (DM choke or leakage of CM choke): 100 µH
  C_X2: 100 nF, X2-rated
  C_Y3, C_Y4: 1 nF each (to earth/chassis — optional second stage)
```

**Complete filter topology:**

```
L (Live) ─── F1 ─── MOV1 ─── L1a ─────────────── L2 ─── Bridge (+)
              │       │        │                    │
              ├─CX1──┤       CY1↕  ↕CY2            ├─CX2─┤
              │               │                    │
N (Neutral) ──────────────── L1b ──────────────────────── Bridge (-)
              │               │
            Safety GND  ─────┘ (Earth connection)
```

**Required attenuation calculation:**

At fsw = 65 kHz, noise level at LISN without filter (estimate):
```
DM: I_switch ≈ 3A peak, V_DM ≈ 3A × 25Ω = 75 V = 137.5 dBµV
CM: I_CM = C_stray × dV/dt ≈ 100pF × 1.9V/ns = 190 mA
    V_CM ≈ 0.19A × 50Ω = 9.5 V = 139.5 dBµV

CISPR limit at 65 kHz: 66 dBµV (class B, quasi-peak)
Required attenuation: 139 - 66 - 6 = 67 dB
```

**Verify filter achieves 67 dB at 65 kHz:**

DM filter (L2 + CX2 = 100µH + 100nF):
```
f_LC_DM = 1/(2π√(100µH × 100nF)) = 1/(2π × 100µs) = 1.59 kHz

At 65 kHz: attenuation = 40 × log(65/1.59) = 40 × 1.61 = 64.3 dB (Stage 2 alone)
With Stage 1 L1-CX1 (10mH effective DM leakage + 470nF):
f_LC_1 = 1/(2π√(10µH × 470nF)) ≈ 1/(2π × 68.6µs) = 2.32 kHz [leakage ≈ 1% of CM = 100µH]
At 65 kHz: additional 40 × log(65/2.32) = 40 × 1.447 = 57.9 dB

Total DM: ~64 + 58 = 80 dB (better than required 67 dB with margin ✓)
```

---

## Advanced (Questions 12–16)

---

### Q12. How does the transformer inter-winding capacitance contribute to CM EMI and how do you mitigate it?

**Answer:**

**The coupling mechanism:**

In an isolated flyback transformer, the primary switch node voltage swings from 0 to Vin + reflected voltage (100–400+ V) at each switching transition. The inter-winding capacitance (C_ps between primary and secondary) couples this voltage to the secondary:
```
I_CM_via_transformer = C_ps × dV_SW/dt

For C_ps = 50 pF, dV_SW = 400V, t_rise = 50ns:
I_CM = 50×10⁻¹² × 400/50×10⁻⁹ = 50×10⁻¹² × 8×10⁹ = 400 mA — very significant
```

This current flows into the secondary ground and returns via the Y-capacitors and safety earth. It appears as common-mode noise on both output conductors.

**Mitigation — Faraday shield:**

Insert a thin copper layer (shielding winding) between primary and secondary. Connect this shield to primary ground (chassis-referenced primary GND):

```
Primary winding ─── Shield (→ primary GND) ─── Insulation ─── Secondary winding
```

The CM current from the primary now flows: switch node → C_ps_to_shield → shield → primary GND.
It does NOT flow to the secondary. The secondary is isolated from the primary switch node voltage swing.

**Important:** The shield must form a single-turn winding (copper foil layer), connected at only ONE end to prevent it from being a shorted turn (which would cause circulating currents and high loss). Both ends connected = shorted turn = transformer failure.

**Quantification of reduction:**

A properly designed Faraday shield can reduce the effective C_ps by 10–50× (reducing CM noise proportionally).

**Trade-off:** The shield adds leakage inductance between primary and secondary (another copper layer between them). For minimising leakage, the shield should be as thin as possible (0.05–0.1 mm copper foil).

---

### Q13. Explain the LISN and how it is used for conducted EMI measurement.

**Answer:**

**LISN — Line Impedance Stabilisation Network:**

The LISN is a passive network that:
1. Presents a defined impedance to the EUT (Equipment Under Test) at high frequency
2. Provides isolation of the EUT from mains impedance variations
3. Presents a 50 Ω measurement port for the spectrum analyser

**LISN circuit (per conductor, CISPR 16 specification):**
```
Mains terminal ──── 50 µH ──── EUT terminal
                      │
                    1000 µF (to earth)  [passes mains frequency, blocks HF to mains]
                      │
                    50 Ω resistor ──── Measurement port (to SA)
                      │
                    0.1 µF ──── Earth ground  [blocks mains DC, passes HF to measurement]
```

**Impedance presented to EUT:**

At mains frequency (50/60 Hz): impedance ≈ 0 Ω (50 µH is ~15 mΩ at 50 Hz, 1000 µF shorts mains)
At 150 kHz and above: impedance ≈ 50 Ω (50 µH is high impedance, 1000 µF is short, 50Ω dominates)

The LISN standardises the source impedance seen by the EUT regardless of the actual mains impedance at the test facility. This ensures repeatability of measurements between laboratories.

**Using a LISN for DM/CM separation:**

Two LISNs (one per conductor):
```
V_L = V_DM/2 + V_CM   (measurement on L conductor)
V_N = -V_DM/2 + V_CM  (measurement on N conductor)

DM voltage: V_DM = V_L - V_N = (V_L - V_N)
CM voltage: V_CM = (V_L + V_N)/2

Total conducted noise on one line: V_L = combination of DM and CM
```

**EMI pre-compliance using a DIY LISN:**

Commercial LISNs (Fischer F-50-4) cost $3,000+. A DIY LISN can be built from:
- 50 µH inductors (two, in series on each conductor)
- 10 µF capacitors (to earth)
- 50 Ω / 1W resistors with BNC connector

This provides a reasonable approximation for pre-compliance work (±6 dB accuracy) at a fraction of the cost.

---

## Quick Reference: EMC Filter Design Checklist

```
INPUT FILTER DESIGN CHECKLIST:

1. Estimate unfiltered DM and CM noise levels at fsw
2. Calculate required attenuation (noise - limit - 6dB margin)
3. Design DM filter: LC corner at fsw / 10^(attenuation_dB/40)
4. Design CM filter: CM choke L_CM with Y-caps
5. Add damping to DM filter (R-C damping to prevent Middlebrook instability)
6. Verify Y-cap leakage current ≤ safety limit
7. Rate X-caps for mains voltage + transient margin
8. Choose CM choke with I_rated ≥ I_in_max

EMI COMPLIANCE STANDARDS QUICK REFERENCE:
CISPR 32 Class B:  66 dBµV QP at 150-500 kHz; 56 dBµV QP at 500k-30 MHz
CISPR 32 Class A:  79 dBµV QP at 150-500 kHz; 73 dBµV QP at 500k-30 MHz
FCC Part 15B:      Same as CISPR 22 Class B (nearly equivalent)
IEC 62368-1:       References CISPR 32 for emissions

COMPONENT RATINGS:
X2 capacitor: 275V AC continuous, 2.5 kV peak pulse
Y2 capacitor: 300V AC continuous, 5 kV peak pulse
CM choke voltage: ≥ mains_peak + 20% margin
```
