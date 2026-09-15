---
title: stepCAN
slug: stepcan
sidebar_position: 1
sidebar_label: stepCAN
description: A CANopen I/O bus whose node addresses come from the hardware, for DIN-rail backplanes and distributed installations alike.
tags: [stepcan, canopen, fieldbus]
---

# stepCAN

**Revision 0.2 · 2026-09-15**

A modular I/O bus that derives every node's address from the hardware itself.

One CAN segment carries the data. Alongside it runs a one-directional RS-485 **chain** carrying a single static level — *you are next* — from each module to its immediate physical neighbour. That one wire pair is what turns an unordered network into an ordered rack, in under a second, with no address switches and no commissioning step.

The same electrical design serves two mechanical arrangements: modules packed onto a DIN-rail backplane, and modules spread across a machine or a building.

## Design envelope

| Parameter | Value |
|---|---|
| Modules per segment | 64 |
| CAN bit rate | 500 kbit/s |
| Bus length | 25–30 m in the distributed profile (there is a *minimum*, see [CAN physical layer](./physical-layer/can.md)) |
| Protocol | CANopen — CiA 301 v4.2.0, CiA 305 v3.0.0, CiA 401-1 v3.1.0 |
| Chain channel | RS-485, static level, one direction |
| Cable | Cat5e F/UTP, 4 pairs |
| Connector | RJ45 per CiA 303-1, or board-to-board |
| Enumeration time | under 1 second for 64 modules |
| Clock source | quartz crystal, ±100 ppm or better — **mandatory** |

## Where to start

```cards
- title: Introduction
  href: ./overview/introduction.md
  icon: rocket
  text: What stepCAN is, what it is for, and how the document is put together.
- title: Architecture
  href: ./overview/architecture.md
  icon: sitemap
  text: The two channels, and why the chain exists rather than address switches or LSS Fastscan.
- title: Deployment profiles
  href: ./overview/deployment-profiles.md
  icon: layer-group
  text: DIN-rail backplane, distributed across a machine or house, and the mix of the two.
- title: CAN physical layer
  href: ./physical-layer/can.md
  icon: wave-square
  text: Transceiver selection for 64 nodes, bus loading, stub budget, bit timing and clock accuracy.
- title: Enumeration
  href: ./protocol/enumeration.md
  icon: list-ol
  text: The core mechanism — how 64 modules get ordered node-IDs in under a second.
- title: Verification status
  href: ./reference/verification.md
  icon: clipboard-check
  text: Every open item from revision 0.1, now closed, with the sources that closed it.
```

## What changed in revision 0.2

Eight items that revision 0.1 left open have been researched and closed. Three of them changed the design:

!!! warning "Design changes in this revision"
    - **A quartz crystal is now mandatory** and the STM32 internal RC oscillator is prohibited as a CAN clock source. The bit timing of this specification allows ±4854 ppm of total clock error; the internal oscillator is 2× to 6× outside that. See [CAN physical layer](./physical-layer/can.md).
    - **SIT1050T is NRND** — not recommended for new designs — and publishes no ESD figure in any datasheet revision. **SIT1050Q** replaces it: same pinout, same guaranteed 30 kΩ differential input resistance, a published ±8 kV HBM rating and an AEC-Q100 claim.
    - **SIT1051A and SIT1042 fail the 64-node loading requirement.** Their differential input resistance is 19 kΩ minimum, not the 45 kΩ a first research pass reported. Revision 0.1 recommended SIT1051AT/3 as an upgrade path; that recommendation is withdrawn.

Full detail, with the sources, is in [Verification status](./reference/verification.md).
