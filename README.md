# Event-Based Photodiode Sensing & TIA Circuit Optimization

> Analog front-end circuit design and SPICE simulation for an in-sensor event-detection system using dual electrostatically-doped silicon photodiodes and an OPA818 transimpedance amplifier.

---

## Project Overview

This project focuses on the design, simulation, and optimization of an analog readout circuit for an **in-sensor event-detection system** based on dual-gate silicon photodiodes. The circuit converts nanoampere-level photocurrents from a 2R1C photodiode network into measurable voltage spikes that encode light ON/OFF events — mimicking how the human retina detects temporal changes in the visual field.

The work is part of ongoing research in the **Xu Lab at UMass Amherst** (Department of Electrical and Computer Engineering), building on published work in [*Nature Electronics* (Jang et al., 2022)](https://doi.org/10.1038/s41928-022-00819-6) and the lab's own amorphous-silicon photodiode array platform.

---

## Circuit Architecture

```
[PD₁] ──R──┐
            ├──── I_net ──► [OPA818 TIA] ──► V_out ──► Oscilloscope
[PD₂] ──R──┤
       ──C──┘
```

The **2R1C network** pairs two photodiodes (PD₁ and PD₂) in complementary gate configurations:
- **PD₁** (1R branch): responds immediately to changes in light power (ΔP_light)
- **PD₂** (1R1C branch): responds with an RC time delay

When light turns ON or OFF, the two branches produce a net differential current — a transient spike — captured and amplified by the **OPA818 TIA** on a custom PCB.

The TIA closed-loop transfer function is:

$$A_{CL}(s) = -\frac{2R_F}{R} \cdot \frac{1 + s\frac{RC}{2}}{1 + sR_FC_F}$$

- **DC Gain:** −2R_F / R
- **Zero:** F_Z = 2 / (πRC)
- **Pole:** F_P = 1 / (2πR_FC_F)

---

## Research Objectives

The primary challenges investigated in this project:

### 1. Light ON/OFF Spike Amplitude Mismatch
The positive (light-ON) and negative (light-OFF) output spikes exhibit different amplitudes, which is undesirable for symmetric event detection. Two possible causes were investigated:

- **TIA feedback resistor R_F:** Increasing R_F raises DC gain but lowers bandwidth (pole F_P = 1/R_FC_F shifts left), potentially placing both spikes in the roll-off region of the Bode plot at different gain levels.
- **2R1C resistor R:** Adjusting R shifts the circuit zero (F_Z = 2/RC), changing how the gain is distributed across the ON and OFF spike frequencies.

### 2. Post-Event Overshoot / Ringing After Light-OFF Spike
A transient bump appears after the negative light-OFF spike. Two hypotheses were examined:

- **RC bandwidth limiting:** A larger R in the 2R1C reduces the bump but slows overall spike response — a direct speed vs. stability tradeoff.
- **Second-order parasitics:** When real photodiode parasitics (junction capacitance C_j, gate-source parasitic capacitance C_gs) are included in the model, the transfer function develops **complex poles and zeros**, introducing RLC-like resonance behavior even without physical inductance. This may be the root cause of the ringing.

---

## Tools & Methods

| Tool | Purpose |
|------|---------|
| **LTspice** | SPICE simulation of the full 2R1C + TIA circuit |
| **OPA818 SPICE Model** | Texas Instruments op-amp model (loaded into LTspice) |
| **Bode Plot Analysis** | Frequency-domain gain and phase response under varying R, R_F, C_F |
| **Transient Analysis** | Time-domain spike shape simulation across kHz–MHz regimes |
| **Oscilloscope** | Hardware validation of simulated waveforms on physical PCB |

---

## Key Design Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Photodiode current range | ~nA | Electrostatically-doped p-i-n silicon PDs |
| 2R1C resistor R | 0.5–1 MΩ (swept) | Controls zero frequency and spike speed |
| TIA feedback resistor R_F | 1–100 MΩ (swept) | Controls DC gain and bandwidth |
| TIA feedback capacitor C_F | 2–100 pF (swept) | Controls stability and high-freq roll-off |
| Target spike frequency range | kHz–MHz | Human vision ≈ kHz; faster enables new applications |
| TIA device | OPA818 | High-bandwidth, low-noise op-amp |

---

## Simulation Results Summary

### Effect of C_F on TIA stability
- **C_F = 100 pF:** Pole at ~kHz range; filters high-frequency noise; stable but reduces bandwidth
- **C_F = 2 pF:** Pole shifts to ~MHz range; higher bandwidth but susceptible to noise and oscillation

### Effect of R_F on amplitude mismatch
- Higher R_F increases gain but narrows bandwidth, causing ON and OFF spikes to land at different points on the roll-off curve → **different gain → amplitude mismatch**
- Lower R_F reduces mismatch but sacrifices sensitivity (lower gain for nA-level currents)

### Effect of R on overshoot
- R = 0.5 MΩ: Clean spikes, no visible bump after light-OFF spike
- R = 1 MΩ: Bump appears after light-OFF spike; confirmed in simulation
- Increasing R reduces bump but slows trise/tfall — direct speed-stability tradeoff

### Real photodiode model (with C_gs)
- Including gate-source parasitic capacitance C_gs introduces **second-order terms** in the transfer function
- Results in complex pole-zero pairs → Bode plot shows resonant peak behavior
- This likely contributes to observed ringing independent of R selection

---

## Repository Contents

```
├── README.md                          # This file
├── simulation/
│   └── HA_event_complete_with_Cgs_1stage_B2ckt_MD.asc   # LTspice schematic
├── docs/
│   ├── transfer_function_bode.png     # Slide 3: Transfer function + Bode plots
│   ├── amplitude_mismatch_RF.png      # Slide 4: Mismatch vs R_F sweep
│   └── amplitude_mismatch_R.png       # Slide 6: Mismatch vs 2R1C R sweep
└── references/
│   ├── Jang_et_al_2022_Nature_Electronics.pdf
│   └── Dual_Gate_Photodiode_UMass.pdf
```

---

## Background & Motivation

Traditional CMOS image sensors separate sensing (front-end photodiode array) from processing (back-end digital pipeline). **In-sensor computing** integrates processing directly at the sensing layer, reducing data transfer overhead, latency, and power — analogous to how the human retina performs early visual processing before signals reach the brain.

This circuit implements the **readout front-end** for a gate-tunable, dual-photodiode event-detection unit. The electrostatically-doped photodiodes (fabricated at UMass Amherst) can have their photocurrent direction and amplitude programmed via gate voltage V_p, enabling analog multiply-accumulate operations directly at the sensor level.

The TIA is the critical interface between the nA-level photocurrent differential and a measurable voltage output — making its gain-bandwidth optimization central to the system's overall performance.

---

## References

1. Jang, H. et al. *In-sensor optoelectronic computing using electrostatically doped silicon.* Nature Electronics 5, 519–525 (2022). https://doi.org/10.1038/s41928-022-00819-6

2. Xiong, Z. et al. *Parallelizing analog in-sensor visual processing with arrays of gate-tunable silicon photodetectors.* University of Massachusetts Amherst, Xu Lab (2024).

3. Texas Instruments OPA818 Datasheet & SPICE Model. https://www.ti.com/product/OPA818

---

## Acknowledgements

This work was conducted under the supervision of the **Xu Lab, Department of Electrical and Computer Engineering, University of Massachusetts Amherst.** Circuit simulation and analysis performed by Adithya Kumar as part of graduate research contributions (Dec 2025).
