---
title: Power distribution
slug: power
sidebar_position: 10
sidebar_label: Power
description: Local, group and cable-powered options, voltage drop over Cat5e, injection strategy and where isolation belongs.
tags: [stepcan, power, cable]
---

# Power distribution

## Options

| Option | Where | Notes |
|---|---|---|
| **Local 24 V per module or per enclosure** | Distributed and mixed profiles | **Recommended.** Each enclosure has a supply anyway. |
| **Group supply over the backplane connector** | Backplane profile | Pins 7/8 carry the group's 24 V through every module. Rate the contacts ≥2 A and limit groups to 16 modules. |
| **Bus-powered over the FTP pair 7/8** | Short distributed runs only | Severely limited — see below. |

## Voltage drop over Cat5e

Cat5e 24 AWG solid is 93.8 mΩ/m per conductor, so 187.6 mΩ/m of loop.

| Length | Current | Loop R | Drop | Loss |
|---|---|---|---|---|
| 10 m | 0.5 A | 1.68 Ω | 0.84 V | 0.42 W |
| 10 m | 1.0 A | 1.68 Ω | 1.68 V | 1.68 W |
| 20 m | 0.5 A | 3.37 Ω | 1.68 V | 0.84 W |
| 20 m | 1.0 A | 3.37 Ω | 3.37 V | 3.37 W |
| 30 m | 0.5 A | 5.05 Ω | 2.53 V | 1.26 W |
| 30 m | 1.0 A | 5.05 Ω | 5.05 V | 5.05 W |
| 30 m | 2.0 A | 5.05 Ω | **10.10 V** | 20.2 W |

!!! danger "Do not attempt to feed 64 modules from one end over the FTP pair"
    At a modest 100 mA per module that is **6.4 A at the head** — far beyond 24 AWG's current rating and producing a drop that leaves the far modules unpowered. Cat5e power is for a handful of modules on a short run, nothing more.

## Recommended strategy

1. **Power each enclosure or backplane group locally** from its own 24 V supply.
2. Carry 0 V between groups on pair 7/8 as a **reference**, not as a supply, so the CAN common-mode reference is continuous along the bus.
3. If you do distribute power over the cable, **inject 24 V every 8 modules** and size for ≤0.5 A per segment.
4. Fuse or current-limit each injection point. A shorted module must not take down the rack.

## Isolation

For a 30 m in-building bus with a continuous 0 V reference, bus isolation is unnecessary and costs bit-timing margin. Isolate at the **I/O terminals**: optocoupled or magnetically isolated digital inputs, relay or opto-isolated outputs, isolated analogue front-ends. That is where the field-side energy is.

If a run crosses between separately-earthed buildings, isolate the bus — with fast digital isolators, not optocouplers. See [Protection](../physical-layer/protection.md#isolation) and the propagation note in [CAN physical layer](../physical-layer/can.md#clock-accuracy-a-crystal-is-mandatory).
