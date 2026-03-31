# PCB Layout for Power Electronics — Interview Preparation

## Overview

PCB layout is where theory meets reality in power supply design. Poor layout causes EMI failures, unexpected oscillations, thermal hotspots, and efficiency degradation that no amount of schematic redesign can fix. This topic is frequently tested in hardware interviews because it distinguishes engineers with hands-on experience from those who only understand theory.

---

## Key Principles Reference

### Current Loop Area and Inductance
```
Parasitic loop inductance: L_loop ≈ µ0 × A_loop / (π × d_wire)   [simplified]

For a PCB trace loop: L_loop ≈ µ0 × area / width × length_factor

Rule of thumb: 1 nH per mm of trace length for typical trace widths
Switching current loop inductance causes V_spike = L_loop × dI/dt
```

### Trace Width vs Current
```
IPC-2221 (external trace, 1 oz copper, ΔT = 10°C):
  Width (mm) = (I / (0.048 × ΔT^0.44))^(1/0.725)   [I in amps]

Common approximations:
  1 oz copper (35µm): 400 mA/mm for ΔT=10°C, 600 mA/mm for ΔT=20°C
  2 oz copper (70µm): ~2× current capacity for same width

Online calculator: Saturn PCB Toolkit (recommended for real designs)
```

### Parasitic Capacitance (Parallel Planes)
```
C_plane = ε0 × εr × A / d   [F]

For FR4 (εr ≈ 4.2):
  2 oz planes, 0.1 mm dielectric: C ≈ 370 pF/cm²
  Useful for decoupling on inner planes
```

### Gate Drive Loop
```
Gate current at turn-on: I_gate_peak = (Vdriver - Vth) / R_gate
dV_gate/dt = I_gate / C_iss   [gate voltage slew rate]
dI_D/dt = gm × dV_gate/dt    [drain current slew rate during Miller plateau]
```

---

## Fundamentals (Questions 1–6)

---

### Q1. What are the critical current loops in a buck converter and why must they be minimised?

**Answer:**

A buck converter has two critical switching current loops that change abruptly each switching cycle:

**High-frequency (switching) current loop — switch ON:**
```
Path: Input capacitor (+) → High-side FET (drain to source) → Switch node → Inductor → Load → Input capacitor (-)
```
During on-time, this loop carries the inductor current (rising ramp).

**High-frequency (switching) current loop — switch OFF (synchronous buck):**
```
Path: Output node → Inductor → Switch node → Low-side FET (source to drain) → Ground → Output capacitor
```
During off-time, current flows through the low-side FET.

**Why minimise loop area:**

Any loop with rapidly changing current (dI/dt) creates a magnetic field that induces noise in nearby circuits:
```
V_induced = M × dI/dt   [mutual inductance M with nearby loop]
```

Additionally, the loop has parasitic inductance L_loop. The fast current change creates a voltage spike:
```
V_spike = L_loop × dI/dt
```

For dI/dt = 1 A/ns (typical for fast FETs) and L_loop = 5 nH (achievable with poor layout):
```
V_spike = 5 nH × 10⁹ A/s = 5 V
```
This 5 V spike appears on the switch node and input rail, creating EMI and stressing components.

**Layout rule:**

Place the input decoupling capacitor(s) as close as possible to the FET source (high-side) and the FET drain. The path input cap → PFET → switch node → NFET → ground must be as short as possible:
```
Best:    Input cap directly beside FETs (< 5 mm total loop)
Typical: Input cap within 10 mm of FETs
Poor:    Input cap > 20 mm from FETs (adds significant loop inductance)
```

---

### Q2. Describe the placement priority order for components in a buck converter layout.

**Answer:**

Component placement follows the power flow path. Start with the highest-current, highest-frequency elements and work outward.

**Priority order (most critical first):**

**1. FET pair (high-side and low-side MOSFETs):**
Place side by side, as close as physically possible. The switch node (connection between FETs) should be a single copper pour, not a long trace.

**2. Input decoupling capacitor:**
Place the bulk input decoupling capacitor (electrolytic or large ceramic) with its positive terminal at the high-side FET drain and negative terminal at the low-side FET source. The loop from (+) to drain to source to (-) must be the tightest loop on the board.

**3. Bootstrap capacitor (if applicable):**
The bootstrap capacitor charges during low-side conduction and must be directly connected between BOOT and SW nodes of the driver IC. Place within 5 mm of the driver IC.

**4. Gate drive resistors:**
Place directly at the gate pins of each FET (within 3–5 mm). Long gate traces add inductance and can cause ringing or oscillation of the gate signal.

**5. Inductor:**
Connected to the switch node. Place on the same layer, close to the FET pair. Orient the inductor so the fringing field from the gap faces away from sensitive signals.

**6. Output capacitors:**
Place near the inductor's output terminal and the load's power rails. For multi-capacitor output (ceramic + electrolytic), connect ceramics closest to the IC.

**7. Feedback divider:**
Place close to the output voltage, but routed carefully to avoid switching noise. Use Kelvin sensing (separate sense traces) for accuracy.

**8. PWM controller IC:**
Can be placed with some separation from the high-power section, but gate drive paths must be short. The VCC decoupling cap must be directly at the IC's VCC and GND pins.

**9. Signal-level components (error amplifier, compensation, reference):**
Furthest from the switching noise, near the IC, in a quiet area of the board.

---

### Q3. What is a thermal via and how is it designed for a power device?

**Answer:**

**Purpose:**

A thermal via is a through-hole via that transfers heat from a component's thermal pad (top copper) to a thermal plane on inner or bottom layers, or to a heatsink. It dramatically reduces the junction-to-board thermal resistance.

**Physical mechanism:**

Heat conducted from the device thermal pad into the copper on the top layer spreads laterally and flows down through the via barrels (copper plating). The heat exits through the inner or bottom copper planes, which act as heat spreaders.

**Design parameters:**

1. **Via diameter:** 0.3–0.5 mm for thermal vias. Smaller diameter → higher via resistance per via but allows more vias per area. Larger diameter → lower resistance but fewer fit under a small pad.

2. **Via plating thickness:** IPC standard is 25 µm (1 mil) minimum. Thermal resistance of one via:
   ```
   Rth_via = length / (k_Cu × A_cross-section)
   = 1.6mm / (385 W/m·K × π × (0.175mm)² ) [for 0.3mm via, 1.6mm board]
   = 1.6×10⁻³ / (385 × 9.62×10⁻⁸)
   = 1.6×10⁻³ / 37.0×10⁻⁶
   = 43.2 K/W per via
   ```

3. **Number of vias:** Multiple vias in parallel divide the thermal resistance:
   ```
   Rth_vias_total = Rth_via / N_vias

   For 9 vias: Rth_total = 43/9 = 4.8 K/W
   ```

4. **Via pattern:** Place vias in a grid pattern under the thermal pad, typically 1 via per 0.5–1 mm². For a 5×5 mm² thermal pad: 25–100 vias.

5. **Solder filling:** Standard vias under a thermal pad get filled with solder during reflow, significantly improving thermal conductance (solder k ≈ 50 W/m·K vs air k = 0.025 W/m·K). For press-fit vias (no solder fill), the thermal resistance is dominated by the air void.

**Design checklist:**
- Specify via tenting (top and bottom) or no tenting (open) depending on solder fill requirement
- If vias are not solder-filled, use copper-filled vias (specify on fab note) for best thermal conductance
- Ensure vias connect to unbroken copper plane on adjacent layer (not a floating island)

---

### Q4. How do you implement Kelvin sensing for accurate output voltage feedback?

**Answer:**

**The problem without Kelvin sensing:**

The output voltage sense point is typically connected to the feedback network via traces that also carry load current. Even a small trace resistance (e.g., 5 mΩ in 50 A of current) creates a 250 mV error at the sense point. The converter regulates the voltage at the sense point, not at the load — so the load sees lower voltage at heavy load.

**Kelvin sensing principle:**

Route two dedicated sense traces from the exact point where regulation is desired (at the load, or at the output connector):
- One trace: positive sense (Vsense+)
- One trace: negative sense (Vsense-)

These traces carry only the very small current through the feedback resistor divider (typically 1–100 µA) — negligible voltage drop. They are routed separately from the power current path.

**Implementation:**

```
Power path:
Output cap (+) ──────── [power trace, thick, high current] ──── Load (+)
Output cap (-) ──────── [power trace, thick, high current] ──── Load (-)

Kelvin sense traces (thin, separate route):
Load (+) ─── [sense trace, thin, ~0.2mm width OK] ─── R1 ─── FB pin
Load (-) ─── [sense trace, thin] ─────────────────────────── GND of controller
```

**Key rules:**
1. **Branch point at the load, not the capacitor:** Connect sense traces at the load connector pins, not at the output capacitor pads. This compensates for trace resistance between capacitor and load.

2. **Parallel routing, no current loops:** Route the two sense traces in parallel (differential pair style) to reject common-mode noise. Do not loop them around high-dI/dt areas.

3. **No additional components in sense path:** No vias, connectors, or thermal pads in the sense path — each connection adds resistance.

4. **Remote sensing in multi-board systems:** For a converter with an output cable, route the sense leads back alongside the power conductors. Twist the pair to reject induced noise. This is "remote sensing" — the converter regulates voltage at the load regardless of cable resistance.

---

### Q5. What is a copper pour for ground and how should it be implemented in a switching power supply?

**Answer:**

**Purpose of copper pour (polygon pour):**

A copper pour fills unused PCB areas with copper connected to a net (usually ground or a power plane). Benefits:
- Low-impedance return current path
- Thermal spreading (if on top/bottom layer)
- EMI shielding for signals under the pour
- Mechanical stiffness (minor)

**Power supply ground pour rules:**

**1. Single-point ground (star ground):**
For a switching power supply, separate the "noisy" ground (power ground — FET source, input cap (-), inductor return) from the "quiet" ground (signal ground — IC logic, error amplifier, feedback). Connect them at a single point near the supply output or the output capacitor. This prevents the large dI/dt currents in the power ground from inducing noise in the signal ground.

```
Signal GND ────┐
               └──── Star point ──── Power GND
```

**2. Split ground planes:**
On multilayer boards, a common approach is:
- Top layer: power components and copper pour
- Ground plane: second layer, solid copper for return current
- Power plane: third layer (or combined with top)
- Bottom layer: signal and control routing

The ground plane provides a low-inductance return path for all current loops. The critical switching current loop return goes through the shortest path on the ground plane directly below the switching components.

**3. Pour around high-dV/dt nodes:**
Avoid placing a solid copper pour adjacent to the switch node (SW). A large pour near SW has parasitic capacitance (C_stray × dV/dt = I_displacement), which creates common-mode current through the board to the chassis or other nearby components.

**4. Pour under the gate drive IC:**
A solid copper pour (GND) under the gate driver IC reduces ground bounce and provides a low-impedance return path for gate currents.

---

### Q6. How do you route the gate drive signal to minimise noise?

**Answer:**

The gate drive signal is the critical control signal that determines when the FET switches. Noise on the gate can cause:
- Premature turn-on (shoot-through in synchronous converters)
- Oscillation during transitions (parasitic L-C ringing)
- Increased EMI (gate ringing at hundreds of MHz)
- FET destruction (gate overvoltage)

**Gate drive routing rules:**

**1. Short traces (< 20 mm ideal):**
Every mm of gate trace adds approximately 1 nH of inductance. With gate-source capacitance Cgs = 3000 pF, a 10 nH gate trace creates a resonance at:
```
fr = 1/(2π×√(L×C)) = 1/(2π×√(10nH×3000pF)) = 1/(2π×5.48ns) = 29 MHz
```
This resonance rings on the gate signal and increases EMI.

**2. Keep return path adjacent:**
Route the gate drive signal trace directly alongside its return (source of the FET for N-channel). If using a ground plane, the return is directly below the trace on the ground plane — the current loop is determined by the trace height above the plane, not the route length.

**3. Kelvin gate connection:**
Use a separate ground connection for gate drive return (source of the FET) separate from the power source connection. This prevents the large dI/dt power current from entering the gate drive return path and creating gate voltage noise:
```
Gate driver ground: connect to FET source Kelvin pad (separate from power source pad)
Power source: connect to power ground plane (heavy connection)
```
Many power FET packages have separate Kelvin source pins (e.g., TO-263-7, TOLL packages) specifically for this purpose.

**4. Gate resistor placement:**
Place the gate resistor(s) (R_g, R_g_off) directly at the gate pin, not at the driver. This way, the gate trace is on the "quiet" side (after the resistor), and the resonant energy is dissipated before it reaches the gate.

**5. Avoid crosstalk between high-side and low-side gate drives:**
Route high-side and low-side gate traces on different layers or with ample separation. Capacitive coupling between the two can cause unwanted cross-conduction.

---

## Intermediate (Questions 7–14)

---

### Q7. What is the "hot loop" in a switching converter and how do you find it on a schematic?

**Answer:**

The "hot loop" is the electrical loop that carries the highest dI/dt — the fastest-changing current in the circuit. In a buck converter, the hot loop is the switching current loop that includes the input capacitor, the FET pair, and the switch node.

**Identifying the hot loop:**

Trace the path of current that changes abruptly (step change) at the switching transitions:

**At turn-ON of high-side FET:**
```
Current suddenly flows from Cin(+) → Q1_drain → Q1_source → SW node → ...
The loop: Cin → Q1_drain → SW → Q2_drain (while Q2 is off) → Q2_source → GND → Cin(-)
```

At this instant, dI/dt can be 1–10 A/ns depending on the FET and gate drive.

**Hot loop components:**
- Input capacitor (electrolytic + paralleled ceramics)
- High-side FET
- Low-side FET (or freewheeling diode)
- Switch node copper (connecting FET to inductor)

The inductor is NOT in the hot loop because the inductor current changes slowly (L × dI/dt = V, so dI/dt = V/L which is finite and relatively slow at high L).

**Layout minimisation:**

Draw an arrow from Cin+ through Q1 through SW through Q2 through GND back to Cin-. The area enclosed by this arrow (projected onto the PCB plane) is the hot loop area. Minimise this area by:
1. Placing Cin immediately beside Q1/Q2
2. Using the shortest possible connection from Cin(-) to Q2 source
3. Using wide, short copper pours rather than traces

**Common mistake:** Engineers often focus on making the output trace wide (for lower DCR) while neglecting the input hot loop. A wide output trace has almost no benefit on EMI because the inductor current changes slowly. The hot loop, which changes abruptly, is the dominant EMI source.

---

### Q8. How does a ground plane affect power supply performance?

**Answer:**

A continuous ground plane on an inner layer of a multilayer PCB provides:

**1. Low impedance return path:**

Current returning from the load to the source follows the path of least impedance. At high frequency, this is the path of least inductance, which is directly below the forward current trace (image current). A solid ground plane enables this natural current return, minimising loop area and inductance.

```
Return current spreads to width of about 3× the height above the plane:
  If trace is 0.2 mm above the plane, return current spreads ~0.6 mm wide
  Loop inductance ≈ µ0 × h / width × L_trace ≈ 0.25 nH/mm (typical)
  vs. point-to-point trace: ~1 nH/mm
```

**2. EMI reduction:**

With a solid ground plane, the return current is immediately below the forward trace. The forward and return currents are close together, causing their magnetic fields to cancel:
```
Effective radiation ∝ Area_loop = trace_length × trace_height_above_plane
vs. no ground plane: ∝ trace_length × trace_separation
```
The ground plane reduces the effective loop area by 5–50× compared to no ground plane.

**3. Common-mode impedance:**

The ground plane provides a reference for all signals on the board. Without a solid plane, different circuit sections can have different ground potentials at high frequency → common-mode noise between sections.

**4. Thermal management:**

A copper plane spreads heat from hot components. 2 oz copper plane: thermal resistivity ≈ 1.3 °C·cm/W per square.

**When NOT to use a solid ground plane:**

Beneath the switch node (SW) in a buck converter, a solid ground plane creates stray capacitance:
```
C_stray = ε0 × εr × A / d
```
This capacitance resonates with the switch node inductance, creating ringing. Optimal: remove the ground plane from directly below the SW node copper pour (create a "keepout" for the ground plane under SW node).

---

### Q9. Describe the layout considerations for an optocoupler in an isolated power supply.

**Answer:**

**Optocoupler function:** Transfers the error signal across the isolation barrier from secondary (output) side to primary (controller) side.

**Layout considerations:**

**1. Isolation clearance on PCB:**

The PCB must maintain the same clearance and creepage distances as the transformer:
```
Clearance (air gap between primary and secondary traces): ≥ 4 mm (for 250V)
Creepage (surface distance): ≥ 8 mm (for 250V, PD2, reinforced insulation)
```

This means a physical gap (slot routed in the PCB, or just no copper) must be present between primary-side traces and secondary-side traces where the optocoupler bridges them. The slot prevents arcing and surface leakage.

**2. Optocoupler placement:**

Place the optocoupler with its LED side on the secondary, collector/emitter side on the primary. The device itself provides the internal isolation (minimum 5kV rated for most safety optocouplers). The PCB must maintain separation outside the package.

**3. LED drive circuit:**

The TL431 (or similar) controls the LED current through a series resistor. This circuit is on the secondary side:
```
Secondary ground → TL431 → LED (anode) → LED (cathode) → R_series → Vout
```
Place the TL431 and its compensation components close together on the secondary side. Avoid routing these signal traces near the primary-side switching node.

**4. Collector pull-up:**

The optocoupler collector connects to the controller's FB pin through a pull-up resistor to VCC. This pull-up determines the maximum feedback current and thus part of the loop gain. Place the pull-up resistor directly at the IC FB pin (reduces trace capacitance to FB, which can add a pole in the feedback path).

**5. PCB slot requirement:**

For IEC 62368-1 / UL compliance, route a slot (routed gap, typically 0.5–1.0 mm wide) across the PCB between primary and secondary copper. The slot must be visible in the Gerber files and verified in fabrication. The optocoupler package pins straddle this slot.

---

### Q10. What is spread spectrum modulation (SSM) in a switching converter and how does it affect layout?

**Answer:**

**Spread Spectrum Modulation:**

SSM varies the switching frequency slightly (±3–10%) around the nominal frequency, typically using a triangular or pseudorandom modulation. The energy that would concentrate at a single frequency (fsw) and its harmonics is spread over a frequency band.

**Effect on EMI spectrum:**
```
Without SSM: EMI peak at fsw = 200 kHz → measured EMI = high
With SSM (±5%): frequency spread over 190–210 kHz → same total energy,
but spread over 20 kHz bandwidth → peak EMI reduced by:
  reduction ≈ 20 × log(Δf_spread / Δf_resolution) dB
  For 1 kHz RBW analyser: 20 × log(20,000/1,000) = 26 dB reduction
```

This can give 10–15 dB improvement in measured conducted EMI (using the 9 kHz resolution bandwidth of CISPR standards) without any hardware filter changes.

**Layout impacts of SSM:**

1. **Current ripple variation:** As fsw changes, inductor current ripple changes (ΔIL = Vout×(1-D)/(fsw×L)). The output voltage ripple also varies slightly. Design output capacitor for the minimum switching frequency (highest ripple).

2. **Loop stability:** If the control loop bandwidth fc is close to the SSM modulation frequency, the modulation can interact with the closed loop → output voltage variation at the SSM rate (audible if the modulation rate is 20Hz–20kHz). Keep fc >> SSM modulation rate.

3. **Magnetic component design:** The inductor must handle the lowest switching frequency in the spread range. Core loss may increase slightly (lower fsw → larger ΔB per cycle at the same duty cycle). Verify the inductor design at fsw_min.

4. **No layout change needed:** SSM is implemented in the PWM controller IC (via dithering the oscillator). No additional external components in the layout — it is a feature of the controller's timing circuit.

---

### Q11. How does PCB stackup affect power supply performance?

**Answer:**

**Common stackups for 4-layer power supply board:**

```
Layer 1 (top):    Component placement, power traces, switch node copper
Layer 2:          Ground plane (solid copper)
Layer 3:          Power plane (Vin, Vout) or signal routing
Layer 4 (bottom): Signal routing, secondary components
```

**Performance impacts:**

**1. Ground plane proximity:**

The dielectric between Layer 1 and Layer 2 determines the trace-to-plane separation h. Lower h → lower trace inductance (better), but higher stray capacitance to plane.
```
For 4-layer board, dielectric between L1 and L2 is typically 0.1–0.2 mm
  L_trace ≈ µ0/π × ln(2h/r_trace) per unit length ≈ 0.3 nH/mm (at h=0.1mm)
```

**2. Switch node keepout:**

Do not route ground plane directly below the switch node copper (L1). Create a polygon keepout on L2 below the SW area. Otherwise, C_stray = ε0 × εr × A_SW / h resonates with L_loop.

**3. Split plane for primary/secondary:**

In isolated supplies, split the ground plane at the isolation barrier. Primary GND and secondary GND are separate on L2, connected only through the Y capacitor(s) at the designated point. This prevents primary switching current from flowing in the secondary GND plane.

**4. Controlled impedance:**

For gate drive traces and high-speed digital control signals, controlled impedance (50Ω) may be needed. The trace width for 50Ω on FR4 with 0.1mm dielectric: typically 0.2 mm width.

**5. Thermal spreading:**

2 oz copper (70 µm) on an inner plane layer spreads heat from the FET thermal vias much more effectively than 1 oz copper. Specify heavier copper on planes for high-power designs.

---

### Q12. What are the layout rules for decoupling capacitors?

**Answer:**

Decoupling capacitors (bypass capacitors) suppress voltage ripple and high-frequency noise on power supply rails. Their effectiveness depends critically on placement.

**Types and placement:**

**1. Bulk decoupling (electrolytic or large ceramic, 10–100 µF):**
Purpose: Store charge for slow load transients; filter low-frequency ripple.
Placement: Within 20 mm of the load or converter output. Exact placement less critical (impedance at these frequencies is not dominated by inductance).

**2. High-frequency decoupling (ceramic, 0.1–1 µF):**
Purpose: Filter switching frequency harmonics and provide charge for fast transients.
Placement: Within 5 mm of the IC power pins. Route VCC pin → capacitor → GND as the shortest possible path. The capacitor must be between VCC and GND, not offset.

**3. Ultra-high-frequency decoupling (10–100 nF, 0402 or 0201 size):**
Purpose: Filter > 10 MHz noise; maintain low PDN impedance at processor speeds.
Placement: Directly at device pins, as close as physically possible.

**Practical rules:**

1. **One capacitor per power pin:** IC datasheets specify recommended decoupling per pin. Follow these recommendations.

2. **Via placement:** Place the decoupling capacitor VCC via on the IC VCC pad side and GND via on the GND pad side. Do not use the IC's own VCC pin via as the capacitor connection — this adds via inductance in series with the capacitor.

3. **Parallel multiple capacitors:** Place them with the smallest one (least series inductance) closest to the IC pin. Larger values further away. Combined impedance is lower than any single capacitor.

4. **Avoid daisy-chaining:** Each capacitor should connect directly to the VCC/GND planes, not in a chain (A→B→C→IC). Daisy-chaining means current must pass through capacitor A's series inductance to reach B → net inductance increases.

5. **Do not share vias between adjacent decoupling capacitors:** Each cap should have its own VCC and GND vias for lowest loop inductance.

---

### Q13. How does layout affect the measurement of output voltage ripple?

**Answer:**

Output voltage ripple measurement is notoriously prone to measurement errors that make the ripple appear much worse than it actually is. Understanding layout-related measurement artefacts is essential.

**Common measurement error — probe ground lead loop:**

When probing with an oscilloscope probe with a ground lead:
```
Layout (wrong): Probe tip at Vout, Ground clip at GND ← 10 cm loop
Result: The 10 cm ground lead loop picks up magnetic fields from the inductor and switch node
→ Displays 100–500 mV of "ripple" that is actually interference, not real ripple
Actual ripple may be 20 mV
```

**Correct measurement technique:**

1. **Short ground spring (co-axial tip):** Use the spring ground attachment that comes with the probe (or a BNC-terminated coax probe). The ground is right next to the probe tip — loop area approaches zero.

2. **Surface mount test points:** Solder a 0402 ferrite bead or resistor (0 Ω or 33Ω) in series with a SMD test point directly at the output capacitor. Connect probe tip to test point, probe ground spring to adjacent GND via.

3. **Differential measurement:** Use a differential probe with both + and − tips placed at the measurement point. No ground lead needed — the differential probe is immune to the common-mode interference.

4. **BNC header:** For critical ripple measurement, solder a BNC connector directly to the board at the measurement point (with short ground connection). Measure with 50Ω transmission line.

**ESR spike vs capacitive ripple:**

The measured ripple waveform consists of:
1. **Capacitive ripple:** ΔVout = ΔIL / (8×fsw×C) — sinusoidal, peak at mid-half-cycle
2. **ESR spike:** ΔVout_ESR = ΔIL × ESR — sharp spike at the switching transitions

The ESR spike can appear much larger than the capacitive ripple in scope captures. If a high-ESR electrolytic is replaced with a low-ESR polymer or ceramic, the spike disappears but the capacitive ripple component may remain (or decrease). Always measure with both the long-lead and short-lead methods to distinguish real from artefact.

---

### Q14. What is via stitching and when is it used in power supply layouts?

**Answer:**

**Via stitching:**

Via stitching places multiple vias around the perimeter of a copper pour, at regular intervals, connecting it to the copper pours on adjacent layers. The vias mechanically stitch the layers together and provide low-impedance connections at closely spaced intervals.

**Use cases in power supplies:**

**1. Ground plane stitching (seam vias):**
Along the boundary between primary and secondary ground areas, and along the edges of the board, stitching vias connect the ground plane to the chassis ground (through mounting points). This provides a low-impedance RF path to chassis.

**2. Shield layer stitching:**
A copper pour on the top or bottom layer intended to shield a sensitive area (e.g., under a sensitive analog circuit, or around a clock oscillator) is stitched with vias to the ground plane beneath it:
```
Grid of vias: one via per λ/20 at the highest frequency of interest
For 100 MHz: λ = 3m, λ/20 = 15 cm → 1 via every 15 cm (very relaxed for power supply)
For 1 GHz:   λ/20 = 1.5 cm → 1 via every 1.5 cm (tighter)
```

**3. Current-carrying plane stitching:**
A thick copper current path on one layer can be stitched to a parallel path on another layer to reduce total resistance and current density. Multiple vias in parallel carry the current:
```
Resistance of N vias in parallel (each 0.5 mm diameter, 1 mm long):
R_via = ρ_Cu × L / (N × A_cu_plating) = 1.72×10⁻⁸ × 0.001 / (N × π × 0.5mm × 0.025mm)
      ≈ 0.43 mΩ per via → 10 vias = 0.043 mΩ (negligible for most power currents)
```

**4. Thermal stitching:**
Under high-power components, an array of thermal vias stitches the top copper (component thermal pad) to the ground plane on the next layer. The ground plane then acts as a heat spreader, distributing heat over a large area before it reaches the board edge or heatsink.

---

## Advanced (Questions 15–18)

---

### Q15. How do you lay out a high-current bus bar for a 100+ amp converter?

**Answer:**

At currents exceeding 50–100A, PCB copper traces become impractical (too wide, too thick). Bus bars — thick copper or aluminium conductors — carry the bulk current.

**Bus bar design:**

1. **Material and cross-section:**
   ```
   Copper resistivity: 1.72×10⁻⁸ Ω·m
   For 100A with 1 mΩ resistance:
   A = ρ × L / R = 1.72×10⁻⁸ × 0.1m / 0.001 = 1.72×10⁻⁶ m² = 1.72 mm²
   For 5 mm wide bar: thickness = 1.72/5 = 0.34 mm (just standard 0.5mm Cu strip)
   ```

2. **Inductance minimisation:**
   Bus bars carry high dI/dt. The inductance of a flat bus bar:
   ```
   L_busbar ≈ µ0 × l/π × [ln(2l/(w+t)) - 1 + (w+t)/(2l) + ...]  [H/m × length]
   ```
   Minimise by: keeping bars short, using laminated bus bars (positive and negative bars stacked with thin insulation between), which causes the return current to be adjacent to the forward current.

3. **Laminated bus bar:**
   Stack (+) and (-) bus bars with 0.1–0.5 mm insulation (polyimide or FR4). The mutual inductance between the opposite current paths nearly cancels the self-inductance:
   ```
   L_laminated = µ0 × d / w × L_length   [d = separation, w = bar width, L = length]
   At d=0.5mm, w=50mm: L_per_mm ≈ 0.013 nH/mm  (very low)
   vs. single conductor: ≈ 1 nH/mm
   ```

4. **Connections:**
   Use bolted connections with Belleville washers (spring washers) to maintain contact pressure over thermal cycling. Torque to specification. Apply anti-oxidant compound (Noalox or similar) for aluminium connections.

5. **Thermal management:**
   Bus bars at high current generate heat (I²R). Tin-plating the surface improves emissivity (heat radiation). Forced air cooling may be needed for currents above 200A.

---

## Quick Reference: Layout Checklist

```
CRITICAL CURRENT LOOPS:
[ ] Input cap adjacent to FETs (hot loop < 5 mm²)
[ ] Switch node copper minimal area
[ ] Gate drive traces < 20 mm, Kelvin source connection
[ ] Gate resistors at FET gate pins

GROUND AND PLANES:
[ ] Solid ground plane on inner layer
[ ] Split plane at isolation barrier (if isolated)
[ ] No ground plane below switch node
[ ] Star ground connecting power and signal GND

THERMAL:
[ ] Thermal vias under power components (≥ 9 vias for SOT-223 / TO-252)
[ ] Thermal vias to inner copper plane
[ ] No solder mask on thermal pad opening
[ ] Heatsink/thermal pad specified to manufacturer

SIGNAL INTEGRITY:
[ ] Kelvin sensing traces from load
[ ] Feedback traces far from switch node
[ ] Decoupling caps at each IC power pin (< 3mm)
[ ] Bootstrap cap directly at BOOT-SW pins

SAFETY (if isolated):
[ ] Creepage slot in PCB across isolation barrier
[ ] Minimum 8mm creepage, 4mm clearance across barrier
[ ] Y capacitor at designated single crossing point
```
