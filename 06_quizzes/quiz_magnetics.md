# Quiz: Magnetics Design for Power Converters

20 multiple-choice questions covering inductor design, transformer design, core materials, and magnetics analysis. Each question has four options; only one is correct. Explanations cover why wrong answers are wrong.

**Scoring guide:** 18-20 correct = expert level; 14-17 = solid intermediate; 10-13 = review the magnetics section; below 10 = study Faraday's law and core fundamentals first.

---

## Q1

An inductor core has an AL value of 2500 nH/turn². How many turns are needed to achieve an inductance of 10µH?

- A) 2 turns
- B) 4 turns
- C) 6 turns
- D) 25 turns

**Correct answer: A**

**Explanation:**

The AL value (inductance factor) relates inductance to turns by:
```
L = AL × N²
N = sqrt(L / AL)
```

Converting units: AL = 2500 nH/turn² = 2500 × 10⁻⁹ H/turn², L = 10µH = 10,000 nH.

```
N = sqrt(10,000 nH / 2500 nH/turn²) = sqrt(4) = 2 turns
```

**Important formula to remember:**
```
L = AL × N²    →    N = sqrt(L / AL)
```

**Why B is wrong:** N = 4 would give L = 2500 × 16 = 40,000 nH = 40µH, not 10µH.

**Why C is wrong:** N = 6 gives L = 2500 × 36 = 90µH.

**Why D is wrong:** N = 25 gives L = 2500 × 625 = 1,562,500 nH ≈ 1.56mH, far too large.

---

## Q2

A transformer has a turns ratio of 10:1 (primary to secondary). If the primary has an inductance of 1mH (measured at the primary terminals with secondary open), what is the secondary inductance measured at the secondary terminals with the primary open?

- A) 1mH
- B) 0.1mH
- C) 0.01mH
- D) 10mH

**Correct answer: C**

**Explanation:**

The inductance of a winding scales with the square of the turns ratio. If the primary has N1 turns and inductance L1, the secondary with N2 turns has inductance:

```
L2 = L1 × (N2/N1)²
```

With N1/N2 = 10:1 → N2/N1 = 1/10:

```
L2 = 1mH × (1/10)² = 1mH × (1/100) = 0.01mH = 10µH
```

This makes physical sense because inductance depends on N²/R_reluctance. Both windings share the same core (same reluctance), so L ∝ N². The secondary has 1/10 the turns, so 1/100 the inductance.

**Why A is wrong:** 1mH would mean the inductance is the same regardless of turns — this ignores the N² dependence. Only if N1 = N2 would L1 = L2.

**Why B is wrong:** 0.1mH would correspond to scaling by 1/10 (turns ratio), not 1/100 (turns ratio squared). A common error is to forget the squaring: L scales as N², not N.

**Why D is wrong:** 10mH would be 10× the primary inductance, which would require N2/N1 = √10 ≈ 3.16:1, not 1:10.

---

## Q3

What is the purpose of an air gap in a power inductor core?

- A) To reduce the core's permeability and increase the energy storage capability
- B) To increase the core's inductance for a given number of turns
- C) To reduce eddy current losses in the core
- D) To provide a path for heat to escape from the winding

**Correct answer: A**

**Explanation:**

The energy stored in an inductor is E = ½ × L × I². For a given peak current, energy storage is proportional to L. However, the core material saturates at a maximum flux density B_sat. Without an air gap, the inductor saturates at a relatively low current, limiting energy storage.

An air gap increases the total reluctance of the magnetic circuit:
```
R_total = R_core + R_gap ≈ R_gap = l_gap / (µ0 × Ae)    (since µ_core >> µ0)
```

Higher total reluctance means:
- Lower inductance for same N: L = N² / R_total (so L decreases with gap)
- The same peak H field (N×I_peak/l_total) requires more Ampere-turns to reach B_sat
- Peak energy storable before saturation increases: E_max = ½ × L × I_sat²

The air gap "linearises" the inductance vs. current characteristic, allowing the inductor to operate at higher currents without saturating. The energy density in the gap is very high (E_gap = ½ × µ0 × H² per unit volume).

**Critical insight:** The air gap reduces inductance, which might seem counterproductive. But the saturation current increases more than the inductance decreases, so the peak storable energy E_max = ½ × L × I_sat² INCREASES overall.

**Why B is wrong:** This is the opposite of the truth. An air gap REDUCES inductance for a given N. To restore inductance after adding a gap, more turns are needed. The air gap's benefit is not higher inductance — it is higher saturation current.

**Why C is wrong:** Eddy current losses occur in the core material due to circulating currents induced by changing flux. An air gap does not prevent flux from entering the core (it only changes the reluctance), so eddy current losses in the core are not directly reduced by the air gap. (However, a gap near the winding can increase AC copper losses due to fringing flux — the opposite effect.)

**Why D is wrong:** Air gaps are magnetic gaps, not thermal gaps. Heat removal from the winding is managed by thermal paths to the core and then to ambient (heatsinking), not by air gaps in the magnetic path.

---

## Q4

The Steinmetz equation for core loss is Pv = Cm × f^α × B_peak^β. For a typical MnZn ferrite (N87), what are the approximate values of α and β?

- A) α = 1, β = 2 (linear in frequency, quadratic in flux)
- B) α ≈ 1.7, β ≈ 2.6
- C) α ≈ 1.0, β ≈ 1.0
- D) α ≈ 3.0, β ≈ 4.0

**Correct answer: B**

**Explanation:**

The Steinmetz parameters for MnZn power ferrite (N87 or equivalent, at temperatures around 100°C where core loss is minimised):

- α (frequency exponent): approximately 1.7 to 1.9 for N87 at 100kHz-1MHz. Often quoted as 1.74.
- β (flux density exponent): approximately 2.4 to 2.7 for N87. Often quoted as 2.58.

These exponents are empirically derived from curve fitting to measured core loss data. They are NOT simply 2 and 2 (which would be eddy current dominated) nor 1 and 1 (which would imply linear loss).

**Physical basis:**
- α > 1: Core loss increases faster than linearly with frequency. The excess above 1.0 is due to anomalous eddy current losses and domain wall movement losses.
- β > 2: Core loss increases faster than squared with flux density. Hysteresis loss (the area of the B-H loop) grows non-linearly with peak flux density.

**Why A is wrong:** α = 1 is the low-frequency approximation where hysteresis dominates purely (Physt ∝ f × B²_peak). At practical power frequencies, anomalous eddy current and excess losses cause α > 1. The Steinmetz equation with α = β = integer values is a simplification not useful for accurate design.

**Why C is wrong:** α = β = 1 implies loss proportional to f × B, which is far too optimistic. Real ferrite losses increase much faster with both frequency and flux density.

**Why D is wrong:** α = 3 and β = 4 are too high. These values would suggest extremely rapid loss increase with frequency and flux, characteristic of materials with very high eddy current losses. Power ferrites are specifically optimised to have low eddy currents (achieved via high resistivity, grain structure), giving moderate α and β values.

---

## Q5

A Faraday shield (electrostatic shield) is sometimes placed between the primary and secondary windings of a power transformer. What is its function?

- A) To increase the mutual coupling (reduce leakage inductance) between primary and secondary
- B) To reduce the common-mode noise current that flows through the inter-winding capacitance
- C) To improve the core's saturation characteristics by concentrating flux
- D) To provide thermal isolation between the hot primary and cooler secondary

**Correct answer: B**

**Explanation:**

Inter-winding capacitance (C_ps) exists between the primary and secondary windings due to their proximity. In a flyback or forward converter, the primary switching node swings at high dV/dt (e.g., 48V × 400kHz = dV/dt >> 10V/µs). This dV/dt drives displacement current through C_ps to the secondary:

```
I_CM = C_ps × dV/dt
```

This common-mode current propagates into the secondary circuit and the output, contributing to conducted EMI (measured by the LISN on the primary side as return path current). This is the primary mechanism for common-mode EMI generation in isolated converters.

A Faraday shield is a thin conductive sheet (copper foil, typically one turn, or copper tape) inserted between primary and secondary windings. It is:
- Connected to a quiet ground reference (often the primary-side return, or a safety earth)
- NOT closed as a shorted turn (which would reduce coupling and add loss)

The shield intercepts the capacitive displacement current and diverts it to the quiet ground, preventing it from reaching the secondary. The CM noise current now flows:

```
Primary switching node → C_ps1 → Faraday shield → ground (primary return)
```

Instead of completing through the secondary circuit.

**Why A is wrong:** A Faraday shield actually slightly INCREASES leakage inductance because it introduces another conductor layer and spacing. The shield does not link magnetic flux and therefore doesn't help with magnetic coupling. Reducing leakage inductance requires interleaving (P-S-P or S-P-S winding arrangements).

**Why C is wrong:** The Faraday shield is a thin electrostatic conductor, not a magnetic material. It has no effect on the core's flux distribution or saturation characteristics.

**Why D is wrong:** While the Faraday shield is typically made of copper with a thin dielectric (tape insulation), it is not an effective thermal barrier. The thermal resistance of a thin copper sheet is negligible. Thermal isolation between windings is not a function of the Faraday shield.

---

## Q6

In a flyback converter, the magnetising inductance of the transformer is 500µH. The converter operates at fsw = 100kHz with a duty cycle of D = 0.45. What is the peak magnetising current if Vin = 100V?

- A) 0.45A
- B) 0.9A
- C) 1.8A
- D) Cannot be determined without knowing the turns ratio

**Correct answer: B**

**Explanation:**

The magnetising inductance Lm is reflected to the primary side. During the switch on-time, Vin is applied across Lm:

```
Vm = Vin = 100V during ton
ton = D / fsw = 0.45 / 100kHz = 4.5µs

ΔIm = Vin × ton / Lm = 100V × 4.5µs / 500µH
    = 100 × 4.5×10⁻⁶ / (500×10⁻⁶)
    = 450×10⁻⁶ / 500×10⁻⁶ = 0.9A
```

In a DCM flyback (or at boundary), the magnetising current starts at zero and ramps to I_peak = 0.9A during the on-time. In a CCM flyback, there is also a DC bias.

The question asks for peak magnetising current assuming the converter starts each cycle from zero (DCM or boundary mode). The peak is 0.9A.

**Why A is wrong:** 0.45A is half the correct value; it would arise from using ton/2 or Vin/2 in error.

**Why C is wrong:** 1.8A is double the correct value (a factor-of-2 error). Using the full period instead of the on-time would give Vin × Tsw / Lm = 100 × 10µs / 500µH = 2A.

**Why D is wrong:** The peak magnetising current is determined by the primary-side voltage and the magnetising inductance on the primary side. The turns ratio determines how this magnetising current maps to secondary-side quantities, but the magnetising current itself is a primary-side quantity that can be calculated from Vin, D, fsw, and Lm. No turns ratio needed.

---

## Q7

What is the "skin depth" and why does it matter for inductor winding design at high frequency?

- A) The thickness of the core material that participates in magnetic flux conduction
- B) The depth below a conductor's surface at which the current density falls to 1/e of its surface value, causing AC resistance to increase at high frequency
- C) The minimum wire diameter needed to prevent mechanical damage during winding
- D) The maximum conductor thickness before eddy currents dominate core loss

**Correct answer: B**

**Explanation:**

At DC, current distributes uniformly across the cross-section of a conductor. At AC frequencies, the changing magnetic field induces eddy currents within the conductor that oppose the primary current near the conductor's centre, forcing the current toward the surface. This is the skin effect.

The skin depth δ is:
```
δ = sqrt(ρ / (π × f × µ))
```

For copper at room temperature: ρ = 1.68×10⁻⁸ Ω·m, µ ≈ µ0 = 4π×10⁻⁷ H/m:
```
δ(copper) = 66.5mm / sqrt(f_Hz) = 66.5mm / sqrt(f)
```

At 100kHz: δ = 66.5mm/√100,000 = 66.5mm/316 ≈ 0.21mm = 210µm
At 400kHz: δ ≈ 105µm = 0.105mm

If the wire diameter is much larger than 2δ (twice the skin depth = skin layer on both sides), the current only flows near the surface. The effective conducting cross-section is smaller than the physical cross-section, so the AC resistance RAC > RDC.

For a round wire of diameter d:
- If d << 2δ: RAC ≈ RDC (full cross-section conducts)
- If d >> 2δ: RAC ≈ RDC × d / (4δ) (current confined to a surface annulus of area ≈ π × d × δ, against π × d² / 4 at DC)

**Design rule:** Choose wire diameter ≤ 2δ at the operating frequency to keep RAC ≈ RDC. At 400kHz: use wire ≤ 0.21mm diameter (approximately AWG 33). For higher current, parallel multiple thin strands (or use Litz wire).

**Why A is wrong:** The skin depth of the core refers to a separate concept (how deeply electromagnetic fields penetrate the core material, related to core eddy currents). This is important for core design but is separate from the skin depth in the winding conductor.

**Why C is wrong:** Minimum wire diameter for mechanical handling is a manufacturing constraint (typically AWG 40 or so is the limit for manual winding). This is unrelated to the electromagnetic skin depth.

**Why D is wrong:** Eddy currents in the core material are related to the core material's skin depth, not the winding conductor's skin depth. Eddy current loss in the core is minimised by using high-resistivity materials and thin laminations or small grain sizes.

---

## Q8

A buck converter operates at 300kHz with Vin = 48V and Vout = 12V. The inductor has L = 10µH. What is the volt-second product (volt-seconds) applied to the inductor during each on-time?

- A) 8.33 V·µs
- B) 25 V·µs
- C) 30 V·µs
- D) 40 V·µs

**Correct answer: C**

**Explanation:**

In a buck converter, the voltage across the inductor during the on-time is:
```
VL_on = Vin - Vout = 48 - 12 = 36V
```

Duty cycle: D = Vout/Vin = 12/48 = 0.25

On-time: ton = D / fsw = 0.25 / 300kHz = 0.833µs

Volt-seconds = VL_on × ton = 36V × 0.833µs = 30 V·µs

Check with the off-time (volt-second balance): Vout × (1-D) / fsw = 12 × 0.75 / 300kHz = 30 V·µs. The two must be equal, and they are.

```
λ = (Vin - Vout) × ton = (Vin - Vout) × D / fsw = 36V × 0.25 / 300kHz = 30 V·µs
```

The key point: the volt-seconds determine the flux swing in the core (ΔB = λ / (N × Ae)), which sets both the turns needed and the core loss. The inductance value (10µH) is not needed to find λ; it sets the ripple current, ΔI = λ / L = 3A.

**Why D is wrong:** 40 V·µs = Vin × ton = 48V × 0.833µs — it forgets to subtract Vout from the inductor voltage.

**Why A and B are wrong:** they do not follow from volt-second balance for these conditions (8.33 V·µs would be 10V × ton, and 25 V·µs would be 30V × ton).

---

## Q9

What is the primary advantage of using Litz wire in a high-frequency power inductor compared to solid round wire of the same outer diameter?

- A) Lower cost due to simpler manufacturing
- B) Higher temperature rating due to better insulation
- C) Lower AC resistance because many thin strands reduce skin and proximity effect losses
- D) Higher current carrying capacity due to better thermal conduction

**Correct answer: C**

**Explanation:**

Litz wire (from Litzendraht, German for "woven wire") consists of many individually insulated thin wire strands, twisted together in a specific pattern designed to equalise the flux linkage (and therefore current distribution) among all strands.

**How it works:**

In solid round wire, skin effect limits current to the surface (depth δ). Interior copper is wasted.

In Litz wire, each strand is thin (diameter < 2δ), so current distributes uniformly across each strand cross-section. The twisting pattern ensures each strand spends equal time near the surface and near the centre of the bundle, so all strands carry equal current.

The result: for the same total copper area, Litz wire has lower AC resistance than solid wire at high frequency.

**Example:** At 200kHz, δ = 148µm. A 1mm solid wire (d/δ ≈ 6.8) has RAC ≈ (d/(4δ) + 0.25) × RDC ≈ 1.9 × RDC from skin effect alone. The same copper area (0.79mm²) in Litz needs about 157 strands of AWG 40 (79µm diameter, well within 2δ), giving RAC ≈ 1.05 × RDC — about 1.8× lower AC resistance, before counting proximity effect, which usually makes the advantage of Litz in a multi-layer winding much larger.

**Why A is wrong:** Litz wire is significantly more expensive than solid wire due to the complex manufacturing process (stranding, twisting, applying individual insulation to each strand, then bundling). Cost is a disadvantage of Litz wire.

**Why B is wrong:** The temperature rating depends on the insulation material on each strand (typically polyimide or polyurethane rated 130-200°C) — similar to solid wire. Litz wire has the same temperature rating class as solid wire with comparable insulation. There is no inherent thermal advantage.

**Why D is wrong:** Current carrying capacity is determined by I²×R heating in the wire. Since Litz wire has lower RAC, it does run cooler at the same AC current — this could be viewed as higher effective current capacity. However, the thermal conduction is not inherently better; in fact, the air gaps between strands can make thermal conductivity slightly lower than solid wire. The current capacity advantage of Litz wire at high frequency comes from lower AC resistance, not better thermal conduction.

---

## Q10

In a push-pull converter transformer, a "DC bias" or "flux imbalance" problem can cause core saturation. Which winding technique best mitigates this without active current balancing?

- A) Separate primary and secondary on different core legs
- B) A DC blocking capacitor in series with the primary winding
- C) Increasing the air gap in the core
- D) Using a toroidal core instead of an E-core

**Correct answer: B**

**Explanation:**

As explained in Q15 of the topology quiz, flux walking occurs when the two half-cycles of the push-pull driver apply slightly asymmetric volt-seconds to the primary. The flux accumulates unidirectionally each cycle until the core saturates.

A DC blocking capacitor in series with the primary winding prevents DC current from flowing. If one switch on-time is slightly longer, more charge accumulates on one side. The capacitor charges accordingly, creating a voltage that opposes and equalises the volt-seconds automatically:

When switch 1 is on slightly longer: capacitor charges slightly positive on one plate. During switch 2's turn, the capacitor voltage subtracts from Vin, reducing the effective volt-seconds for switch 2's half-cycle. The system self-corrects.

The capacitor must be large enough that the primary current does not charge it by much within one on-time:
```
C ≥ I_primary × ton / ΔV_cap_allowed
```

Typical: a few µF with adequate voltage rating.

**Why A is wrong:** Separating primary and secondary on different core legs (as in some E-I or planar designs) does not prevent flux imbalance in the primary. The two primary half-windings still apply volt-seconds to a shared magnetic circuit.

**Why C is wrong:** An air gap lowers the effective permeability, so more magnetomotive force (magnetising current) is needed to reach saturation. However, it does not prevent flux accumulation — it only raises the threshold at which saturation occurs. The flux imbalance mechanism is unchanged; saturation just occurs at a higher peak magnetising current. An air gap also reduces inductance and increases magnetising current ripple.

**Why D is wrong:** The core geometry (toroidal vs. E-core) affects winding ease, leakage inductance, and EMI characteristics. It does not address the fundamental volt-second imbalance that drives flux walking. Toroidal cores are actually harder to wind balanced push-pull primaries on, which can worsen imbalance.

---

## Q11

How does increasing the switching frequency of a power converter affect the size of its magnetic components?

- A) Higher frequency allows smaller magnetic components because the required volt-seconds per cycle decreases
- B) Higher frequency requires larger magnetic components to maintain the same inductance at higher ripple current
- C) Switching frequency has no effect on magnetic size — only inductance value matters
- D) Higher frequency always reduces magnetics size until core loss becomes dominant, at which point size must increase to manage loss

**Correct answer: D**

**Explanation:**

Increasing switching frequency is the primary tool for reducing magnetic component size in power converters, but it has a limit:

**Why higher frequency initially reduces size:**

For a given inductance, the inductor volt-second product (λ = L × ΔI) can be maintained with a smaller number of turns or smaller core if the frequency increases:

```
L = N × Ae × ΔB / ΔI     (from Faraday's law)
Required Ae × N = L × ΔI / ΔB
```

For fixed ΔB (core loss constraint) and ΔI, reducing L (because higher fsw allows smaller inductance for the same ripple ratio) reduces Ae × N. Smaller Ae means smaller core. N can also be reduced proportionally.

Alternatively, for fixed inductance value: higher fsw still means smaller L is needed because the converter can use a smaller L for the same ripple current (ΔI = VL × ton / L, smaller ton allows smaller L for same ΔI).

**The ceiling — core loss:**

Core loss Pv = Cm × f^α × B_peak^β grows with frequency. At some point, the core must be made larger to reduce B_peak (and thus core loss) to an acceptable level. This reverses the size reduction trend.

For a given ferrite material, there is an optimal frequency range where the core volume is minimised. Beyond this frequency, the core must be derated (lower ΔB) to control loss, and the size increases again. For N87 ferrite, this optimal range is roughly 200kHz-800kHz depending on the application.

**Why A is wrong (partially):** The mechanism in A is correct (volt-seconds decrease with frequency for fixed ΔI), but the answer ignores the core loss limit, which is the key nuance that makes D the better answer.

**Why B is wrong:** This is entirely backward. Higher frequency allows SMALLER magnetic components. Higher frequency means shorter on-time, smaller required volt-seconds, smaller required core and windings.

**Why C is wrong:** Switching frequency directly and strongly affects magnetic component size through the volt-second product and core loss. Claiming no effect is incorrect.

---

## Q12

What does "proximity effect" cause in transformer windings, and how does it differ from skin effect?

- A) Proximity effect and skin effect are different names for the same phenomenon
- B) Proximity effect causes current redistribution within a conductor due to the magnetic field of an ADJACENT conductor, while skin effect is caused by the conductor's own field
- C) Proximity effect increases the DC resistance of windings due to physical proximity to the core
- D) Proximity effect only affects the secondary winding because it is closer to the output

**Correct answer: B**

**Explanation:**

**Skin effect:** A conductor carrying high-frequency current generates an alternating magnetic field around itself. This field induces eddy currents within the conductor that oppose the current near the centre, pushing the current to the surface. Cause: the conductor's own field. Effect: current crowds toward the conductor surface.

**Proximity effect:** In a multi-layer winding, adjacent conductors carrying alternating current generate fields that penetrate neighbouring conductors. These induced fields drive eddy currents in the adjacent conductors, causing non-uniform current distribution within them. Cause: the field of neighbouring conductors. Effect: can significantly increase AC resistance beyond what skin effect alone predicts.

In a multi-layer winding (e.g., 5 layers of wire), the proximity effect compounds. Each layer "sees" the field of all adjacent layers:

- Layer 1 (innermost): sees fields from layers 2-5
- Layer 5 (outermost): sees fields from layers 1-4

Dowell's method quantifies the combined skin + proximity effect loss as a function of the ratio of wire diameter to skin depth and the number of winding layers. For a 5-layer winding at 500kHz, the AC/DC resistance ratio can be 10-50× — far more than skin effect alone would predict.

**Why A is wrong:** They are distinct mechanisms with different mathematical treatments. Skin effect depends only on the conductor's own geometry; proximity effect depends on the winding architecture (number of layers, layer thickness, interlaying with secondary).

**Why C is wrong:** Proximity effect does not affect DC resistance. DC resistance is purely a function of conductor material resistivity, length, and cross-section — proximity is a frequency-dependent (AC) phenomenon. Near the core, the core's AC field does affect the winding, but this is also a frequency-dependent mechanism, not a change in DC resistance.

**Why D is wrong:** Proximity effect affects ALL windings — primary, secondary, and any bias windings. The primary winding, typically multi-layered to accommodate high voltage turns, can suffer worse proximity effect than the secondary (which may be a single layer of heavy wire for low-voltage, high-current applications).

---

## Q13

What is the iGSE (Improved Generalised Steinmetz Equation) and why is it used instead of the standard Steinmetz equation for power converter applications?

- A) iGSE includes the effect of DC bias on core loss
- B) iGSE handles non-sinusoidal flux waveforms by integrating the instantaneous loss rate based on dB/dt
- C) iGSE uses a more accurate physical model by accounting for temperature effects
- D) iGSE is used only for laminated iron cores, not for ferrites

**Correct answer: B**

**Explanation:**

The standard Steinmetz equation Pv = Cm × f^α × B_peak^β was curve-fitted to sinusoidal excitation data. Power converters drive magnetic cores with non-sinusoidal (trapezoidal, square, triangular) flux waveforms. Applying the Steinmetz equation with the equivalent frequency of the non-sinusoidal waveform can produce significant errors.

The iGSE handles non-sinusoidal excitation by computing an instantaneous loss rate based on the rate of change of flux density:

```
Pv = (1/T) ∫₀ᵀ ki × |dB/dt|^α × ΔB^(β-α) dt
```

where ΔB = 2 × B_peak and ki is related to the Steinmetz coefficients by:

```
ki = Cm / [(2π)^(α-1) × ∫₀²π |cos(θ)|^α × 2^(β-α) dθ]
```

For a triangular flux waveform (typical of a buck or boost inductor):

```
P_iGSE = ki × (2 × ΔB/Δt)^α × ΔB^(β-α)  ×  (fraction of period at each dB/dt)
```

This integrates the loss over each segment of the waveform (rising and falling portions separately), capturing the actual dB/dt experienced by the core.

**Why this matters:** For a buck converter at D = 0.25, the inductor flux ramps up steeply for 25% of the period and ramps down gradually for 75%. The rising dB/dt is 3× steeper than the falling. The iGSE correctly accounts for this asymmetry; the standard Steinmetz equation (with equivalent frequency) gives an approximation that may be 20-50% inaccurate.

**Why A is wrong:** DC bias effects on core loss are addressed by separate models (modified Steinmetz equation with DC bias term, or measured data with bias). iGSE specifically addresses non-sinusoidal WAVEFORM SHAPE, not DC offset.

**Why C is wrong:** Temperature effects are handled by using temperature-dependent Steinmetz parameters (Cm, α, β measured at different temperatures) or by separate empirical corrections. iGSE does not incorporate temperature as a fundamental improvement.

**Why D is wrong:** iGSE was developed specifically for ferrite cores used in switching power supplies. Laminated iron cores at power line frequency are typically characterised by different methods (loss separation into hysteresis, eddy current, and excess loss components). iGSE is most relevant for high-frequency ferrite applications.

---

## Q14

In a planar transformer, what is the primary advantage over a conventional wound transformer of equivalent power rating?

- A) Planar transformers have no leakage inductance
- B) Lower profile, tighter manufacturing tolerances, and better thermal management due to the PCB-integrated winding structure
- C) Planar transformers can operate at lower frequency because the winding resistance is lower
- D) Planar transformers are less expensive due to automated PCB manufacturing

**Correct answer: B**

**Explanation:**

Planar transformers use PCB copper layers as the winding conductors, instead of wound wire. This provides several advantages:

**Low profile:** The transformer height is limited by the core and PCB stack. Heights of 2-5mm are achievable for transformers handling 50-200W, far lower than equivalent wound transformers (which might be 15-25mm tall).

**Manufacturing consistency:** PCB etching is highly repeatable to ±50µm tolerances. Conventional wire winding has operator-dependent variability in layer spacing, turn position, and therefore leakage inductance. Planar transformers have very consistent electrical characteristics part-to-part.

**Thermal management:** The PCB winding is a flat, planar surface that makes good contact with heat-spreading structures (heatsinks, heat spreaders, cold plates). Heat extraction from the winding is more effective than in wound transformers where the winding is buried inside a bobbin.

**Interleaving:** PCB layers naturally enable complex P-S-P or S-P-S-P interleaving schemes that minimise leakage inductance and proximity effect. This is difficult to achieve consistently with manual winding.

**Why A is wrong:** Planar transformers do have leakage inductance — it arises from incomplete flux linkage between primary and secondary traces. Planar transformers can achieve lower leakage than wound transformers due to consistent interleaving, but leakage is not zero.

**Why C is wrong:** The relationship is opposite. Planar transformers with their flat, thin conductors have HIGHER AC resistance per unit length than thick round wire at the same DC resistance — the conductor cross-section is limited by PCB thickness (typically 35µm or 70µm copper). To compensate, planar designs operate at HIGHER frequency where smaller inductance (fewer turns) is acceptable. Planar transformers are specifically suited to high-frequency applications.

**Why D is wrong:** Planar transformers are typically more expensive than wound transformers of equivalent power rating. The PCB layer count (often 8-16 layers for complex transformers), special core shapes (E-E, E-I planar cores), and assembly complexity drive costs up. The premium is justified by the performance and profile benefits.

---

## Q15

What does "AL value" mean for a magnetic core, and what conditions must be specified when using it?

- A) AL is the inductance per turn (linear relationship): L = AL × N
- B) AL is the inductance per turn-squared: L = AL × N²; it depends on core geometry and permeability and must be specified with no air gap (or a specific gap)
- C) AL is the absolute maximum inductance the core can support before saturation
- D) AL is the core loss coefficient analogous to Cm in the Steinmetz equation

**Correct answer: B**

**Explanation:**

AL (the "inductance factor" or "inductance per unit turns-squared") is defined such that:
```
L = AL × N²
```

Units are typically nH/turn² or µH/turn² (the "per turn²" is implied).

**Physical basis:**

Inductance is L = N² / R_reluctance = N² × µ × Ae / l_e

For a given core: AL = µeff × Ae / l_e = µr × µ0 × Ae / l_e

**Conditions that must be specified:**

1. **Gap condition:** AL changes dramatically with air gap. An ungapped ferrite E25 core might have AL = 2500 nH/turn², but with a 300µm gap, AL drops to perhaps 200 nH/turn². Core datasheets specify AL for the ungapped core; the designer must calculate the gapped AL.

2. **Material and temperature:** Permeability of ferrite changes with temperature (typically -40% to +100% over -40°C to 120°C). AL changes proportionally. Some datasheets provide AL at 25°C only.

3. **DC bias:** Ferrite permeability decreases with DC bias (approaching saturation). The AL value is a small-signal parameter measured at low AC excitation. Under DC bias, the effective AL is lower.

**Why A is wrong:** L = AL × N is incorrect. The correct relationship is L = AL × N². The N² dependence comes from Faraday's law (voltage ∝ N × dΦ/dt = N × Ae × dB/dt) and Ampere's law (H × l = N × I), combining to give L = N² × µ × Ae / l. Forgetting the squaring is a common and costly error.

**Why C is wrong:** AL does not specify the maximum inductance before saturation. The saturation limit is determined separately by B_sat, Ae, and N: I_sat = B_sat × Ae × (1/L) × N = B_sat × l_e / (µ × N). The AL value itself is measured under small-signal conditions with no saturation.

**Why D is wrong:** The Steinmetz coefficient Cm has units of W/(m³ × Hz^α × T^β) and characterises core loss per unit volume. AL has units of inductance per turn² and characterises energy storage. They are completely different quantities.

---

## Q16

A transformer's leakage inductance is measured as 5µH (primary-referred) with the secondary shorted. The transformer has a turns ratio of 5:1 (primary to secondary). What is the leakage inductance referred to the secondary?

- A) 5µH
- B) 1µH
- C) 0.2µH
- D) 25µH

**Correct answer: C**

**Explanation:**

When reflecting impedance (including inductance) across a transformer, the rule is:

```
Z_secondary = Z_primary / n²
```

where n = N_primary / N_secondary = 5.

For inductance:
```
L_secondary = L_primary / n² = 5µH / 25 = 0.2µH
```

Alternatively, from the secondary perspective: if you measure the leakage inductance at the secondary terminals with the primary shorted, you would measure L_secondary = L_primary / n². The secondary has fewer turns (N2 = N1/5), so the leakage flux linking the secondary represents a much smaller inductance than when viewed from the primary.

**Practical significance:** For a 5:1 step-down transformer in a flyback converter, the 5µH primary leakage inductance becomes 0.2µH when reflected to the secondary. This 0.2µH combined with the secondary ESL can cause ringing on the secondary rectifier voltage.

Conversely, the 5µH primary leakage inductance causes a voltage spike when the switch opens: V_spike = L_ll × dI/dt, which can be large enough to destroy the primary MOSFET.

**Why A is wrong:** 5µH = L_primary. Leakage inductance transforms as n², not unity. The secondary and primary do not have the same leakage inductance.

**Why B is wrong:** 1µH = 5µH / 5 = L_primary / n. This is the result of dividing by n instead of n². A common error is to forget that inductance scales as turns-squared.

**Why D is wrong:** 25µH = 5µH × 5 = L_primary × n. This is the leakage inductance if the roles were reversed (5:1 step-UP transformer from secondary to primary), or if the n² scaling is applied in the wrong direction.

---

## Q17

What is the Boucherot cell (snubber) used on a transformer secondary, and how does it affect efficiency?

- A) An RC circuit that dampens the leakage inductance / diode capacitance resonance at the expense of some power dissipation
- B) A lossless resonant circuit that transfers energy from the snubber to the output
- C) A saturable inductor that clamps the current spike from reverse recovery
- D) A Zener diode that clamps the secondary voltage to a safe level

**Correct answer: A**

**Explanation:**

At the moment a secondary rectifier diode turns off, its junction capacitance (Cj) and the leakage inductance (Ll) form an LC circuit. The residual energy in Ll rings with Cj:

```
ω_ring = 1 / sqrt(Ll_sec × Cj)
V_ring_peak ≈ Vout × (1 + Q_ring)    where Q_ring = sqrt(Ll_sec / Cj) / R_damp
```

Without damping, the ringing can reach 2-3× Vout and can exceed the diode's reverse voltage rating, causing failure or EMI.

A Boucherot (or RC snubber) cell consists of a resistor R_s and capacitor C_s in series, placed in parallel with the diode (or across the secondary):
- C_s resonates with Ll to reduce the peak voltage
- R_s damps the oscillation by absorbing energy

R_s dissipates about ½ × C_s × V² each time C_s charges and again each time it discharges, so the snubber power is approximately:
```
P_snubber ≈ C_s × V² × fsw
```

This is a direct efficiency penalty. For example: C_s = 1nF, V = 50V, fsw = 200kHz:
```
P_snubber = 1nF × 50² × 200kHz = 1e-9 × 2500 × 200e3 = 0.5W
```

Design choice: make C_s large enough to adequately clamp the ring, but not so large that snubber loss is excessive. R_s is chosen to damp the ring: R_s ≈ sqrt(Ll_sec / C_s).

**Why B is wrong:** A lossless snubber is theoretically possible (e.g., active clamp) but the Boucherot cell is specifically a LOSSY RC snubber. The energy absorbed by R_s is dissipated as heat, not transferred to the output. Any snubber with a resistor is lossy by definition.

**Why C is wrong:** A saturable inductor (saturation core) is a different protection approach: when current exceeds a threshold, the inductor saturates and presents low impedance, bypassing the current spike. This is a distinct circuit element, not an RC snubber.

**Why D is wrong:** A Zener diode clamp is a voltage-clamping approach (also lossy), but it is not the Boucherot cell. Zener clamps are hard voltage clamps; RC snubbers are softer damped clamps. The Boucherot cell specifically refers to the RC combination.

---

## Q18

Why are MnZn ferrites not recommended for use above ~2MHz, and what alternative material is used instead?

- A) MnZn ferrite saturates at low flux density above 2MHz; NiZn ferrite has higher B_sat
- B) MnZn ferrite has increasing permeability above 2MHz, making inductance unpredictable; NiZn is more stable
- C) MnZn ferrite has low electrical resistivity relative to NiZn, causing eddy current losses to dominate above 2MHz; NiZn ferrite has much higher resistivity
- D) MnZn ferrite requires sintering at higher temperatures than are available above 2MHz; amorphous cores are used instead

**Correct answer: C**

**Explanation:**

The key distinction between MnZn and NiZn ferrites is electrical resistivity:

| Material | Resistivity | Application frequency |
|---|---|---|
| MnZn ferrite | ~1 Ω·m | DC to ~2MHz |
| NiZn ferrite | ~10⁵ Ω·m | 1MHz to >100MHz |

Eddy current losses in the core scale as:

```
P_eddy ∝ f² × B² / ρ
```

At low frequencies (< 500kHz), MnZn's resistivity (ρ ≈ 1 Ω·m) is high enough that eddy currents are small. As frequency increases, eddy currents grow as f². Above ~2MHz, eddy currents in MnZn become dominant, causing:
- Rapidly increasing core loss
- Decreasing effective permeability (eddy currents shield the flux from the core interior)
- Degraded Q factor in inductors and resonant circuits

NiZn ferrite has 10⁵× higher resistivity, so eddy currents are suppressed up to much higher frequencies. NiZn is therefore the material of choice for EMI filters, CM chokes, and high-frequency power inductors operating above 2MHz.

**Trade-off:** NiZn has lower initial permeability (µi ≈ 100-1000) compared to MnZn (µi ≈ 1000-10,000), so NiZn inductors need more turns for the same inductance.

**Why A is wrong:** MnZn and NiZn have similar B_sat values (300-500mT). Saturation flux density is not the reason for the frequency limit. B_sat does decrease somewhat at high temperature, but this is not the MnZn frequency limitation.

**Why B is wrong:** MnZn permeability DECREASES above the resonance frequency (Snoek's limit), not increases. The complex permeability µ' decreases and µ'' (loss component) increases. This is another way of saying core loss increases, but the mechanism is better described by resistivity and eddy currents.

**Why D is wrong:** Sintering temperature is a manufacturing process parameter, not a function of operating frequency. The same MnZn material sintered at the same temperature can be used at DC and at 1MHz — its limiting frequency is not determined by the manufacturing temperature.

---

## Q19

A coupled inductor (two-phase buck converter) uses two inductors wound on the same core with inverse coupling (windings wound such that their fluxes oppose). What is the main benefit?

- A) The inductance seen by each phase is doubled compared to an uncoupled inductor
- B) The effective inductance for current ripple cancellation is high, but the transient inductance (seen during load steps) is low
- C) The coupled inductor has zero core loss because the fluxes cancel
- D) The coupled inductor allows the converter to operate at twice the switching frequency without additional switches

**Correct answer: B**

**Explanation:**

In a two-phase interleaved buck with inversely coupled inductors (coupling coefficient k < 0 by convention for opposing winds):

Write the mutual inductance as M = -α × L, with 0 < α < 1 for inverse coupling (Wong, Xu and Lee, IEEE Trans. Power Electronics, 2001).

**Transient:** During a load step both phase currents change together, and the inductance they see is:
```
L_transient = L + M = L × (1 - α)
```

**Ripple (steady-state):** For D < 0.5, the equivalent inductance that sets each phase's current ripple is:
```
L_ss = L × (1 - α²) / (1 - α × D/(1-D))
```

Example, α = 0.8 and D = 0.25: L_transient = 0.2L, while L_ss = 0.36L / 0.733 = 0.49L. The ripple sees 2.5× the inductance that limits the transient. An uncoupled inductor has L_ss = L_transient, so to get the same ripple it would need to be 0.49L and would slow the transient by the same 2.5×. (As α → 1, both inductances fall toward zero, so the coupling is chosen at an intermediate value.)

This decoupling of ripple inductance from transient inductance is the primary benefit.

**Practical realisation:** A 4-layer or 8-layer PCB planar coupled inductor, or a toroidal core with bifilar winding (two wires wound simultaneously but with opposite current direction), or a specially configured EE core with shared center leg.

**Why A is wrong:** Neither inductance is doubled. Inverse coupling lowers the inductance each phase sees; the benefit is the RATIO of steady-state to transient inductance, not a larger inductance.

**Why C is wrong:** The fluxes from the two phases partially cancel in the core, but not completely (because the two phases carry slightly different instantaneous currents due to ripple). Even if the DC fluxes cancelled perfectly, the AC flux (from ripple) still exists and causes core loss. Coupled inductors do benefit from reduced flux swing (which reduces core loss), but zero core loss is not achievable.

**Why D is wrong:** The coupled inductor does not change the effective switching frequency for the converter's control or switching losses. The interleaving of two phases at fsw creates an apparent 2×fsw ripple at the output (as discussed in the topology quiz), but this is an interleaving benefit, not a coupled inductor benefit specifically.

---

## Q20

What is the area-product (Ap) method for selecting a magnetic core, and what quantities does it estimate?

- A) The minimum core cross-sectional area needed to avoid saturation at peak flux density
- B) A figure of merit (Ae × Aw) that estimates the required core size based on stored energy (or power throughput), peak flux density, and winding current density
- C) The surface area of the core needed for adequate heat dissipation
- D) The product of the AL value and the core volume, used to compare cores of different materials

**Correct answer: B**

**Explanation:**

The area product Ap = Ae × Aw combines:
- Ae = effective cross-sectional area of the core (determines flux capacity, B_peak × N = λ/Ae)
- Aw = winding window area (determines how many turns of a given wire diameter can fit)

**Derivation:**

For an inductor with peak current I_peak and inductance L:

Energy stored: E = ½ × L × I_peak²

From Faraday's law: L × I_peak = N × Ae × B_peak → N = L × I_peak / (Ae × B_peak)

From winding: all N turns must fit in window Aw with wire cross-section Aw_wire:
```
N × Aw_wire = ku × Aw    (where ku = fill factor ≈ 0.4-0.5)
Aw_wire = I_peak / J    (where J = current density, typically 3-5 A/mm²)
```

Combining:
```
Ae × Aw = (L × I_peak²) / (ku × J × B_peak) = 2E / (ku × J × B_peak)
```

Given a target B_peak, current density J, and fill factor ku, calculate the required Ap. Then search the core catalogue for a core with Ap ≥ Ap_required.

**For transformers:** Ap = Pout / (ku × J × B_peak × fsw × (Ae × constant)) — a similar energy/flux/window argument.

**Why A is wrong:** The minimum Ae for saturation avoidance is just: Ae ≥ N × ΔΦ / ΔB = L × ΔI / (N × ΔB). This is a single area calculation, not the area product. The area product combines both the core area AND the winding window to estimate total core size.

**Why C is wrong:** Surface area for heat dissipation is a thermal calculation, not the area product method. The thermal calculation estimates maximum power loss and required surface area for convection (P_loss = h × A_surface × ΔT). Unrelated to Ap.

**Why D is wrong:** AL × V_core has no standard meaning in magnetics design. The area product Ap = Ae × Aw is a core size metric, not a performance metric combining AL with volume.

---

*End of Quiz — Magnetics Design*

**Answer Key:** 1-A, 2-C, 3-A, 4-B, 5-B, 6-B, 7-B, 8-C, 9-C, 10-B, 11-D, 12-B, 13-B, 14-B, 15-B, 16-C, 17-A, 18-C, 19-B, 20-B
