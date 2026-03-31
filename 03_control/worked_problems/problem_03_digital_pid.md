# Worked Problem 03: Digital PID Controller Implementation

## Problem Statement

Implement a digital PID voltage controller for a current-mode buck converter using the bilinear (Tustin) transform. The design must address:

1. Discretization of the analog compensator
2. Coefficient calculation
3. Anti-windup implementation
4. Fixed-point implementation considerations

**Converter specifications:**
```
Vin  = 12 V
Vout = 3.3 V
Iout_max = 5 A
fsw  = 400 kHz    (switching frequency)
fs   = 400 kHz    (sampling frequency, one ADC sample per period)
Ts   = 2.5 µs     (sampling period)
```

**Plant (current-mode, simplified):**
```
Gplant(s) = R_load / (1 + s × R_load × C)

R_load = 3.3/5 = 0.66 Ω  (full load)
C      = 220 µF
Output pole: fp = 1/(2π × 0.66 × 220µF) = 1099 Hz ≈ 1.1 kHz
```

**ADC:**
```
12-bit, range 0 to 3.3V → resolution = 3.3/4096 = 0.806 mV/count
Reference: VREF = 1.65V → ADC count = 2048 at Vout = 3.3V × (1.65/3.3) = 1.65V
(using resistor divider: top = bottom = R → Vfb = Vout/2 = 1.65V at nominal Vout)
```

**DPWM:** 12-bit, 0 to 4095 counts = 0% to 100% duty cycle

**Controller IC:** ARM Cortex-M4 with FPU, 168 MHz clock, hardware 12-bit ADC and timer-based PWM

---

## Step 1: Design the Analog Compensator

**Target crossover frequency:**
```
fc_max = fs/10 = 400kHz/10 = 40 kHz

Choose fc = 30 kHz (conservative, leaves room for delay phase loss)
```

**Phase budget at 30 kHz:**
```
Plant phase at 30 kHz:
  φ_plant = -arctan(30k/1.1k) = -arctan(27.3) = -87.9° ≈ -88°

Computational delay phase (assume 1.5 × Ts):
  τ = 1.5 × 2.5µs = 3.75 µs
  φ_delay = -360° × 30kHz × 3.75µs = -40.5°

Total plant+delay phase at 30 kHz: -88° + (-40.5°) = -128.5°

Required loop phase for PM = 55°: -(180° - 55°) = -125°
Required compensator phase: -125° - (-128.5°) = +3.5°
```

A proportional-integral (PI) controller (Type II without explicit HF pole) provides phase:
```
φ_PI = -90° + arctan(fc/fz)

Setting φ_PI = +3.5°:
-90° + arctan(30k/fz) = 3.5°
arctan(30k/fz) = 93.5° → impossible (arctan ≤ 90°)
```

Since we need the compensator to provide near +90° boost (to overcome the -90° from the integrator), place the zero well below fc:

**Choose fz = 500 Hz (well below fc = 30 kHz):**
```
φ_PI = -90° + arctan(30k/500) = -90° + arctan(60) = -90° + 89.05° = -0.95°

Total loop phase at 30 kHz:
  -128.5° + (-0.95°) = -129.45°
PM = 180° - 129.45° = 50.55° > 45° ✓ (just barely — acceptable for this example)
```

To increase PM to 55°, move fz lower or add a derivative term. For this problem, a PD-I structure (derivative on measurement) improves PM without HF noise amplification.

**Use a PI controller with fz = 500 Hz and verify:**
```
Kp × (1 + ωz/s) with ωz = 2π × 500 = 3142 rad/s
C(s) = Kp × (1 + s/ωz) / (s/ωz) = Kp × (s + ωz) / s  × (ωz/ωz)
     = Kp + Kp/ωz × ωz/s
     = Kp + Ki/s    where Ki = Kp × ωz = Kp × 3142
```

**Set Kp from gain condition at fc = 30 kHz:**
```
|T(j2π×30kHz)| = 1 (= 0 dB)

|C(j2π×30kHz)| × |GPWM| × |Gplant(j2π×30kHz)| × |H| = 1

|Gplant(30kHz)| = R_load / √(1 + (30k/1.1k)²) = 0.66/27.3 = 0.0242

GPWM = 1/Vm: assume Vm = 0.5V → GPWM = 2 V⁻¹
H = 0.5 (feedback divider: Vfb/Vout = 1.65V/3.3V = 0.5)

At 30 kHz, PI compensator gain:
|C(j×2π×30kHz)| = Kp × √(1 + (30k/500)²) / (30k/500)
               = Kp × √(1 + 3600) / 60
               = Kp × 60.008 / 60
               ≈ Kp   (since fz << fc)

Setting up equation:
Kp × 2 × 0.0242 × 0.5 = 1
Kp × 0.0242 = 1
Kp = 1 / 0.0242 = 41.3
```

**Analog PI parameters:**
```
Kp = 41.3
Ki = Kp × ωz = 41.3 × 3142 = 129,765 rad/s  (≈ 1.3 × 10⁵)
```

---

## Step 2: Discretize Using Bilinear Transform

**Bilinear transform with prewarping at fc = 30 kHz:**

Prewarped angular frequency:
```
ω_prewarp = (2/Ts) × tan(ω_c × Ts/2)
          = (2/2.5µs) × tan(2π × 30kHz × 2.5µs/2)
          = 800,000 × tan(π × 0.075)
          = 800,000 × tan(0.2356 rad)
          = 800,000 × 0.2439
          = 195,120 rad/s
```

The prewarped analog zero frequency:
```
ω_z_prewarp = ω_z × (ω_prewarp / ω_c)
            = 3142 × (195,120 / 188,496)
            = 3142 × 1.0351
            = 3252 rad/s
            → fz_prewarp = 517 Hz
```

(Small correction — the prewarping only matters for precision near Nyquist; at 30 kHz with fs=400 kHz, the correction is minor: 3.5%. For this problem, proceed with unprewarped values for clarity.)

**Bilinear substitution:** `s → (2/Ts) × (z-1)/(z+1)`

For the PI controller C(s) = Kp + Ki/s:

```
C(z) = Kp + Ki × [Ts/(2) × (z+1)/(z-1)]   [bilinear: 1/s → Ts/2 × (z+1)/(z-1)]

C(z) = Kp + (Ki × Ts/2) × (z+1)/(z-1)

Multiply through by (z-1):
C(z) × (z-1) = Kp × (z-1) + Ki × Ts/2 × (z+1)
             = (Kp + Ki×Ts/2) × z + (-Kp + Ki×Ts/2)
```

**Difference equation form:**

The output u[n] and error e[n] are related by:

```
u[n] × (z-1) = (a0 × z + a1) × e(z)

In time domain:
u[n] - u[n-1] = a0 × e[n] + a1 × e[n-1]
u[n] = u[n-1] + a0 × e[n] + a1 × e[n-1]
```

**Calculate coefficients:**
```
a0 = Kp + Ki × Ts/2
   = 41.3 + 129,765 × (2.5 × 10⁻⁶)/2
   = 41.3 + 129,765 × 1.25 × 10⁻⁶
   = 41.3 + 0.1622
   = 41.46

a1 = -Kp + Ki × Ts/2
   = -41.3 + 0.1622
   = -41.14
```

**Complete difference equation:**
```
u[n] = u[n-1] + 41.46 × e[n] + (-41.14) × e[n-1]
```

where:
- `e[n]` = error at current sample = (setpoint count - ADC count)
- `u[n]` = duty cycle command (DPWM count 0–4095)
- `u[n-1]` = previous duty cycle command
- `e[n-1]` = previous error

**Verify discrete-time pole:**

The characteristic equation of the difference equation is:
```
z - 1 = 0  →  z = 1
```

This is the integrator pole exactly on the unit circle — correct for a PI controller (pure integral at DC).

The closed-loop stability is verified by finding poles of:
```
1 + C(z) × G(z) = 0
```

Compute G(z) using bilinear transform of Gplant(s):
```
Gplant(s) = R_load / (1 + s × R_load × C) = 0.66 / (1 + s × 0.66 × 220µF)
           = 0.66 / (1 + s × 0.1452 ms)
           = 0.66 × ω_p / (s + ω_p)    where ω_p = 6912 rad/s (= 2π × 1099 Hz)

Bilinear: s → (2/Ts)(z-1)/(z+1) = 800k(z-1)/(z+1)

G(z) = 0.66 × 6912 / (800000(z-1)/(z+1) + 6912)
     = 0.66 × 6912 × (z+1) / (800000(z-1) + 6912(z+1))
     = 4561.9(z+1) / ((800000 + 6912)z + (-800000 + 6912))
     = 4561.9(z+1) / (806912z - 793088)
     = 0.00565(z+1) / (z - 0.9828)
```

This matches the expected discrete first-order system with a pole near z = 1 (at 0.9828, i.e., inside the unit circle ✓).

---

## Step 3: Anti-Windup Implementation

**Why anti-windup is critical here:**

During startup, Vout rises from 0V to 3.3V. During this time, error e[n] is large (3.3V/0.806mV = 4096 ADC counts). The integrator accumulates:

Without anti-windup:
```
After 1ms: u_integrated ≈ a0 × error × samples = 41.46 × 4096 × 400 = 68 million counts
```
This vastly exceeds the DPWM range of 0–4095.

When Vout reaches 3.3V and error drops to 0, the integrator still holds ~68 million. The output duty cycle remains at maximum for a very long time while the integrator unwinds → severe overshoot.

**Implementation with back-calculation anti-windup:**

```c
#include <stdint.h>
#include <stdbool.h>

// Controller parameters (computed from design)
#define KP          (41.46f)      // Proportional gain
#define KI_TS_HALF  (0.1622f)     // Ki * Ts/2 (integral coefficient)
#define A0          (41.46f)      // = Kp + Ki*Ts/2
#define A1          (-41.14f)     // = -Kp + Ki*Ts/2

// DPWM limits
#define DUTY_MIN    (0.0f)        // 0% duty cycle
#define DUTY_MAX    (4095.0f)     // 100% duty cycle (12-bit DPWM)

// Anti-windup tracking time constant: typically Tt = sqrt(Ti * Td) or Kp/Ki
// Ti = Kp/Ki = 41.3/129765 = 318µs = 1/3142 rad/s (= 1/ωz)
// For PI, set Tt = Ti = 318µs → Tt_inv = 1/Ti = 3142/s
// Per sample: Tt_per_sample = ωz * Ts = 3142 × 2.5µs = 0.00786
#define ANTIWINDUP_GAIN  (0.00786f)   // = ωz × Ts = Ts/Ti

// Controller state
static float integral_state = 0.0f;  // Integral accumulator
static float prev_error     = 0.0f;  // e[n-1]

// ADC setpoint (12-bit count for nominal Vout = 3.3V with /2 divider)
// Vfb = Vout/2 = 1.65V → ADC count = 1.65/3.3 * 4096 = 2048
#define SETPOINT_COUNT  (2048u)

void controller_init(void) {
    integral_state = 0.0f;
    prev_error     = 0.0f;
}

// Called from switching frequency ISR (every 2.5µs)
// adc_count: raw 12-bit ADC reading (0 to 4095)
// Returns: DPWM count (0 to 4095)
uint16_t controller_update(uint16_t adc_count) {
    // 1. Calculate error (setpoint - measurement)
    float error = (float)SETPOINT_COUNT - (float)adc_count;

    // 2. Proportional term
    float p_term = KP * error;

    // 3. Integral state (accumulated from previous cycles)
    float i_term = integral_state;

    // 4. Compute unsaturated output
    //    Difference equation: u[n] = u[n-1] + a0*e[n] + a1*e[n-1]
    //    Here restructured as: p + i where i accumulates the running sum
    float u_unsat = p_term + i_term;

    // 5. Saturate output
    float u_sat = u_unsat;
    if (u_sat > DUTY_MAX) u_sat = DUTY_MAX;
    if (u_sat < DUTY_MIN) u_sat = DUTY_MIN;

    // 6. Update integral with difference equation + anti-windup back-calculation
    //    Next integral = current integral + a0*e[n] + a1*e[n-1] + antiwindup
    //    Note: a0*e[n] + a1*e[n-1] represents the bilinear integral increment
    float integral_increment = A0 * error + A1 * prev_error;
    float antiwindup_correction = ANTIWINDUP_GAIN * (u_sat - u_unsat);

    integral_state += integral_increment - p_term + antiwindup_correction;
    // Note: we subtract p_term because i_term = u - p_term (integral is total minus proportional)

    // Clamp integral to prevent accumulator from going wildly out of range
    if (integral_state > DUTY_MAX) integral_state = DUTY_MAX;
    if (integral_state < DUTY_MIN) integral_state = DUTY_MIN;

    // 7. Save error for next cycle
    prev_error = error;

    // 8. Return saturated duty cycle as integer
    return (uint16_t)u_sat;
}
```

**Simplified version (easier to understand the algorithm):**

```c
// Clean PI with anti-windup (equivalent, clearer structure)
static float integral = 0.0f;
static float prev_err = 0.0f;

uint16_t pid_simple(uint16_t adc) {
    float e = (float)SETPOINT_COUNT - (float)adc;

    // Bilinear integral: u[n] = u[n-1] + a0*e[n] + a1*e[n-1]
    // Equivalently: integral += A0*e + A1*prev_e
    float u_new = integral + A0 * e + A1 * prev_err;

    // Saturate
    float u_sat = u_new < DUTY_MIN ? DUTY_MIN : (u_new > DUTY_MAX ? DUTY_MAX : u_new);

    // Anti-windup: if saturated, correct the integral before storing
    // correction = antiwindup_gain * (saturated - unsaturated)
    integral = u_sat - A0 * e;  // Store what u would need to be (without this cycle's p term)
    integral += ANTIWINDUP_GAIN * (u_sat - u_new);  // Back-calculation correction

    prev_err = e;
    return (uint16_t)u_sat;
}
```

---

## Step 4: Fixed-Point Implementation Considerations

**Evaluate if floating-point is feasible:**
```
Cortex-M4 FPU: single-precision float operations in 1–14 cycles
ISR budget: 2.5µs × 168 MHz = 420 clock cycles total ISR budget

PI controller operations:
  1 subtraction (error)     : 1 cycle
  2 multiplications (A0*e, A1*prev_e): 1 cycle each (FPU MAC)
  1 addition                : 1 cycle
  2 comparisons (saturation): 2 cycles
  Anti-windup               : 3-4 cycles
  Overhead (save/restore, ISR entry): ~20 cycles

Total: ~30 cycles out of 420 → 7% CPU load
Floating-point is feasible on Cortex-M4F for this design.
```

**If fixed-point is required (e.g., Cortex-M0 without FPU):**

Choose Q-formats based on value ranges:

| Variable         | Range              | Q-format | Bits Needed |
|------------------|--------------------|----------|-------------|
| error (ADC cnt)  | -4096 to +4096     | Q0       | 13-bit int  |
| A0 coefficient   | 41.46              | Q7 (×128)| int ≈ 5313  |
| A1 coefficient   | -41.14             | Q7       | int ≈ -5267 |
| integral_state   | 0 to 4095          | Q7       | int 0-524160|
| duty cycle output| 0 to 4095          | Q0       | 12-bit int  |

**Fixed-point coefficient representation:**
```c
// Q7 format: value × 128 (7 fractional bits)
#define A0_Q7    ((int32_t)(41.46f * 128))   // = 5307
#define A1_Q7    ((int32_t)(-41.14f * 128))  // = -5266

// Integral state in Q7 (wider accumulator to prevent rounding)
static int32_t integral_q7 = 0;  // 32-bit accumulator, Q7 format

int16_t pid_fixed_point(int16_t adc_count) {
    // Error: Q0 (integer ADC counts)
    int32_t error = (int32_t)SETPOINT_COUNT - adc_count;

    // Multiply: error (Q0) × A0 (Q7) → result in Q7
    // Use 64-bit intermediate to prevent overflow
    int64_t increment = (int64_t)A0_Q7 * error + (int64_t)A1_Q7 * prev_error_q0;

    // Update integral (Q7 accumulator)
    integral_q7 += (int32_t)(increment >> 0);  // Still in Q7

    // Convert to duty cycle (Q0) by right-shifting 7 bits
    int32_t duty_q0 = integral_q7 >> 7;

    // Saturate
    if (duty_q0 > 4095) { duty_q0 = 4095; integral_q7 = 4095 << 7; }
    if (duty_q0 < 0)    { duty_q0 = 0;    integral_q7 = 0; }

    prev_error_q0 = error;
    return (int16_t)duty_q0;
}
```

**Critical fixed-point pitfalls:**

1. **Intermediate overflow:** Multiplying Q7 × Q0 gives Q7. With A0=5307 and error=4096: product = 21.7 million — fits in int32 (max ~2.1 billion). Safe here.

2. **Coefficient rounding:** A0_Q7 = 41.46 × 128 = 5307.0 → stored as 5307 exactly. Error = 0.00039% — negligible.

3. **Integral precision at small errors:** When error = 1 ADC count, integral increment = 5307 (Q7) per sample. After 7 bits of shift, this is 41 counts of duty change per sample for a 1-count error. At 400 kHz sampling and 30 kHz crossover, this is appropriate.

4. **Verify no limit cycle:**
   ```
   Minimum duty step = 1 DPWM count
   Minimum Vout change per DPWM step = Vin × (1/4096) = 12/4096 = 2.93 mV
   ADC resolution = 0.806 mV/count
   2.93 mV > 0.806 mV → 3.6 ADC counts per DPWM step

   This means 3-4 ADC counts of error produces the same DPWM output → limit cycling possible.
   Solution: use sigma-delta DPWM or accept 3-count voltage regulation band.

   Alternatively: verify DPWM resolution requirement:
   N_dpwm > N_adc + log2(Vin/Vout) = 12 + log2(12/3.3) = 12 + 1.86 = 13.86 → 14-bit DPWM needed
   A 12-bit DPWM at 12V/3.3V = 3.6:1 ratio will limit cycle — use dithering.
   ```

---

## Step 5: Complete Implementation Verification

**Verify difference equation coefficients by checking DC gain:**

At DC (z = 1), the PI controller should have infinite gain (pure integrator):
```
C(z=1) = ... (z-1 denominator = 0 → infinite gain ✓)
```

**Verify at Nyquist (z = -1):**
```
C(z=-1) = a0 × (-1) + a1 × 1 / ((-1) - 1) ... complex calculation, but gain finite ✓
```

**Step response test (open-loop, no feedback):**

Apply e[n] = 1.0 (unit step, in ADC counts) and verify u[n] ramps linearly (PI integrator response):
```
n=0: u[0] = 0 + 41.46×1 + (-41.14)×0 = 41.46
n=1: u[1] = 41.46 + 41.46×1 + (-41.14)×1 = 41.46 + 41.46 - 41.14 = 41.78
n=2: u[2] = 41.78 + 41.46×1 + (-41.14)×1 = 41.78 + 0.32 = 42.10
...
Rate of increase per sample ≈ (A0 + A1) = 41.46 - 41.14 = 0.32 counts/sample
= Ki × Ts = 129,765 × 2.5µs = 0.324 counts/sample ✓
```

The ramp rate of the integral equals Ki × Ts — the bilinear discretization correctly captures the integral gain.

---

## Summary: Design Results

| Parameter           | Value          | Notes                              |
|---------------------|----------------|------------------------------------|
| Target fc           | 30 kHz         | = fs/13 (conservative)             |
| Zero frequency fz   | 500 Hz         | Placed well below fc               |
| Analog Kp           | 41.3           | Set for 0 dB loop gain at fc       |
| Analog Ki           | 129,765 rad/s  | = Kp × ωz                         |
| Bilinear A0         | 41.46          | = Kp + Ki×Ts/2                    |
| Bilinear A1         | -41.14         | = -Kp + Ki×Ts/2                   |
| Anti-windup gain    | 0.00786/sample | = ωz × Ts = Ts/Ti                 |
| Duty cycle range    | 0–4095         | 12-bit DPWM                        |
| Estimated PM        | ~50°           | Verify on hardware Bode plot       |
| DPWM limit cycle    | Possible       | Use dithering or 14-bit DPWM      |
