# Core Materials and Loss Calculation — Interview Preparation

## Overview

Selecting the correct magnetic core material is as important as sizing the core. Different materials have dramatically different loss characteristics, permeability vs frequency behaviour, saturation flux density, and Curie temperatures. This section covers the major material families, loss modelling (Steinmetz equation), and the B-H curve as a design tool. Material selection questions appear in almost every magnetics-focused power electronics interview.

---

## Key Equations Reference

### Steinmetz Equation (Core Loss)
```
P_core = Cm × f^α × B_peak^β × Ve   [W, for sinusoidal excitation]

or in loss density:   Pv = Cm × f^α × B_peak^β   [W/m³ or W/cm³ or mW/cm³]

Typical parameters for MnZn ferrite (e.g., N87, 3C94):
  Cm ≈ 1.5–5×10⁻⁷ (SI units, W/(m³·Hz^α·T^β))
  α ≈ 1.5–1.8  (frequency exponent)
  β ≈ 2.5–3.0  (flux density exponent)
```

### Improved Steinmetz Equation (iSE) for Non-Sinusoidal Waveforms
```
Pv = Cm × feq^(α-1) × (ΔB)^β × fsw

feq = (2 / (ΔB^2 × π²)) × ∫(dB/dt)² dt   [equivalent frequency]
ΔB = peak-to-peak flux density swing
```

### Skin Depth in Core Material
```
δ_core = √(2ρ / (ω × µ))   [m]

For ferrite: ρ = 10² to 10³ Ω·m → skin depth >> core dimensions (non-issue)
For powdered iron: ρ = 10⁻⁴ Ω·m → skin depth can matter at very high f
For laminated silicon steel: ρ = 4×10⁻⁷ Ω·m → thin laminations needed above 1 kHz
```

### Permeability vs Frequency
```
Initial permeability µi: measured at low frequency, low flux density
Complex permeability: µ = µ' - jµ''
  µ': real part (energy storage), µ'': imaginary part (loss)
  Loss tangent: tan(δ) = µ''/µ' → increases with frequency
```

### Copper Loss vs Core Loss Trade-off (optimal B)
```
Optimal B_max (minimising total loss):
  dP_total/dB = 0 → P_Cu ≈ P_core × β/(α × 2)   [approximate]
  → At optimum: P_Cu ≈ P_core for typical ferrites (β/2α ≈ 0.8)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. What are the main ferrite types used in switching power supplies and when do you use each?

**Answer:**

Ferrites are ceramic magnetic materials made from iron oxide combined with other metal oxides. The two main types for power electronics are:

**MnZn (Manganese-Zinc) ferrite:**

- Composition: MnO + ZnO + Fe₂O₃ sintered ceramic
- Relative permeability (µᵢ): 1,000 to 15,000 (high — lots of inductance per turn)
- Saturation flux density (B_sat): 400–500 mT at 25°C (drops to 200–350 mT at 100°C)
- Resistivity: 1–10 Ω·m (low for a ceramic — eddy currents possible)
- Frequency range: DC to ~3 MHz (optimal: 10 kHz to 1 MHz)
- Loss characteristic: very low loss at 100 kHz–500 kHz

**Applications:** The standard choice for power inductors and transformers in switching supplies at 20 kHz to 1 MHz. Examples: Ferroxcube 3C94, TDK N87/N95, Fair-Rite #77/#78 material.

**NiZn (Nickel-Zinc) ferrite:**

- Composition: NiO + ZnO + Fe₂O₃
- Relative permeability: 10 to 2,000 (lower than MnZn)
- B_sat: 150–350 mT (lower than MnZn)
- Resistivity: 10⁴ to 10⁶ Ω·m (very high — no eddy currents in core)
- Frequency range: 1 MHz to >1 GHz (optimal: 1–300 MHz)
- Loss: very low at high frequency due to high resistivity

**Applications:** EMI filter beads, high-frequency signal transformers, antenna rods, common-mode chokes at radio frequencies. NOT used for power inductors (too low B_sat and µᵢ).

**When to use which:**

| Requirement                     | Use MnZn              | Use NiZn              |
|---------------------------------|-----------------------|-----------------------|
| Power inductor at 100–500 kHz   | Yes (first choice)    | No (low µᵢ, low B_sat)|
| Power transformer at 500 kHz    | Yes                   | Marginal              |
| EMI suppression bead (>1 MHz)   | Poor (too lossy)      | Yes                   |
| Common-mode choke (50–400 kHz)  | Yes                   | Maybe (depends on f)  |
| Very high frequency (>3 MHz)    | No                    | Yes                   |

---

### Q2. What are powder core materials and when are they preferred over ferrite?

**Answer:**

**What are powder cores:**

Powder cores are made by compressing fine metallic particles (iron, iron alloy, or permalloy) mixed with insulating binder. The insulation between particles acts as a distributed air gap — unlike ferrite where the gap is concentrated, the gap in powder cores is spread throughout the volume.

**Major types:**

**1. Powdered iron (carbonyl iron, Micrometals series):**
- µᵢ = 10–100
- B_sat = 1.0–1.4 T (very high compared to ferrite)
- Cost: low
- Loss: high at frequencies above 100 kHz (iron resistivity is low → eddy currents)
- Best for: 50/60 Hz inductors, PFC inductors at 50–100 kHz, DC chokes

**2. Kool Mµ (iron-silicon-aluminum, Magnetics Inc.):**
- µᵢ = 26–125
- B_sat = 1.0 T
- Loss: lower than powdered iron, acceptable to 500 kHz
- Best for: PFC inductors (50–130 kHz), buck inductors, energy storage

**3. MPP (Molypermalloy Powder, 80% Ni-20% Fe-Mo):**
- µᵢ = 14–550 (widest range)
- B_sat = 0.75 T
- Loss: very low, excellent to 1 MHz
- Cost: highest
- Best for: precision inductors, audio, telecommunications, wherever core loss must be minimised

**4. High Flux (50% Ni-50% Fe):**
- µᵢ = 14–160
- B_sat = 1.5 T (highest of powder cores)
- Best for: energy storage with high DC bias, output inductors where flux is high

**Advantages of powder cores over ferrite:**

1. **Soft saturation:** Powder cores saturate gradually (permeability decreases softly) rather than abruptly. This gives some overcurrent tolerance that ferrite does not — the inductance rolls off gracefully rather than collapsing.

2. **High B_sat:** 1.0–1.5 T vs 0.3–0.5 T for ferrite → smaller core for the same energy storage.

3. **Distributed gap:** No fringing field from a discrete air gap → lower radiated EMI from the inductor.

4. **Stable with temperature:** Powder core permeability and B_sat are more stable across temperature than ferrite.

**Disadvantages:**

1. **Higher core loss at high frequency:** Metallic particles have lower resistivity than ferrite → more eddy currents at high frequency.

2. **Lower permeability:** Fewer turns for same inductance → more copper, more DCR.

3. **Larger volume:** For the same inductance, need more core volume than ferrite (lower µᵢ).

**Selection guide:** Use ferrite above 200 kHz for minimum core loss. Use powder cores for PFC inductors (50–130 kHz, high bias current), DC smoothing chokes (low frequency, high current), and wherever a soft saturation characteristic is desired.

---

### Q3. What is the B-H curve and how do you read it to predict inductor performance?

**Answer:**

**The B-H curve (magnetisation curve):**

The B-H curve plots magnetic flux density B (Tesla) on the vertical axis against magnetic field intensity H (A/m) on the horizontal axis.

**Key features:**

```
B
↑          ...................B_sat (saturation)
|       ./
|     ./
|    /  ← Linear region (slope = µ0 × µr)
|   /
|  / ← Knee (onset of saturation)
| /
|/___________________________________________→ H
0        H_c (coercivity)
```

**Reading the B-H curve:**

1. **Initial slope (origin):** The slope dB/dH = µ = µ0 × µr gives the initial permeability µᵢ. A steeper slope = higher µᵢ = more inductance per turn.

2. **Linear region:** For B << B_sat, inductance is constant and the material behaves linearly. Design operating point should be within this region.

3. **Knee / onset of saturation:** Above the knee, the slope (= µᵢ) decreases. The inductance begins to drop. The knee is often defined at the point where µᵣ drops to 70% or 50% of µᵢ.

4. **Saturation (B_sat):** The plateau where dB/dH → µ0 (permeability equals that of free space). Inductance is essentially zero — the core is saturated.

5. **Coercivity (H_c):** The H field required to reduce B to zero after magnetisation. For soft ferrites: very low H_c (< 50 A/m) → narrow hysteresis loop → low hysteresis loss.

**Hysteresis loop (dynamic excitation):**

Under AC excitation, the B-H curve traces a closed loop (hysteresis loop). The area of this loop equals the energy lost per unit volume per cycle:
```
W_hys = ∮ H dB   [J/m³ per cycle]
P_hys = W_hys × fsw   [W/m³]
```

Wider loop = more hysteresis loss. Soft ferrites and amorphous materials have very narrow loops (low hysteresis loss). Hard ferrites (permanent magnets) have wide loops.

**Practical use in design:**

- Read B_sat at the operating temperature (not just 25°C)
- Design peak flux B_pk at 60–70% of B_sat for temperature margin
- Check that B_pk is within the linear region (not in the knee) for small-signal inductance accuracy
- The slope at the operating B level gives the incremental permeability and thus the actual inductance at DC bias

---

### Q4. What is the Steinmetz equation and what are its limitations?

**Answer:**

**The Steinmetz equation:**

The classical Steinmetz equation models core loss per unit volume under sinusoidal excitation:
```
Pv = Cm × f^α × B_peak^β

where Pv = core loss density [W/m³ or mW/cm³]
      f   = frequency [Hz]
      B_peak = peak flux density [T]
      Cm, α, β = material-dependent constants (fitted to measured data)
```

**Physical basis:**

Core loss has two main components:
1. **Hysteresis loss:** Proportional to frequency (one loop per cycle) and B^n (n = 1.6–2 for original Steinmetz)
2. **Eddy current loss:** Proportional to f² and B² (classical eddy current in conducting slabs)

The Steinmetz equation is a curve-fit that lumps both effects into a single power-law expression. It is empirical, not derived from first principles.

**Using Steinmetz parameters:**

From the TDK N87 ferrite datasheet:
```
At 100°C: Cm = 2.23×10⁻⁷ (W·s^α·T^(-β)/m³), α = 1.58, β = 2.65
At 25°C: higher losses (temperature affects Cm)

For B_peak = 150 mT, f = 200 kHz:
Pv = 2.23×10⁻⁷ × (200,000)^1.58 × (0.150)^2.65
   = 2.23×10⁻⁷ × 1.34×10⁸ × 0.0112
   = 2.23×10⁻⁷ × 1.50×10⁶
   = 334,500 W/m³ = 335 mW/cm³

For Ve = 2000 mm³ = 2.0 cm³:
P_core = 335 × 2.0 = 670 mW = 0.67 W
```

**Limitations:**

1. **Sinusoidal waveform only:** The Steinmetz equation is valid for sinusoidal flux density. Switching converters produce trapezoidal flux waveforms (triangular ripple). The error can be 30–100%.

2. **No DC bias effect:** Core loss under DC bias differs from the loss at the same AC amplitude without DC bias (DC bias moves the operating point on the B-H curve, altering the incremental loop area). The Steinmetz equation does not account for this.

3. **Temperature dependence:** Cm, α, β all change with temperature. N87 ferrite has minimum loss near 100°C — using room temperature parameters underestimates loss at operating temperature.

4. **Minor loop effects:** Under partial reversal (minor hysteresis loops), the Steinmetz equation is not directly applicable.

**Corrections for switching waveforms:**

**Improved Generalized Steinmetz Equation (iGSE):**
```
Pv = (1/Ts) × ∫ Cm × |dB/dt|^α × (ΔB)^(β-α) dt
```

This integrates the instantaneous loss over the waveform period, giving more accurate results for non-sinusoidal waveforms. The equivalent frequency concept is:
```
feq = (2/π²) × (1/ΔB²) × ∫(dB/dt)² dt
Pv ≈ Cm × feq^(α-1) × ΔB^β × fsw
```

For a triangular flux waveform (buck converter ripple), iGSE typically predicts 20–40% less loss than classical Steinmetz with peak B value.

---

### Q5. What is permeability and how does it change with frequency and temperature?

**Answer:**

**Definition:**

Permeability µ describes the ability of a material to concentrate magnetic flux:
```
B = µ × H = µ0 × µr × H
```
- µ0 = 4π × 10⁻⁷ H/m (permeability of free space)
- µr = relative permeability (dimensionless, material property)

**Types of permeability:**

- **Initial permeability (µᵢ):** measured at very low H and low frequency — a material constant
- **Amplitude permeability (µa):** ratio of B_peak / (µ0 × H_peak) at finite amplitude — decreases as B approaches B_sat
- **Incremental permeability (µΔ):** dB/dH at a given DC bias — determines actual inductance at a bias point
- **Complex permeability (µ = µ' - jµ''):** used for AC modelling at high frequencies; µ'' represents loss

**Frequency dependence:**

At low frequencies, µᵢ is high and constant. As frequency increases:

1. **Snoek's law:** For ferrites, µᵢ × fres ≈ constant (fres = ferromagnetic resonance frequency). Higher µᵢ → lower fres → permeability rolls off sooner.

   For MnZn ferrites: µᵢ = 2000, fres ≈ 2 MHz (µᵢ × fres ≈ 4000 MHz, Snoek product)
   For NiZn ferrites: µᵢ = 200, fres ≈ 50 MHz (µᵢ × fres ≈ 10,000 MHz, higher Snoek product)

2. **Above fres:** Real permeability µ' drops sharply. Imaginary part µ'' peaks. The core becomes lossy and non-inductive.

**Temperature dependence:**

MnZn ferrite permeability variation with temperature:
- µᵢ increases as temperature rises from -40°C to the Curie temperature (typically 200°C for power ferrites, 130°C for some MnZn)
- B_sat decreases monotonically with temperature: typically 30–40% drop from 25°C to 100°C
- Core loss has a minimum at some temperature (often 70–100°C for power grade MnZn) — this is exploited by designing for operation at that temperature

**Design implications:**

Always characterise the material at the operating temperature:
- Minimum B_sat at maximum temperature → worst-case saturation margin
- Loss at operating temperature (may be lower than at room temperature for MnZn ferrite — beneficial)
- Permeability at operating temperature → actual inductance at temperature

Check the datasheet's B-H curve and Pv-vs-T curve at the target operating temperature range.

---

### Q6. What are amorphous and nanocrystalline materials and what advantages do they offer?

**Answer:**

**Amorphous metals:**

Amorphous (non-crystalline) metal alloys are produced by rapidly quenching molten metal so that atoms solidify without forming a crystal lattice. The resulting material has very low hysteresis loss because there are no grain boundaries to impede domain wall motion.

**Common types:**
- Metglas 2605SA1 (Fe-Si-B): µᵢ ≈ 100,000–300,000 at low frequency; B_sat = 1.56 T; Curie T = 395°C
- Metglas 2714A (Co-Fe-Ni-Mo-Si-B): µᵢ ≈ 250,000; very low core loss

**Advantages over ferrite:**
- Much higher B_sat (1.5 T vs 0.4 T for ferrite) → smaller core for same energy
- Very low coercivity (Hc < 1 A/m vs 10–100 A/m for ferrite) → very low hysteresis loss
- Usable at 50/60 Hz for power transformers and at 10–100 kHz for switching supplies

**Disadvantages:**
- Available only as thin ribbons (25–50 µm), must be wound in toroid form (cutting is difficult)
- Higher eddy current loss than ferrite at high frequency (lower resistivity than ferrite, must be used as thin ribbon)
- Brittle, difficult to work with mechanically

**Applications:** Current transformers, precision inductors, 60 Hz power transformers where low loss is critical, PFC inductors.

**Nanocrystalline materials:**

Produced by partially crystallising an amorphous metal base. Nano-scale crystals (10–20 nm) are embedded in an amorphous matrix. This gives:
- Very high permeability: µᵢ up to 100,000
- Low saturation: B_sat ≈ 1.2 T
- Very low core loss: lower than both MnZn ferrite and amorphous at 10–100 kHz
- Good temperature stability (Curie T > 450°C for nanocrystalline)

**Common brands:** Vitroperm (Vacuumschmelze), Finemet (Hitachi Metals)

**Applications:** Common-mode chokes with very high impedance at low frequency (EMC filters), current sense transformers, high-efficiency switching transformer cores where ferrite size is prohibitive.

**Performance comparison at 100 kHz, 100 mT:**

| Material              | Pv (mW/cm³)  | B_sat (T) | µi              |
|-----------------------|--------------|-----------|-----------------|
| N87 MnZn ferrite      | 100–150      | 0.49      | 2,000           |
| Kool Mµ powder        | 500–1000     | 1.0       | 26–125          |
| Powdered iron         | 2000–5000    | 1.4       | 10–100          |
| Amorphous (Metglas)   | 50–100       | 1.56      | 100,000+        |
| Nanocrystalline       | 20–50        | 1.2       | 10,000–100,000  |

---

## Intermediate (Questions 7–13)

---

### Q7. How do you apply the Steinmetz equation to a switching power supply with a non-sinusoidal flux waveform?

**Answer:**

**The problem:**

A buck converter produces a triangular flux density waveform in the inductor core:
```
B(t): ramps up during on-time at slope (Vin-Vout)/(Np×Ae×L)
       ramps down during off-time at slope -Vout/(Np×Ae×L)
Peak-to-peak ΔB = Vin×D×Ts / (Np×Ae)
```

This is not sinusoidal. Classical Steinmetz (which uses B_peak from a sinusoidal waveform) gives incorrect results if applied naively.

**Method 1: Use B_peak as if sinusoidal (simple but inaccurate):**
```
Pv ≈ Cm × f^α × (ΔB/2)^β   [treating ΔB/2 as if it were B_peak_sine]
```
This overestimates loss because for the same peak-to-peak swing, a triangular waveform has less harmonic content than a sine wave. Error: typically 20–100% overestimate.

**Method 2: Improved Generalised Steinmetz Equation (iGSE):**

For a piecewise linear waveform (like a triangular ripple), the iGSE integral simplifies:

For a triangular waveform with on-slope m1 = dB/dt_on and off-slope m2 = -dB/dt_off:
```
feq = (1/π²) × [m1² × D + m2² × (1-D)] / (ΔB/2)² × (1/fsw)
```

Or using the simplified formula for a symmetrical triangle (m1 = m2 = ΔB × fsw):
```
feq = (2/π²) × (ΔB × fsw × 2/ΔB)² / fsw² × 1 = 8fsw/π² × (ratio of slopes)
```

For a non-symmetrical triangle (D ≠ 0.5):
```
feq = (2×fsw/π²) × [D × m1² + (1-D) × m2²] / (ΔB/2)²

With m1 = ΔB/(D×Ts) and m2 = ΔB/((1-D)×Ts):
feq = (2×fsw²/π²) × [D/(D×Ts)² + (1-D)/((1-D)×Ts)²] × (2/ΔB)² × ΔB²/4
    = (2/π²) × fsw² × [1/(D×Ts) + 1/((1-D)×Ts)]
    = (2×fsw/π²) × [1/D + 1/(1-D)]
    = (2×fsw/π²) / (D×(1-D))
```

Steinmetz loss with iGSE:
```
Pv_iGSE = Cm × feq^(α-1) × ΔB^β × fsw
```

**Example (buck converter):**
```
fsw = 200 kHz, D = 0.4, ΔB = 80 mT, N87 at 100°C: Cm=2.23×10⁻⁷, α=1.58, β=2.65

feq = (2 × 200k / π²) / (0.4 × 0.6) = 40,528 / 0.24 = 168,867 Hz

Pv = 2.23×10⁻⁷ × (168,867)^0.58 × (0.08)^2.65 × 200,000
   = 2.23×10⁻⁷ × 3,024 × 0.000329 × 200,000
   = 2.23×10⁻⁷ × 1.990×10⁵
   = 44.4 mW/cm³

Compared to classical Steinmetz with B_peak = ΔB/2 = 40 mT:
Pv_classical = 2.23×10⁻⁷ × (200k)^1.58 × (0.040)^2.65
             = 2.23×10⁻⁷ × 1.27×10⁸ × 0.000490 × ...  [much higher]
```

---

### Q8. How does core loss change with temperature and how do you account for it in design?

**Answer:**

**Temperature dependence of core loss:**

MnZn ferrite core loss exhibits a strong and non-monotonic dependence on temperature:

- At low temperatures (< room temperature): loss is HIGH
- At room temperature: moderate loss
- At moderate temperature (60–100°C): loss reaches a MINIMUM (for power-grade ferrites)
- At high temperature (> 150°C): loss increases sharply and material approaches Curie point

This unusual behaviour means that MnZn ferrites are often designed to run at 80–100°C to minimise core losses — counter-intuitive (unlike copper, which always benefits from lower temperature).

**Quantitative example (N87 ferrite):**

At B_peak = 200 mT, f = 100 kHz:
```
Temperature   Pv (mW/cm³)
25°C          ~300
60°C          ~200
100°C         ~150  (minimum)
120°C         ~200
150°C         ~500
```

**Design implications:**

1. **Do not use room-temperature Pv for final design:** The minimum loss occurs at the operating temperature. Using 25°C data overestimates loss for converters that run at 80–100°C.

2. **Thermal runaway risk:** Above the loss minimum, loss increases with temperature. If the core heats up due to increasing loss, and the heating causes more loss, a runaway condition can develop. More likely in isolated environments (no airflow) or overloaded converters.

3. **B_sat derating:** B_sat drops with temperature. The core must not saturate at the maximum operating temperature, not just at 25°C. N87 B_sat ≈ 490 mT at 25°C, ≈ 330 mT at 100°C — a 33% reduction.

4. **Combined derating:** At maximum temperature: lower B_sat AND potentially higher core loss (if operating above the loss minimum). Both effects constrain the design at high temperature.

**Accounting for temperature in design:**

1. Calculate operating temperature first (iterate): estimate losses → calculate ΔT → get operating T
2. Look up Pv at that temperature from the datasheet plot
3. Recalculate losses with temperature-corrected parameters
4. Verify B_sat at operating temperature provides adequate margin
5. Iterate until convergence (usually 2–3 iterations)

---

### Q9. What is the Curie temperature and why does it matter for converter reliability?

**Answer:**

**Curie temperature (Tc):**

The Curie temperature is the temperature above which a ferromagnetic material loses its permanent magnetism and becomes paramagnetic (µr → 1). For practical magnetic design, it marks the temperature at which the material becomes useless as a magnetic core.

**Values for common materials:**

| Material              | Curie Temperature |
|-----------------------|-------------------|
| MnZn ferrite (power)  | 200–230°C         |
| NiZn ferrite          | 300–350°C         |
| Powdered iron         | 770°C             |
| Silicon steel         | 745°C             |
| Amorphous (Metglas)   | 370–430°C         |
| Nanocrystalline       | 450–600°C         |

**Why it matters:**

A converter operating near Tc (due to overload, cooling failure, or ambient extremes) will have:
1. Rapidly declining B_sat → easier saturation at lower currents
2. Declining permeability → inductance collapse → uncontrolled current rise
3. Above Tc: inductance → 0 → no current limiting → component failure

**Safety margin:**

Design so the core temperature under any fault condition (including loss of cooling) stays at least 50°C below Tc:
```
T_core_max ≤ Tc - 50°C

For N87 ferrite (Tc = 220°C): T_core_max ≤ 170°C
```

**Practical thermal margins:**

Insulation class of winding wire limits practical temperature more than Curie temperature for most power ferrites:
- Class B wire insulation: 130°C max
- Class F wire: 155°C max
- Class H wire: 180°C max

For ferrite cores with Tc = 220°C: the wire insulation typically fails before the core reaches Curie. Specifying class F or H wire provides temperature margin.

For amorphous material (Tc = 395°C): much more headroom — the winding insulation and bobbin material are the limiting factors.

---

### Q10. Explain how to perform a loss breakdown analysis for a complete inductor (copper vs core).

**Answer:**

**Loss breakdown procedure:**

**Given:**
```
Inductor: L=10µH, N=8 turns on E25/13/7 core, 3-layer winding
Converter: buck, Vin=12V, Vout=5V, Iout=3A, fsw=200kHz, D=0.417
Core material: N87 ferrite, Ve=2.1cm³, Ae=52mm²
Wire: AWG 26 (d=0.405mm, Aw=0.129mm²), MLT=30mm
```

**Step 1: Flux density swing**
```
ΔB = Vin × D × Ts / (N × Ae) = 12 × 0.417 × 5µs / (8 × 52mm²)
   = 12 × 0.417 × 5×10⁻⁶ / (8 × 52×10⁻⁶)
   = 25×10⁻⁶ / 416×10⁻⁶
   = 0.0600 T = 60 mT
B_peak = ΔB/2 = 30 mT  (or 60mT if unipolar — for an inductor in a buck, it's unipolar:
        B goes from B_avg - ΔB/2 to B_avg + ΔB/2, NOT from 0 to ΔB)
```

**Step 2: Core loss (iGSE)**
```
feq = (2 × fsw/π²) / (D × (1-D)) = (2 × 200k/9.87) / (0.417 × 0.583)
    = 40,528 / 0.243 = 166,700 Hz

N87 parameters at 100°C: α=1.58, β=2.65, Cm=2.23×10⁻⁷

Pv = Cm × feq^(α-1) × ΔB^β × fsw
   = 2.23×10⁻⁷ × (166,700)^0.58 × (0.060)^2.65 × 200,000
   = 2.23×10⁻⁷ × 3,000 × 0.000195 × 200,000
   = 2.23×10⁻⁷ × 1.17×10⁵
   = 26.1 mW/cm³

P_core = 26.1 × 2.1 = 54.8 mW ≈ 0.055 W
```

**Step 3: DC copper loss**
```
DCR = ρ × MLT × N / Aw = 1.72×10⁻⁸ × 0.030 × 8 / 0.129×10⁻⁶
    = 4.13×10⁻⁹ / 1.29×10⁻⁷ = 0.032 Ω = 32 mΩ

I_rms ≈ Iout = 3A (small ripple approximation)
P_Cu_DC = 3² × 0.032 = 0.288 W
```

**Step 4: AC copper loss**
```
Skin depth at 200 kHz: δ = 66.5/√200000 = 0.149 mm
Wire diameter: d = 0.405 mm → d/2δ = 1.36 → some skin effect

Dowell factor (approximate, 3 layers):
Δ = d/δ × √(fill factor) ≈ 1.36 × √0.5 = 0.96

F_R (3 layers at Δ=0.96):
F_skin ≈ 0.96 × [sinh(1.92)+sin(1.92)] / [cosh(1.92)-cos(1.92)] ≈ 1.15
F_prox = (9-1)/3 × Δ × [sinh(Δ)-sin(Δ)] / [cosh(Δ)+cos(Δ)]
       = (8/3) × 0.96 × [sinh(0.96)-sin(0.96)] / [cosh(0.96)+cos(0.96)]
       = 2.67 × 0.96 × [1.150 - 0.820] / [1.571 + 0.574]
       = 2.56 × 0.330 / 2.145
       = 0.394

F_R = F_skin + F_prox = 1.15 + 0.394 = 1.54

P_Cu_AC = F_R × P_Cu_DC × (ΔIL/Iout)²  [AC loss applies to ripple current only]
ΔIL = ΔB × N × Ae / L = 0.060 × 8 × 52×10⁻⁶ / 10×10⁻⁶ = 2.496 A/10 = ...
[recalculate: ΔIL = (Vin-Vout)×D/(fsw×L) = 7×0.417/(200k×10µ) = 1.46 A]
I_AC_rms = ΔIL / (2√3) = 1.46/3.46 = 0.42 A

P_Cu_AC = (F_R - 1) × R_DC × I_AC_rms² = (1.54-1) × 0.032 × 0.42² = 0.54 × 0.032 × 0.18 = 3.1 mW
```

**Total losses:**
```
P_Cu = 0.288 + 0.003 = 0.291 W   (AC loss is small here — low ripple ratio)
P_core = 0.055 W
P_total = 0.346 W

Loss split: Cu = 84%, Core = 16%
Copper loss dominates → consider increasing wire gauge (AWG 24) or reducing turns (if saturation allows)
```

---

### Q11. What is permeability vs DC bias and why must inductors be characterised at operating current?

**Answer:**

**Permeability vs DC bias characteristic:**

When a DC bias current flows through an inductor, it establishes a steady-state magnetic flux in the core. As DC bias increases:

1. The core operates at higher average B
2. The B-H curve's slope (= incremental permeability) decreases in the saturation region
3. The effective inductance L = µ_incremental × N² × Ae / le decreases

**Shape of L vs I_DC curve:**

- At I = 0: L = L_rated (maximum)
- As I increases: L remains flat until approaching H_sat
- Near saturation: L begins to drop (initially slowly, then rapidly)
- At I_sat: L = 70% of L_rated (typical definition)
- Above I_sat: L falls sharply

Ferrite has a fairly abrupt saturation (clear knee). Powder cores (Kool Mµ, MPP) have a much softer saturation (gradual, gentle decline).

**Why this matters for converter design:**

A buck converter designed for L = 10 µH at I_DC = 0 may actually have L = 6 µH at I_DC = 5A (if the core is not sized properly). Consequences:
```
ΔIL_actual = Vout × (1-D) / (fsw × L_actual)
           = Vout × (1-D) / (fsw × 6µH)  [50% larger than designed]
```

The converter may enter DCM at rated load, the filter capacitor needs to be larger to handle the extra ripple, and the peak current is higher than planned (saturation risk worsens).

**How to correctly characterise:**

LCR meters with DC bias capability (e.g., Keysight E4980A with DC bias port) apply a known DC current while measuring inductance with a small AC signal at the switching frequency:
```
Measurement setup:
1. Set DC bias to I_avg (average operating current)
2. Measure L at the operating frequency
3. Report L(I_bias) — this is the actual design value
```

Manufacturer datasheets typically show L vs I_DC curves. Always use the L value at I_DC = I_nominal for design calculations.

**Powder core advantage:**

MPP powder core at 50% saturation might retain 95% of its zero-bias inductance. A ferrite core at the same relative bias might retain only 80% (depending on design margins and gap). For applications where inductor saturation on transients is a concern, powder cores provide more graceful degradation.

---

### Q12. Compare core losses in a full-bridge converter versus a flyback converter for the same output power.

**Answer:**

**Full-bridge transformer (bipolar excitation):**

The core is driven with alternating positive and negative volt-seconds. The flux swings from +B_max to -B_max each cycle:
```
ΔB = 2 × B_max  [peak-to-peak swing]
B_max = Vin × D × Ts / (2 × Np × Ae)   [factor 2 from bipolar drive]
```

The entire B-H loop is traversed each cycle → full hysteresis loop area is lost each cycle.

**Flyback inductor (unipolar excitation, CCM):**

The flux swings from (B_DC - ΔB/2) to (B_DC + ΔB/2). It never crosses zero (in CCM). Only a minor hysteresis loop is traversed:
```
ΔB = L × ΔIL / (N × Ae)   [peak-to-peak ripple only]
B_DC = L × I_avg / (N × Ae)   [DC bias — major contribution]
```

**Core loss comparison at same Pout:**

For the full-bridge: ΔB = 2×B_max. The Steinmetz loss at B_max = 150mT:
```
Pv_FB = Cm × f^α × (150mT)^β
```

For the flyback in CCM: ΔB might be 40mT (small ripple on large DC bias):
```
Pv_flyback ≈ Cm × f^α × (20mT)^β  [using ΔB/2 as B_peak of the AC ripple]
```

At β = 2.65 and ratios of 150mT vs 20mT:
```
Ratio = (150/20)^2.65 = 7.5^2.65 = 284

Full-bridge core loss is 284× higher per unit volume than flyback at the same frequency.
```

But the full-bridge core is smaller (Ae larger, B_max can be higher as it's symmetric). The comparison at the same Pout requires also accounting for:
- Different core sizes needed for the two topologies
- Full-bridge transfers energy efficiently every half cycle; flyback every full cycle (flyback core works harder in terms of energy per unit volume)

**Practical takeaway:**

The full-bridge transformer has relatively small AC flux swing (B_max typically 100–200 mT bipolar) because it operates at high efficiency. The flyback's DC bias adds a large average flux but does NOT contribute to core loss — only the AC ripple does. For similar power and frequency, the full-bridge core often has lower total core loss in absolute terms (better core utilisation), but the flyback may have lower core loss density (less total swing) — depends on the specific design.

---

## Advanced (Questions 13–16)

---

### Q13. Explain the Modified Steinmetz Equation (MSE) and when it is preferred over iGSE.

**Answer:**

**The Modified Steinmetz Equation (MSE):**

The MSE was proposed by Reinert et al. (1999) as a simpler alternative to iGSE for non-sinusoidal waveforms:

```
Pv_MSE = Cm × feq^(α-1) × (ΔB)^β × fsw

where feq = (2/(ΔB²π²)) × ∫(dB/dt)² dt   [same equivalent frequency as iGSE]
```

This is identical in form to iGSE. The difference lies in how ΔB is defined and whether the DC bias is included.

**Key differences between MSE and iGSE:**

| Aspect                | MSE                        | iGSE                         |
|-----------------------|----------------------------|------------------------------|
| DC bias effect        | Does not account for it    | Does not account for it      |
| Minor loops           | Not handled explicitly     | Handled by decomposition      |
| Accuracy              | ~10-20% error typical      | ~5-10% error for simple waves |
| Complexity            | Simple to compute           | More complex integral          |
| Computational cost    | Low                        | Moderate                       |

**When MSE is preferred:**

- Quick estimates and design iterations where 20% accuracy is acceptable
- Piecewise linear waveforms where the integral is easy to compute analytically
- Tool-chain limitations (some simulators only implement MSE)

**When iGSE is needed:**

- High accuracy (< 10% error) required for thermal management
- Complex waveforms (resonant converters, partial switching)
- Minor loop decomposition (asymmetric duty cycles, varying flux excitation)

**DC-bias corrected Steinmetz:**

Neither MSE nor iGSE accounts for DC bias effects on core loss. At high DC bias (close to saturation), the incremental B-H slope is reduced, and the incremental loop area changes. The "DC bias enhanced Steinmetz" or experimental measurements at operating DC bias levels are required for accurate prediction in high-bias applications (such as flyback or PFC inductors).

---

### Q14. How do you choose between ferrite, powder cores, and amorphous material for a PFC inductor at 100 kHz?

**Answer:**

**PFC inductor requirements:**

A boost PFC inductor at 100 kHz (typical) must:
- Handle high DC bias (output voltage ~400V creates large magnetising current at low input voltage)
- Handle large current ripple (CCM PFC: ΔIL = 20-40% of peak input current)
- Operate over a wide current range (from near-zero at zero-crossing to I_peak at peak input)
- Maintain inductance over the full current range (saturation not allowed at peak)

**Ferrite (MnZn):**

Advantages:
- Very low core loss at 100 kHz (lowest of the three options in mW/cm³)
- High B_sat allows higher peak currents per unit volume

Disadvantages:
- Hard saturation: L collapses abruptly at I_sat → must be oversized for peak current with margin
- Gap required for energy storage → fringing EMI
- Inductance drops significantly near saturation (less forgiving)

Verdict: Feasible but requires careful design. Gap placement (pot core or section of E core) can minimise fringing. Lower total core loss.

**Powder cores (Kool Mµ or Kool Mµ Hf):**

Advantages:
- Soft saturation: L rolls off gradually → self-protecting (inductance softly decreases as current exceeds rated)
- Distributed gap: no concentrated fringing field → lower EMI
- B_sat ≈ 1.0 T → better energy storage per unit volume

Disadvantages:
- Higher core loss than ferrite at 100 kHz (but "Kool Mµ Hf" grade reduces this to comparable)
- More turns needed (lower µᵢ) → more DCR → more copper loss

Verdict: Industry standard choice for 50–130 kHz PFC inductors. Soft saturation, distributed gap, good energy density, acceptable core loss.

**Amorphous / nanocrystalline:**

Advantages:
- Lowest core loss of all three (important if 100 kHz is above the ferrite loss minimum)
- High B_sat (amorphous: 1.56 T, nanocrystalline: 1.2 T)
- Very high permeability with gap (gives flexibility in turns number)

Disadvantages:
- Only available as wound toroids (no E-cores for easy gapping)
- Brittle (difficult to cut/gap)
- Higher cost
- Gapping requires cutting the toroid → significant mechanical challenge

Verdict: Used in high-end power supplies where efficiency premium justifies the cost and manufacturing complexity. Very common in high-efficiency server power (> 95% efficiency target).

**Recommendation matrix:**

| Priority                   | Best Choice        |
|----------------------------|--------------------|
| Lowest cost                | Ferrite (gapped E) |
| Easiest manufacturing      | Kool Mµ toroid     |
| Lowest core loss           | Amorphous toroid   |
| Soft saturation            | Kool Mµ            |
| Highest power density      | Amorphous          |
| Standard PFC (fsw=65-130kHz)| Kool Mµ toroid    |

---

## Quick Reference: Core Material Comparison

| Property           | MnZn Ferrite  | NiZn Ferrite  | Kool Mµ     | MPP          | Amorphous   | Nanocrystalline |
|--------------------|---------------|---------------|-------------|--------------|-------------|-----------------|
| µᵢ (typical)       | 1000–15000    | 10–2000       | 26–125      | 14–550       | 10000+      | 10000–100000    |
| B_sat (T)          | 0.4–0.5       | 0.3–0.4       | 1.0         | 0.75         | 1.56        | 1.2             |
| Core loss (mW/cm³) at 100kHz, 100mT | 50–150 | very high | 300–800 | 100–400 | 50–100 | 20–50       |
| Freq range         | 1k–3M Hz      | 1M–1GHz       | DC–2M Hz    | DC–2M Hz     | DC–500k Hz  | DC–500k Hz      |
| Curie temp (°C)    | 200–230       | 300–500       | 770         | 450          | 395         | 600             |
| Saturation type    | Hard          | Hard          | Soft        | Soft         | Moderate    | Moderate        |
| Available shapes   | E/I/toroid    | E/I/toroid    | Toroid, E   | Toroid only  | Toroid only | Toroid only     |
| Relative cost      | Low           | Low           | Medium      | High         | Medium      | High            |
| Primary use        | Power magnetics| EMI, HF       | PFC, output | Precision    | PFC, CT    | CM chokes, CT   |
