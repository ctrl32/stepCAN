---
title: Reference BOM
slug: bom
sidebar_position: 50
sidebar_label: BOM
description: Bus interface bill of materials per module, with indicative pricing and second-source notes.
tags: [stepcan, bom, components]
---

# Reference BOM

Bus interface section only, per module. Prices are indicative 1000-off figures gathered 2026-09-15, not quotations.

| Ref | Part | Function | Qty | ~$ each |
|---|---|---|---|---|
| U1 | **SIT1050Q** | CAN transceiver, 30 kΩ min R_diff, ±8 kV HBM | 1 | 0.35 |
| U2 | **SP3490** | Chain transceiver, full duplex, 8-pin | 1 | 0.60 |
| X1 | **Crystal, ±100 ppm, 8/16 MHz** | **CAN clock — mandatory, TR-5** | 1 | 0.25 |
| J1, J2 | RJ45 shielded, or 8-way B2B | IN / OUT ports | 2 | 0.80 |
| GDT1, GDT2 | **Bourns GDT35-09-S1-RP** | 3-electrode GDT, 90 V | 2 | 0.88 |
| L1, L3 | **TDK ACT45B-510-2P** | CM choke, CAN, 51 µH | 2 | 0.74 |
| L2, L4 | TDK ACT45B-101-2P | CM choke, chain, 100 µH | 2 | 0.89 |
| D1, D2 | **Nexperia PESD2CANFD24V-T** | CAN TVS, 6 pF | 2 | 0.15 |
| D3, D4 | **Bourns CDSOT23-SM712** | Chain TVS, −7/+12 V | 2 | 0.40 |
| R1, R2 | 4.7 kΩ 1 % | Chain fail-safe bias | 2 | 0.01 |
| Rs1–Rs4 | 10 Ω pulse-proof | Chain coordinating impedance | 4 | 0.03 |
| Rs5, Rs6 | 3.3 Ω pulse-proof | CAN coordinating impedance | 2 | 0.03 |
| RT1 | 120 Ω 1 % 0.25 W | CAN termination (switched) | 1 | 0.02 |
| RT2 | 120 Ω 1 % | Chain termination — **footprint only, DNP** | 1 | — |
| Q1 | Logic-level N-MOSFET | Termination switch | 1 | 0.08 |
| | | **Bus interface subtotal** | | **≈ $9.45** |

Variants:

| Variant | Omits | Subtotal |
|---|---|---|
| Full cascade, cable may leave the building | — | ≈ $9.45 |
| In-building only, GDT not populated | 2 × GDT | ≈ $7.70 |
| Backplane-only module, TVS only | GDT, chokes, series R | ≈ $3.55 |

## Component notes

!!! warning "Availability — second-source everything before committing"
    At the time of research: **PESD2CANFD24V-TR** showed zero stock with 2027 delivery dates at one major distributor; **CDSOT23-SM712** had stock but a 24-week factory lead time; **NUP2105LT1G** a 52-week lead time. Keep the alternates in [Protection](../physical-layer/protection.md) on the drawing.

!!! danger "Parts that must not be substituted casually"
    - **The CAN transceiver** — substituting a 19 kΩ part (TJA1042, SIT1042, SIT1051A) breaks the 64-node bus. TR-1 is the rule; check the *minimum*, not the typical.
    - **The CAN TVS** — substituting NUP2105L breaks the node-spacing budget. Check the capacitance.
    - **The crystal** — substituting a ceramic resonator breaks the clock-tolerance budget and fails only in the field, over temperature.

    These three substitutions all look harmless on a BOM review and all produce intermittent field failures rather than bench failures. They are the ones to flag in a change-control process.

## Alternates

| Position | First choice | Alternate | Note |
|---|---|---|---|
| CAN transceiver | SIT1050Q | SIT1050 (existing stock only) | SIT1050T is NRND and publishes no ESD rating |
| | | TCAN1042V, TCAN1044V | 30–40 kΩ, real IEC ESD ratings, needs a 5 V rail |
| | | ~~SN65HVD230~~ | **Not at 64 nodes over 30 m** — capacitance fails the spacing rule |
| CAN TVS | PESD2CANFD24V-T | ESDCAN03-2BWY (3 pF) | Lowest capacitance; 3.7 A rating |
| | | PESD2CANFD24L-T (9–10 pF) | Higher surge rating, tighter spacing budget |
| Chain TVS | CDSOT23-SM712 | Littelfuse SM712-02HTG | Equivalent, higher peak rating |
| GDT | Bourns GDT35-09-S1-RP | Bourns 2036-09-SMLF | Specify the **-SMLF** RoHS variant; 2036-15-SM is obsolete |
| | | Littelfuse SL1021A090 | **Not CG3** — that is a 2-electrode high-voltage family |
| CM choke | TDK ACT45B-510-2P | Würth 744232101 | Higher DCR (5.5 Ω) — only acceptable on the chain pair |
