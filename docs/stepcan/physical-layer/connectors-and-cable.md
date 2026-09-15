---
title: Connectors and cable
slug: connectors-and-cable
sidebar_position: 10
sidebar_label: Connectors and cable
description: RJ45 pin assignment per CiA 303-1, the board-to-board backplane connector, Cat5e specifications and the deviations it introduces.
tags: [stepcan, cable, connector, cia-303]
---

# Connectors and cable

## RJ45 pinout — distributed profile

The pinout follows **CiA 303-1**, the CANopen cabling recommendation, with the two reserved pins carrying the stepCAN chain.

| Pin | T568 pair | Signal | Direction | Notes |
|---|---|---|---|---|
| 1 | 2 | **CAN_H** | bidirectional | Bus, passes through the module |
| 2 | 2 | **CAN_L** | bidirectional | Bus, passes through the module |
| 3 | 3 | **CAN_GND** | — | Bus reference, passes through |
| 4 | **1** | **CHAIN_A** | IN: receiver / OUT: driver | Chain, **does not pass through** |
| 5 | **1** | **CHAIN_B** | IN: receiver / OUT: driver | Chain, **does not pass through** |
| 6 | 3 | CAN_SHLD (optional) | — | Shield bond, passes through |
| 7 | 4 | 0 V | — | Optional auxiliary supply return |
| 8 | 4 | +24 V (CAN_V+) | — | Optional auxiliary supply, 18–30 V DC |

Pins 4 and 5 are the only pair in CiA 303-1 where **both** conductors are unassigned, and they form a native twisted pair in both T568A and T568B. CiA 303-1 states the intended pairing explicitly: *"The pin 3-6 and 1-2 are twisted pairs."*

!!! warning "Conformance caveat — put this in your product documentation"
    CiA 303-1 says nothing about permitted alternative use of reserved pins, and "reserved" in CiA usage normally means reserved for future CiA assignment, not free for vendor use. A stepCAN port is therefore **CiA 303-1-compatible but not CiA 303-1-conformant**.

    A strictly conformant CANopen device plugged into a stepCAN port will leave pins 4–5 unconnected and interoperate normally on the CAN pair; it simply will not participate in the chain, and the head will see the rack as ending there. Suggested wording:

    > Pins 4 and 5, reserved in CiA 303-1, carry a manufacturer-specific auxiliary signal. Devices conforming strictly to CiA 303-1 leave these pins unconnected and interoperate normally on the CAN pair.

### Version dependency

CiA 303-1 has been revised and the pin 7 assignment changed:

| Version | Date | Pin 7 |
|---|---|---|
| v1.0 (DR-303-1) | 1999-10-10 | CAN_GND |
| v1.9.0 | 2017-09-18 | Optional separate ground, distinct from CAN_GND on pin 3 |
| **v2.0.1 (current)** | **2023-03-27** | — |

This specification follows the v1.9.0 assignment and treats pin 7 as an optional 0 V return. Cite the version you conform to.

!!! note "CiA 303-1 is a Technical Report, not a normative specification"
    Its status is TR — *recommendations*. Where this document treats a CiA 303-1 figure as a requirement, that is stepCAN's choice, not an obligation the CiA document imposes. It is also **free to download** from can-cia.org after registering an account.

## Board-to-board connector — backplane profile

An 8-way 2 mm or 2.54 mm pitch header and socket pair, or a card-edge connector, carrying the same eight signals in the same order.

| Requirement | Reason |
|---|---|
| Ground and supply contacts mate **first**, signals last | Hot-swap: no signal contact before the module's ground reference exists |
| Contact resistance ≤10 mΩ | CiA 303-1 specifies 2.5–10 mΩ for CANopen connectors |
| Keying or asymmetry preventing reversed mating | A reversed module shorts +24 V onto the chain pair |
| Current rating ≥2 A on pins 7/8 | Group supply passes through every module |

## Cable

**Specified cable: Cat5e F/UTP (overall foil shield), solid conductor, 24 AWG.**

| Property | Cat5e F/UTP | CiA 303-1 recommendation | Assessment |
|---|---|---|---|
| Characteristic impedance | 100 Ω ±15 % | 120 Ω | **Deviation — analysed below** |
| Mutual capacitance | **48 pF/m nominal**, 45–56 pF/m range | not specified | Higher than CAN cable's 39–40 pF/m; this *helps* the node-spacing rule |
| Conductor cross-section | 0.205 mm² (24 AWG) | 0.25–0.34 mm² for 0–40 m | Thinner |
| Loop resistance | 187.6 mΩ/m (93.8 mΩ/m per conductor) | ≤70 mΩ/m for 0–40 m | **Deviation — analysed below** |
| Propagation delay | 4.98 ns/m at NVP 0.67 | 5 ns/m assumed | Matches almost exactly |
| Pairs available | 4 | — | Exactly what stepCAN needs |

### Where the 48 pF/m comes from

Datasheets publish three different capacitance specifications and they are not interchangeable. Getting the wrong one is the most common error in this calculation.

| Specification | What it measures | Typical Cat5e value | Use in the node-spacing rule? |
|---|---|---|---|
| **Mutual capacitance** | Conductor to conductor within one pair, at 1 kHz | 47–56 pF/m | **Yes — this is the one** |
| Capacitance unbalance to ground | Differential imbalance of the two conductors' capacitance to ground | ≤330 pF/100 m (TIA limit) | No — a balance metric, ~1/300th the magnitude |
| Capacitance to shield | Each conductor to the foil | rarely published | No — common mode |

Manufacturer figures for 24 AWG solid Cat5e:

| Manufacturer | Part | Construction | Mutual capacitance |
|---|---|---|---|
| **Draka / Prysmian** | UC300 S24 Cat.5e F/UTP | F/UTP, solid | **48 pF/m** |
| Draka / Prysmian | UC300 S24 Cat.5e U/UTP | U/UTP, solid | 48 pF/m |
| Helukabel | HELUKAT 200IND | SF/UTP, stranded | 47 pF/m |
| Lapp | ETHERLINE H Cat.5e | SF/UTP, solid | 50 pF/m |
| Leviton | 21X-UTP5P | U/UTP, solid | 52 pF/m |
| Belden | 1583E | U/UTP, solid | 56 pF/m — *this is the TIA ceiling restated* |

Theory agrees: for a lossless 100 Ω line, `C = 1/(Z₀ × NVP × c)` gives 46.3 pF/m at NVP 0.72 and 52.1 pF/m at NVP 0.64, bracketing every genuine measured nominal.

!!! danger "56 pF/m is a specification ceiling, not a typical value"
    TIA/EIA-568-B.2 §4.3.4.3 states that mutual capacitance *"should not exceed 5.6 nF per 100 m"* for category 5e — note "should", it is advisory. Any datasheet reading exactly 56 pF/m is restating the limit, not reporting a measurement. Several vendors do exactly this.

    Worse, in the node-spacing formula the required spacing is **inversely** proportional to capacitance, so 56 pF/m is the *least* conservative number you could substitute. Use 48 pF/m nominal and 45 pF/m for worst case.

**F/UTP and U/UTP of the same design have the same mutual capacitance** — Draka publishes 48 pF/m identically for the F/UTP-PE, F/UTP-PVC and U/UTP-PVC variants of one product. The foil is a common-mode screen and barely couples to the differential mode. This matters because several manufacturers publish mutual capacitance only for their U/UTP parts.

### The 100 Ω impedance deviation

Cat5e is a 100 Ω medium terminated into 120 Ω. The reflection coefficient is:

```
Γ = |(120 − 100) / (120 + 100)| ≈ 0.09
```

About 9 % of the incident amplitude returns at each terminator. At 500 kbit/s the bit period is 2000 ns and the round trip on 30 m is 300 ns, so any reflection has long decayed before the sample point at 1750 ns. The mismatch is real and worth documenting, but it is not a functional risk at this bit rate and length. It would become one at 1 Mbit/s over 30 m — one more reason this specification targets 500 kbit/s.

!!! warning "Do not 'fix' the mismatch by lowering the termination"
    This is counter-intuitive and it is the single easiest way to break a 64-node bus.

    | Termination | Driver load at 64 nodes | Verdict |
    |---|---|---|
    | 2 × 120 Ω | 53.2 Ω | **Compliant** |
    | 2 × 110 Ω | 49.2 Ω | Below the ISO 50 Ω minimum |
    | 2 × 100 Ω | 45.2 Ω | Extended range only; non-compliant at normal |

    At low node counts the impedance match would win the argument. At 64 nodes the node loading does. **Termination stays at 120 Ω.**

### The DC resistance deviation

Cat5e 24 AWG solid is 93.8 mΩ/m per conductor, against CiA 303-1's recommended ≤70 mΩ/m for a 0–40 m bus, and its 0.205 mm² is below the recommended 0.25 mm². The deviation is real. Its consequence is not large:

```
cable loop resistance, 30 m  = 2 × 30 × 0.0938  = 5.63 Ω
far terminator branch        = 120 + 5.63       = 125.63 Ω
driver load (64 nodes)       = 120 ∥ 125.63 ∥ 469 = 54.3 Ω
far-node differential        = V_bus × 120 / 125.63 = 0.955 × V_bus
```

So the far node sees **95.5 %** of the bus differential voltage. At the ISO minimum dominant output of 1.5 V that is **1.43 V**, against a receiver dominant threshold of 0.9 V — a margin of 1.6×. At a typical 2.0 V driver it is 1.91 V, a margin of 2.1×.

The deviation costs 4.5 % of the differential voltage and is acceptable. Note also that adding the cable resistance slightly *raises* the driver load, from 53.2 Ω to 54.3 Ω, which is in the helpful direction.

!!! note "Do not parallel conductors across pairs to reduce the resistance"
    It is tempting to press a spare pair into service by paralleling a second conductor onto CAN_H and CAN_L. Do not. It destroys the twisted-pair geometry that gives the pair its impedance and its common-mode rejection, and the cure is far worse than the 4.5 % it treats. The legitimate uses for the spare pairs are the chain, the 0 V reference and auxiliary power.

### Comparison with dedicated 120 Ω CAN cable

| Parameter | Cat5e F/UTP 24 AWG | Lapp UNITRONIC BUS CAN A | Belden 3084A DeviceNet |
|---|---|---|---|
| Impedance | 100 Ω ±15 % | 120 Ω ±15 % | 120 Ω |
| Mutual capacitance | 48 pF/m | 40 pF/m | 39.4 pF/m |
| Conductor | 0.205 mm² solid | 0.22 mm² | 24 AWG stranded |
| Loop resistance over 30 m | 5.63 Ω | 5.58 Ω | 5.51 Ω |
| Propagation delay over 30 m | 149 ns | 132 ns | 134 ns |
| Availability in building wiring | universal | specialist | specialist |
| Cost | baseline | 3–4× | 3–4× |

**The only thing genuinely lost is the impedance match.** Loop resistance is equal, propagation delay differs by 17 ns on a 2000 ns bit, and capacitance being higher helps rather than hurts. For a machine or a house, Cat5e's availability, cost, connector ecosystem and installer familiarity are worth more than 9 % of reflection.

## Straight-through wiring and chain polarity

Modules are connected with **standard straight-through (non-crossover) patch cable**. Pin *n* of the OUT connector reaches pin *n* of the next module's IN connector. This works naturally for the chain because the two ports are wired to different halves of one transceiver:

| Connector | Pin 4 | Pin 5 | Connected to |
|---|---|---|---|
| **IN** (upstream) | CHAIN_A | CHAIN_B | SP3490 receiver inputs **A** and **B** |
| **OUT** (downstream) | CHAIN_A | CHAIN_B | SP3490 driver outputs **Y** and **Z** |

So module *n*'s **Y** meets module *n+1*'s **A**, and **Z** meets **B**. Pin 4 always carries the non-inverting conductor.

!!! tip "Fit a polarity swap option on the first prototype"
    Getting Y/Z and A/B the wrong way round is the most likely bring-up error in this design, and it fails in a way that looks like a firmware bug: the chain works exactly inverted. Two 0 Ω links in a swap pattern on the OUT pair let you correct it with a soldering iron instead of a board respin.
