---
title: stepCAN chain
slug: chain
sidebar_position: 30
sidebar_label: stepCAN chain
description: The SP3490 full-duplex transceiver, the chain schematic, mandatory fail-safe biasing, debounce timing and the supply budget.
tags: [stepcan, rs-485, sp3490]
---

# stepCAN chain physical layer

## Transceiver — SP3490

The chain uses **one** RS-485 transceiver per module. This is possible because the SP3490 is a **full-duplex** part: its driver (Y/Z) and receiver (A/B) are electrically independent ports, which maps exactly onto the module's OUT and IN connectors.

| Parameter | SP3490 | Consequence |
|---|---|---|
| Duplex | **Full — Y/Z driver, A/B receiver** | One chip serves both ports |
| Package | **8-pin NSOIC** | Same footprint class as a half-duplex part |
| Enable pins | **None** — driver and receiver always on | Two MCU pins, nothing to get wrong in firmware |
| Supply | 3.0 / 3.3 / 3.6 V | Runs directly off the module's 3.3 V rail |
| Max data rate | 10 Mbit/s | Irrelevant — the chain carries a static level |
| Receiver thresholds | V_IT± = ±200 mV | Symmetric — **not** true fail-safe |
| Fail-safe | **Open-circuit only** | **External bias resistors are mandatory** |
| ESD, bus pins | **±2 kV HBM only; no IEC 61000-4-2 figure** | **External TVS is mandatory** |
| Driver short-circuit current | ±250 mA, −7 V ≤ V_O ≤ +12 V | |
| R_IN | 12 kΩ min / 15 kΩ typ | Not a node-count limit here — see below |
| I_CC, quiescent | 1 mA typ / 2 mA max | With-load figure derived below |
| θ_JA, 8-pin NSOIC | 128.4 °C/W | |

**Pinout (8-pin NSOIC):**

| Pin | Name | Function | Connects to |
|---|---|---|---|
| 1 | VCC | +3.3 V | Module 3.3 V rail |
| 2 | R | Receiver output | MCU `CHAIN_IN` (GPIO input) |
| 3 | D | Driver input | MCU `CHAIN_OUT` (GPIO output) |
| 4 | GND | Ground | Module 0 V |
| 5 | **Y** | Driver out, non-inverting | **OUT connector pin 4** |
| 6 | **Z** | Driver out, inverting | **OUT connector pin 5** |
| 7 | **B** | Receiver in, inverting | **IN connector pin 5** |
| 8 | **A** | Receiver in, non-inverting | **IN connector pin 4** |

The SP3491 is the same silicon with DE/RE enables in a 14-pin package; not needed here and it costs board area. The SP3485 is half-duplex and cannot do this job with one chip.

## Schematic

```
                            +3.3V
                              │
                            ┌─┴─┐
                            │R1 │ 4k7            ┌────────
 IN connector               └─┬─┘                │
 (upstream, RJ45 or B2B)      │                  │
                              │                  │
   pin 4 ──[Rs1 10R]──────────┼──────────────────┤ 8  A      ┌─────────────┐
                              │                  │           │             │
                            ┌─┴─┐ RT 120R        │  SP3490   │  2  R ──────┼──▶ MCU CHAIN_IN
                            │DNP│ (see below)    │           │             │
                            └─┬─┘                │           │  3  D ◀─────┼── MCU CHAIN_OUT
                              │                  │           │             │
   pin 5 ──[Rs2 10R]──────────┼──────────────────┤ 7  B      └─────────────┘
                              │                  │
                            ┌─┴─┐                │            1 VCC ── +3.3V
                            │R2 │ 4k7            │            4 GND ── 0V
                            └─┬─┘                │
                              │                  │
                             GND                 │
                                                 │
 OUT connector                                   │
 (downstream)                                    │
                                                 │
   pin 4 ──[Rs3 10R]──────────────────────────── ┤ 5  Y
   pin 5 ──[Rs4 10R]──────────────────────────── ┤ 6  Z
                                                 └────────

 Protection on both ports: GDT to PE at the connector, common-mode choke,
 SM712 TVS after the choke. See the Protection page.
```

!!! tip "The structural point: IN and OUT are separate nets"
    Nothing bridges them on the PCB. The "series" connection of the chain is made by the transceiver and the firmware, not by copper.

    The alternative — bridging the incoming pair to the outgoing pair through a relay or analogue switch — puts **128 contacts in series** across a 64-module rack. One oxidised contact anywhere and the rack does not start. It also accumulates on-resistance: 64 analogue switches at 3 Ω each would put 190 Ω in series with a 60 Ω bus.

## Fail-safe biasing — mandatory

The SP3490's receiver has symmetric ±200 mV thresholds and only open-circuit fail-safe. A module whose upstream neighbour is absent, unpowered or not yet booted sees a floating IN pair and will read noise.

!!! danger "Without bias, the head will enumerate modules that do not exist"
    This is not a subtle degradation. It is a hard functional failure and it is the first thing to check at bring-up.

Two valid configurations:

| Configuration | R1, R2 | RT | V_bias | Driver load | I per module | 64 modules |
|---|---|---|---|---|---|---|
| **Unterminated (recommended)** | **4.7 kΩ** | not fitted | **1850 mV** | 21.4 kΩ | 0.15 mA | **10 mA** |
| Terminated (chain reused for data) | 560 Ω | 120 Ω | 319 mV | 1.24 kΩ | 2.7 mA | 170 mA |

`V_bias` for the unterminated case is the divider working into the receiver's 12 kΩ input resistance:

```
V_bias = 3.3 V × 12 k / (4.7 k + 12 k + 4.7 k) = 1.85 V
```

Both clear the 200 mV threshold; the unterminated configuration clears it by 9×.

**Why unterminated is correct here.** The chain carries a DC level, not data. Reflections on a static signal are irrelevant, and with a slope-limited transition the longest plausible link is still electrically short. Omitting the 120 Ω saves 160 mA across a 64-module rack and lets the bias resistors be 8× larger.

**Fit the 120 Ω footprint but leave it unpopulated.** It costs nothing and preserves the option of using the chain as a real data channel later — firmware update, diagnostics — at which point you swap to the 560 Ω bias values.

!!! info "TR-3 — chain idle polarity"
    The bias network shall pull A above B, and the idle (unbiased, undriven) state shall therefore be read by firmware as **"not enabled"**. An unpowered or absent upstream neighbour must never present as *enabled*.

## Timing and debounce

The chain level is static, so firmware treats it as a mechanical contact:

| Parameter | Value | Reason |
|---|---|---|
| Debounce time | **1 ms stable** before a level change is accepted | Rejects EFT-induced glitches without slowing enumeration meaningfully |
| Assert-to-probe delay | 2 ms after driving CHAIN_OUT before the head probes | Lets the downstream receiver settle and its firmware notice |
| Loss-of-chain reaction | Immediate on a debounced 1→0 transition | Module drops its node-ID and cascades downstream |

## Why the 32-node RS-485 limit does not apply

RS-485 transceivers carry a "32 unit loads" figure derived from TIA/EIA-485-A, and the SP3490 at 12 kΩ is one unit load. This is frequently — and here, wrongly — read as "no more than 32 SP3490s in a system".

The limit is **per electrical segment**, and a stepCAN chain segment has exactly **two** devices on it: one driver and one receiver, on one span of cable between two physical neighbours. A 64-module rack contains 64 independent two-device segments, not one 64-device bus. The unit-load budget is used at 2 of 32 on every one of them.

This is a further argument for the chain-of-links topology over any bridged-line scheme, where all 64 transceivers really would share one segment and the limit really would bite.

## Supply budget

MaxLinear specifies I_CC only for the unloaded condition, in every revision back to the 2002 Sipex original. There is no Typical Performance Characteristics section and no supply-current curve. The with-load figure must be derived; the load is a resistor network this specification defines, so the derivation is exact.

Assume the driver outputs sit at the rails — at a ~13 kΩ total load the drop across the driver's output impedance is negligible, and this assumption maximises the computed current.

**Worst case: driver opposing the bias network (A = 0 V, B = 3.3 V)**

```
4.7 kΩ pull-up,   3.3 V across it   →  702 µA   sunk by output A
4.7 kΩ pull-down, 3.3 V across it   →  702 µA   sourced by output B
12 kΩ receiver,   3.3 V across it   →  275 µA   sourced by B, sunk by A

current through the chip via B   = 702 + 275  =  977 µA
total drawn from the 3.3 V rail  = 702 + 977  = 1679 µA ≈ 1.68 mA
```

| | Quiescent | + Load (worst case) | **Total** |
|---|---|---|---|
| Typical | 1.00 mA | 1.68 mA | **2.68 mA** |
| **Maximum** | 2.00 mA | 1.68 mA | **≈ 3.7 mA** |
| Best-case polarity | 2.00 mA | 0.28 mA | 2.28 mA |

!!! note "Do not add the receiver input current separately"
    The datasheet's I_IN figure (1.0 mA at V_IN = 12 V) and the 12 kΩ input resistance are the same physical effect — 12 V ÷ 12 kΩ = 1.0 mA. Adding both double-counts. At 0–3.3 V operating levels the relevant number is V/R_IN, far below the worst-case 1.0 mA figure.

**Across a 64-module rack: about 171 mA typical.** Negligible against any plausible supply.

**Thermal:** on-chip dissipation is 6.6 mW of the 12.1 mW total, giving a junction rise of 6.6 mW × 128.4 °C/W = **0.85 °C**. Continuous driver operation at this load needs no derating, no thermal relief and no copper pour.
