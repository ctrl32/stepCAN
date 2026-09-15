---
title: Introduction
slug: introduction
sidebar_position: 10
sidebar_label: Introduction
description: What stepCAN is, the design goals it serves, its operating envelope, and how evidence is tagged in this document.
tags: [stepcan, canopen]
---

# Introduction

## What stepCAN is

stepCAN is a bus for modular, processor-per-module I/O. Every module carries its own MCU, runs a full CANopen stack, and talks to a head station over a single CAN segment. What distinguishes stepCAN from plain CANopen is a second, deliberately trivial channel running alongside the CAN pair: a one-directional RS-485 **chain** that carries nothing but the statement *"you are next"* from each module to its immediate physical neighbour.

That one wire pair is what turns an unordered CAN network into an ordered rack. On power-up the head walks the chain module by module, and because exactly one module is listening at any instant, it can hand out CANopen node-IDs in strict physical order with no address switches, no serial-number scanning and no commissioning step.

The name is literal: addresses are assigned *step by step* along the chain.

## Design goals

| Goal | How stepCAN meets it |
|---|---|
| Node address = physical position | Hardware chain, resolved at every power-up |
| No DIP switches, no rotary encoders, no commissioning tool | Enumeration is automatic and takes under a second for 64 modules |
| Replacing a failed module needs no configuration | Node-ID is never stored in the module; it is re-derived on each boot |
| Wrong module in the wrong slot is detected, not silently accepted | Head stores a `position → identity` map and compares it at every start |
| Works both densely packed and spread out | One electrical design, two mechanical profiles |
| Standards-based where it costs nothing | CANopen application layer, CiA 303-1 connector |
| Fully protected ports | Three-stage GDT → choke → TVS cascade |

## Design envelope

| Parameter | Value |
|---|---|
| Maximum modules per segment | 64 |
| CAN bit rate | 500 kbit/s |
| Maximum bus length | 30 m |
| **Minimum bus length, distributed profile** | **64 × 0.4 m ≈ 25 m at full node count** — see the node-spacing rule in [CAN physical layer](../physical-layer/can.md) |
| Protocol | CANopen — CiA 301 v4.2.0, CiA 305 v3.0.0 LSS, CiA 401-1 v3.1.0 device profile |
| Chain channel | RS-485, static level, one direction |
| Cable | Cat5e F/UTP, 4 pairs, 24 AWG solid |
| Connector | RJ45 per CiA 303-1 v2.0.1, or board-to-board in the backplane profile |
| Module supply | 24 V DC, locally or over the cable |
| CAN clock source | Quartz crystal, ±100 ppm or better — mandatory |

The minimum-length entry is the counter-intuitive one, and it is real. It is explained under [bus capacitance and node spacing](../physical-layer/can.md#bus-capacitance-and-node-spacing).

## How to read the evidence in this document

Every electrical figure falls into one of three categories, kept visually distinct:

- **Quoted** — verbatim from a standard or a manufacturer's datasheet. The source is in [Sources](../reference/sources.md).
- **Derived** — calculated here from quoted figures. The calculation is always shown.
- **Open** — could not be established from public sources. As of revision 0.2 there are none left at the top level; [Verification status](../reference/verification.md) records what each one turned out to be and what remains partially settled.

## Corrections to widely repeated claims

Six things are stated confidently across the internet, in datasheets and in application notes, and all six are wrong in ways that matter to this design. They are collected in [Verification status](../reference/verification.md#corrections-to-widely-repeated-claims); the three that change component selection are:

!!! danger "These will mislead you if you trust them"
    1. **ISO 11898-2 does not define "unit load" and states no maximum node count.** That is RS-485 terminology borrowed into CAN marketing. The real constraint is differential input resistance.
    2. **The minimum CAN driver load is 50 Ω, not 45 Ω.** 45 Ω is the lower bound of the *optional extended* range. Designing to 45 Ω makes 19 kΩ transceivers look acceptable at 64 nodes when they are not.
    3. **The cable-length condition for surge testing is in Table 2 of EN 61000-6-2, not Table 3.** Table 3 is DC power ports. It has been Table 2 since 1999.
