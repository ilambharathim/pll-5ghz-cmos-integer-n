# Integer-N Phase-Locked Loop — ~5 GHz, 90 nm CMOS

> A fully transistor-level integer-N PLL designed and simulated in Cadence Virtuoso/Spectre targeting ~5 GHz output frequency with a ÷48 feedback divider and 100 MHz reference input.

---

## Overview

This project implements a complete integer-N PLL in 90 nm CMOS (GPDK090) at the transistor level using Cadence Virtuoso. The loop comprises a Phase-Frequency Detector (PFD), Charge Pump (CP), passive loop filter, current-starved 5-stage ring oscillator VCO, and a ÷48 frequency divider built from True Single-Phase Clocked (TSPC) D flip-flops. Cadence Spectre transient and parametric simulations verify the individual blocks and demonstrate closed-loop PLL locking behaviour.

---

## Problem Statement

Designing a fully integrated integer-N PLL in nanometer CMOS requires careful co-design of several interdependent analog and mixed-signal blocks. Key challenges include:

- Achieving a linear, wide-range VCO tuning curve in the gigahertz regime using a ring oscillator topology
- Designing a dead-zone-free PFD with reliable reset logic
- Sizing the loop filter passives (R, C) to ensure loop stability and adequate phase margin
- Implementing a high-speed integer frequency divider using TSPC logic compatible with gigahertz-range VCO output

---

## Objectives

- Design and simulate each PLL block at transistor level: PFD, CP, LPF, VCO, Frequency Divider
- Achieve VCO operation in the ~4.8–4.85 GHz range using a 5-stage current-starved ring oscillator
- Implement a ÷48 divider to bring the VCO output down to the 100 MHz reference
- Demonstrate closed-loop transient locking with VREF = 100 MHz
- Validate each block individually before closed-loop integration

---

## System Architecture

```
VREF (100 MHz)
      │
      ▼
┌─────────────┐    UP     ┌─────────────┐
│     PFD     │ ────────► │  Charge     │
│  (2×DFF     │    DOWN   │  Pump       │
│   + AND)    │ ────────► │  (Icp=100µA)│
└─────────────┘           └──────┬──────┘
      ▲                          │  VC
      │                          ▼
      │                  ┌─────────────────────┐
      │                  │  Passive Loop Filter │
      │                  │  R=57.45kΩ           │
      │                  │  C1=164pF, C2=116fF  │
      │                  └──────────┬──────────┘
      │                             │  VC
      │                             ▼
      │                  ┌─────────────────────┐
      │                  │  Ring Oscillator VCO │
      │                  │  5-stage current-    │
      │                  │  starved (ROSINY)    │
      │                  │  ~4.8–4.85 GHz       │
      │                  └──────────┬──────────┘
      │                             │  VOUT (~4.814 GHz)
      │                             ▼
      │                  ┌─────────────────────┐
      │                  │  Frequency Divider   │
      │     VDIV         │  ÷48 (6× TSPC DFF   │
      └──────────────────│  + AND chain)        │
         (100.3 MHz)     └─────────────────────┘
```

---

## Circuit Blocks

### Phase-Frequency Detector (PFD)
- Architecture: Two D flip-flops with active-low reset via AND gate
- Inputs: VREF (reference), VDIV (divided feedback)
- Outputs: UP (QA), DOWN (QB)
- Flip-flop type: NOR-based DFF
- Function: Generates UP/DOWN pulses proportional to phase/frequency error between VREF and VDIV

### Charge Pump (CP)
- Architecture: PMOS/NMOS current mirror with inverter-controlled switch
- Bias current: 100 µA (Idc)
- PMOS/NMOS devices: GPDK090 `gpdk090_pmos1v` / `gpdk090_nmos1v`, W=120n, L=100n
- Output: Control voltage VC to loop filter
- Function: Sources/sinks charge packets to the loop filter proportional to UP/DOWN pulse widths

### Passive Loop Filter (LPF)
- Topology: Second-order passive RC
- R = 57.45 kΩ
- C1 = 164 pF
- C2 = 116 fF
- Function: Converts charge pump current pulses into a stable DC control voltage VC

### Voltage-Controlled Oscillator (VCO)
- Topology: 5-stage current-starved ring oscillator (ROSINY cell)
- Technology: GPDK090 `gpdk090_pmos1v` / `gpdk090_nmos1v`
- Total transistors: 26 instances
- Tuning input: VC
- Output frequency range: ~4.795 GHz – 4.849 GHz
- Function: Generates output clock VOUT whose frequency is controlled by VC

### ÷48 Frequency Divider
- Architecture: 6 cascaded TSPC DFFs with AND gate feedback for modulus control
- TSPC DFF: 8 PMOS + 3 NMOS per stage, W=120n, L=100n
- Input: VOUT from VCO
- Output: DIV_OUT at 1/48 of VCO frequency
- Simulation verified: Input ~2.083 MHz → Output 100 MHz (in standalone testbench)

### TSPC D Flip-Flop
- Topology: True Single-Phase Clocked (TSPC) DFF
- Devices: `gpdk090_pmos1v` (W=120n) and `gpdk090_nmos1v` (W=120n, L=100n)
- Used in: ÷48 divider (6 instances)
- Advantage: Single clock phase, suitable for high-speed digital operation at GHz frequencies

---

## Tools & Technology

| Tool / Technology | Purpose |
|---|---|
| Cadence Virtuoso 6.1.8 | Transistor-level schematic capture |
| Cadence Spectre | Transient and parametric simulation |
| Cadence ADE L | Simulation setup, parametric sweep, output measurement |
| GPDK090 (90 nm CMOS PDK) | Technology library (`gpdk090_pmos1v`, `gpdk090_nmos1v`) |

---

## Simulation Results

All results obtained from Cadence Spectre simulation. No hardware measurements were performed.

### VCO Tuning Curve (Parametric Sweep)

| VC (V) | VCO Frequency (GHz) |
|---|---|
| 0.1 | 4.795 |
| 0.2 | 4.801 |
| 0.3 | 4.816 |
| 0.4 | 4.835 |
| 0.5 | 4.845 |
| 0.6 | 4.848 |
| 0.7 | 4.848 |
| 0.8 | 4.849 |
| 0.9 | 4.849 |
| 1.0 | 4.849 |

> Source: `docs/simulation/vco_frequency_table.png` — Cadence ADE parametric sweep

**VCO Gain (Kvco):** Peak ~186 MHz/V at VC ≈ 0.3 V

> Source: `docs/simulation/vco_kvco_table.png` — derivative of frequency vs. VC

### ÷48 Divider (Standalone)

| Signal | Frequency |
|---|---|
| DIV_IN (input) | 2.083 MHz |
| DIV_OUT (output) | 100.0 MHz |

> Source: `docs/simulation/divider_48_frequency_table.png`

### Closed-Loop PLL (Transient — 1.5 µs)

| Signal | Measured Value |
|---|---|
| VREF | 100 MHz |
| VDIV (feedback) | 100.3 MHz |
| VOUT (VCO) | 4.814 GHz |

> Source: `docs/simulation/pll_ade_output_measurements.png` — Cadence ADE outputs panel

### PLL Transient Waveforms (Closed Loop)

The transient simulation (0–400 ns shown) captures:
- VREF: 100 MHz reference clock (1.5 V swing)
- VDIV: Divided feedback approaching lock with VREF
- UP/DOWN: PFD error pulses narrowing as loop approaches lock
- VC: Control voltage stabilising to a near-DC level
- VOUT: VCO output (fully saturated due to gigahertz frequency — visible in frequency domain)

> Source: `docs/simulation/pll_transient_locked.png`

---

## Repository Structure

```
pll-5ghz-cmos-integer-n/
│
├── README.md
├── .gitignore
│
└── docs/
    ├── schematics/
    │   ├── pll_top_schematic.png          ← Full PLL top-level
    │   ├── pfd_schematic.png              ← Phase-frequency detector
    │   ├── charge_pump_schematic.png      ← Charge pump (transistor level)
    │   ├── charge_pump_testbench.png      ← CP+PFD testbench
    │   ├── dff_schematic.png              ← NOR-based DFF (used in PFD)
    │   ├── tspc_dff_schematic.png         ← TSPC DFF (used in divider)
    │   ├── divider_48_schematic.png       ← ÷48 divider (6× TSPC)
    │   └── vco_ring_osc_schematic.png     ← 5-stage ring oscillator VCO
    │
    ├── simulation/
    │   ├── vco_tuning_curve_frequency.png ← Frequency vs. VC (tuning curve)
    │   ├── vco_tuning_curve_kvco.png      ← Kvco (dF/dVC) vs. VC
    │   ├── vco_parametric_sweep_results.png ← ADE parametric sweep output
    │   ├── vco_kvco_table.png             ← Tabulated Kvco values
    │   ├── vco_frequency_table.png        ← Tabulated VCO frequencies
    │   ├── divider_48_transient.png       ← Divider transient waveform
    │   ├── divider_48_frequency_table.png ← Divider frequency measurement
    │   ├── pll_transient_locked.png       ← Full PLL transient (lock behaviour)
    │   └── pll_ade_output_measurements.png ← ADE output: VREF, VDIV, VOUT freq.
    │
    └── references/
        └── ring_oscillator_reference.pdf  ← Ring oscillator design reference
```

---

## Verification

All verification was performed at schematic level using Cadence Spectre. No post-layout or DRC/LVS verification was performed in this work.

| Block | Verification Type | Result |
|---|---|---|
| VCO | Transient + parametric sweep (VC sweep 0.1–1.0 V) | Frequency range 4.795–4.849 GHz |
| VCO Kvco | Derivative of tuning curve | Peak ~186 MHz/V @ VC≈0.3 V |
| ÷48 Divider | Standalone transient simulation | Correct ÷48 division verified |
| Charge Pump + PFD | Combined testbench | UP/DOWN pulse generation verified |
| Full PLL (closed-loop) | Transient simulation (1.5 µs) | VOUT ~4.814 GHz, VDIV ~100.3 MHz |

---

## Applications

- Wireless communication transceivers (frequency synthesis)
- Clock generation and distribution (SoC, processors)
- Frequency multiplication in mixed-signal ICs
- Educational reference for integer-N PLL design in CMOS

---

## Limitations

- This is a schematic-level design only; post-layout parasitics have not been accounted for
- No DRC, LVS, or physical layout was performed
- The current-starved ring oscillator VCO has a narrow tuning range (~54 MHz over 0.1–1.0 V) — a limitation of the ring topology compared to LC-VCO implementations
- The ÷48 standalone testbench uses a lower-frequency input (MHz range); full GHz-speed divider verification with VCO output was not shown separately
- Loop lock time and phase noise were not explicitly extracted in this work
- No corner or Monte Carlo simulations were performed

---

## Future Work

- Perform physical layout in Cadence Virtuoso Layout Editor
- Run DRC and LVS verification (Cadence PVS or Calibre)
- Extract post-layout parasitics and re-simulate (post-layout simulation)
- Characterize phase noise of VCO and closed-loop PLL
- Measure lock time from initial transient
- Explore LC-VCO replacement for wider tuning range and lower phase noise
- Add ÷N programmability to enable multi-channel frequency synthesis

---

## Getting Started

This repository contains schematic screenshots and simulation results from Cadence Virtuoso. There are no standalone netlists or SPICE files committed.

To reproduce the design:

1. Install **Cadence Virtuoso 6.1.8** with **GPDK090** PDK
2. Recreate the schematics from the images in `docs/schematics/`
3. Set up Cadence Spectre transient analyses as shown in `docs/simulation/pll_ade_output_measurements.png`
4. Run parametric sweep on VC (0.1 V to 1.0 V, step 0.1 V) to reproduce the VCO tuning curve

---

## References

- Ring Oscillator design reference: `docs/references/ring_oscillator_reference.pdf`
- B. Razavi, *Design of Analog CMOS Integrated Circuits*, McGraw-Hill
- GPDK090 PDK Documentation (Cadence Generic Process Design Kit, 90 nm)
