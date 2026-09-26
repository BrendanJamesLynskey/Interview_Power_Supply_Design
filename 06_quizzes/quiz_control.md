# Quiz: Control Theory for Power Converters

20 multiple-choice questions covering feedback control, stability analysis, compensator design, and digital control. Each question has four options; only one is correct. Explanations cover why wrong answers are wrong.

**Scoring guide:** 18-20 correct = expert level; 14-17 = solid intermediate; 10-13 = review the control section; below 10 = start with Bode plot fundamentals.

---

## Q1

A Type II compensator is designed for a voltage-mode buck converter. The compensator has one pole at the origin, one zero at 3kHz, and one pole at 20kHz. What is the phase contribution of this compensator at the crossover frequency of 10kHz?

- A) -90°
- B) -43°
- C) +30°
- D) +45°

**Correct answer: B**

**Explanation:**

A Type II compensator has the transfer function:
```
Gc(s) = K × (1 + s/ωz) / [s × (1 + s/ωp)]
```

The phase contribution is:
```
∠Gc(jω) = -90° + arctan(f/fz) - arctan(f/fp)
```

At f = 10kHz, fz = 3kHz, fp = 20kHz:
```
∠Gc = -90° + arctan(10/3) - arctan(10/20)
     = -90° + arctan(3.33) - arctan(0.5)
     = -90° + 73.3° - 26.6°
     = -43.3°
```

So the compensator's phase at 10kHz is about -43° (option B). Relative to a pure integrator (-90°), the zero-pole pair is providing a phase boost of 73.3° - 26.6° = +46.7° at this frequency. (The maximum boost occurs at sqrt(fz × fp) = sqrt(3k × 20k) = 7.75kHz, where the phase is -42.4°, so 10kHz is close to optimum.)

**For the interview, the key formula is:**
```
φ_Type_II = -90° + arctan(fc/fz) - arctan(fc/fp)
```
At fc = 10kHz, fz = 3kHz, fp = 20kHz: ≈ -43°

**Why A is wrong:** -90° is the phase of a pure integrator (Type I compensator with no zero or additional pole). The zero in the Type II compensator adds positive phase relative to the pure integrator.

**Why C is wrong:** +30° is not reachable: a Type II compensator's phase stays between -90° and 0° because the integrator contributes -90° at every frequency and the zero adds at most +90°.

**Why D is wrong:** +45° confuses the phase boost (+46.7°, measured relative to the integrator) with the absolute phase. The Type II compensator never reaches positive absolute phase — it always has the integrator's -90° as a baseline.

---

## Q2

What is the "gain crossover frequency" of a control loop, and why does it matter?

- A) The frequency at which the phase of the open-loop transfer function crosses -180°
- B) The frequency at which the magnitude of the open-loop gain crosses 0dB (unity gain)
- C) The frequency at which the closed-loop bandwidth is maximised
- D) The switching frequency divided by π

**Correct answer: B**

**Explanation:**

The gain crossover frequency fc (or ωc) is defined as the frequency where:
```
|T(jωc)| = 1   (or equivalently, 0 dB)
```

where T(s) is the open-loop gain (loop gain). This is the frequency at which the control loop transitions from "loop gain > 1" (controlling region) to "loop gain < 1" (open-loop region).

Its importance:
- The crossover frequency approximates the closed-loop bandwidth — the speed of the closed-loop system response
- The phase of T(jωc) determines the phase margin: PM = 180° + ∠T(jωc)
- Selecting fc too high risks instability (insufficient phase margin) and can interact with sampling delays or parasitics
- Selecting fc too low makes the converter slow to respond to load transients

**Why A is wrong:** The frequency where phase crosses -180° is the phase crossover frequency (ωpc). This determines the gain margin: GM = -|T(jωpc)| in dB. It is a separate and equally important stability metric.

**Why C is wrong:** Maximising closed-loop bandwidth is a design objective, not a definition. The closed-loop -3dB bandwidth is related to but not identical to the gain crossover frequency. A well-designed loop has closed-loop bandwidth ≈ fc.

**Why D is wrong:** fsw/π has no standard control-theoretic meaning. While there is a rule of thumb that fc should be ≤ fsw/10 or fsw/5, and the Nyquist frequency for a digital system sampled at fsw is fsw/2, none of these equal fsw/π in a meaningful way.

---

## Q3

A boost converter in CCM has a right-half-plane (RHP) zero. What is the effect of the RHP zero on the Bode plot?

- A) It increases magnitude and adds positive phase (like a left-half-plane zero)
- B) It increases magnitude and subtracts phase (unlike a left-half-plane zero)
- C) It decreases magnitude and subtracts phase (like a left-half-plane pole)
- D) It decreases magnitude and adds positive phase

**Correct answer: B**

**Explanation:**

A right-half-plane zero at s = +ωz has the transfer function factor: (1 - s/ωz).

**Bode magnitude:** |1 - jω/ωz| = sqrt(1 + (ω/ωz)²)

This is identical in magnitude to a left-half-plane zero (1 + s/ωz) — the magnitude curve is the same: flat at 1 for ω << ωz, rising at +20dB/decade for ω >> ωz.

**Bode phase:** ∠(1 - jω/ωz) = arctan(-ω/ωz) = -arctan(ω/ωz)

For a left-half-plane zero (1 + jω/ωz): phase = +arctan(ω/ωz) (positive)
For a right-half-plane zero (1 - jω/ωz): phase = -arctan(ω/ωz) (negative)

The RHP zero adds magnitude gain (like a regular zero) but subtracts phase (like a pole). This makes it particularly dangerous in control design: it contributes +20dB/decade of gain (helping to push the loop gain above 0dB), while simultaneously reducing phase margin.

The physical manifestation: when duty cycle increases, the output initially decreases (energy is stored in the inductor, stealing from the output temporarily) before eventually increasing. This non-minimum-phase response is the time-domain equivalent of the RHP zero.

**Why A is wrong:** A left-half-plane zero increases magnitude AND adds positive phase. The RHP zero adds magnitude but subtracts phase — the phase behaviour is opposite.

**Why C is wrong:** A left-half-plane pole decreases magnitude at +20dB/decade and subtracts phase. The RHP zero increases magnitude and subtracts phase — the magnitude behaviour is opposite (though the phase behaviour is the same).

**Why D is wrong:** No standard transfer function element decreases magnitude and adds positive phase at the same frequency simultaneously. This does not correspond to any physical RHP zero or pole.

---

## Q4

Peak current mode control (PCMC) inherently provides which of the following?

- A) A double pole at half the switching frequency in the current loop
- B) Noise immunity superior to voltage mode control
- C) An inner current loop that makes the inductor appear as a voltage-controlled current source, simplifying the outer voltage loop
- D) Immunity to subharmonic oscillation for all duty cycles

**Correct answer: C**

**Explanation:**

In PCMC, the inner current loop samples the inductor current each switching cycle and compares it to a reference (set by the error amplifier output). When the inductor current reaches the reference, the switch turns off.

This inner loop has the effect of making the inductor + switch combination look like a transconductance stage: the control voltage (error amp output) directly commands the inductor peak current. For the outer voltage loop, the inductor no longer presents as an L with a double pole — it behaves more like a current source with a single pole at the output RC filter.

Practical result: the voltage loop plant for PCMC has a dominant single pole (at 1/2π×R×C) rather than the double LC pole of voltage mode control. This makes the outer voltage loop much easier to compensate — a simple Type II (or even Type I) compensator is usually sufficient.

**Why A is wrong:** The double pole at half the switching frequency (fs/2) is a sampling effect (the subharmonic pole) that exists in the PCMC current loop and is a PROBLEM, not a benefit. At duty cycles above 0.5, this pole causes subharmonic oscillation. It is mitigated by slope compensation.

**Why B is wrong:** PCMC is actually more susceptible to noise than voltage mode control in some respects. The current comparator triggers on the instantaneous inductor current waveform, which has switching noise superimposed. A small noise spike can cause a premature trip of the comparator (false turn-off), causing jitter and potentially subharmonic behaviour. Voltage mode control integrates the error signal, providing better noise filtering.

**Why D is wrong:** PCMC is susceptible to subharmonic oscillation when D > 0.5 (WITHOUT slope compensation). Slope compensation (adding a ramp to the current sense signal) is required to stabilise PCMC at D > 0.5.

---

## Q5

A converter loop is measured to have phase margin PM = 10° and gain margin GM = 3dB. What would you expect to observe in the step response?

- A) Clean, fast step response with no overshoot
- B) Heavy ringing that takes many cycles to settle (near-unstable)
- C) Critically damped response (no overshoot, slightly sluggish)
- D) The converter oscillates continuously at the crossover frequency

**Correct answer: B**

**Explanation:**

Phase margin and gain margin are related to the damping of the closed-loop system. Very low values indicate a lightly damped system:

- PM = 10° corresponds to a damping ratio ζ ≈ PM/100 ≈ 0.10 (rough approximation), giving oscillatory step response with large overshoot (~73% overshoot for ζ = 0.1)
- GM = 3dB means the loop gain can only increase by a factor of √2 (≈1.41×) before the system becomes unstable

The step response will show significant ringing at approximately the crossover frequency, with the ringing decaying very slowly (low ζ). This is the hallmark of a nearly-unstable control loop.

**Typical acceptable specifications:** PM > 45°, GM > 6dB. At PM = 45°, ζ ≈ 0.4-0.5, giving ~16-25% overshoot with reasonable settling. At PM = 60°, ζ ≈ 0.6, giving ~5-10% overshoot.

**Why A is wrong:** Clean fast response with no overshoot corresponds to PM > 60°, GM > 10dB. PM = 10° is extremely marginal — the response would be far from clean.

**Why C is wrong:** Critical damping (ζ = 1) corresponds to PM ≈ 76° in many systems. PM = 10° is nowhere near critically damped.

**Why D is wrong:** Continuous oscillation indicates PM ≤ 0° (unstable system). PM = 10° is positive, so the system is theoretically stable. However, in practice, component tolerances and nonlinearities can push a PM = 10° design into oscillation. The system is at risk but not guaranteed to oscillate.

---

## Q6

In a voltage mode controlled buck converter, what is the primary effect of the ESR zero on the control loop design?

- A) It destabilises the loop by adding a right-half-plane zero
- B) It provides helpful phase boost at frequencies above the ESR zero, potentially allowing a higher crossover frequency
- C) It eliminates the need for a compensator zero
- D) It must always be cancelled by a compensator pole at the same frequency

**Correct answer: B**

**Explanation:**

The ESR zero arises from the output capacitor's equivalent series resistance:
```
ωz_ESR = 1 / (ESR × C)
```

This is a left-half-plane zero in the control-to-output transfer function. It causes the magnitude to level off (or rise at +20dB/decade) above the zero frequency, and adds positive phase. This phase boost is beneficial:

- The LC double pole causes -180° of phase lag at frequencies just above ω0
- The ESR zero adds positive phase back, partially recovering some phase
- If the ESR zero is below the crossover frequency, the phase margin can be improved significantly

For electrolytic capacitors: ESR is high, so f_ESR = 1/(2π×ESR×C) is at low frequency (often 10-100kHz range). This places the ESR zero usefully within the control bandwidth.

For ceramic capacitors: ESR is very low (milliohms), so f_ESR = 1/(2π×ESR×C) is at very high frequency (MHz range), well above the crossover. No phase benefit within the control bandwidth.

**Design implication:** With ceramic output capacitors, the ESR zero disappears from the compensation challenge, but all the phase recovery from ESR is also gone. The compensator must provide all necessary phase boost.

**Why A is wrong:** The ESR zero is a LEFT-half-plane zero, not a right-half-plane zero. It adds positive phase (beneficial), not negative phase. A right-half-plane zero would destabilise the loop; the ESR zero does the opposite.

**Why C is wrong:** The ESR zero helps, but it rarely eliminates the need for a compensator zero entirely. The LC double pole requires significant phase boost near ω0, and the ESR zero may not be at the right frequency or magnitude to provide all the needed boost. A compensator with a zero near the LC resonance is typically still needed.

**Why D is wrong:** Cancelling the ESR zero with a compensator pole is an option but not a requirement. The ESR zero is beneficial — placing a compensator pole at the same location would neutralise the benefit. Cancellation is sometimes done for notching purposes (e.g., to prevent the ESR zero from extending the bandwidth too far), but is not always required or even desirable.

---

## Q7

What is the Middlebrook Criterion for stability of a cascaded power system (source converter + load converter)?

- A) The load converter's input impedance must be resistive at all frequencies
- B) The source converter's output impedance must be less than the load converter's input impedance at all frequencies
- C) The source and load converters must have the same crossover frequency
- D) The total impedance of the cascade must be greater than 6dB above 0Ω

**Correct answer: B**

**Explanation:**

When a power converter (load converter) is connected to the output of another converter (source converter), the interaction between the source output impedance Z_s and the load input impedance Z_in can destabilise the system even if each converter is individually stable.

Middlebrook's criterion (minor loop gain criterion) states that the system is stable if:
```
|Z_s(jω)| < |Z_in(jω)|  for all ω
```

Or equivalently, the ratio Z_s/Z_in (the "minor loop gain") must have gain less than 0dB at all frequencies.

Physical intuition: If the source has high output impedance and the load has low input impedance (because switching converters draw constant power, presenting a negative incremental impedance), the source cannot maintain its voltage. This creates an instability driven by the negative impedance interaction.

Practical implication: The source converter should have low output impedance (wide bandwidth, good regulation), and the load converter should have high input impedance (good input filter, or sufficient input capacitance). EMI filters can actually violate this criterion if they create a high-impedance resonance that exceeds the load input impedance.

**Why A is wrong:** Load converter input impedance does not need to be resistive. All switching converters present dynamic impedances that vary with frequency. The criterion is about the relative magnitude at each frequency, not the phase (though phase does matter for a more precise stability analysis).

**Why C is wrong:** Matching crossover frequencies is not required and may actually cause problems. Each converter should be designed with adequate stability margins independently.

**Why D is wrong:** This is not a standard criterion. The Middlebrook criterion compares impedances to each other, not to an absolute 0Ω reference.

---

## Q8

In digital control of a power converter, what is the "computational delay" and how does it affect stability?

- A) The time for the ADC to complete a conversion, which reduces the gain at DC
- B) The one-sample delay between measuring a quantity and applying the corrective duty cycle update, which adds phase lag at high frequencies
- C) The quantisation error in the DPWM, which causes limit cycling
- D) The startup delay before the digital controller begins switching

**Correct answer: B**

**Explanation:**

In a digitally controlled converter, the control algorithm runs in a microcontroller or FPGA. The sequence each switching period is:

1. ADC converts Vout to a digital value (ADC conversion time)
2. Digital PID or compensator runs (computation time: typically 1 clock cycle or more)
3. Duty cycle register is updated in the DPWM
4. New duty cycle takes effect at the next PWM carrier event

The total delay from when the output voltage is sampled to when the compensating duty cycle is applied is typically one full switching period (Tsw) or more. This is a pure time delay:

```
Hdelay(s) = e^(-s×Td)    where Td ≈ 1.5 × Tsw typically
```

In the frequency domain, a pure delay adds phase lag:
```
∠Hdelay(jω) = -ω × Td
```

At the crossover frequency fc:
```
Phase lag = -360° × fc × Td = -360° × fc × 1.5/fsw
```

For fc = fsw/10 (a common rule of thumb): Phase lag = -360° × 0.1 × 1.5 = -54°

This 54° of phase lag directly reduces the phase margin. Combined with the plant phase and compensator phase, this computational delay can consume most of the available phase margin, which is why digital converters are often limited to lower bandwidth than analog designs at the same switching frequency.

**Mitigation:** Predict-ahead algorithms, faster processors, pipelined ADC and computation to reduce Td.

**Why A is wrong:** ADC conversion time does contribute to the total delay, but it affects phase (lag) not DC gain. DC gain is unaffected by a time delay (|e^(-jωT)| = 1 for all ω).

**Why C is wrong:** DPWM quantisation causing limit cycling is a separate issue related to the resolution of the duty cycle register, not to computational delay. Limit cycling occurs when the minimum duty cycle step causes an output voltage oscillation that exceeds the ADC's least significant bit.

**Why D is wrong:** Startup delay is a one-time event during power-on and has no effect on steady-state loop stability.

---

## Q9

A Type III compensator is needed for a voltage-mode buck converter with a low-ESR ceramic output capacitor. Why is Type III needed instead of Type II?

- A) Type II cannot achieve sufficient DC gain for zero steady-state error
- B) Ceramic capacitors cause the ESR zero to disappear from the useful frequency range, so the compensator must supply two zeros to provide adequate phase boost at the LC resonance
- C) Type III has higher gain at the crossover frequency, allowing faster transient response
- D) Type II is only suitable for current-mode control; Type III is required for voltage-mode

**Correct answer: B**

**Explanation:**

For a voltage-mode buck converter, the plant transfer function has:
- A double pole at the LC resonance (ω0 = 1/√(LC))
- An ESR zero at ω_ESR = 1/(ESR × C)
- A flat gain above ω_ESR

With electrolytic capacitors (high ESR): f_ESR = 1/(2π × 50mΩ × 470µF) ≈ 6.8kHz. The ESR zero occurs within the control bandwidth and provides significant phase boost there. A Type II compensator (one zero, two poles including the integrator) may be sufficient.

With ceramic capacitors (low ESR: 2mΩ): f_ESR = 1/(2π × 2mΩ × 22µF) ≈ 3.6MHz. This is far above any reasonable crossover frequency (typically < fsw/5). The ESR zero provides NO phase help in the control bandwidth.

Without the ESR zero, the plant phase at the crossover frequency is dominated by:
- The LC double pole: -180° of phase lag approaching ω0, recovering toward -90° well above ω0
- No phase recovery from ESR zero within the bandwidth

A Type II compensator provides only one zero (one phase boost hump). This may not be enough to recover the loop phase from approximately -180° above the LC resonance to -135° (a PM of 45°) at the desired crossover frequency above ω0.

A Type III compensator provides two zeros, placed to straddle the LC resonance, providing more phase boost and enabling higher crossover frequencies with ceramic capacitor designs.

**Why A is wrong:** Both Type II and Type III have an integrator (pole at origin), providing infinite DC gain and zero steady-state error in steady state. Type III's advantage is phase, not DC gain.

**Why C is wrong:** Type III does not have higher gain at the crossover frequency relative to Type II — both can be designed to cross 0dB at any desired frequency. The gain is set by the component values, not the type. The key advantage is the phase boost from the second zero.

**Why D is wrong:** Both Type II and Type III are used with voltage-mode control. Current-mode control simplifies the outer voltage loop (single pole plant), often allowing a Type II or even Type I compensator. Type III is specifically motivated by the need for more phase boost in voltage-mode designs with low-ESR output capacitors.

---

## Q10

What is anti-windup in a digital PID controller, and why is it necessary?

- A) A method to prevent the integral term from growing without bound when the output is saturated
- B) A method to reduce the sampling rate of the ADC when the error is small
- C) A filter on the derivative term to prevent noise amplification
- D) A technique to prevent the duty cycle from exceeding 100% during startup

**Correct answer: A**

**Explanation:**

The integral term in a PID controller accumulates error over time: integral += Ki × error × dt. In steady state, the integral settles at the value needed to maintain zero error.

During startup or after a large load step, the error may be large for many sampling periods. The integral accumulates a large value. If the controller output is also limited (saturated at max_duty or min_duty), the actuator cannot respond proportionally to the large integral. The integral continues to "wind up" to a very large value.

When the error finally decreases (the output reaches setpoint), the integral is so large that it drives the controller output past saturation in the opposite direction, causing significant overshoot. The system may oscillate for many cycles while the integral slowly winds down.

**Anti-windup implementations:**

1. **Integrator clamping:** Stop accumulating integral when the output is saturated.
   ```c
   if (output < out_max && output > out_min)
       integral += Ki * error;
   ```

2. **Back-calculation:** Feed the saturation error back to discharge the integrator:
   ```c
   integral += Ki * error + (u_sat - u_unsat) / Tt;
   ```
   where Tt is a tracking time constant. When saturated, (u_sat - u_unsat) is nonzero and drives the integral back toward the saturation boundary.

Anti-windup is essential for power converters that have hard limits (duty cycle 0-100%, current limits, voltage limits). Without it, soft-start, fault recovery, and large load steps can cause severe overshoot and oscillation.

**Why B is wrong:** Reducing ADC sampling rate based on error magnitude is not a standard anti-windup technique. It would degrade control performance at small errors and doesn't address integral accumulation.

**Why C is wrong:** Filtering the derivative term (derivative filtering) is a separate technique to prevent noise amplification. The derivative term responds to the rate of change of error — noise causes large spikes. A low-pass filter on the derivative is commonly used: D_term = Kd × (error - prev_error) / (Ts × (1 + s/ωd_filter)). This is unrelated to integral windup.

**Why D is wrong:** Preventing duty cycle from exceeding 100% is output clamping (saturation limiting), which is a separate hardware or firmware feature. While output clamping is part of the system that creates the windup problem, it is not the definition of anti-windup.

---

## Q11

A digital controller uses a 10-bit ADC to measure output voltage. The output voltage range is 0-5V. The controller uses a 10-bit DPWM at 200kHz. What is the minimum output voltage ripple that can be achieved before limit cycling occurs?

- A) About 5mV (one ADC LSB)
- B) About 0.5mV
- C) About 50mV
- D) Cannot be determined without knowing the loop gain

**Correct answer: A**

**Explanation:**

Limit cycling occurs when the controller cannot find a stable duty cycle that eliminates the output voltage error. The condition is:

For a 10-bit ADC over 0-5V range:
```
ADC LSB = 5V / 2^10 = 5V / 1024 ≈ 4.88mV ≈ 5mV
```

For a 10-bit DPWM at 200kHz:
```
DPWM LSB = 1/200kHz / 1024 ≈ 4.88ns ≈ 5ns minimum duty cycle step
```

Minimum duty cycle resolution: 1/1024 ≈ 0.098% → minimum voltage step at output:
```
ΔVout per DPWM step = (Vin × 1/1024) × (Vout/Vin) ≈ Vout / 1024
```

For Vout = 3.3V: ΔVout = 3.3V/1024 ≈ 3.2mV

Limit cycling occurs when the minimum duty cycle step produces an output change larger than one ADC LSB. In this case, the duty cycle alternates between two adjacent values, causing the output to oscillate between two ADC codes. The output ripple = one ADC LSB ≈ 5mV.

**The key condition for NO limit cycling:**
```
ΔVDPWM_step ≤ ADC_LSB
```

If the DPWM resolution is coarser than the ADC (fewer bits), limit cycling is almost certain. If DPWM has more bits than the ADC, the condition can be met.

**Why B is wrong:** 0.5mV is below the ADC resolution — the controller cannot even detect a 0.5mV change. All errors smaller than one ADC LSB (5mV) are invisible to the digital controller.

**Why C is wrong:** 50mV is 10× the ADC LSB. Limit cycling due to ADC quantisation would show up as ripple at the ADC resolution (5mV), not 50mV. 50mV ripple would require 10 LSB amplitude oscillation, which is possible if the DPWM resolution is much coarser but is not the minimum limit cycling condition.

**Why D is wrong:** The minimum limit cycling amplitude is determined by the ADC resolution (for ADC-resolution-limited limit cycling). While loop gain affects the loop dynamics and how quickly limit cycling oscillations grow, the minimum detectable error and minimum correctable duty cycle step determine whether limit cycling can be avoided — and these are set by bit resolutions, not loop gain.

---

## Q12

What is the gain margin of a control loop, and how is it measured from a Bode plot?

- A) The additional gain (in dB) that can be added at the gain crossover frequency before instability
- B) The additional gain (in dB) that can be added at the phase crossover frequency before the gain reaches 0dB
- C) The ratio of the open-loop gain to the closed-loop gain at DC
- D) The frequency range between the gain crossover and the phase crossover frequencies

**Correct answer: B**

**Explanation:**

The gain margin (GM) is defined as the negative of the open-loop gain in dB at the phase crossover frequency ωpc (where the phase = -180°):

```
GM = -|T(jωpc)|_dB  =  -20×log10(|T(jωpc)|)
```

If |T(jωpc)| = 0.1 (-20dB), then GM = +20dB. This means the open-loop gain can be increased by 20dB before the system crosses 0dB at ωpc and becomes unstable.

**How to read from Bode plot:**
1. Find where the phase curve crosses -180° → this is ωpc
2. Read the magnitude at ωpc from the magnitude plot (e.g., -12dB)
3. Gain margin = -(magnitude at ωpc) = +12dB

A positive GM means the system is stable. Minimum recommended: GM > 6dB (most standards), often >10dB for robustness.

**Why A is wrong:** The additional gain at the GAIN crossover frequency relates to phase margin, not gain margin. At the gain crossover (0dB crossing), the relevant metric is how much phase is available before reaching -180° (phase margin).

**Why C is wrong:** The ratio of open-loop to closed-loop gain at DC is related to the loop gain at DC (which affects DC regulation accuracy), not the gain margin. The gain margin is a stability metric evaluated at a specific frequency, not at DC.

**Why D is wrong:** The frequency difference between gain crossover and phase crossover is related to the "stability range" in frequency, but this is not the definition of gain margin. Gain margin is a magnitude measurement in dB, not a frequency difference.

---

## Q13

What happens to the control-to-output transfer function of a boost converter in CCM if the load resistance doubles (load halves)?

- A) The LC resonant frequency increases by √2
- B) The RHP zero frequency halves and moves toward the crossover frequency
- C) The DC gain doubles and the LC double pole location is unchanged
- D) The damping of the LC double pole decreases and the RHP zero moves to higher frequency

**Correct answer: D**

**Explanation:**

For a CCM boost converter, the control-to-output transfer function has:

**DC gain:** Gdc = Vout / (D' × R × il) — depends on operating point

**LC double pole:** ω0 = D' / √(L×C), where D' = (1-D)

The Q-factor of the LC double pole (Erickson and Maksimović, *Fundamentals of Power Electronics*):
```
Q = D' × R × √(C/L)
```

"Load halves" means Iout halves, which means R doubles (for fixed Vout). Then:
- Q doubles, so the double pole is LESS damped (a higher resonant peak in the plant).
- **RHP zero:** ωz_RHP = D'² × R / L doubles, so the RHP zero moves to HIGHER frequency, away from crossover (better for stability).
- ω0 is unchanged (D is fixed by Vin/Vout in CCM), and the ideal DC gain Vout/D' does not depend on R.

Option D states exactly this: damping decreases and the RHP zero moves higher.

**Why A is wrong:** The LC resonant frequency ω0 = D'/√(LC) depends on D' (duty cycle), L, and C — not on load resistance. Changing load at fixed Vout maintains fixed D in steady state, so ω0 is unchanged.

**Why B is wrong:** The RHP zero ωz_RHP = D'² × R / L INCREASES when R increases (load halves). Halving the load moves the RHP zero AWAY from the crossover, not toward it.

**Why C is wrong:** In the ideal CCM model the control-to-output DC gain is Vout/D', independent of R, so it does not double when R doubles. Furthermore, the LC pole location is not independent of load (Q changes with R, affecting the shape of the resonance peak even though ω0 location is unchanged).

---

## Q14

A Type II compensator for a current-mode buck converter is designed with a zero at fz = 500Hz and a pole at fp = 20kHz, targeting 10kHz crossover. After PCB assembly, the crossover is measured at 7kHz instead. What likely changed?

- A) The output capacitor is larger than specified
- B) The feedback resistor divider has an incorrect ratio
- C) The PWM modulator gain is different from the designed value
- D) Any of the above could cause this

**Correct answer: D**

**Explanation:**

The crossover frequency is where the total open-loop gain T(s) crosses 0dB:

```
T(s) = Gcompensator(s) × Gplant(s) × HPWM
```

A lower-than-expected crossover means the total loop gain at 10kHz is less than 0dB. This can be caused by any factor that reduces the gain:

**A: Larger output capacitor.** In current-mode control, the plant is approximately a single-pole system with dominant pole at 1/(2π×R×C). Larger C moves the pole lower → more gain reduction at 10kHz. Also, the single pole reduces gain at -20dB/decade above the pole, so if the pole moves from 1kHz to 500Hz, gain at 10kHz is 6dB lower. This shifts the 0dB crossing lower. Plausible cause.

**B: Incorrect feedback divider.** The feedback divider sets the DC operating point (Vout setpoint). If the divider ratio is wrong, the compensator output at DC changes, which can affect the operating point and indirectly the loop gain. However, a feedback resistor error changes the setpoint, not directly the gain at 10kHz. This could affect Vout, changing D, which changes the plant gain. Indirectly plausible.

More directly: if the sense resistor in the feedback network (the one that sets the compensator input gain) is wrong, it directly scales the loop gain. Plausible.

**C: PWM modulator gain.** HPWM = 1/V_ramp (for voltage mode) or depends on the current slope (for current mode). If the slope compensation level, current sense resistor, or modulator ramp amplitude is different from what was designed, the modulator gain changes, shifting the entire gain curve up or down.

All three are plausible causes of a reduced crossover frequency. Answer D is correct.

**In practice:** When the crossover is wrong, check: (1) measure the actual open-loop gain vs. frequency with a frequency response analyser, (2) identify which stage (compensator, plant, or modulator) has the wrong gain, (3) fix the component values.

---

## Q15

What is the purpose of slope compensation in peak current mode control?

- A) To compensate for the slope of the inductor current ripple and linearise the modulator gain
- B) To prevent subharmonic oscillation when the duty cycle exceeds 0.5
- C) To improve the load transient response speed
- D) To increase the maximum peak current limit

**Correct answer: B**

**Explanation:**

In peak current mode control without slope compensation, the sampled-data control loop has a subharmonic oscillation mode that goes unstable when D > 0.5.

The underlying cause: if a perturbation ΔI0 occurs in the inductor current at the start of a cycle:
- The current ramps up at slope m1 (on-time slope)
- The peak current limit is hit at a slightly different time
- The current ramps down at slope m2 (off-time slope)
- At the end of the cycle, the error is: ΔI1 = -ΔI0 × (m2/m1)

For stability, |m2/m1| < 1, which requires m2 < m1.

For a buck converter: m1 = (Vin - Vout)/L and m2 = Vout/L.
m2/m1 = Vout/(Vin - Vout) = D/(1-D)

At D = 0.5: m2/m1 = 1 (marginal stability, oscillation at fs/2)
At D > 0.5: m2/m1 > 1 (unstable)

Slope compensation adds an artificial ramp of slope mc to the current sense signal. A perturbation then decays by the factor (m2 - mc)/(m1 + mc) each cycle, so the condition becomes |(m2 - mc)/(m1 + mc)| < 1, i.e. mc > (m2 - m1)/2. Choosing mc ≥ m2/2 satisfies this for every duty cycle from 0 to 1 (and mc = m2 gives deadbeat, one-cycle correction).

**Why A is wrong:** Linearising the modulator gain is a separate effect of slope compensation but not its primary purpose. The modulator gain in PCMC does vary with duty cycle without slope compensation, and adding slope compensation makes it more constant — but the primary motivation for slope compensation is stability, not gain linearisation.

**Why C is wrong:** Slope compensation affects the current loop dynamics and can actually slow transient response slightly (by adding damping). The transient response speed is primarily set by the voltage loop bandwidth, not the slope compensation.

**Why D is wrong:** The peak current limit is set by the current sense resistor and comparator threshold. Slope compensation does not change the maximum peak current — it adds a ramp to the sensed current signal, effectively reducing the effective peak current that triggers the comparator in inverse proportion to the slope amount. It makes the limit duty-cycle-dependent, not higher.

---

## Q16

In a control loop Bode plot, what is the significance of the "conditional stability" condition?

- A) The loop is stable only when the load is above a minimum threshold
- B) The loop is stable with normal gain but would become unstable if the gain were reduced
- C) The loop is stable at room temperature but unstable at high temperature
- D) The loop requires an external trigger to start up stably

**Correct answer: B**

**Explanation:**

A conditionally stable loop has an unusual phase Bode plot characteristic: the phase crosses -180° multiple times. A typical stable system has phase going monotonically more negative with frequency, crossing -180° once (if it crosses at all). A conditionally stable system might have a phase curve like:

```
Phase:  0° at DC → -180° at f1 → back up to -90° → down to -180° at f2 → continues negative
```

In this case, the loop is stable (positive phase margin) at the normal crossover frequency, because the phase is above -180° there. However, if the gain is reduced (shifting the magnitude curve down), the new crossover frequency shifts to a lower frequency — potentially landing in the region where phase = -180° at f1. The loop becomes unstable with LOWER gain.

This violates the intuitive notion that "reducing gain makes a system safer." Conditionally stable loops can fail during:
- Soft start (low gain at startup)
- Light load conditions (if gain varies with load)
- Intentional gain reduction for power saving modes
- Component tolerances that reduce gain

**Detection:** On the Bode plot, the phase crosses -180° before the gain crosses 0dB at a lower frequency. The gain margin is measured at the NEAREST phase crossover to 0dB.

**Why A is wrong:** Load-dependent stability is a real phenomenon (especially in boost converters where the RHP zero and Q change with load), but it is not the definition of conditional stability. Conditional stability specifically refers to gain-level-dependent stability.

**Why C is wrong:** Temperature-dependent stability is caused by component parameter drift with temperature (Rds_on, capacitance, etc.). This is a practical concern but is not the definition of conditional stability.

**Why D is wrong:** Startup stability issues can be caused by conditional stability, but the definition of conditional stability is the gain-dependent inversion of the expected gain-stability relationship.

---

## Q17

For a 400kHz switching converter with a digital controller running at 400kHz sample rate, what is the maximum achievable closed-loop bandwidth?

- A) 400kHz
- B) 200kHz
- C) About 20-40kHz (5-10% of switching frequency)
- D) About 100kHz (Nyquist frequency / 2)

**Correct answer: C**

**Explanation:**

The theoretical Nyquist limit for a 400kHz sampling system is 200kHz. However, several effects conspire to limit the practical bandwidth much lower:

1. **Computational delay:** Approximately 1.5 × Tsw = 3.75µs of pure delay, causing phase lag of 360° × fc × 1.5/fsw. At fc = 40kHz, this is 54° of phase lag — consuming most of the available phase margin.

2. **ZOH (zero-order hold) effect:** The DPWM holds the duty cycle constant for one period (DAC-equivalent). This introduces a ZOH transfer function:
   ```
   ZOH(s) = (1 - e^(-sT)) / (sT)
   ```
   which has a phase lag of 90° at the Nyquist frequency and approximately 90° × (fc/fNyquist) at lower frequencies.

3. **Anti-aliasing filter:** Required before the ADC to prevent aliasing. Adds additional phase lag within the bandwidth.

Combined effect: the maximum practical closed-loop bandwidth is typically 1/5 to 1/10 of the switching frequency:

- Analog controller: fc ≤ fsw/5 (limited by PCM subharmonic or aliasing from switching harmonics)
- Digital controller: fc ≤ fsw/10 (additional computational delay and ZOH effects)

For 400kHz switching: practical digital bandwidth ≈ 40kHz.

**Why A is wrong:** 400kHz equals the sampling rate. The Nyquist theorem prohibits meaningful signal representation above half the sampling rate. Even below Nyquist, control bandwidth is far more constrained by delay.

**Why B is wrong:** 200kHz is the Nyquist frequency (half the sample rate). While this is the theoretical maximum for information content, the phase lag accumulated from delays and ZOH effects makes any crossover near 200kHz essentially impossible to stabilise with positive phase margin.

**Why D is wrong:** 100kHz (Nyquist/2 = fsw/4) is more conservative than B but still too high. The computational delay alone at 100kHz crossover would consume: 360° × 100k × 1.5/400k = 135° of phase lag. This leaves no phase margin for the plant poles and is impractical.

---

## Q18

What is the difference between "voltage feedforward" and "feedback" in a switching power supply?

- A) Feedback controls output voltage; feedforward controls output current
- B) Feedforward uses the output voltage to pre-correct for input voltage changes before the feedback loop acts, reducing the time required for the feedback loop to respond
- C) Feedforward controls the duty cycle proportionally to the input voltage change, pre-compensating for line disturbances before they affect the output
- D) B and C describe the same thing; both are correct

**Correct answer: C**

**Explanation:**

C describes voltage feedforward. B describes the right idea (correcting for line changes before the feedback loop has to act) but gets the sensed quantity wrong: feedforward measures the INPUT voltage, not the output voltage. Sensing the output voltage is what feedback does. So B is wrong, and therefore D is wrong too.

**The concept:** When the input voltage changes (e.g., Vin steps up 20%), the output would transiently rise before the feedback loop responds. Voltage feedforward detects the Vin change and pre-adjusts the duty cycle immediately — before the output has changed. The feedback loop then only needs to correct the residual error. This dramatically improves input-to-output transient response (audio susceptibility).

**The implementation:** For a buck converter, Vout = D × Vin. To maintain constant Vout when Vin changes, the duty cycle must adjust inversely: D_new = Vout / Vin_new. If Vin increases 20%, D must decrease by 20%. Voltage feedforward accomplishes this by modifying the PWM ramp height proportional to Vin (so the duty cycle at the same comparator threshold automatically scales inversely with Vin).

**Why feedforward matters:**
- Without feedforward: a 20% Vin step causes a brief output disturbance; the feedback loop corrects it over several switching cycles (time = 1/(2π × fc))
- With feedforward: the duty cycle adjusts within one switching cycle; output disturbance is minimised

In practice, feedforward is implemented by:
- Making the PWM sawtooth amplitude proportional to Vin (analog)
- Dividing the error amp output by a measured Vin value in the duty cycle calculation (digital)

**Why A is wrong:** Feedback controls output voltage; feedforward is also related to output voltage control (not current). Both are voltage control techniques. Output current control (current limiting, current sharing) is a separate function.

---

## Q19

What is the "Nyquist criterion" as applied to power converter control stability?

- A) The open-loop gain must be less than unity at all frequencies above the Nyquist frequency
- B) A closed-loop system is stable if the Nyquist plot of the open-loop transfer function stays inside the unit circle
- C) The switching frequency must be at least twice the closed-loop bandwidth
- D) For stability, the number of counter-clockwise encirclements of (-1, 0) by the open-loop Nyquist plot must equal the number of open-loop poles in the right half-plane

**Correct answer: D**

**Explanation:**

The Nyquist stability criterion (for a unity feedback system with open-loop transfer function T(s)) states:

```
N = Z - P
```

where:
- N = number of clockwise encirclements of the critical point (-1 + j0) in the Nyquist plot of T(jω) as ω goes from -∞ to +∞
- Z = number of closed-loop poles in the right half plane (RHP)
- P = number of open-loop poles in the RHP

For a stable system: Z = 0 (no closed-loop RHP poles).
Therefore for stability: N = -P, i.e., counterclockwise encirclements of (-1,0) equal the number of open-loop RHP poles.

For a minimum-phase open-loop system (P = 0): stability requires N = 0 (no encirclements of -1). This simplifies to the familiar: phase is above -180° at the crossover frequency (phase margin > 0°).

The power of the Nyquist criterion is that it handles non-minimum-phase systems (with open-loop RHP poles or zeros) correctly. Bode plot analysis can give wrong answers for conditionally stable or non-minimum-phase systems.

**Why A is wrong:** This describes a sampling theorem constraint (signal content above Nyquist), not the Nyquist stability criterion. The two "Nyquist" concepts are completely separate.

**Why B is wrong:** Staying inside the unit circle (|T| < 1 everywhere) is sufficient for stability (the small-gain theorem) but is not the Nyquist criterion, and it is far too restrictive: every useful control loop has |T| > 1 at low frequency. The Nyquist criterion counts encirclements of (-1, 0).

**Why C is wrong:** This is the Shannon-Nyquist sampling theorem (sampling frequency > 2 × signal bandwidth). Again unrelated to the Nyquist stability criterion.

---

## Q20

A buck converter's open-loop Bode plot shows the gain crossing 0dB at 15kHz with a phase of -110°. What is the phase margin, and is the design acceptable?

- A) PM = -110°, not acceptable (unstable)
- B) PM = 70°, acceptable
- C) PM = 110°, acceptable
- D) PM = 70°, marginally acceptable (needs review)

**Correct answer: B**

**Explanation:**

Phase margin is defined as:
```
PM = 180° + ∠T(jωc)
```

where ∠T(jωc) is the phase of the open-loop transfer function at the gain crossover frequency.

```
PM = 180° + (-110°) = 70°
```

**Is 70° acceptable?** Yes, 70° is an excellent phase margin. The generally accepted minimum is 45°, and 60° is the target for most power supply designs. 70° provides generous margin against:
- Component tolerances (a capacitor ±20% tolerance shifts the LC pole and can reduce PM by 10-20°)
- Temperature effects (component values drift)
- Load changes (the plant transfer function changes with operating point)

**Typical targets:**
- PM > 45° = minimum acceptable
- PM = 60° = good design target
- PM > 70° = conservative/robust design (may indicate unnecessarily slow bandwidth — check if crossover could be pushed higher for better transient response)

At PM = 70°, the damping ratio of the dominant closed-loop poles is approximately ζ ≈ 0.65, giving a step response with ~8% overshoot and good settling.

**Why A is wrong:** Phase margin uses the formula PM = 180° + phase. The phase of -110° does NOT mean the PM is -110°. PM = 180° + (-110°) = +70°. A phase of -180° exactly would give PM = 0° (marginally stable). Phase of -110° gives positive PM = 70° (stable).

**Why C is wrong:** 110° would be the result of misreading the phase as positive, or confusing the phase angle with PM by a sign error. PM = 180° - 110° = 70°, not 180° - 70° = 110°.

**Why D is wrong:** 70° is not merely "marginally acceptable" — it is well within the acceptable range and represents a robust design. "Marginally acceptable" would describe PM in the 45-55° range where there is limited margin against parameter variations.

---

*End of Quiz — Control Theory*

**Answer Key:** 1-B, 2-B, 3-B, 4-C, 5-B, 6-B, 7-B, 8-B, 9-B, 10-A, 11-A, 12-B, 13-D, 14-D, 15-B, 16-B, 17-C, 18-C, 19-D, 20-B
