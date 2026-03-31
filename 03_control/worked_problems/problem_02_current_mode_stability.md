# Worked Problem 02: Current-Mode Control Stability and Slope Compensation

## Problem Statement

A peak current-mode controlled (PCMC) buck converter has the following specifications:

```
Vin  = 48 V
Vout = 12 V
D    = Vout/Vin = 0.25  (nominal duty cycle)
L    = 20 µH
fsw  = 300 kHz
Ts   = 1/fsw = 3.33 µs
```

**Part A:** Determine whether subharmonic oscillation will occur without slope compensation.

**Part B:** Calculate the minimum slope compensation ramp required to eliminate subharmonic oscillation at all duty cycles up to D = 0.9.

**Part C:** Calculate the optimal slope compensation for best transient performance.

**Part D:** Analyse what happens when slope compensation is set too high.

---

## Background: Subharmonic Oscillation Mechanism

In peak current-mode control, the duty cycle each cycle is determined by when the inductor current ramp reaches the control threshold set by the voltage loop error amplifier.

A perturbation δ in the inductor current at the start of a cycle propagates from cycle to cycle. The growth or decay of this perturbation determines stability.

**Without slope compensation:**

```
Inductor on-slope (current rising):   m1 = (Vin - Vout) / L
Inductor off-slope (current falling): m2 = Vout / L

Perturbation gain (per cycle): A = -(m2 / m1)
```

If |A| > 1: perturbation grows → subharmonic oscillation (unstable).
If |A| < 1: perturbation decays → stable.
If |A| = 1: perturbation neither grows nor decays → marginally stable.

**Critical condition:**
```
|A| = m2/m1 = Vout/(Vin - Vout) = D/(1-D)

At D = 0.5: m2/m1 = 1  → marginally stable
At D > 0.5: m2/m1 > 1  → unstable
```

---

## Part A: Stability Assessment Without Slope Compensation

**Calculate on-slope and off-slope:**

```
m1 = (Vin - Vout) / L = (48 - 12) / 20µH = 36 / 20µH = 1.8 A/µs

m2 = Vout / L = 12 / 20µH = 0.6 A/µs
```

**Perturbation gain at nominal D = 0.25:**
```
A = -(m2/m1) = -(0.6 / 1.8) = -0.333

|A| = 0.333 < 1  →  STABLE at D = 0.25
```

**Maximum stable duty cycle without slope compensation:**
```
|A| = 1 when m2/m1 = 1 → D/(1-D) = 1 → D = 0.5

So stability limit is D = 0.5.
```

**Conclusion for Part A:**
At D = 0.25, the converter is stable without slope compensation. However:
- If Vin drops from 48V to 26.7V while Vout stays at 12V, D rises to 0.45 (still stable).
- If Vin drops to 24V, D = 0.5 → marginally stable.
- If the voltage loop demands Vout higher (e.g., during a load step transient), D could transiently exceed 0.5 → momentary instability and output oscillation.

**For a design with Vin_min = 24V:** D_max = 12/24 = 0.5. This is right at the stability boundary. Slope compensation is needed.

**For Vin that could go down to 20V:** D_max = 12/20 = 0.6 > 0.5 → subharmonic oscillation without slope compensation.

---

## Part B: Minimum Slope Compensation for D up to 0.9

**With slope compensation ramp mc added to the sensed current (or subtracted from the control reference):**

```
Perturbation gain with slope compensation:
A = -(m2 - mc) / (m1 + mc)
```

For stability: |A| ≤ 1 (equality gives marginal stability; choose |A| < 1 for margin).

**For stability at all D up to D_max = 0.9:**

At D = 0.9 (worst case):
```
m1 = (Vin_for_D0.9 - Vout) / L

We need to find Vin that gives D = 0.9:
D = Vout/Vin → Vin = Vout/D = 12/0.9 = 13.3 V

m1 = (13.3 - 12) / 20µH = 1.3 / 20µH = 0.065 A/µs
m2 = Vout / L = 12 / 20µH = 0.6 A/µs   [unchanged, only Vout/L]

Note: m2/m1 = 0.6/0.065 = 9.23  → extremely high gain, very unstable without compensation
```

**Stability condition at D = 0.9:**
```
(m2 - mc) / (m1 + mc) ≤ 1

m2 - mc ≤ m1 + mc
m2 - m1 ≤ 2mc
mc ≥ (m2 - m1) / 2
   ≥ (0.6 - 0.065) / 2
   ≥ 0.535 / 2
   = 0.268 A/µs
```

**But the standard design recommendation is mc ≥ m2/2:**

This ensures stability for any duty cycle (even D → 1):
```
mc_min = m2 / 2 = 0.6 / 2 = 0.3 A/µs
```

**Verify at D = 0.9 with mc = 0.3 A/µs:**
```
A = -(m2 - mc) / (m1 + mc)
  = -(0.6 - 0.3) / (0.065 + 0.3)
  = -(0.3) / (0.365)
  = -0.822

|A| = 0.822 < 1  ✓  Stable
```

**Convert slope to voltage (for circuit design):**

The slope compensation is usually added as a ramp voltage to the current sense signal or subtracted from the error amplifier output. If the current sense gain (across Rs or CT) converts inductor current to voltage with gain Ki (V/A):

```
mc_voltage = mc × Ki

If Ki = 0.1 V/A (typical for 100mΩ sense resistor):
mc_voltage = 0.3 A/µs × 0.1 V/A = 0.03 V/µs = 30 mV/µs
```

Over the maximum on-time at D = 0.9, Ts = 3.33 µs:
```
Max on-time: t_on = D × Ts = 0.9 × 3.33µs = 3.0 µs
Ramp amplitude at max D: 30 mV/µs × 3.0 µs = 90 mV
```

---

## Part C: Optimal Slope Compensation

**Optimal choice: mc = m2/2**

This is the Erickson-Maksimovic recommendation for optimal transient performance. At the optimal slope:

```
mc_optimal = m2/2 = Vout / (2L) = 12 / (2 × 20µH) = 0.3 A/µs
```

**Why mc = m2/2 is optimal:**

1. **Stability:** Ensures |A| < 1 for all duty cycles (as shown above).

2. **Transient response:** With mc = m2/2, the current loop responds to control signal changes in a way that is optimally damped — neither too slow (over-compensated) nor ringing (under-compensated).

3. **Mathematical basis:** The sampled-data model of the current loop shows that the current loop bandwidth is maximised at mc = m2/2 without causing subharmonic oscillation. The current loop gain has a pole at z = -1 exactly at the Nyquist frequency (fs/2), which means the current ripple correction happens in exactly one cycle.

**Practical ramp generation circuit:**

```
A resistor-capacitor network connected to the oscillator timing capacitor generates a ramp:

If the oscillator uses a capacitor Cosc charged by Iosc:
  Ramp slope = Iosc / Cosc  [V/s]

Add this ramp via a resistor Rslope to the current sense pin:
  mc_added_voltage = (Iosc / Cosc) × (Rslope / (Rslope + Rs_input))

Adjust Rslope to achieve mc_voltage = mc × Ki = 0.3 × 0.1 = 0.03 V/µs
```

---

## Part D: Effect of Excessive Slope Compensation

**With mc >> m2 (over-compensation):**

As mc increases beyond m2/2:
```
A = -(m2 - mc) / (m1 + mc)
```

When mc > m2: (m2 - mc) becomes negative → A > 0 → perturbations no longer oscillate but instead decay monotonically (no oscillation, but slower convergence).

When mc >> m1, m2:
```
A ≈ -(mc/mc) = -1   → marginally stable again (but now at Nyquist frequency due to the -1 sign)
```

As mc → ∞:
```
A → -1  [purely oscillatory at fs/2, borderline unstable in magnitude]
```

**Effect on current loop bandwidth:**

The inner current loop bandwidth degrades as slope compensation increases. With high mc, the current loop acts like a voltage-mode converter with a slow inner loop.

Specifically, the current loop gain at the Nyquist frequency becomes:
```
|Hid(e^{jπ})| = (m2 - mc) / (m1 + mc)
```

Excessive slope reduces this gain, meaning the current loop responds sluggishly to control commands. The advantage of current-mode (fast inner loop, simple outer loop compensation) is lost.

**Effect on converter audio susceptibility:**

With very high slope compensation, the line-to-output audio susceptibility reverts to that of a voltage-mode converter — the input voltage feedforward effect of current-mode control disappears. For a boost or flyback designed to reject input ripple via current-mode control, excessive slope compensation re-introduces input ripple at the output.

**Quantitative bandwidth degradation:**

The effective bandwidth of the current loop with slope compensation:
```
f_inner_3dB ≈ (m1 + mc) / (2π × L × (m2 + mc)) × R_load ... [simplified]
```

As mc increases from 0 to m2, the bandwidth decreases by approximately 50%.
At mc = 5 × m2, the bandwidth is approximately 6× lower than without compensation.

**Recommended maximum:**
```
mc_max ≤ m2   [to preserve current-mode advantages]
```

---

## Summary Table

| Condition               | mc (A/µs) | Stability at D=0.9 | |A| at D=0.9 | Bandwidth |
|-------------------------|-----------|---------------------|------------|-----------|
| No slope comp           | 0         | Unstable            | 9.23       | Maximum   |
| Minimum stable          | 0.268     | Marginally stable   | 1.0        | High      |
| Optimal (mc = m2/2)     | 0.3       | Stable              | 0.82       | Optimal   |
| Moderate overcomp.      | 0.6       | Stable              | 0.0        | Reduced   |
| Excessive               | 3.0       | Stable              | 0.73 (neg) | Very low  |

---

## Key Equations Summary

```
Inductor on-slope:    m1 = (Vin - Vout) / L   [A/s]
Inductor off-slope:   m2 = Vout / L            [A/s]

Perturbation gain (no compensation):   A = -(m2/m1)
Perturbation gain (with compensation): A = -(m2 - mc)/(m1 + mc)

Stability condition:  |A| < 1

Minimum slope (general):   mc ≥ (m2 - m1)/2
Minimum slope (for all D): mc ≥ m2/2   (ensures stability as D → 1)
Optimal slope:             mc = m2/2 = Vout/(2L)

Slope in voltage (referred to current sense signal):
  mc_V = mc × Rs   [V/µs]  where Rs = current sense resistance
```

---

## Interview Debrief: What Examiners Test

**Level 1 (fundamentals):**
- "At what duty cycle does a peak current mode converter become unstable without slope compensation?"
- Expected answer: D > 0.5

**Level 2 (derivation):**
- "Derive the minimum slope compensation needed for a buck converter."
- Expected: draw the inductor current waveform with perturbation, show geometric relationship, derive mc ≥ m2/2.

**Level 3 (design judgment):**
- "If your converter needs to operate from D = 0.2 to D = 0.85, what slope compensation do you specify?"
- Expected: calculate m2 at worst case Vin (lowest Vin → highest D), apply mc = m2/2 at that point, note this must hold at all operating conditions.

**Level 4 (trade-offs):**
- "Your customer says the converter has poor line rejection after adding slope compensation. Why?"
- Expected: excessive slope compensation reduces current-mode input feedforward — converter behaves like voltage mode. The input ripple appears at the output. Solution: optimise slope to minimum required (mc = m2/2, not more).
