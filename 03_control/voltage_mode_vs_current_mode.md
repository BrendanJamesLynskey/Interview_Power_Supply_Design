# Voltage-Mode vs. Current-Mode Control — Interview Preparation

## Overview

The choice of control architecture determines the converter's stability, transient response, noise immunity, and compensation complexity. Voltage-mode control (VMC) and current-mode control (CMC) represent the two fundamental approaches. Advanced variants — constant on-time (COT), average current mode, V² control — offer different trade-off points. This is one of the most commonly tested topics in power electronics interviews.

---

## Key Concepts Reference

### Voltage-Mode Control (VMC)

```
Error amplifier compares Vout to Vref → Vc (control voltage)
Comparator: Vc vs. sawtooth ramp → duty cycle
Single-loop (no inner current loop)
Plant transfer function (CCM buck): Gvd = Vin / (1 + s/ωz_esr) / [(1 + s×Q/ωLC + s²/ωLC²)]
Double LC pole at: ω_LC = 1/√(L×C), Q = √(L/C)/R
ESR zero at: ω_ESR = 1/(ESR×Cout)
```

### Peak Current-Mode Control (PCM)

```
Inner loop: sense primary/inductor current → compare to error amp output Vc
When IL reaches Vc: switch turns off (peak current limited)
Outer loop: error amp adjusts Vc to regulate Vout
Effective plant (current to output): integrator (1/sC) × load
Slope compensation required for D > 0.5 to prevent subharmonic oscillation
```

### Average Current-Mode Control (ACM)

```
Inner loop: current error amplifier drives average inductor current to reference
Slower current loop than PCM; more accurate current tracking
No subharmonic oscillation without slope compensation (slopes average, not peak)
Used in PFC boost, battery chargers, current-source output converters
```

### Constant On-Time (COT) Control

```
On-time Ton = K/Vin (constant for given Vin; adjusts D via frequency)
Frequency varies with load: f = Vout / (Vin × Ton)
Fast transient response (direct Vout sensing → immediate Ton trigger)
Switching frequency varies → complicates EMI filter design
Pseudo-fixed frequency variants use clock synchronisation
```

### V² Control

```
Both inductor current ripple and capacitor voltage ripple are fed back
V²: Vout ripple directly added to feedback → very fast output voltage ripple rejection
Instantaneous inductor current information in Vout ripple (via ESR) enables single-cycle response
Requires ESR-dominated output (works poorly with ceramic caps — too low ESR)
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Explain voltage-mode control for a buck converter. What are its advantages and limitations?

**Answer:**

**Operation:**

VMC uses a single feedback loop comparing the output voltage to a reference:
1. Error amplifier: `Vc = Av × (Vref - Vout)` where Av is the error amp gain
2. PWM comparator: `D = Vc / Vramp` where Vramp is the peak-to-peak sawtooth amplitude
3. Result: `D × Vin ≈ Vout` through the LC filter

**Plant transfer function (CCM buck, VMC):**

```
Gvd(s) = Vin / Vramp × [1 + s/ωz_esr] / [1 + s/(ωLC × Q) + (s/ωLC)²]

ωLC = 1/√(LC)
Q = R_load × √(C/L) / (1 + R_load × ESR × C × ωLC²)... [simplified: Q ≈ R_load/√(L/C)]
ωz_esr = 1/(ESR × Cout)
```

The double pole at ωLC creates a 180° phase shift in one decade — the compensator must recover this phase before the crossover frequency.

**Advantages of VMC:**
1. **Simple implementation:** Single comparator, one modulator.
2. **No current sensing:** No sense resistor or current transformer needed — less cost, no power loss.
3. **Works well with ceramic capacitors:** No ESR zero required to stabilise; capacitive output works fine.
4. **Noise immunity:** Output voltage measured directly; no switching noise in the feedback path (unlike current sensing).
5. **Consistent behaviour across load:** Plant does not change significantly with load in CCM.

**Limitations of VMC:**
1. **Double LC pole:** Requires Type III compensator (three poles, two zeros) to achieve adequate phase margin — more design complexity.
2. **Slow response to line transients:** Input voltage change affects output only after passing through the LC filter. No feedforward.
3. **Volt-second imbalance in bridge topologies:** No current limiting in inner loop — core saturation risk.
4. **Slower transient response vs. CMC:** Current mode responds to inductor current directly; VMC must wait for voltage to change.

---

### Q2. Explain peak current-mode control. How does it simplify compensation?

**Answer:**

**Operation:**

Peak CMC adds an inner current loop around the inductor (or primary) current:
1. Current sense: `Vsense = IL × Rsense` (resistor) or via current transformer
2. Inner comparison: when IL reaches Vc (error amp output), the switch turns OFF
3. Outer loop: Vc is adjusted to regulate Vout

**Why compensation is simpler:**

The inner current loop effectively converts the inductor into a voltage-controlled current source. The plant seen by the outer voltage loop is no longer:
```
Gvd(s) = Vin × [...LC resonance...] / [...]   [double pole]
```

Instead, it becomes approximately:
```
Gvc(s) ≈ 1 / (sC × Rload / (Rload + 1/sC)) ≈ 1/sC  [integrating behaviour]
          (single pole from output capacitor + load, not LC double pole)
```

The double pole is eliminated (inductor is controlled as a current source), leaving only a single output pole. A Type II compensator (one pole, one zero) is sufficient.

**Benefits:**
1. **Simpler compensation:** Type II instead of Type III
2. **Automatic current limiting:** Inner loop clamps peak current directly
3. **Fast line rejection:** Input voltage change immediately changes IL ramp slope → duty cycle adjusts within one cycle
4. **Natural current sharing in parallel converters:** Phases share the same Vc setpoint → current sharing inherent

**Disadvantages:**
1. **Slope compensation required for D > 0.5** (see Q4)
2. **Noise sensitivity:** Current sense signal is at the switching node — high-noise environment
3. **Gain variation with load:** Plant gain depends on Rload; compensation may need to accommodate gain variation
4. **RHP zero in boost/flyback:** Inner loop changes but does not eliminate RHP zero

---

### Q3. What is the transfer function of a VMC buck converter? Derive the key poles and zeros.

**Answer:**

**System model:**

For an ideal CCM buck converter (no parasitics initially):
```
Switch model: Vswitch = D × Vin  (averaged)
LC filter:    Vout(s) = Vswitch / (1 + sL/R + s²LC)
```

Adding ESR to the capacitor:
```
Cout impedance: Z_C(s) = ESR + 1/(sCout)
LC filter with load R: Vout = Vswitch × (Z_C || R) / (sL + Z_C || R)
```

After algebra, the control-to-output transfer function (control voltage Vc to Vout):
```
Gvd(s) = Vin/Vramp × (1 + s × ESR × Cout) / (1 + s × (L/R + ESR × Cout) + s² × L × Cout)
```

**Identifying poles and zeros:**

**ESR zero (numerator = 0):**
```
1 + s × ESR × Cout = 0
s_z = -1/(ESR × Cout)
f_z_esr = 1/(2π × ESR × Cout)
```
This zero comes from the ESR — it partially cancels the phase drop from the double pole. With ceramic caps (ESR ≈ 1 mΩ), this zero is at very high frequency (MHz) and provides no compensation help in the crossover region.

**Double pole (denominator = 0):**
```
1 + s × (L/R + ESR × C) + s² × LC = 0
ωn = 1/√(LC)
Q = ωn × L/R + ωn × ESR × C ≈ R/√(L/C) for low ESR
```

The double pole creates:
- -40 dB/decade gain rolloff above ωn
- 180° phase shift over one decade around ωn
- Peaking in gain at ωn for high Q (resonance)

**For a buck with L=6.8µH, C=47µF, R=1.65Ω (5V/3A load):**
```
f_LC = 1/(2π × √(6.8e-6 × 47e-6)) = 1/(2π × 1.788e-5) = 8.9 kHz
Q ≈ R/√(L/C) = 1.65/√(6.8e-6/47e-6) = 1.65/0.38 = 4.34  [resonance peak of +12.7 dB]
f_z_esr (ESR=10mΩ) = 1/(2π × 10e-3 × 47e-6) = 338 kHz
```

The compensator must cross over well below 338 kHz and must provide at least 180° phase recovery around 8.9 kHz to maintain stability.

---

### Q4. What is subharmonic oscillation in peak current-mode control and how is slope compensation applied?

**Answer:**

**Subharmonic oscillation mechanism:**

In peak CMC, the inductor current ripple ramps up during the on-time (slope = (Vin-Vout)/L = m1) and ramps down during the off-time (slope = Vout/L = m2).

Consider a perturbation: the inductor current starts slightly higher than nominal at the beginning of a cycle. The peak comparator turns the switch off slightly earlier → shorter on-time → current at end of cycle is slightly lower than nominal (the perturbation has changed sign and grown).

**For D < 0.5 (Vout < Vin/2):** m1 < m2. The off-time slope is steeper, so the perturbation recovers and diminishes each cycle → stable.

**For D > 0.5 (Vout > Vin/2):** m1 > m2. The on-time slope is steeper, so the perturbation grows each cycle → diverges into a period-2 oscillation (subharmonic at fsw/2).

**Mathematical criterion:**
```
Stable when: m2 > m1/2
             Vout/L > (Vin-Vout)/(2L)
             2×Vout > Vin - Vout
             3×Vout > Vin
             D < 2/3  [strict stability]
For guaranteed stability: D < 0.5
```

**Slope compensation:**

Add an artificial slope Sa to the current sense signal (or subtract from the control reference):
```
Slope-compensated peak threshold: V_th(t) = Vc - Sa × t  (decreasing ramp)
```

The added slope effectively damps the perturbation. Stability criterion with compensation:
```
Stable for all D when: Sa ≥ m2/2 = Vout/(2L)  [for m2 = Vout/L]
```

Typical choice: Sa = m2/2 (minimum for stability at all D) to Sa = m2 (generous margin, used in most controller ICs).

**Practical implementation:**

Controller ICs (e.g., UC3845, LM5030) generate slope compensation internally from the oscillator ramp. The slope can often be set by an external resistor or capacitor.

**Effect on gain:**

Slope compensation reduces the DC gain of the current loop by a factor:
```
Gain_reduction = m2 / (m2 + Sa)
```
For Sa = m2: gain reduced by 50%. This must be accounted for in outer loop compensation.

**Common interview mistake:** Stating that slope compensation is needed only for D > 0.5. In reality, for robust designs with converter input voltage variation, slope compensation is used whenever D could approach or exceed 0.5 under any operating condition.

---

### Q5. What is constant on-time (COT) control and what are its advantages for fast transient response?

**Answer:**

**COT operation:**

In COT control:
1. Output voltage is monitored continuously.
2. When Vout drops below the reference: on-time triggers immediately (switch turns on).
3. On-time duration: `Ton = K/Vin` (constant for given Vin, where K = L × Ipeak_ripple × Vin adjustment)
4. After Ton, switch turns off; circuit waits for next trigger.

**Switching frequency in steady state:**
```
D = Ton / Ts = Ton × fsw = K/Vin × fsw
Vout = D × Vin = K × fsw
→ fsw = Vout / K = Vout × Vin / (K × Vin) = Vout / Ton_typical
```

Frequency self-adjusts to maintain volt-second balance without explicit duty cycle control.

**Transient response advantage:**

The COT trigger is based on direct Vout comparison — there is no carrier ramp to wait for (unlike PWM). The moment Vout drops below the threshold, the switch turns on immediately. This provides sub-cycle response to load steps.

**Time to respond to a load step (COT):**
- Essentially zero delay (next on-time starts immediately when Vout drops)
- Only limitation is inductor slew rate (di/dt = (Vin-Vout)/L)

Compare to PWM VMC:
- Must wait for the next rising edge of the carrier ramp (up to Ts delay)
- Then error amp output changes duty cycle

COT can respond within nanoseconds while VMC may wait one full switching period (µs at typical fsw).

**Disadvantages of COT:**
1. **Variable switching frequency:** Frequency depends on Vin and load → EMI spectrum changes; harder to filter.
2. **Noise sensitivity:** Any noise coupling to the Vout sensing node can cause spurious triggering.
3. **Ceramic capacitor issues:** COT relies on ESR-induced voltage ripple to trigger the comparator. With very low ESR ceramics, the Vout ripple is capacitive (not at ESR zero) → triggering may become unreliable. Solution: add artificial ripple injection or use "ripple injection" variant.
4. **Multiple outputs:** Difficult to synchronise multiple COT converters.

**Pseudo-fixed frequency COT (e.g., Richtek RT7800, Maxim MAX17498):**
Adds a clock synchronisation mechanism that forces on-time to start at a fixed frequency unless transient demands immediate response. Gets best of both worlds: fixed frequency for EMI, immediate response for transients.

---

### Q6. Compare voltage-mode and peak current-mode control for a synchronous buck converter. Which is preferred in practice and why?

**Answer:**

**Comparison table:**

| Feature | Voltage Mode | Peak Current Mode |
|---------|-------------|-------------------|
| Compensator complexity | Type III (3 poles, 2 zeros) | Type II (2 poles, 1 zero) |
| Current limiting | Not inherent (separate OCP) | Inherent (cycle-by-cycle) |
| Line rejection | Requires feedforward | Inherent (fast) |
| Noise sensitivity | Low (voltage feedback clean) | High (switching noise) |
| Transient response | Moderate | Fast (one-cycle response) |
| Slope compensation | Not needed | Required for D > 0.5 |
| Multi-phase current sharing | External circuit needed | Inherent (share Vc) |
| Ceramic cap compatibility | Excellent | Excellent |
| Electrolytic cap stability | Easier (ESR zero helps) | Easier (single pole plant) |
| D > 0.5 operation | No issues | Requires slope comp |

**Industry practice:**

**Peak current mode dominates in most DC-DC converters** for the following reasons:
1. Type II compensator is simpler to design and more robust across operating conditions.
2. Cycle-by-cycle current limiting is a safety requirement in most applications.
3. Line-input rejection is critical in non-isolated converters with varying Vin.
4. Multi-phase VRMs inherently share current through the common Vc — essential for balanced phase operation.

**Voltage mode is preferred when:**
1. Output uses ceramic-only capacitors (no ESR zero available; PCM has no advantage in this case).
2. Very high switching frequency (>3 MHz) where current sense propagation delay becomes problematic.
3. The sensing environment is too noisy for reliable current sensing.
4. Multi-output isolated converters where secondary current sensing is impractical.

**Modern trend:** Many modern GaN-based converters and high-density designs use VMC with digital control and input voltage feedforward (Vin feedforward eliminates the line rejection advantage of CMC). Digital compensators can implement Type III easily, removing the compensator complexity argument against VMC.

---

## Intermediate (Questions 7–12)

---

### Q7. What is average current-mode control and when is it preferred over peak current mode?

**Answer:**

**Average CMC operation:**

Peak CMC controls the peak inductor current, not its average. For a triangular current waveform:
```
IL_avg = IL_peak - ΔIL/2
```
The error between peak and average = ΔIL/2, which varies with duty cycle and line voltage. This creates a load regulation error in the controlled current.

Average CMC adds a current error amplifier (CA) that integrates the difference between the sensed current and the reference:
```
Vc_CA = H(s) × (I_ref - IL_sensed)
```
The current loop drives the average inductor current to the reference, not just the peak.

**When average CMC is preferred:**

1. **Power factor correction (PFC):** The inductor current must track a sinusoidal reference accurately. PCM creates a 50% current ripple error at the peak; ACM provides the accurate average tracking needed for high power factor.

2. **Battery chargers:** Constant current charging requires precise average current control. PCM introduces ripple-dependent error.

3. **LED drivers:** Constant average current for luminance regulation.

4. **Current source outputs:** Any application requiring precise output current (not voltage) regulation.

**Compensation for average CMC:**

The inner current loop has a transfer function similar to a buck plant (inductor + capacitor seen from current perspective). The current amplifier typically uses a Type II compensator with crossover well below fsw/2.

The outer voltage loop (if present) closes outside the current loop.

**Key advantage over PCM:**

No slope compensation required — the averaging action inherently prevents subharmonic oscillation (the ramp slopes are averaged, not compared at peak).

---

### Q8. What is V² control and how does it achieve single-cycle transient response?

**Answer:**

**V² control principle:**

In V² control, both the output voltage error (Vout - Vref) and the AC ripple of Vout are fed back. The switching threshold is the sum of these:
```
V_compare = Vout_ripple + Vout_error_compensated
```

**Why ripple feedback works:**

In a buck converter with an output capacitor that has significant ESR:
```
Vout_ripple ≈ IL_ripple × ESR
```

The output voltage ripple directly reflects the inductor current ripple. Therefore, feeding back Vout ripple is equivalent to feeding back inductor current ripple — without a separate current sensor.

This creates an inner current loop from the voltage measurement alone.

**Single-cycle transient response:**

When a load step occurs:
1. Vout drops immediately (capacitor voltage changes).
2. The drop in Vout directly decreases the threshold for the PWM comparator.
3. The duty cycle increases within the same switching cycle.
4. The inductor current begins to ramp up immediately.

This is as fast as direct voltage sensing can go — no delay from current sensing, no carrier ramp delay (similar to COT).

**Dependency on ESR:**

V² control requires that Vout ripple reflects IL ripple — this requires ESR to be significant:
```
ESR_min ≈ 1/(8 × fsw × Cout)  [ESR zero should be below 8×fsw]
```

With ceramic capacitors (ESR < 1 mΩ), the ripple is capacitive-dominated and does not proportionally represent IL. V² control becomes unreliable and may exhibit incorrect behaviour or instability.

**Solution for ceramic capacitors:**

Inject artificial ripple proportional to inductor current into the feedback path:
- Use a sense resistor or DCR sensing to inject a voltage proportional to IL into the error amp output
- This creates "emulated V² control" compatible with ceramic capacitors

Used in Texas Instruments TPS548D22, Intersil ISL8002 families.

---

### Q9. Describe the inner and outer loop structure of a current-mode controlled boost PFC converter.

**Answer:**

**PFC boost converter requirements:**

The PFC boost must:
1. Regulate output voltage (DC bus, e.g., 400V) — outer voltage loop
2. Shape input current to follow sinusoidal input voltage (power factor correction) — inner current loop
3. Not disturb current regulation at 100/120 Hz (output voltage ripple) — loop design critical

**Inner current loop:**

Average current mode is standard for PFC:
```
I_ref = |sin(ωline × t)| × I_amplitude   [full-wave rectified sine reference]
Current error amp: Vc_inner = Ki(s) × (I_ref - IL)
PWM: D = Vc_inner / Vramp
```

The inner current loop must have bandwidth >> 2×fline (>240 Hz for 120 Hz line) to accurately track the sinusoidal reference. Typical current loop bandwidth: 10–50 kHz.

**Outer voltage loop:**

Regulates 400V DC bus. Must be slow (bandwidth < 20 Hz) to avoid distorting the sinusoidal current reference:
```
I_amplitude = Gv(s) × (Vref - Vout)
```

If the outer loop is fast (>50 Hz), it would adjust I_amplitude rapidly in response to the 100/120 Hz output ripple — this modulates the peak current reference at 100/120 Hz, creating 3rd harmonic distortion in the input current.

**Loop bandwidth requirement for PFC:**
- Inner current loop: > 10 kHz (fast enough to track 60 Hz sinusoid)
- Outer voltage loop: < 10 Hz (slow enough to not react to 120 Hz ripple)

**Input voltage feedforward:**

The amplitude of I_ref is divided by Vin_rms² (after low-pass filtering to get the RMS value):
```
I_amplitude = Gv(s) × (Vref - Vout) / Vin_rms²
```

This provides line voltage feedforward — if Vin increases, the current reference decreases proportionally to maintain constant power, improving harmonic distortion performance.

---

### Q10. What is the control-to-output transfer function for a CCM peak current-mode controlled buck? How does slope compensation affect it?

**Answer:**

**Without slope compensation:**

The inner current loop transforms the plant. The effective transfer function from control voltage Vc to output voltage Vout is approximately:
```
Gvc(s) ≈ Ri/R × 1/(1 + s × C × Rp)  × 1/s... [single-pole approximation]
```

More precisely, the sampled-data model (He's model) shows that PCM introduces a sampled pair of poles (subharmonic poles) at fn = fsw/2:
```
He(s) = 1/(1 + s × Ts/(2π²) + (s/(π × fsw))²)
```

These subharmonic poles add phase at fsw/2. With slope compensation, these poles move to higher frequency or are damped.

**With slope compensation (Sa compensation ratio Q_sc):**

The PCM transfer function modifications:
```
Q_PCM = -1/(π × (Mc - 0.5))
where Mc = (m1 + Sa) / m2, m1 = (Vin-Vout)/L, m2 = Vout/L
```

For minimum slope compensation (Sa = m2/2, Mc = 1.5):
```
Q_PCM = -1/(π × 1.0) = -0.318   → negative Q (over-damped sampled poles)
```

This means the sampled poles are well-damped at fsw/2, and the current loop behaves stably at all duty cycles.

**Practical effect on outer voltage loop:**

The simplified control-to-output (after inner loop) for PCM is approximately:
```
Gvc(s) ≈ Ri_eff / (Cout × s) × 1/(1 + s/ωp2)

Single pole at: ωp = 2/(Rload × Cout)  [output pole from capacitor and load]
No double LC pole (inductor is "absorbed" into the current source model)
```

Type II compensator (one integrator + one zero) is sufficient for the outer loop.

---

### Q11. How does a VRM (CPU voltage regulator) controller achieve fast transient response through current mode and adaptive techniques?

**Answer:**

**VRM transient challenge:**

Modern CPUs demand:
- 50–100A load steps with < 1 ns rise time
- Output voltage maintained within ±50mV (50mV window on 1V rail)
- Response must be complete within 10–100 µs

**Layer 1 — Bulk output capacitors:**

During the first 1–50 µs, before inductors can respond, bulk output capacitors supply current:
```
Capacitor slew rate: ΔI = C × dV/dt → capacitor maintains Vout during transient
Typical: 500–1000 µF of mixed MLCC + polymer
```

**Layer 2 — Peak current mode with fast gate timing:**

PCM allows the VRM to detect (via inductor current comparison) that more current is needed and immediately ramp up within one cycle. The gate turn-on delay from the gate driver (5–20 ns) is the bottleneck, not the control loop.

**Layer 3 — Adaptive on-time extension:**

During a large load step, some VRM controllers detect a fast Vout undershoot and temporarily extend the on-time beyond the nominal duty cycle:
```
If dVout/dt < threshold (fast drop detected):
    Extend on-time → faster inductor ramp → reaches new load sooner
```

This "bang-bang" or "emergency response" mode bypasses the normal control loop for the first few cycles after a load step.

**Layer 4 — Active voltage positioning (AVP):**

AVP pre-positions the steady-state voltage:
- At light load: Vout is higher (headroom for undershoot)
- At heavy load: Vout is lower (headroom for overshoot)

This doubles the effective transient window without requiring larger capacitors.

**Layer 5 — Adaptive dead time:**

Optimising the body diode conduction time (dead time between HS/LS gate signals) increases efficiency, which indirectly allows higher operating frequency (higher fsw → faster inductor response per unit time).

---

### Q12. Design a slope compensation network for a peak CMC buck converter. Given: Vin=12V, Vout=5V, L=10µH, fsw=200kHz, Rsense=50mΩ.

**Answer:**

**Step 1 — Calculate inductor slopes:**
```
m1 (rising slope) = (Vin - Vout) / L = (12 - 5) / 10e-6 = 700,000 A/s = 0.7 A/µs
m2 (falling slope) = Vout / L = 5 / 10e-6 = 500,000 A/s = 0.5 A/µs
```

**Step 2 — Convert to sense voltage slopes:**
```
Vsense ramp: ms1 = m1 × Rsense = 0.7e6 × 0.050 = 35,000 V/s = 35 mV/µs
             ms2 = m2 × Rsense = 0.5e6 × 0.050 = 25,000 V/s = 25 mV/µs
```

**Step 3 — Duty cycle check:**
```
D = Vout / Vin = 5/12 = 0.417 < 0.5 → strictly speaking, no slope compensation required
```

But for robust design (Vin could be 8V → D = 0.625 > 0.5), add slope compensation anyway.

**Step 4 — Minimum slope compensation:**
```
Sa_min = ms2/2 = 25/2 = 12.5 mV/µs
```

Choose Sa = ms2 = 25 mV/µs (full m2 slope) for better phase margin and stability.

**Step 5 — Implement slope compensation:**

Most controller ICs generate slope compensation internally from the oscillator. For external implementation:

Method: Add a resistor (Rsc) from the oscillator capacitor to the current sense pin.

The oscillator cap charges from 0 to Vosc during Ts:
```
Vosc_slope = Vosc / Ts = Vosc × fsw
```

Attenuated through Rsc and Rsense:
```
Sa = Vosc × fsw × Rsense / (Rsc + Rsense) ≈ Vosc × fsw × Rsense / Rsc  (if Rsc >> Rsense)
```

For Vosc = 2V (typical oscillator amplitude), Sa target = 25 mV/µs = 25 V/ms = 25,000 V/s:
```
Sa = 2 × 200e3 × 0.050 / Rsc = 20,000 / Rsc
Rsc = 20,000 / 25,000 = 0.8 Ω → use 1 kΩ (check the actual oscillator circuit — this formula depends on specific IC topology)
```

**Verify:** Check that the slope compensation reduces Q_PCM to a stable value:
```
Mc = 1 + Sa/ms2 = 1 + 25/25 = 2
Q_PCM = -1/(π × (Mc - 0.5)) = -1/(π × 1.5) = -0.212
```

The current loop is well-damped (Q = -0.212 is stable and effectively overdamped).

---

## Advanced (Questions 13–16)

---

### Q13. Compare COT, VMC, and PCM for a 5MHz, 1.0V/10A smartphone PMIC application.

**Answer:**

**Application context:**

Smartphone PMIC requires:
- Very small inductor (L = 0.47–1 µH, constrained by height < 1 mm)
- Ceramic output capacitors only (2–5 × 10 µF, 1.5V-rated)
- High switching frequency to minimise L and C size
- Fast transient response for CPU/GPU load steps
- Battery input (3.0–4.5V), 1.0V output

**D at full battery:** D = 1.0/4.5 = 0.22

**COT analysis:**
- Fast transient: immediately turns on when Vout drops → excellent
- Variable frequency: challenging at 5 MHz — already difficult to filter
- Ceramic cap issue: low ESR means no natural ripple trigger → requires ripple injection
- Best for: very fast response with small L and no ceramic cap ripple issues (with injection)

**VMC analysis:**
- Type III compensator at 5 MHz is complex (poles and zeros must be at multi-MHz)
- LC pole at: fLC = 1/(2π × √(0.47e-6 × 30e-6)) = 1/(2π × 3.76e-6) = 42 kHz
- High Q at light load → stability challenges
- ESR zero at: 1/(2π × 0.001 × 30e-6) = 5.3 MHz — not useful within bandwidth
- Not ideal without significant digital assistance

**PCM analysis:**
- D = 0.22 < 0.5 → no slope compensation needed
- Current sensing at 5 MHz requires very fast sense amplifier (bandwidth > 50 MHz)
- Propagation delays become significant at 5 MHz (20 ns = 10% of period at 5 MHz)
- Natural current limiting — good for battery-powered designs
- Good for 5 MHz but requires fast gate driver IC and careful layout

**Recommendation for 5 MHz smartphone PMIC:**

COT with ripple injection (most modern PMIC designs at >2 MHz use COT variants):
- TI TPS62870, Renesas ISL91107, Maxim MAX20001 — all use COT or pseudo-COT at 2–5 MHz
- Constant (or pseudo-constant) on-time with artificial ripple injection solves the ceramic cap limitation
- Achieves sub-cycle response for CPU load steps
- Frequency dithering manages EMI despite variable frequency

---

### Q14. What is the effect of right-half-plane (RHP) zero on current-mode controlled boost converter bandwidth?

**Answer:**

**RHP zero in boost/flyback:**

The RHP zero occurs in boost converters (and flyback CCM) because increasing duty cycle initially decreases output current:
- Longer D → longer switch on-time → diode is off longer → less average diode current → output voltage drops initially
- Before recovering: Vout drops even as D increases

**RHP zero location (CCM boost, current mode):**
```
f_RHP = (1-D)² × Rload / (2π × L)

For Vin=5V, Vout=12V (D=0.583), Rload=14.4Ω (Pout=10W), L=10µH:
D = 1 - 5/12 = 0.583
f_RHP = (0.417)² × 14.4 / (2π × 10e-6)
      = 0.174 × 14.4 / (62.8e-6)
      = 2.505 / 62.8e-6
      = 39.9 kHz
```

**Phase shift contribution:**

The RHP zero at 39.9 kHz adds -90° phase shift (same as a pole) while adding +20 dB/decade gain (same as a zero). The net effect:
- Gain continues to rise past f_RHP
- Phase drops by 90° at f_RHP

**Bandwidth limitation:**

The control loop must cross over below f_RHP/5 to maintain adequate phase margin:
```
f_crossover_max = f_RHP / 5 = 39.9 / 5 = 8 kHz
```

At 8 kHz crossover with a 200 kHz switching frequency, the ratio is only 2.5% — very conservative. Transient response will be slow.

**Mitigation:**
1. Increase L → reduces f_RHP (actually makes it worse): L↑ → f_RHP↓. Use minimum L.
2. Reduce D → higher f_RHP. Not always possible.
3. Increase fsw → allows higher crossover while maintaining sufficient phase at f_RHP.
4. Accept DCM at light load (RHP zero disappears in DCM).

For the example: increasing fsw to 1 MHz allows f_crossover ≈ 20 kHz (limited by f_RHP to 8 kHz — fsw doesn't help the f_RHP limit, only allows the compensator to be designed around it more freely). Only increasing Vout/Vin ratio (lower D) or accepting DCM at light load truly helps.

---

### Q15. What is ripple injection in constant on-time control and why is it necessary with ceramic output capacitors?

**Answer:**

**The ceramic capacitor problem:**

COT control relies on the output voltage ripple to determine when to trigger the next on-time. The trigger fires when Vout drops below the reference — the ripple causes periodic triggering at the desired frequency.

With ESR-dominated capacitors (electrolytic, polymer):
```
Vout_ripple ≈ ΔIL × ESR   [fast ESR ripple, large amplitude]
```
The trigger fires reliably each cycle.

With capacitive-dominated ripple (ceramic caps, very low ESR):
```
Vout_ripple ≈ ΔIL / (8 × fsw × Cout)   [slow, very small amplitude]
```
The ripple amplitude may be only 1–2 mV, insufficient to reliably trigger the comparator each cycle. The result: missed triggers, inconsistent frequency, poor regulation.

**Ripple injection solution:**

Inject a synthetic ripple proportional to inductor current into the feedback voltage:
```
V_fb_modified = Vout / (1 + R2/R1) + V_injected

V_injected = IL × Rin_inject   [via current sense network in parallel with L or Rsense]
```

The injected ripple tracks the inductor current (same waveform shape), providing a reliable periodic signal for the comparator even with ceramic capacitors.

**Injection methods:**

1. **DCR injection network:** RC network across inductor. Time constant = L/DCR. V_C = IL × DCR is injected into the feedback node through a resistor divider.

2. **Sense resistor injection:** If a sense resistor is present, its voltage is directly injected.

3. **Internal artificial ripple (proprietary controller):** Some ICs generate artificial ripple internally from the switching signal (e.g., TI D-CAP, D-CAP2, D-CAP3 architectures).

**D-CAP2 (TI adaptive on-time):**

The D-CAP2 controller (used in TI TPS546D24A and similar) uses the sensed inductor current (via DCR) as the ripple source. This provides a clean, artificial ripple that:
- Is proportional to inductor current (correct waveform shape)
- Works with ceramic output capacitors
- Maintains stable switching frequency (pseudo-fixed via clock sync)
- Provides full average current sensing for sharing

---

### Q16. Summarise the control architecture choices for different converter types in a single comparison table.

**Answer:**

| Topology | Power Range | Recommended Control | Reason |
|----------|-------------|--------------------|----|
| Buck VRM (CPU) | 10–300W | Peak CMC + COT variant | Fast transient, current sharing |
| Buck general | 1–50W | Peak CMC | Simple comp, inherent OCP |
| Boost PFC | 100W–10kW | Average CMC | Accurate sinusoidal current tracking |
| Flyback DCM | 1–100W | VMC or PCM | DCM: no RHP zero; simple plant |
| Flyback CCM | 50–300W | PCM | Manages RHP zero via current loop |
| Forward | 50–500W | PCM | Volt-second balance via current sensing |
| Half/Full bridge | 200W–10kW | PCM with slope comp | Prevents saturation |
| LLC resonant | 100W–10kW | Voltage mode (FM) | Frequency control, no current loop needed |
| Multiphase buck | 10A–200A | PCM (peak or avg) | Phase current sharing |
| Battery charger (CC/CV) | all | Average CMC | Accurate constant-current in CC mode |
| Bidirectional | 1–50kW | VMC or Digital | Complex mode switching |
| PSFB | 500W–10kW | PCM | ZVS control, volt-second balance |
