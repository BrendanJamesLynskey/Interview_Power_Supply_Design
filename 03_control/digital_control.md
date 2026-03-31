# Digital Control Implementation — Interview Preparation

## Overview

Digital control of switching power supplies replaces the analog error amplifier and compensator with a microcontroller or DSP running a discrete-time algorithm. It offers precise coefficient setting, adaptive behavior, and elimination of component tolerances — but introduces new challenges: sampling delays, quantization, computational latency, and discrete-time stability analysis. This topic is increasingly tested at companies building server power, EV chargers, and solar inverters.

---

## Key Equations Reference

### ADC Resolution Requirement
```
Minimum bits N_ADC so that quantization noise < specified resolution:
  N_ADC ≥ log2(Vout_fullscale / ΔV_required)

Example: 3.3V range, 1mV resolution:
  N_ADC ≥ log2(3.3V / 0.001V) = log2(3300) = 11.7 → 12-bit ADC required
```

### DPWM Resolution Requirement (to avoid limit cycling)
```
N_DPWM ≥ log2(Vout / ΔV_LSB_ADC × Vin)  [approximately]

More precisely: N_DPWM > N_ADC + log2(Vin/Vout)
```

### Bilinear (Tustin) Transform
```
s → (2/Ts) × (z-1)/(z+1)

Frequency prewarping: s = (2/Ts) × tan(ω_desired × Ts/2)  → use prewarped s
```

### Forward Euler (Rectangle) Transform
```
s → (z-1)/Ts   [integrator: 1/s → Ts/(z-1)]
```

### PID in z-domain (Bilinear/Tustin)
```
Analog PID: C(s) = Kp + Ki/s + Kd·s

Discretized (Tustin):
C(z) = Kp + Ki·Ts/2·(z+1)/(z-1) + Kd·2/Ts·(z-1)/(z+1)
```

### Difference Equation (Direct Form I PID)
```
u[n] = u[n-1] + a0·e[n] + a1·e[n-1] + a2·e[n-2]

where:
  a0 = Kp + Ki·Ts/2 + Kd/Ts
  a1 = -Kp + Ki·Ts/2 - 2·Kd/Ts
  a2 = Kd/Ts
  e[n] = error at sample n
  u[n] = control output at sample n
```

### Computational Delay Phase Lag
```
φ_delay = -360° × fc × Td   [degrees]

For one-cycle delay (Td = Ts = 1/fs) at fc = fs/10:
  φ_delay = -36°
```

### Limit Cycle Condition
```
Limit cycling occurs when:
  DPWM resolution < ADC resolution × Vin/Vout

If DPWM has fewer effective bits than ADC (referred to input), the controller
continuously oscillates between two duty cycle values.
```

---

## Fundamentals (Questions 1–6)

---

### Q1. Why does digital control introduce additional phase lag, and how does it limit achievable bandwidth?

**Answer:**

Digital control introduces two unavoidable sources of phase lag that analog control does not have:

**1. Sampling delay (Zero-Order Hold effect):**

The ADC samples the output voltage at discrete instants. Between samples, the controller does not respond to changes — it holds its previous output. This zero-order hold introduces a delay of approximately Ts/2 (half a sample period):

```
ZOH transfer function: GZOH(s) = (1 - e^(-sTs))/(sTs) ≈ e^(-sTs/2)
Phase lag at frequency f: φ_ZOH ≈ -360° × f × (Ts/2)   [degrees]
```

**2. Computational delay:**

After sampling, the DSP/MCU requires time to:
- Complete the ADC conversion
- Read the ADC result
- Execute the PID/compensator algorithm
- Load the new duty cycle into the DPWM register

This computation takes at minimum one switching period (the new duty cycle is applied in the next cycle). Additional pipeline stages can increase this.

Total delay: Td ≈ 1.5 × Ts (worst case: 0.5Ts from ZOH + 1Ts from computation).

**Combined phase lag at crossover fc:**
```
φ_total = -360° × fc × Td = -360° × fc × 1.5/fs

If fc = fs/10:  φ_total = -360° × 0.1 × 1.5 = -54°
If fc = fs/5:   φ_total = -360° × 0.2 × 1.5 = -108°
```

This means a digital controller targeting fc = fs/5 must accept 108° of phase lag from delays alone, leaving very little budget for plant phase lag.

**Practical crossover limit:**
To maintain PM ≥ 45°, the delay-limited maximum crossover is approximately:
```
fc_max ≈ (PM_budget) / (360° × Td) = 135° / (360° × 1.5Ts) = 0.25/Ts = fs/4
```
But plant phase lag typically consumes another 60–90°, making fc_max ≈ fs/10 to fs/15 a realistic constraint.

---

### Q2. What ADC resolution is required for a 5V output with 1mV load regulation requirement?

**Answer:**

**Derivation of minimum ADC bits:**

The ADC must resolve voltage changes smaller than the required regulation accuracy. One LSB (least significant bit) of the ADC represents:
```
V_LSB = V_fullscale / 2^N
```

Requirement: V_LSB ≤ 1 mV
```
V_fullscale / 2^N ≤ 0.001V

If the ADC input range spans 0 to 5V (full-scale = 5V):
2^N ≥ 5V / 0.001V = 5000
N ≥ log2(5000) = 12.3 → need 13-bit ADC
```

**Practical considerations:**

1. **Effective number of bits (ENOB):** ADC noise and nonlinearity reduce effective resolution. A 12-bit ADC may only have 10.5 ENOB. Use ENOB, not nominal bits, in resolution calculations.

2. **Noise averaging:** If the switching ripple on the ADC input is large, the ADC alternately reads above and below the true voltage. Averaging N samples increases effective resolution by log2(N) bits (requires uncorrelated noise):
   ```
   4× oversampling → +1 bit effective resolution
   16× oversampling → +2 bits
   ```
   This is useful for slow-moving feedback variables.

3. **Anti-aliasing filter:** Place an RC low-pass before the ADC with cutoff at fs/2 = Nyquist frequency. This prevents switching noise aliasing into the measured feedback signal. The RC also limits the ADC input bandwidth — the cutoff must be above the required control bandwidth.

4. **Reference accuracy:** ADC accuracy is bounded by the reference voltage accuracy. If VREF has 0.1% tolerance (50 mV on a 5V reference), that directly limits voltage regulation to ≥ 50 mV regardless of ADC resolution.

**Typical designs:**
- 12-bit ADC: adequate for most power supply control (4096 codes over 5V = 1.22 mV/LSB)
- 10-bit ADC: sufficient for loose regulation (4.88 mV/LSB at 5V)
- 16-bit ADC: used for precision measurements and calibration

---

### Q3. What is DPWM resolution and why can insufficient resolution cause limit cycling?

**Answer:**

**DPWM resolution:**

The digital PWM (DPWM) generates the switching duty cycle from a digital count. If the DPWM has N_dpwm bits, the duty cycle can take 2^N_dpwm discrete values:
```
ΔD = 1 / 2^N_dpwm   [smallest duty cycle step]
ΔVout = ΔD × Vin    [resulting output voltage step for a buck, open loop]
```

**Limit cycling mechanism:**

Suppose the ADC has 1 mV resolution but the DPWM can only change Vout by 2 mV per step (at a given Vin). The controller is in a dilemma:
- Current duty cycle: Vout = 3.299V (ADC reads -1 mV error)
- Next duty cycle step up: Vout = 3.301V (ADC reads +1 mV error)
- No duty cycle value produces exactly 3.300V

The controller oscillates between the two adjacent duty cycle values: D_n and D_n+1, producing a limit cycle at the switching frequency (or a subharmonic). The output voltage dithers between 3.299V and 3.301V continuously.

**Quantitative limit cycle condition:**
```
Limit cycling occurs if:
  ΔVout per DPWM step > ΔVout per ADC step (referred to same domain)

  (Vin / 2^N_dpwm) > (Vout / 2^N_adc)   [approximately]

  → 2^N_dpwm < 2^N_adc × Vin/Vout
  → N_dpwm < N_adc + log2(Vin/Vout)
```

**Numerical example:**
```
Buck: Vin=12V, Vout=3.3V, N_adc=10 bits
Required N_dpwm > 10 + log2(12/3.3) = 10 + 1.86 = 11.86 → need 12-bit DPWM
```

**Solutions when N_dpwm is insufficient:**
1. Use a higher-resolution DPWM (increase counter frequency: N = log2(fsw_counter/fsw))
2. Dithering: alternate between two adjacent DPWM codes to achieve average intermediate value
3. Sigma-delta DPWM: use a sigma-delta modulator to achieve high effective resolution
4. Reduce Vin/Vout ratio through topology choice

---

### Q4. Explain the z-domain and how it relates to the s-domain for control design.

**Answer:**

**The z-domain:**

The z-transform is to discrete-time systems what the Laplace transform is to continuous-time. For a sequence sampled at rate Ts:
```
Z{x[n]} = X(z) = Σ x[n] × z^(-n)    n = 0 to ∞
```

The variable z represents a one-sample advance: z^(-1) = unit delay.

**Relationship to s-domain:**
```
z = e^(s·Ts)

Equivalently: s = (1/Ts) × ln(z)
```

**The unit circle:**
In the z-plane, the stability boundary is the unit circle (|z| = 1). This corresponds to the imaginary axis (s = jω) in the s-plane.
- Poles inside unit circle |z| < 1: stable (decay in time domain)
- Poles on unit circle |z| = 1: marginally stable (sustained oscillation)
- Poles outside unit circle |z| > 1: unstable (growing oscillation)

**The z = -1 point (Nyquist frequency):**
z = -1 corresponds to s = jπ/Ts = j·ωs/2 (half the sampling frequency). A pole at z = -1 oscillates at exactly half the sampling rate — the fastest oscillation representable in the discrete system.

**Discretization of a compensator:**

Given an analog compensator C(s), convert to C(z) using a transform method:

1. **Bilinear (Tustin):** `s = (2/Ts)·(z-1)/(z+1)`
   - Best frequency response match; maps jω axis to unit circle bijectively
   - Requires frequency prewarping to match a specific analog frequency exactly

2. **Forward Euler:** `s = (z-1)/Ts`
   - Simple; unstable analog poles can map to stable z-poles
   - Accuracy decreases as fsw/fs ratio increases

3. **Backward Euler:** `s = (z-1)/(z·Ts)`
   - Always stable if analog design is stable; slightly conservative

4. **Impulse invariance:** preserves impulse response exactly; less common for power supply control

**Practical design flow:**
1. Design analog compensator C(s) using Bode plot methods
2. Choose sampling rate fs ≥ 10× the crossover frequency
3. Discretize using bilinear transform with prewarping at fc
4. Verify z-domain stability (all poles inside unit circle)
5. Implement as difference equation in firmware

---

### Q5. Describe the anti-windup problem and three methods to prevent it.

**Answer:**

**The windup problem:**

An integrator in a PID controller accumulates the error signal continuously. When the converter output is far from setpoint (e.g., during startup, current limit, or a large load step), the error is large for an extended period. The integrator output grows very large (winds up) in an attempt to correct the error.

When the error finally reaches zero (output reaches setpoint), the integrator has accumulated a large value that continues to push the output past the setpoint (overshoot). It then takes additional time for the integrator to "unwind" through the opposite error. The result is large overshoot and slow settling.

**In power supplies specifically:**
During a large load step (e.g., 0A to full load), the output voltage dips. The integrator rapidly accumulates. When the output recovers, the integrator's large positive value drives the duty cycle higher than needed, causing Vout to overshoot above the setpoint.

**Method 1 — Clamping (output limiting):**
Limit the integrator output (or the total PID output) to the range [D_min, D_max]:
```c
integral += Ki * error * Ts;
integral = clamp(integral, OUT_MIN, OUT_MAX);
u = Kp * error + integral + Kd * (error - prev_error) / Ts;
u = clamp(u, OUT_MIN, OUT_MAX);
```
Simple to implement. Drawback: the integrator still winds up until the output reaches the limit — the clamping happens after windup occurs. The integrator state and output state diverge.

**Method 2 — Conditional integration (back-calculation):**
Stop integrating when the output is saturated:
```c
u_unsat = Kp * error + integral;          // compute unsaturated output
u_sat = clamp(u_unsat, OUT_MIN, OUT_MAX); // apply saturation
// Anti-windup: back-calculate correction
integral += Ki * error * Ts + (u_sat - u_unsat) / Tt;  // Tt = tracking time constant
```
The term `(u_sat - u_unsat)/Tt` drives the integrator back when the output saturates. Tt is tuned — typically Tt ≈ √(Ti × Td) where Ti, Td are the integral/derivative time constants. This is the most effective method.

**Method 3 — Integrator hold:**
Freeze integration when the output hits a limit:
```c
if (u > OUT_MAX || u < OUT_MIN) {
    // Don't update integral
} else {
    integral += Ki * error * Ts;
}
```
Simple and robust. May cause a step in the integrator when coming out of saturation (the suddenly resumed integration can cause a step change in output). Add filtering on the integral resumption for smooth behavior.

**Common mistake:** Forgetting anti-windup in current-limit mode. If the current limit clamps the duty cycle during a heavy load step, the voltage loop integrator winds up. When current limit clears, the voltage loop over-drives the converter until the integrator unwinds — causing secondary overshoot or instability.

---

### Q6. What is the bilinear (Tustin) transform and why is it preferred over forward Euler?

**Answer:**

**Bilinear transform definition:**
```
s ← (2/Ts) × (z - 1) / (z + 1)
```

This maps the entire left-half s-plane (stable region) to the interior of the unit circle in the z-plane — guaranteeing that a stable analog compensator becomes a stable digital one.

**Forward Euler transform:**
```
s ← (z - 1) / Ts
```

This maps the left-half s-plane to a circle of radius 1/2 centered at z = 1/2 in the z-plane. The mapping is accurate near z = 1 (DC, low frequencies) but distorts at higher frequencies. Some stable analog poles can map to outside the unit circle (unstable digital poles) if the sampling rate is too low.

**Comparison:**

| Property                | Bilinear (Tustin)              | Forward Euler                  |
|-------------------------|--------------------------------|--------------------------------|
| Stability preservation  | Always (if analog is stable)   | Only for adequate fs           |
| DC gain match           | Exact                          | Exact                          |
| HF frequency response   | Compressed (warped)            | Distorted, may be inaccurate   |
| Nyquist boundary        | Maps exactly to unit circle    | Does not map jω to unit circle |
| Phase response          | Good with prewarping           | Can have significant error     |
| Computational cost      | Same as Euler                  | Slightly simpler               |

**Frequency warping:**
The bilinear transform compresses the frequency axis. A continuous-time frequency ω_a maps to digital frequency ω_d:
```
ω_a = (2/Ts) × tan(ω_d × Ts / 2)
```

For low frequencies (ω_d × Ts << 1): ω_a ≈ ω_d (good match)
At Nyquist (ω_d = π/Ts): ω_a → ∞ (all frequencies compressed into finite range)

**Frequency prewarping:**
To ensure the compensator behaves correctly at a specific frequency ω_design (e.g., the crossover frequency):
1. Compute the prewarped analog frequency: `ω_pre = (2/Ts) × tan(ω_design × Ts/2)`
2. Design the analog compensator at ω_pre instead of ω_design
3. Apply bilinear transform → digital compensator has correct behavior at ω_design

**When forward Euler is acceptable:**
Forward Euler is fine for slow integrators in control where fs >> fc by 100:1 or more. It is simpler to implement in resource-constrained firmware. For power supply control where fs/fc ≈ 10:1, bilinear gives significantly more accurate phase response.

---

## Intermediate (Questions 7–12)

---

### Q7. Implement a digital PID controller in C, including anti-windup. Explain each component.

**Answer:**

```c
#include <stdint.h>
#include <float.h>

// PID controller state (persistent between calls)
typedef struct {
    float Kp;           // Proportional gain
    float Ki;           // Integral gain (Ki = Kp * Ts / Ti)
    float Kd;           // Derivative gain (Kd = Kp * Td / Ts)
    float integral;     // Accumulated integral state
    float prev_error;   // Previous error for derivative
    float out_min;      // Output minimum clamp (e.g., 0.0 = 0% duty cycle)
    float out_max;      // Output maximum clamp (e.g., 1.0 = 100% duty cycle)
    float Tt;           // Anti-windup tracking time constant
} PID_t;

// Initialize PID with given gains and limits
void PID_init(PID_t *pid, float Kp, float Ki, float Kd,
              float out_min, float out_max) {
    pid->Kp        = Kp;
    pid->Ki        = Ki;
    pid->Kd        = Kd;
    pid->integral  = 0.0f;
    pid->prev_error = 0.0f;
    pid->out_min   = out_min;
    pid->out_max   = out_max;
    // Tracking time constant: sqrt(Ti * Td) heuristic, or set manually
    pid->Tt        = (Ki > 0.0f) ? (Kp / Ki) : FLT_MAX;
}

// Compute PID output for current sample
// setpoint: desired output voltage
// measurement: ADC reading of actual output voltage
// Returns: duty cycle command [out_min, out_max]
float PID_update(PID_t *pid, float setpoint, float measurement) {
    // 1. Compute error
    float error = setpoint - measurement;

    // 2. Proportional term
    float p_term = pid->Kp * error;

    // 3. Integral term (accumulated, not yet applied to output)
    //    Anti-windup back-calculation is applied after computing unsaturated output
    float i_term = pid->integral;

    // 4. Derivative term (on measurement, not error — avoids derivative kick on setpoint step)
    float d_term = pid->Kd * (pid->prev_error - error); // negative sign: d/dt error, backwards difference
    pid->prev_error = error;

    // 5. Sum unsaturated output
    float u_unsat = p_term + i_term + d_term;

    // 6. Saturate output to limits
    float u_sat = u_unsat;
    if (u_sat > pid->out_max) u_sat = pid->out_max;
    if (u_sat < pid->out_min) u_sat = pid->out_min;

    // 7. Update integral with anti-windup back-calculation
    //    The correction term (u_sat - u_unsat) / Tt is nonzero only when saturated
    pid->integral += pid->Ki * error + (u_sat - u_unsat) / pid->Tt;

    // Clamp integral itself to prevent slow windup beyond limits
    if (pid->integral > pid->out_max) pid->integral = pid->out_max;
    if (pid->integral < pid->out_min) pid->integral = pid->out_min;

    return u_sat;
}

// Example usage in a switching converter ISR (called every switching period)
// Assume 200kHz switching, 12-bit ADC, 3.3V reference
#define VREF_SETPOINT   (2048)   // 12-bit ADC count for 1.65V (example)
#define FS_HZ           (200000.0f)
#define TS              (1.0f / FS_HZ)  // 5µs

// Tuned PID gains (would be calculated from plant model or empirically)
static PID_t voltage_pid;

void converter_init(void) {
    // Example gains for a current-mode buck: single pole plant
    // Kp = 2.0, Ti = 50µs, Td = 0 (PI controller)
    float Kp = 2.0f;
    float Ti = 50e-6f;     // Integration time constant
    float Ki = Kp * TS / Ti;  // = 2.0 * 5e-6 / 50e-6 = 0.2
    float Kd = 0.0f;
    PID_init(&voltage_pid, Kp, Ki, Kd, 0.05f, 0.95f); // 5% to 95% duty
}

void switching_ISR(void) {
    // Read ADC (12-bit, 0-4095 counts)
    uint16_t adc_raw = read_adc();

    // Convert to normalized voltage (0.0 to 1.0)
    float v_meas = (float)adc_raw / 4095.0f;
    float v_set  = (float)VREF_SETPOINT / 4095.0f;

    // Run PID
    float duty = PID_update(&voltage_pid, v_set, v_meas);

    // Load duty cycle into DPWM register
    set_dpwm_duty(duty);
}
```

**Key design decisions in the code:**

1. **Derivative on measurement (not error):** `d_term = Kd × (prev_error - error)` uses the change in error. Some implementations differentiate measurement directly: `d_term = -Kd × (measurement - prev_measurement)`. This prevents the "derivative kick" that occurs when the setpoint steps (error discontinuity) — differentiating the setpoint step generates a large spurious derivative term.

2. **Anti-windup back-calculation:** The line `integral += Ki * error + (u_sat - u_unsat) / Tt` is the back-calculation. When unsaturated: u_sat = u_unsat, the correction is zero. When saturated: the correction drives the integral toward a value that would produce the saturated output without the integrator contributing excess.

3. **Integral clamping:** Even with back-calculation, clamp the integral to prevent excursion beyond physical limits.

---

### Q8. How do you calculate PID coefficients from a plant model, and what is the design procedure?

**Answer:**

**Step 1: Determine the plant transfer function**

For a current-mode controlled buck converter:
```
Gplant(s) = Vout / (1 + s × R_load × C)   [first-order approximation]

Single pole at: fp = 1/(2π × R_load × C)
```
Example: R_load = 5Ω, C = 100µF → fp = 318 Hz

**Step 2: Choose crossover frequency**
```
fc = min(fs/10, fsw/10) = 200kHz/10 = 20 kHz   [digital controller limit]
```

**Step 3: Design analog compensator**

For a first-order plant and PI controller (Type II with no derivative):
```
C(s) = Kp × (1 + s/ωz) / (s/ωz)   [PI form]

Place zero below fc: fz = fc/3 = 6.7 kHz
Adjust Kp so |T(j2π×fc)| = 0 dB
```

Computing required compensator gain at fc:
```
|Gplant(j2π×20kHz)| = 1/√(1 + (20k/318)²) ≈ -35.9 dB (318/20000 ≈ 0.016)
Required |Gcomp(j2π×20kHz)| = +35.9 dB to achieve 0 dB loop gain at fc
```

**Step 4: Convert to discrete-time (bilinear)**

For a PI controller C(s) = Kp + Ki/s, bilinear discretization:
```
C(z) = Kp + Ki × Ts/2 × (z+1)/(z-1)

Difference equation:
u[n] = u[n-1] + (Kp + Ki×Ts/2) × e[n] + (-Kp + Ki×Ts/2) × e[n-1]

Identifying coefficients:
  a0 = Kp + Ki×Ts/2
  a1 = -Kp + Ki×Ts/2
u[n] = u[n-1] + a0×e[n] + a1×e[n-1]
```

**Numerical example:**
```
Ts = 5µs (200 kHz), Kp = 2.0, Ki = 2 × 2π × 6700 = 84,000 rad/s

Bilinear:
  a0 = 2.0 + 84000 × 5µs/2 = 2.0 + 0.21 = 2.21
  a1 = -2.0 + 84000 × 5µs/2 = -2.0 + 0.21 = -1.79
```

**Step 5: Verify in simulation**
Run a z-domain simulation (MATLAB/Simulink, Python with scipy.signal, or custom code). Check:
- Closed-loop poles inside unit circle
- Step response settling time and overshoot
- Frequency response (Bode plot of C(z) × Gplant(z))

---

### Q9. What is fixed-point arithmetic and what overflow and precision risks does it introduce in digital PID?

**Answer:**

**Fixed-point arithmetic:**

In fixed-point, numbers are stored as integers with an implied binary decimal point. For example, a Q15 format stores numbers in [-1, 1) using a 16-bit integer with 15 fractional bits: value = integer / 2^15.

**Why use fixed-point:**
- Microcontrollers without FPU (most Cortex-M0, M0+, some M3) cannot do floating-point in hardware
- Fixed-point runs at integer speed (multiply: 1 clock, vs 10–50 clocks for software float)
- Deterministic timing — FPU operations on Cortex-M4F take 1–14 cycles; integer is always 1

**Risks:**

**1. Overflow:**
Multiplying two 16-bit numbers gives a 32-bit result. If this 32-bit result is truncated back to 16 bits without proper scaling, the top bits are lost — catastrophic overflow. The output wraps around and produces a completely wrong duty cycle.

Prevention:
```c
// WRONG: potential overflow
int16_t kp = 0x4000;  // Q15 = 0.5
int16_t err = 0x6000; // Q15 = 0.75
int16_t result = (kp * err) >> 15;  // Intermediate: 0x18000000 — truncated wrong!

// CORRECT: use 32-bit intermediate
int32_t result32 = ((int32_t)kp * err) >> 15;
int16_t result = (int16_t)result32;  // Now correct
```

**2. Coefficient quantization:**
Compensator coefficients (a0, a1, a2 in the difference equation) are computed from floating-point design and then quantized to fixed-point. If the quantization step is large relative to the coefficient value, the digital compensator behaves differently than designed.

Example: a1 = -1.79 in Q15 = -1.79 × 32768 = -58,654 counts. Representable as 16-bit signed integer — fine for Q15.
But if a1 = -0.00123: in Q15 = -0.00123 × 32768 = -40.3 → rounds to -40 → actual value -40/32768 = -0.00122. Error is 0.8% — usually acceptable.
If a coefficient is very small or very close to another coefficient, precision loss can destabilize the controller.

**3. Integral accumulation (windup/precision):**
The integral state accumulates additions each sample. If the integral resolution is too coarse (stored in Q15), tiny per-sample increments round to zero and integration never occurs. Store the integral in a wider accumulator (32-bit or 64-bit).

**4. Limit cycle due to fixed-point quantization:**
Even without DPWM resolution issues, fixed-point quantization of the error signal can cause limit cycling if the quantization step maps to a non-zero duty cycle change that keeps cycling through two values.

**Best practices:**
- Use 32-bit or 64-bit accumulators for integral states
- Keep track of the Q-format throughout all operations
- Saturate explicitly before narrowing (don't truncate)
- Test with known inputs and verify outputs match floating-point reference

---

### Q10. How does digital control handle soft start?

**Answer:**

**Purpose of soft start:**
Limit inrush current during converter startup. As the reference rises from 0 to nominal, the duty cycle increases gradually — preventing the inductor from being driven to saturation and the input capacitors from seeing high surge current.

**Analog soft start:**
A capacitor on the soft-start pin charges from a current source, creating a ramp. The error amplifier reference is the lower of the soft-start ramp and the setpoint.

**Digital soft start implementation:**

```c
// Digital soft start: ramp the setpoint over time
static float ss_target;    // Ramping setpoint
static float final_set;    // Desired final setpoint

#define SS_RATE  (0.001f / 200000.0f)  // 1mV per sample = 5V in 5ms at 200kHz

void soft_start_step(void) {
    if (ss_target < final_set) {
        ss_target += SS_RATE;
        if (ss_target > final_set) ss_target = final_set;
    }
}

void switching_ISR(void) {
    soft_start_step();
    float v_meas = read_vout_normalized();
    float duty = PID_update(&pid, ss_target, v_meas);
    set_dpwm(duty);
}
```

**Digital advantages over analog:**
1. **Programmable rate:** Change the soft start time in firmware without hardware changes
2. **Multi-rail sequencing:** Precisely time multiple output rails relative to each other by programming different start delays in software
3. **Retry on fault:** After an OCP or OVP event, reset the soft start and ramp again — clean re-start
4. **Pre-biased start:** If the output has a pre-existing voltage (from another source), detect it and start the ramp from that voltage rather than from 0 — prevents reverse inductor current

**Anti-windup during soft start:**
During soft start, the output is below setpoint and rising. The integral accumulates normally. If the ramp rate is slower than what the inductor could support, the integral winds up. When soft start completes, the integral has overshot — Vout overshoots above the final setpoint.

Fix: During soft start, disable or reduce integral gain. Resume full integral only when |error| < threshold (near setpoint).

---

### Q11. Explain computational delay compensation techniques for digital controllers.

**Answer:**

The computational delay is the time between sampling the ADC output and applying the new duty cycle. It degrades phase margin and limits achievable bandwidth.

**Sources of delay:**
1. ADC conversion time: 0.5–5µs depending on resolution and architecture
2. ADC result read and scaling: 0.1–0.5µs
3. PID computation: 0.1–2µs depending on algorithm and clock speed
4. DPWM register update: must wait for end of current period (up to 1 full Ts)

Total delay: typically 1–2 switching periods.

**Technique 1 — Double update rate (interleaved sampling):**
Sample ADC at 2× the switching frequency (at both the beginning and mid-point of each cycle). Apply the control update immediately after computation, halving the effective delay from Ts to Ts/2. Requires dedicated hardware timer or DMA triggering.

**Technique 2 — Prediction (State estimation):**
Instead of using the sampled value directly, predict what the output will be at the next sample instant using a model:
```
v̂_out[n+1] = v_out[n] + (dv/dt)|measured × Ts
```
The controller acts on the predicted future state, effectively cancelling the delay. This is a simple form of Smith predictor.

**Technique 3 — Smith predictor:**
A formal delay-compensation scheme. An internal model of the plant (without delay) runs in parallel with the actual plant. The controller acts on the model output, and the difference between actual and predicted output corrects the model:
```
u[n] → plant_model → predicted_output
error = setpoint - (actual_output - predicted_output + predicted_output)
```
The Smith predictor moves the delay outside the main control loop, allowing much higher bandwidth. More complex to implement and requires accurate plant model.

**Technique 4 — Reduce computational delay:**
- Sample ADC at the beginning of the period, run computation immediately, update DPWM before the midpoint of the period (applies in the same period, not next)
- Use DMA for ADC transfers (ADC result ready in hardware without CPU overhead)
- Optimize PID code: use pre-computed coefficients, avoid division (multiply by reciprocal), use hardware multiply-accumulate (MAC) instructions

**Quantitative benefit:**
Reducing delay from 1.5Ts to 0.5Ts at fc=fs/10:
```
Phase recovery = 360° × fc × ΔTd = 360° × 0.1 × 1.0 = 36°
```
36° of phase recovery allows fc to be pushed from fs/10 to approximately fs/6, significantly improving transient response.

---

### Q12. How is digital control used to implement adaptive or nonlinear control for improved transient response?

**Answer:**

**Standard linear PID limitation:**
A linear PID with fixed gains is designed for small-signal stability near the operating point. During a large load step (e.g., from 10% to 100% load), the converter operates far from the design point. The fixed PID may:
- Respond too slowly (transient droop exceeds spec)
- Exhibit overshoot (integral windup)

**Approach 1 — Variable-gain (gain scheduled) PID:**
Detect the magnitude of the error signal and increase gains proportionally:
```c
float error = setpoint - measurement;
float scale = 1.0f + K_scale * fabsf(error);  // scale > 1 for large error
pid.Kp_effective = pid.Kp * scale;
// Use effective gains in PID update
```
This is equivalent to nonlinear proportional control: large errors get a boosted response without sacrificing stability at small errors.

**Approach 2 — Hysteretic mode switching:**
Switch between two control modes based on error magnitude:
- Small error (|e| < threshold): use tuned linear PID
- Large error (|e| > threshold): switch to faster mode (e.g., bang-bang, or PID with very high Kp)

The switch must be implemented with hysteresis to prevent chattering at the threshold.

**Approach 3 — Model predictive control (MPC):**
At each sample, predict the output trajectory over a horizon of N steps using the plant model. Choose the input sequence that minimizes a cost function (e.g., quadratic sum of errors and control effort). Apply only the first input, then re-optimize at the next sample.

Benefits: handles constraints naturally (duty cycle limits, current limits), accounts for future behavior. Cost: high computational load — requires matrix operations every sample. Feasible on modern DSPs and MCUs with hardware FPU.

**Approach 4 — V² control (output voltage and its derivative):**
Feed forward the output capacitor current to improve transient response:
```
Duty = D_ff + Kp × e + Ki × ∫e + Kc × C × dVout/dt
```
The derivative of Vout is proportional to the capacitor current, which reflects instantaneous load changes. This enables near-instantaneous response to load steps without relying on the integrator.

**Approach 5 — Digital notch filter for input disturbances:**
If the input has a known interference frequency (e.g., 100 Hz mains ripple), implement a notch filter in the feedback path at that frequency. The loop gain dips at the notch frequency, preventing the controller from chasing the periodic disturbance.

---

## Advanced (Questions 13–17)

---

### Q13. Derive the stability condition for a digital PID from the characteristic equation in the z-domain.

**Answer:**

**Closed-loop characteristic equation:**

For a sampled-data system with digital compensator C(z) and plant (discretized) G(z):
```
Characteristic equation: 1 + C(z) × G(z) = 0

Equivalently: z^n + a_{n-1}·z^{n-1} + ... + a_0 = 0
```

The system is stable if and only if all roots of this polynomial lie strictly inside the unit circle (|z_i| < 1 for all i).

**Jury stability criterion:**

For discrete-time systems, the Jury criterion is the z-domain equivalent of Routh-Hurwitz:

Given characteristic polynomial P(z) = a_n·z^n + a_{n-1}·z^{n-1} + ... + a_0:

Construct the Jury table from the coefficients. Stability conditions:
```
|a_0| < a_n
P(1) > 0
(-1)^n × P(-1) > 0
All Jury table diagonal elements must be positive
```

**Simple PI example:**

For a PI controller C(z) = (a0·z + a1)/(z-1) controlling a first-order plant G(z) = b/(z-p):

Closed-loop: 1 + (a0·z + a1)/((z-1)) × b/(z-p) = 0

```
(z-1)(z-p) + b(a0·z + a1) = 0
z² - (1+p)z + p + b·a0·z + b·a1 = 0
z² + (b·a0 - 1 - p)z + (p + b·a1) = 0
```

Stability requires (Jury conditions for 2nd order):
```
|p + b·a1| < 1          (|a0| < a_n condition, with a_n=1)
1 + (b·a0 - 1 - p) + (p + b·a1) > 0  → b·a0 + b·a1 > 0 → b(a0+a1) > 0
1 - (b·a0 - 1 - p) + (p + b·a1) > 0  → 2 + 2p - b·a0 + b·a1 > 0
```

These inequalities, substituted with the actual plant and compensator values, determine whether the discrete-time controller is stable.

**Practical approach:**
Rather than manually applying Jury, compute the polynomial roots numerically using `numpy.roots()` (Python), `roots()` (MATLAB), or a Durand-Kerner iteration in C. Verify all roots satisfy |z| < 1.

---

### Q14. What is limit cycling in a digital controller and how does it arise from both DPWM resolution and fixed-point arithmetic?

**Answer:**

**Definition:**
A limit cycle is a sustained periodic oscillation in the controller output and (consequently) the power converter output voltage, caused by the finite resolution of digital representations rather than by fundamental instability.

**DPWM-induced limit cycle:**
As derived in Q3: if the smallest duty cycle step ΔD × Vin produces a Vout change larger than one ADC LSB, the controller cannot find a stable equilibrium. It alternates between two adjacent DPWM codes. The output voltage oscillates at the switching frequency (or a subharmonic if the pattern repeats over multiple cycles).

**Fixed-point arithmetic-induced limit cycle:**
In fixed-point PID, the per-sample integral increment is Ki × e[n] × Ts. If Ki is small and Ts is short, this increment may be smaller than 1 LSB in the integral accumulator's fixed-point representation — it rounds to zero. The integrator never advances. Alternatively, it rounds up every other sample, creating a 2-sample oscillation.

**Example:**
```
Ki = 0.1, Ts = 5µs, error = 0.001V
Per-sample increment = 0.1 × 0.001 × 5µs = 5×10^-10 V·s

In Q15 format: 5×10^-10 × 32768 = 1.6×10^-5 counts → rounds to 0
The integrator never moves → steady-state error remains → not a limit cycle, but a DC error.
```

A limit cycle arises when the rounding pattern produces alternating positive and negative increments that cancel only approximately — the accumulator oscillates between two adjacent fixed-point values, each producing a different DPWM output.

**Detection:**
- Measure output voltage with a high-resolution oscilloscope (AC-coupled, high sensitivity)
- Look for periodic oscillation at exactly fsw or fsw/2 with consistent amplitude
- Check if oscillation disappears at a different load (different ADC reading point)

**Elimination:**
1. Increase DPWM resolution (as derived: N_dpwm > N_adc + log2(Vin/Vout))
2. Add dithering to the DPWM (sigma-delta modulation)
3. Use wider accumulator for integral state (32-bit or 64-bit)
4. Increase Ki so per-sample increments are ≥ 1 LSB even at minimum error

---

### Q15. How does a sigma-delta DPWM improve effective resolution and what are its drawbacks?

**Answer:**

**Standard DPWM limitation:**

A DPWM with N bits and switching frequency fsw can only set duty cycle in steps of 1/2^N. With N=12 and fsw=200kHz, the smallest duty step is 1/4096, and the minimum Vout step = Vin/4096. For Vin=12V, ΔVout_min = 2.9mV — too coarse for 1mV resolution requirements.

**Sigma-delta DPWM principle:**

Instead of a binary counter, a sigma-delta modulator generates a PWM pattern that achieves fractional duty cycles by alternating between adjacent integer duty cycles over multiple switching cycles:

```
Desired D = 0.512345 (fractional)
SD output alternates between D=0.512 and D=0.513 in a pattern such that:
  time-average of output = 0.512345
```

The switching pattern is noise-shaped: errors are pushed to high frequencies where they are attenuated by the LC output filter.

**Effective resolution improvement:**
```
With M switching cycles of averaging by the output capacitor:
Effective bits = N_dpwm + log2(M) = N_dpwm + log2(C × R_load × fsw)

For C=100µF, R_load=1Ω, fsw=200kHz:
M = 100µF × 1Ω × 200kHz = 20
Effective additional bits = log2(20) = 4.3
Effective resolution = 12 + 4.3 = 16.3 bits
```

**Implementation:**

```c
// First-order sigma-delta DPWM modulator
// target_d: desired duty in Q16 fixed-point (0 to 65535 = 0% to 100%)
// Returns: integer duty cycle for DPWM counter (0 to MAX_COUNT-1)

static int32_t sd_accumulator = 0;

uint16_t sigma_delta_dpwm(uint16_t target_d, uint16_t max_count) {
    // Accumulate the fractional target
    sd_accumulator += target_d;

    // Output 1 when accumulator overflows
    if (sd_accumulator >= 65536) {
        sd_accumulator -= 65536;
        return max_count; // Full period (100% would be max_count+1, use max_count-1 for safety)
    } else {
        return (uint16_t)((sd_accumulator * max_count) >> 16);  // Proportional
    }
}
```

**Drawbacks of sigma-delta DPWM:**

1. **Increased output ripple:** The alternating duty cycle pattern creates ripple at frequencies below fsw. The output filter must attenuate this additional ripple.

2. **Limit cycling risk:** Poorly designed sigma-delta modulators can produce structured patterns (tones) rather than noise-shaped patterns. These tones appear as spurious frequencies in the output.

3. **Control loop interaction:** The variable duty cycle pattern (from the modulator) interacts with the control loop. The effective open-loop gain is frequency-dependent in a complex way — stability analysis must account for this.

4. **Complexity:** More complex firmware and harder to verify behavior at all conditions.

---

### Q16. Describe a complete digital controller design flow from plant measurement to implemented firmware.

**Answer:**

**Phase 1: Plant characterization**

1. Build the power stage hardware (no control loop yet)
2. Measure the control-to-output transfer function:
   - Apply small-signal perturbation to duty cycle command
   - Measure output voltage response
   - Use swept frequency analysis or step response
3. Fit a transfer function model to measured data:
   - Identify pole and zero frequencies from Bode plot slopes
   - Extract DC gain, Q factor from peak at resonance

**Phase 2: Compensator design (continuous domain)**

4. Choose crossover frequency fc ≤ fs/10 (considering computational delay)
5. Select compensator type: Type II for current-mode, Type III for voltage-mode
6. Place zeros and poles based on plant poles and zeros
7. Set gain to achieve 0 dB loop gain at fc
8. Verify PM ≥ 45°, GM ≥ 10 dB on paper/simulation
9. Simulate closed-loop step response to verify transient specs

**Phase 3: Discretization**

10. Choose sampling rate: typically equal to fsw (sample once per period)
11. Apply bilinear transform with prewarping at fc:
    ```
    ω_prewarp = (2/Ts) × tan(2π × fc × Ts / 2)
    Substitute ω_prewarp for jω in the analog design to get prewarped C(s)
    Then apply: s → (2/Ts)(z-1)/(z+1)
    ```
12. Compute difference equation coefficients (a0, a1, a2...)
13. Check z-domain stability (poles inside unit circle)

**Phase 4: Fixed-point analysis**

14. Determine required Q-format for each coefficient and state variable
15. Verify no overflow for worst-case inputs (maximum duty cycle commands, maximum error)
16. Verify precision of integral accumulation (check minimum step > 0.5 LSB)

**Phase 5: Firmware implementation**

17. Implement difference equation in MCU/DSP interrupt service routine
18. Implement soft start, anti-windup, duty cycle clamping
19. Add protection trips (OCP, OVP, UVLO) that freeze or reset controller

**Phase 6: Verification**

20. Measure digital Bode plot using injection transformer on running hardware
21. Verify PM, GM match design (typically within 5–10° due to modelling error)
22. Perform load step transient tests: verify settling time, peak deviation
23. Test across full input voltage and load range (verify stability at all corners)
24. Thermal testing: verify controller behavior doesn't change at temperature extremes (ADC reference drift, component value shifts)

---

### Q17. What digital control techniques are used in multi-phase interleaved converters?

**Answer:**

**Multi-phase architecture:**
N converters operating at 1/N phase offset, sharing the same output. Each phase has its own controller, but they must be synchronized and current-balanced.

**Synchronization:**
A master timer generates phase-offset PWM clocks for all phases. In digital control:
```
Phase k offset = k × Ts / N   (k = 0, 1, ..., N-1)
```
Each phase controller samples its ADC at its own phase-offset trigger. This interleaves the ADC samples at N× the per-phase switching rate — effectively increasing the aggregate sampling rate.

**Current sharing:**
Each phase measures its inductor current (via DCR sensing or shunt). A current sharing algorithm runs on the master controller:

```c
float avg_current = (I_phase[0] + I_phase[1] + ... + I_phase[N-1]) / N;

for (int k = 0; k < N; k++) {
    float current_error = avg_current - I_phase[k];
    // Adjust each phase's voltage setpoint by a fraction of the current error
    phase_setpoint[k] = main_setpoint + K_share * current_error;
}
```

**Phase shedding (efficiency optimization):**
At light load, shut down some phases. Digital control enables smooth phase addition and removal:
1. Detect when total load current drops below threshold for k phases
2. Ramp down the duty cycle of one phase to zero gradually (avoid current step)
3. Disable the phase
4. Monitor remaining phases for current balance
5. Re-enable phases when load rises again

**Digital implementation benefits over analog:**
- Precise phase offsets (analog approaches use RC timing, which drifts with temperature)
- Programmable number of phases without hardware changes
- Current calibration: correct for sensor gain mismatch in software
- Dynamic phase shedding with programmable thresholds
- Communication with system management controller (PMBus, I²C) for telemetry

**Stability with multiple phases:**
Each phase is a separate control loop. Verify that the closed-loop output impedance of N phases in parallel does not interact with downstream load impedance. The paralleled output impedance is Zout_single / N at low frequencies, rising to Zout_single at frequencies above the current-sharing loop bandwidth.

---

## Quick Reference

| Parameter            | Guideline                                    |
|----------------------|----------------------------------------------|
| ADC bits             | ≥ log2(Vout_range / ΔV_required) + 2 margin |
| DPWM bits            | > ADC bits + log2(Vin/Vout)                  |
| Sampling rate        | ≥ 10 × fc; equals fsw is typical             |
| Max crossover (fc)   | fs/10 (conservative) to fs/5 (aggressive)    |
| Computational delay  | ≈ 1.5 × Ts (one sample + 0.5 ZOH)           |
| Phase loss at fc=fs/10| 360° × 0.1 × 1.5 = 54°                     |
| Discretization method| Bilinear (Tustin) with prewarping            |
| Anti-windup method   | Back-calculation (most effective)            |
