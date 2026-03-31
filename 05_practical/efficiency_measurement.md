# Efficiency Measurement and Loss Analysis

## Overview

Accurate efficiency measurement and loss breakdown analysis are essential skills for power electronics engineers. A converter that simulates well but measures poorly is a design failure — understanding where power is lost and how to measure it correctly separates good engineers from great ones. This topic appears in both design reviews and interviews focused on characterisation methodology.

---

## Key Equations Reference

| Quantity | Equation | Notes |
|---|---|---|
| Efficiency | η = Pout / Pin = Vout×Iout / (Vin×Iin) | Measure all four values simultaneously |
| Power loss | Ploss = Pin - Pout | Small difference of large numbers — accuracy critical |
| Conduction loss (MOSFET) | Pcond = Irms² × Rds_on | Rds_on varies with Tj and Vgs |
| Switching loss | Psw = ½ × Vin × IL × (tr + tf) × fsw | Linear approximation |
| Diode loss | Pd = Vf × If_avg | Vf is forward voltage at operating current |
| Inductor copper loss | PCu = Irms² × DCR | DCR increases with temperature |
| Inductor core loss | Pcore = Pv × Ve | Pv from Steinmetz or datasheet curves |
| Capacitor ESR loss | PESR = Irms² × ESR | ESR often frequency- and temperature-dependent |
| Gate drive loss | Pgate = Qg × Vgs × fsw | Per MOSFET, Qg from datasheet |
| Controller quiescent | Pq = Vcc × Icc | Dominates at light load |
| Snubber loss | Psnub = ½ × C × V² × fsw | RC snubber dissipation |

---

## Tier 1 — Fundamentals

### Question 1: Why is measuring efficiency harder than it looks?

**Question:** A student measures a buck converter and gets η = 96% at full load. The input power is 100W and output power is 96W. What are the key sources of measurement error that could make this result incorrect?

**Answer:**

The fundamental challenge is that efficiency is the ratio of two large, similar numbers. A 4% loss means Ploss = 4W. If each measurement has 1% error, the error in Pin is ±1W and in Pout is ±0.96W — giving a potential error in Ploss of ±1.96W, which is nearly 50% of the actual loss figure.

**Key sources of measurement error:**

**1. Voltmeter placement (4-wire sensing)**

Wrong way: Voltmeter connected at the power supply terminals, not at the converter.

```
Power supply ──[R_wire]── Converter input
         ^                      ^
    Voltmeter?              Correct point
```

Voltage drops across wiring resistance directly corrupt the power calculation. For 100W at 12V with 10mΩ lead resistance and 8.3A, the lead drop is 83mV — a 0.7% error in Vin, giving 0.7% error in Pin.

**Correct practice:** Use 4-wire (Kelvin) measurement. Force current through the heavy power leads; sense voltage directly at the converter terminals using separate high-impedance sense wires.

**2. Averaging vs. true RMS meters**

Standard multimeters measure average and scale by the form factor of a sine wave (1.1107). For non-sinusoidal waveforms (switching converters), this gives incorrect readings.

For DC circuits with low ripple this is usually acceptable for DC voltages, but AC ripple on top of DC can cause averaging errors. Use true-RMS meters or dedicated power analysers.

**3. Current shunt heating**

A 100mΩ current shunt at 8.3A dissipates 8.3² × 0.1 = 6.9W — this is measured as part of "input power" but not delivered to the converter. Use low-value shunts (≤10mΩ typically) and account for shunt voltage drop in Vin measurements.

**4. Measurement timing**

For pulsed or dynamic loads, the efficiency depends on whether Pin and Pout are measured at the same instant. Use a power analyser that samples both simultaneously, or a digitising approach.

**5. Thermal steady state**

Efficiency changes as components warm up. Rds_on increases ~0.5%/°C for silicon MOSFETs, DCR increases ~0.4%/°C for copper. Always wait for thermal equilibrium (typically 15-30 minutes) before recording efficiency.

**Common mistake:** Measuring efficiency too quickly after applying load — the converter runs "cool" initially and shows better efficiency than it achieves in steady state.

---

### Question 2: What equipment is needed for an accurate efficiency measurement?

**Question:** Describe the measurement setup for characterising efficiency of a 200W 48V→12V synchronous buck converter from 10% to 100% load.

**Answer:**

**Minimum equipment:**

- Regulated DC power supply with accurate voltage and current readback, or separate precision voltmeter and ammeter on the input
- Precision electronic load (constant current mode or constant resistance mode)
- Separate precision voltmeters at converter input and output terminals (4-wire)
- Precision ammeters or current shunts at input and output

**Better setup — dedicated power analyser:**

A power analyser (e.g. Yokogawa WT310, Hioki PW3335) simultaneously samples:
- Input voltage and current → Pin with correct phase and RMS handling
- Output voltage and current → Pout
- Computes efficiency in real time

This eliminates timing mismatch errors.

**Wiring discipline:**

```
DC Source → heavy power wire → [input current shunt] → Converter input
                                                              |
                                                    [V_sense hi] ← separate wire
                                                    [V_sense lo] ← separate wire
                                                              |
                                              Converter output → [output current shunt] → E-load
                                                    [V_sense hi] ← separate wire at converter output pin
                                                    [V_sense lo] ← separate wire at converter GND pin
```

**Thermal considerations:**
- Run converter in a temperature-controlled environment, or note ambient temperature
- Allow 15-30 minutes for thermal stabilisation at each load point
- For power modules, mount on a heatsink representative of the final application

**Load sequencing:**
- Sweep from 10% to 100% in steps of 10% (or finer near the efficiency peak)
- Record: Vin, Iin, Vout, Iout, efficiency, output voltage regulation, ambient temperature
- Repeat in reverse order to detect hysteresis (thermal effects)

**Calibration:**
- Zero the current meters with no current flowing
- Verify voltmeter accuracy against a known reference
- Calculate shunt uncertainty: if shunt is ±0.1% and Iout = 16.7A, uncertainty is ±16.7mA

---

### Question 3: What is the DoE (Department of Energy) or 80 PLUS testing methodology?

**Question:** A power supply manufacturer claims 80 PLUS Gold certification. What does this mean in terms of measured efficiency, and how is the measurement conducted?

**Answer:**

**80 PLUS levels (for AC-DC power supplies, 115Vac):**

| Level | 20% load | 50% load | 100% load |
|---|---|---|---|
| 80 PLUS | 80% | 80% | 80% |
| Bronze | 82% | 85% | 82% |
| Silver | 85% | 88% | 85% |
| Gold | 87% | 90% | 87% |
| Platinum | 90% | 92% | 89% |
| Titanium | 90% | 92% | 94% |

**Why these three load points?**

Real-world server load profiles are rarely at 100%. The 50% load point is often where computers operate during typical use. The 20% load point captures light-load (idle/standby) efficiency which matters for data centre PUE calculations.

**Measurement methodology:**
- Input: 115Vac ±1%, 60Hz (or 230V version for Europe)
- Output: Resistive load achieving 20%, 50%, 100% of rated power
- Warm-up: 30 minutes minimum at 100% load before measurement
- Measurement: True RMS power measurement with calibrated equipment
- Power factor also measured (must be >0.9 for some categories)

**Relevance to DC-DC design:**

For DC-DC converters, similar thinking applies: specify efficiency targets at multiple load points, not just full load. Many standards (e.g., Energy Star, DoE Level VI for external power supplies) require minimum efficiency at 25%, 50%, 75%, and 100% of rated output power.

---

## Tier 2 — Intermediate

### Question 4: How do you perform a loss breakdown analysis?

**Question:** You measure a 12V→3.3V, 5A synchronous buck converter running at 400kHz and get η = 88% at full load. The simulation predicts η = 94%. How do you systematically identify where the "missing" efficiency went?

**Answer:**

At full load: Pin = Vout×Iout/η = 3.3×5/0.88 = 18.75W. Ploss_total = Pin - Pout = 18.75 - 16.5 = 2.25W.

Simulation predicts Ploss = 16.5/0.94 × 0.06 = 1.05W. Discrepancy = 1.20W.

**Step 1: Theoretical loss budget**

Calculate expected losses from component datasheets:

| Loss source | Equation | Values | Loss |
|---|---|---|---|
| High-side cond. | IL²_rms × Rds_on × D | (5A)²×25mΩ×0.275 | 172mW |
| Low-side cond. | IL²_rms × Rds_on × (1-D) | (5A)²×12mΩ×0.725 | 218mW |
| Switching (HS) | ½×Vin×IL×(tr+tf)×fsw | ½×12×5×5ns×400k | 60mW |
| Gate drive | 2×Qg×Vgs×fsw | 2×20nC×5V×400kHz | 80mW |
| Inductor DCR | IL²_rms × DCR | 25 × 8mΩ | 200mW |
| Inductor core | Pv × Ve | — | ~50mW |
| Cap ESR | Iripple²×ESR | — | ~20mW |
| Controller Icc | Vcc×Icc | 5V×5mA | 25mW |
| **Total** | | | ~825mW |

Simulation at 94% gives ~1050mW. Measurement at 88% gives 2250mW. Need to find 1400mW discrepancy.

**Step 2: Isolate sections**

*Thermal imaging:* Use an IR camera to identify hot components. A hotter-than-expected inductor points to core or copper loss. A hot controller IC suggests gate drive or quiescent loss.

*Calorimetric measurement:* Place converter in an insulated box with a thermocouple. In steady state, all power loss heats the box at a known rate. This is the most accurate loss measurement method but slow.

**Step 3: Decompose by section**

*Gate drive loss check:* Increase Vgs from 5V to 10V. If efficiency improves, Rds_on was dominant. If efficiency drops, gate drive switching loss was significant.

*Frequency scaling:* Reduce fsw from 400kHz to 200kHz. Switching-related losses (gate drive, transition loss, core loss) halve. Conduction losses stay constant. Measure efficiency delta to separate switching vs. conduction losses.

```
η(400kHz) - η(200kHz) ≈ [2 × P_sw(400kHz)] / Pin
```

*Dead time check:* Body diode conduction during dead time adds Vf_body × Iout × t_dead × fsw per transition. For Vf=0.7V, 5A, 50ns dead time, 400kHz: 2 transitions × 0.7 × 5 × 50n × 400k = 140mW. Optimise dead time.

**Step 4: Common discrepancies between simulation and measurement**

- Rds_on: Simulation uses 25°C value; device runs at 80°C, so Rds_on is 1.5-2× higher
- DCR: Simulation uses 25°C value; copper at 80°C has 20% higher resistivity
- Dead time: Simulation may assume ideal synchronous switching; actual dead time adds body diode loss
- Gate drive: Simulation may not model driver output impedance and Miller plateau correctly
- PCB resistance: Board trace resistance not in simulation

**The most common culprit in simulation-vs-measurement discrepancy is temperature coefficients of Rds_on and DCR.** Always re-check simulation at operating temperature.

---

### Question 5: How does efficiency vary with load, and why?

**Question:** Sketch and explain the efficiency vs. load curve for a typical CCM synchronous buck converter. Why does efficiency peak at an intermediate load rather than at full load?

**Answer:**

**Typical efficiency curve shape:**

```
Efficiency (%)
100 |
    |            ___________
 90 |          /             \
    |         /               \
 80 |        /                 \
    |       /                   \
 70 |      /                     \
    |_____/                       \____
    0    10    20    30    40    50    100
                    Load (%)
```

The curve peaks typically between 30-70% of full load, then decreases.

**Why efficiency drops at light load:**

Fixed losses (independent of output current) dominate at light load:
- Controller quiescent current: Vcc × Icc (e.g., 5V × 5mA = 25mW regardless of load)
- Gate drive loss: Qg × Vgs × fsw (fixed for fixed frequency operation)
- Core loss: proportional to B_peak which varies weakly with load at fixed duty cycle
- Bias supply losses

At 10% load (0.5A out), Pout = 1.65W. Fixed losses of 100mW represent 6% efficiency penalty alone.

**Why efficiency drops at full load:**

Variable losses (proportional to I² or I) dominate at full load:
- Conduction losses: Irms² × Rds_on, Irms² × DCR
- Body diode conduction: Vf × I × t_dead × fsw

At 5A out, I²R losses are 25× higher than at 1A. Even small resistances cause significant loss.

**The crossover point is where fixed losses = variable losses:**

```
P_fixed = P_variable(I)
P_fixed = I² × R_total
I_peak_eff = sqrt(P_fixed / R_total)
```

**Design implications:**
- For a system that mostly runs at light load (e.g., server in idle), optimise for 20-30% load efficiency even at some cost to full-load efficiency
- Use diode emulation (DCM at light load) or burst mode to reduce gate drive losses
- Use smaller Qg switches for light-load optimisation; larger switches for heavy-load optimisation
- Spread-spectrum + burst mode can jointly improve light-load efficiency

**Burst mode operation** (also called PFM at light load): at light load, the controller stops switching for many cycles ("skip" cycles). During the burst, the inductor charges normally. Between bursts, fixed losses stop accumulating. Efficiency improves significantly.

---

### Question 6: How do you measure switching losses separately from conduction losses?

**Question:** Describe an experimental method to separate switching losses from conduction losses in a hard-switched buck converter.

**Answer:**

**Method 1: Frequency scaling**

Since switching losses scale with fsw and conduction losses do not:

```
P_total(f1) = P_cond + P_sw(f1)
P_total(f2) = P_cond + P_sw(f2)

P_sw(f) = Psw_per_cycle × f
Psw_per_cycle = [P_total(f1) - P_total(f2)] / (f1 - f2)
P_cond = P_total(f1) - Psw_per_cycle × f1
```

Practical execution: vary fsw by 2:1 (e.g., 200kHz to 400kHz) using the PWM clock. Measure efficiency at each frequency at the same output power (adjust duty cycle to maintain Vout). The efficiency difference is almost entirely due to switching loss scaling.

**Method 2: Current-voltage waveform measurement**

Directly measure V_ds(t) and I_d(t) during the switching transitions using:
- High-bandwidth current probe on drain node (Rogowski coil or precision coax shunt)
- High-bandwidth differential voltage probe on V_ds

Instantaneous power: p(t) = v_ds(t) × i_d(t)

Switching loss per cycle:
```
E_sw = ∫ v_ds(t) × i_d(t) dt    (integrated over transition)
P_sw = E_sw × fsw
```

**Challenges with waveform method:**
- Probe bandwidth must be >> 1/t_rise (for 5ns rise time, need >200MHz bandwidth)
- Probe ground inductance corrupts the measurement (use coaxial shunt techniques)
- Diode reverse recovery appears as a current spike that must be included
- Oscilloscope time alignment between V and I probes must be calibrated
- Capacitive coupling between probes introduces error

**Method 3: Calorimetric isolation**

Physically separate the power stages: mount the MOSFET on a known thermal mass with thermistor. From the temperature rise rate during a load step, calculate the power dissipated in that component alone. This is slow but accurate and avoids probe bandwidth issues.

**Method 4: Gate charge measurement**

P_gate = Qg × Vgs × fsw (from datasheet, but verify with actual gate charge test circuit)
P_output_cap = ½ × Coss × Vds² × fsw (hard-charged capacitance)
P_body_diode = Qrr × Vin × fsw (reverse recovery)

Sum these switching contributions and compare to P_total - P_cond.

---

### Question 7: What is standby power and why does it matter?

**Question:** A laptop charger (45W rated) draws 180mW with no load attached. What are the sources of this standby power, and what regulations govern it?

**Answer:**

**Sources of standby power (180mW budget):**

In a flyback-based adapter with no load on the secondary:

1. **Primary-side controller:** Vcc × Icc. If Vcc = 20V (from auxiliary winding) and Icc = 3mA: 60mW.

2. **Switching losses at no load:** Even with very short pulses (PFM / burst mode), the MOSFET switches some charge onto Coss periodically. With 5µs burst every 500ms: minimal but measurable.

3. **Resistive dividers:** Voltage sensing dividers from Vout (even at 0V load, the feedback circuit has some current path). Less relevant at no load since Vout may collapse slightly.

4. **Optocoupler and TL431 bias current:** In the secondary regulation circuit, TL431 requires minimum bias (~1mA at 2.5V = 2.5mW). Optocoupler LED bias current.

5. **X capacitor bleed resistor:** To safely discharge the X capacitor after AC removal, a bleed resistor (often 100-470kΩ) across the AC input continuously dissipates P = Vac²/R. At 230Vac, 220kΩ: P = 230²/220k = 240mW. This alone can exceed the standby budget.

6. **EMI filter losses:** Small losses in the CM choke, X/Y caps at line frequency.

**Regulations:**

- **DoE Level VI (US, 2016):** For external power supplies, no-load power ≤ 0.1W for ≤49W output, ≤0.21W for 49-250W.

- **EU CoC Tier 2 (Europe):** Similar limits; ≤0.075W for up to 49W.

- **Energy Star:** Category-specific, generally aligned with DoE Level VI.

**Design techniques to meet limits:**

- Replace X-cap bleed resistor with an active bleed circuit that only discharges after AC removal (detected by missing zero-crossings)
- Use a low-Icc primary controller in skip mode (some controllers draw <200µA in no-load standby)
- Use a separate low-power auxiliary supply for the control IC during no-load
- Minimise optocoupler bias current; use a primary-only feedback scheme for simple adapters

**The X capacitor bleed resistor is the single largest contributor in many designs.** Removing it saves 100-300mW but requires a different discharge method to meet safety standards (IEC 62368 requires Vcap < 60V within 1s of AC removal).

---

### Question 8: How do you characterise efficiency across line and load?

**Question:** Describe how to generate a complete efficiency characterisation map for a 48V-input, 12V-output, 100W DC-DC converter with input range 36-75V.

**Answer:**

**Test matrix:**

Efficiency varies with both input voltage and load current. A complete characterisation requires a 2D sweep.

```
Inputs (Vin):  36V, 48V, 60V, 75V  (4 points)
Loads (Iout):  0.8A, 1.7A, 2.5A, 4.2A, 5.8A, 6.7A, 8.3A  (10-100% in steps)
```

Total: 4 × 7 = 28 measurement points minimum.

**Automated test procedure:**

```python
# Pseudocode for automated efficiency sweep
for vin in [36, 48, 60, 75]:
    source.set_voltage(vin)
    for iout in [0.83, 1.67, 2.50, 3.33, 4.17, 5.00, 5.83, 6.67, 7.50, 8.33]:
        eload.set_current(iout)
        time.sleep(thermal_settle_time)  # e.g. 120 seconds

        pin = meter_in.measure_power()    # Vin × Iin
        pout = meter_out.measure_power()  # Vout × Iout
        eta = pout / pin * 100

        log(vin, iout, eta, temperature_ambient)
```

**What to look for in the characterisation map:**

- Efficiency vs. Vin at fixed load: higher Vin means higher switching losses (½CVin²fsw, higher V across switch), so efficiency usually decreases with Vin for a buck. Boost shows opposite trend.
- Efficiency vs. load at fixed Vin: U-shaped curve as described in Q5.
- Worst-case efficiency: typically at low Vin (maximum duty cycle, maximum current) or high Vin (maximum voltage stress, maximum switching loss). Identify which dominates.
- Peak efficiency point: typically mid-load, nominal Vin.

**Additional characterisation:**

- Temperature sweep: repeat at Tamb = -40°C, +25°C, +85°C. Rds_on, DCR, and Vf all change significantly.
- Efficiency vs. Vout (if adjustable): relevant for VRM (voltage regulator module) designs.
- Dynamic efficiency: efficiency under pulsed load (server power delivery).

**Reporting:**

Present efficiency data as:
1. Efficiency curve at nominal Vin vs. load (primary datasheet curve)
2. Table of efficiency at all test points
3. Contour map (Vin vs. Iout) if a 2D characterisation is done
4. Loss breakdown pie chart at worst-case point

---

## Tier 3 — Advanced

### Question 9: How do you measure and decompose conduction losses in a synchronous rectifier?

**Question:** In a 12V-in, 1.2V-out, 20A synchronous buck converter, the synchronous MOSFET loss is suspected to be higher than simulated. How do you measure it accurately?

**Answer:**

At 1.2V/12V, duty cycle D ≈ 0.10. The synchronous rectifier (SR) is on 90% of the time carrying 20A. Its conduction loss dominates:

Expected: P_SR = Iout² × Rds_on(SR) × (1-D) = 400 × 5mΩ × 0.9 = 1.8W

**Measurement challenge:** At 1.2V output, a 1.8W loss is 7.5% of output power. The drain-source voltage during conduction is only 20A × 5mΩ = 100mV. Measuring this accurately requires precision.

**Method 1: Thermal resistance measurement**

Place the SR MOSFET on a heatsink of known Rth_sa with a calibrated thermistor on the heatsink. With steady DC current through the MOSFET (simulate conduction with a bench supply, not switching), measure ΔT for a known power, establishing:

```
Rth_j_to_heatsink = ΔT / P_applied
```

Then in the actual switching circuit, measure heatsink temperature rise. From Rth and ΔT, back-calculate MOSFET power dissipation.

**Limitation:** In the real circuit, switching losses are mixed with conduction losses in the thermal mass.

**Method 2: Frequency scaling for SR specifically**

The SR dead-time body diode loss = Vf_body × Iout × t_dead × fsw × 2

Measure efficiency with optimised dead time vs. excess dead time to isolate body diode contribution. The remaining SR loss is dominated by I²×Rds_on.

**Method 3: Resistance measurement via voltage drop**

Use a differential amplifier (CMR > 60dB) to measure V_ds of the SR during the on-time:

```
R_effective = <V_ds_on> / Iout
P_SR_cond = Iout² × R_effective × (1-D)
```

The differential amplifier must handle the common-mode swing (the switching node swings from -0.7V to +12V) while resolving 100mV differences. Use a precision instrumentation amplifier (INA128) and AC-couple the switching node to a separate channel to verify timing.

**Method 4: Infrared thermography comparison**

Compare IR temperature maps between: (a) original design, (b) same design with a known-better MOSFET. The temperature differential, combined with the known Rds_on difference, confirms the power loss difference.

**Common finding in low-voltage, high-current designs:**

PCB resistance is often comparable to MOSFET Rds_on. The SR MOSFET may show 5mΩ, but the source pad to ground plane resistance and drain pad to SW node resistance add another 3-5mΩ. This explains simulation-vs-measurement discrepancies. Measure the total resistance from SW node to GND through the SR path using a 4-wire milliohm measurement on the actual PCB.

---

### Question 10: What is the efficiency impact of reverse recovery in a synchronous buck converter?

**Question:** A synchronous buck converter uses a MOSFET as the synchronous rectifier. Explain how body diode reverse recovery affects efficiency and what can be done about it.

**Answer:**

**Physical mechanism:**

When the high-side MOSFET turns on, the low-side (SR) MOSFET body diode must stop conducting. The body diode has stored minority carrier charge Qrr. During the brief dead time where the SR is off but the high-side hasn't yet turned on, the body diode conducts (forward biased). When the high-side turns on, V_sw ramps up, but the body diode cannot block immediately — Qrr must be swept out first.

During the Qrr recovery time, the body diode is reverse-biased while still conducting. The MOSFET drain now sees Vin across it, and a large current spike flows:

```
P_rr = Qrr × Vin × fsw × 2   (both transitions)
```

For Qrr = 100nC (a typical 25V MOSFET body diode), Vin = 12V, fsw = 1MHz:

P_rr = 100nC × 12V × 1MHz × 2 = 2.4W

This is the reverse recovery loss allocated to the high-side MOSFET (it bears the current and voltage simultaneously).

**The reverse recovery also causes:**
- A current spike on V_sw that rings with PCB inductance → EMI
- Increased switching loss in the high-side MOSFET (V×I overlap during recovery)
- Possible turn-off oscillation if the body diode snaps off hard

**Solutions:**

1. **Schottky diode in parallel with the SR body diode:** Schottky has no Qrr (majority carrier device). The Schottky clamps at ~0.3V during dead time and turns off instantly. Eliminates recovery loss entirely. Cost: additional component, board space, reverse leakage current.

2. **Use GaN FETs:** GaN HEMTs have no p-n body diode (they use a reverse conduction path through the 2DEG channel). No Qrr. This is one of the main efficiency advantages of GaN at high frequency.

3. **Minimise dead time:** Shorter dead time = less charge pumped into the body diode. Use adaptive dead time control that measures the V_sw voltage and turns on the SR as soon as V_sw drops below ground. Some gate driver ICs (e.g., LM5113, UCC27714) provide this.

4. **Choose MOSFETs with "fast body diode":** Some MOSFETs are rated with Trr and Qrr specifications. Optimise for low Qrr rather than just low Rds_on in high-frequency designs.

**Trade-off:** Very short dead time risks shoot-through (both FETs on simultaneously). Adaptive dead time circuits must be fast enough to respond to load current changes.

---

### Question 11: How do you account for temperature effects on measured efficiency?

**Question:** You measure 91% efficiency at 25°C ambient. The product specification requires ≥88% efficiency at 70°C ambient. Estimate the efficiency at 70°C and explain what changes.

**Answer:**

**Components that degrade with temperature:**

**1. MOSFET Rds_on:**

Silicon MOSFET Rds_on follows approximately:
```
Rds_on(T) = Rds_on(25°C) × (T/300)^2.3    (T in Kelvin)
```

At Tj = 110°C (70°C ambient + 40°C rise):
```
Rds_on(110°C) = Rds_on(25°C) × (383/298)^2.3 = Rds_on(25°C) × 1.75
```

Conduction loss increases by 75%.

**2. Copper DCR:**

```
DCR(T) = DCR(25°C) × [1 + 0.00393 × (T - 25)]
DCR(85°C) = DCR(25°C) × 1.237
```

A 24% increase in DCR → 24% increase in copper loss.

**3. Diode forward voltage:**

Vf decreases with temperature (~-2mV/°C for silicon). This slightly reduces diode loss but is a secondary effect.

**4. Core loss:**

For N87 MnZn ferrite, core loss peaks around 100°C then decreases. If operating temperature moves toward the peak, core loss increases. If already past the peak, it decreases. Check the datasheet.

**Quantitative estimate:**

Assume at 25°C:
- Conduction loss: 4W (Rds_on + DCR)
- Switching loss: 2W (temperature-insensitive, primarily capacitive)
- Gate drive + fixed: 1W
- Total loss: 7W, Pout = 80W, Pin = 87W, η = 80/87 = 91.9%

At 70°C ambient (assume Tj ≈ 25°C higher → 95°C junction):
- Rds_on increases by ~40%: conduction loss × 1.4 for MOSFET portion
- DCR increases by ~15%: inductor loss × 1.15
- New conduction loss: ~5.2W (rough estimate)
- Total loss: 8.2W
- η = 80/88.2 = 90.7%

**This still meets the 88% specification.** However, the calculation must be done properly using actual component temperatures and the actual Rds_on vs. T curves from the MOSFET datasheet.

**Verification method:**

Run the converter at 70°C ambient in a temperature chamber. Measure efficiency. Also measure Rds_on of MOSFETs in-circuit using the thermal coefficient approach:

```
Rds_on(actual) = V_ds_on / I_d_on
```

measured during the steady-state on-time (requires a current probe and a fast differential voltmeter or scope).

---

### Question 12: What is output ripple and how does it affect efficiency measurements?

**Question:** A switching converter has significant output voltage ripple at the switching frequency. Explain how this affects efficiency measurements and what must be done to measure correctly.

**Answer:**

**The ripple-measurement interaction:**

A real output has: Vout(t) = Vout_DC + Vripple × sin(2π × fsw × t)

The output power is:
```
Pout = Vout_DC × Iout_DC + Vripple_rms × Iripple_rms × cos(φ)
```

If output ripple and load current ripple are in phase (resistive load), this cross-term is positive and the true power exceeds what a DC-only voltmeter would calculate.

**Effect on measurement:**

A true-power analyser measures Vrms × Irms × cos(φ) which correctly handles ripple. A simple DC voltmeter + DC ammeter misses the AC power component. For a converter with 100mVpp ripple into a 12mΩ load resistance:

- AC ripple current: 100mVpp / 12mΩ = 8.3App = 2.9Arms
- AC power at output: 100mV_rms × 2.9Arms = 290mW

This 290mW would be missed by a DC-only measurement, making efficiency appear better than it is.

**Practical guidelines:**

1. Use a power analyser with switching-frequency bandwidth (at least 10× the switching frequency in measurement bandwidth)

2. Alternatively, heavily filter the output before measurement: a large LC filter on the output sense lines removes the switching ripple, leaving only the DC component. Accept that you are measuring DC efficiency, not true efficiency including ripple losses.

3. For load regulation testing, always measure Vout at the converter output terminals (not across the load through long wires), using a 4-wire connection.

4. Document what is included in the efficiency measurement: "DC efficiency" vs. "true power efficiency including AC components."

**When does this matter most?**

- High-ripple designs (ΔIL/Iout > 40%)
- High-ESR output capacitors (electrolytic caps in high-ripple applications)
- Capacitor ESR losses appear in the output terminal voltage ripple and must be accounted for

**Good practice:** Add ceramic bypass capacitors at the output measurement point to absorb switching ripple, and use a separate LC-filtered sense lead for voltage measurement. This isolates the "clean" DC output voltage from the high-frequency ripple for the voltmeter.

---

### Question 13: How is efficiency measured for an AC-DC power supply (PFC + LLC)?

**Question:** A 250W server power supply uses a PFC front-end followed by an LLC resonant converter. Describe the efficiency measurement methodology and what efficiency number is typically reported.

**Answer:**

**System-level vs. stage-level efficiency:**

For a two-stage AC-DC supply, the total efficiency is the product of stage efficiencies:

```
η_total = η_PFC × η_LLC
```

If PFC achieves 96% and LLC achieves 94%, total = 90.2%.

**What is measured and reported:** The system-level efficiency — input AC power to output DC power:

```
η_total = (Vout × Iout) / Pin_AC
```

where Pin_AC is the true AC power (not just apparent power).

**Measurement for AC input:**

True AC power requires a power analyser that handles the non-sinusoidal input current drawn by a PFC converter. The input current waveform is nearly sinusoidal (>0.99 PF for good PFC) but has harmonic content. The power analyser must measure:

```
Pin_AC = Vrms × Irms × PF = Vrms × Irms × cos(φ1) × THD_correction
```

Minimum bandwidth: 50× the line frequency = 3kHz for 60Hz systems (to capture harmonics up to 50th).

**Per-stage measurement:**

To measure each stage separately, probe the intermediate bus (PFC output, typically 380-400V DC):
- PFC efficiency: P_bus_DC / P_AC_in
- LLC efficiency: P_DC_out / P_bus_DC

This requires measuring the intermediate bus voltage and current — practical if the bus is accessible (e.g., measurement points on the PCB).

**Total harmonic distortion impact:**

A PFC with THD = 10% and PF = 0.995 draws slightly more current than a unity-PF, zero-THD source. The true power is still Vrms × Irms × PF, but measuring Vrms × Irms without the PF correction overestimates input power by 1/PF ≈ 0.5%.

**Efficiency reporting for the whole system:**

At rated load (250W), input voltage (230Vac), and equilibrated temperature:

```
η = Pout / Pin_AC_true = 250W / (meter reading in true watts)
```

For 80 PLUS Titanium: must achieve 94% at 100% load, 92% at 50% load, 90% at 20% load.

**What degrades efficiency in this topology:**

- PFC: diode bridge loss, boost inductor loss, MOSFET switching at full line voltage, PFC capacitor ESR
- LLC: transformer copper and core loss, synchronous rectifier conduction loss, resonant tank element losses (Qr cap ESR, resonant inductor DCR)
- Both stages: controller quiescent, gate drive, EMI filter

**Resonant converter efficiency measurement note:**

LLC resonant converters have sinusoidal resonant tank currents that can be large relative to the average DC output current. A current probe measuring the transformer primary current sees a sinusoidal waveform — the RMS value is higher than the DC average by the resonant factor. Efficiency must be computed from true-RMS measurements on both input and output, not from average measurements.

---

## Quick Reference: Efficiency Measurement Checklist

### Pre-test Setup
- [ ] Four-wire (Kelvin) connections at converter input AND output terminals
- [ ] Current meters or shunts calibrated and zeroed
- [ ] True-RMS or power analyser grade instruments
- [ ] Thermal equilibrium protocol defined (minimum time at each load point)
- [ ] Ambient temperature recorded

### Measurement Execution
- [ ] Sweep load from minimum (10%) to maximum (100%) in defined steps
- [ ] Wait for thermal steady state at each point (verify with no change in reading over 2 minutes)
- [ ] Record Vin, Iin, Vout, Iout at each point
- [ ] Calculate η = (Vout × Iout) / (Vin × Iin) at each point
- [ ] Note any protection events or regulation deviations

### Loss Analysis
- [ ] Calculate total loss: Ploss = Pin - Pout
- [ ] Build theoretical loss budget from component datasheets (at operating temperature)
- [ ] Use frequency scaling to separate switching vs. conduction losses
- [ ] Use IR camera or thermocouple mapping to identify hottest components
- [ ] Reconcile simulation prediction vs. measurement

### Documentation
- [ ] Efficiency vs. load curve at nominal Vin
- [ ] Efficiency table at all Vin and Iout test points
- [ ] Worst-case efficiency point identified
- [ ] Loss breakdown pie chart or table
- [ ] Thermal equilibrium temperatures recorded
