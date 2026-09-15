---
title: Sources
slug: sources
sidebar_position: 20
sidebar_label: Sources
description: Standards, application notes and datasheets used in this specification, with how to obtain each.
tags: [stepcan, references, standards]
---

# Sources

## Standards

### How to obtain the CiA documents

| Document | Version | Status | How to get it |
|---|---|---|---|
| **CiA 301** — CANopen application layer and communication profile | 4.2.0, 2011-02-21 | PAS | **Free** from [can-cia.org](https://www.can-cia.org/) after registering an account |
| **CiA 303-1** — Device and network design, Part 1: CANopen physical layer | 2.0.1, 2023-03-27 | TR | **Free** after registration |
| **CiA 303-3** — Device and network design, Part 3: CANopen indicators | 1.4.0, 2012-04-27 | TR | **Free** after registration |
| **CiA 401-1** — CANopen profile for I/O devices, Part 1: Generic I/O modules | 3.1.0, 2013-02-12 | PAS | **Free** after registration |
| **CiA 305** — CANopen layer setting services (LSS) and protocols | 3.0.0, 2013-05-08 | DSP | **Members only.** CiA does not sell it individually |
| CiA 401-B / -C / -F — I/O device profile, restructured | 1.0.0, 2023-06-19 | DSP | Members only |

Free CiA pages citable in place of paywalled text:

- [CANopen overview](https://www.can-cia.org/can-knowledge/canopen)
- [CANopen lower layers](https://www.can-cia.org/can-knowledge/canopen-lower-layers) — bit rates, the 87.5 % sample point, the bus and stub length table
- [CiA 303 series](https://www.can-cia.org/can-knowledge/cia-303-series-canopen-related-documents)
- [CiA 305 LSS](https://www.can-cia.org/can-knowledge/cia-305-layer-setting-services-lss)
- [CiA 401 series](https://www.can-cia.org/can-knowledge/cia-401-series-i/o-device-profile)
- [Technical documents index](https://www.can-cia.org/cia-groups/technical-documents) — the authoritative public list of versions, dates and access status

### ISO and IEC

| Document | Notes |
|---|---|
| **ISO 11898-1:2024** — Data link layer and physical signalling | Current edition. **ISO 11898-1:2015 is withdrawn.** Normative source for the bit-timing segments and both oscillator tolerance conditions |
| **ISO 11898-2:2016** — High-speed medium access unit | Clause 5.3 Table 2 (driver load ranges), clause 5.5 Table 10 (R_diff 12–100 kΩ), Table 12 (loop delay ≤255 ns), Table A.2 (common-mode range) |
| **EN IEC 61000-6-2:2019** — Generic immunity, industrial | Clause 3.5 defines *long distance line*; **Table 2** is signal/control ports |
| **EN 61326-1** — Measurement, control and laboratory equipment | Definition 3.6 and Table 2 footnotes c) and d) state the same rule in freely citable form |
| **EN/IEC 61131-2** — Programmable controllers | Edition 3 aligned its cable-length conditions to the 61000-6-x generic standards |
| IEC 61000-4-2 / -4-4 / -4-5 | ESD, EFT and surge immunity test methods |
| IEC 62305-3 | Tables 8 and 9 — the ≥16 mm² bonding conductor |
| IEC 62561-3 | Isolating spark gaps |
| EN 50310:2016 | Clause 10.2, telecommunications bonding; the ≤1.67 mΩ/m network criterion |
| EN 50325-4:2002 | CANopen as a European standard, derived from CiA 301 but not revised since 2002 |
| TIA/EIA-485-A | The actual source of "unit load" |
| TIA/EIA-568-B.2 | §4.3.4.3 mutual capacitance (advisory ≤5.6 nF/100 m), §4.3.4.4 capacitance unbalance (≤330 pF/100 m), §4.3.4.1 DC resistance |

## Papers and application notes

| Reference | Use in this specification |
|---|---|
| **Hartwich, F. & Bassemir, A., *The Configuration of the CAN Bit Timing*, Robert Bosch GmbH, iCC 1999.** [Kvaser mirror](https://kvaser.com/wp-content/uploads/2014/08/boschcia99-can-bit-timing.pdf) | The two oscillator tolerance conditions and the propagation segment formula |
| **NXP EB181** | States each tolerance condition's purpose, which the Bosch paper does not |
| TI SPNA090 | Corroborates the 10 and 13 bit-time constants |
| **TI SLLA279A** — *Critical Spacing of CAN Bus Connections* | The node-spacing criterion |
| TI SLLA271 — *Common Mode Chokes in CAN Networks* | Choke survey; recommends no specific value |
| **TI SLVA711** — *IEC 61000-4-x Tests for TI Protection Devices* | Test levels and the 42 Ω data-line source impedance |
| TI TIDUB36B | CAN bus protection reference design |
| TI TIDUAS1B, TI SLLA639 | RS-485 surge protection |
| **Renesas AN1978** | Series-resistor values for RS-485 surge coordination |
| Bourns, *Designing Effective Circuit Protection for RS-485/RS-422* | Cascade coordination |
| **ST AN6363** | STM32 internal oscillator accuracy — the basis of TR-5 |
| ABB, *Grounding and cabling of drive systems* | The 3.3 nF / 630 V hybrid bond |
| PROFIBUS/PROFINET, *Functional Bonding and Shielding* | Shield bonding, the ≤10 nF limit, the shield-current remedy hierarchy |
| Analog Devices MT-095 | The λ/20 electrical-length criterion |
| Schneider, *Electrical Installation Guide* | Shield bonding decision criteria, drawing on IEC 61000-5-2 |

## Datasheets

| Part | Function | Note |
|---|---|---|
| **SITCORES SIT1050Q** | CAN transceiver — recommended | 30 kΩ min R_diff, ±8 kV HBM, AEC-Q100 claim |
| SITCORES SIT1050 | CAN transceiver — existing stock only | **NRND; no ESD figure in any revision** |
| SITCORES SIT1051A / SIT1042 | CAN transceiver — **not suitable** | 19 kΩ min R_diff |
| SITCORES SIT65HVD230 | CAN transceiver, 3.3 V | No guaranteed minimum R_diff published |
| TI SN65HVD230 | CAN transceiver, 3.3 V | 40 kΩ min, but 16 pF fails the spacing rule at 64 nodes over 30 m |
| TI TCAN1042 / TCAN1044 | CAN transceiver | 30–40 kΩ, real IEC ESD ratings |
| NXP TJA1042 / TJA1051 / TJA1057 | CAN transceiver — **not suitable** | 19 kΩ min |
| NXP TJA1050 | CAN transceiver, legacy | ±4 kV HBM all pins — **not inheritable by clones** |
| **MaxLinear SP3490 / SP3491** | RS-485 transceiver | Full duplex, 8-pin; no with-load I_CC published |
| Bourns GDT35 series, 2036 series | Gas discharge tubes | 3-electrode, 90 V |
| Littelfuse SL1021A/B | Gas discharge tubes | The 3-electrode data-line family — **not CG3** |
| TDK ACT45B series | Common-mode chokes | 51 µH and 100 µH |
| Nexperia PESD2CANFD24V-T / 24L-T | CAN TVS | 6 pF and 9–10 pF |
| ST ESDCAN03-2BWY | CAN TVS | 3 pF |
| onsemi NUP2105L | CAN TVS — **not suitable** | 26–30 pF |
| Bourns CDSOT23-SM712, Littelfuse SM712 | RS-485 TVS | −7 V / +12 V asymmetric |

## Cable datasheets consulted for the 48 pF/m figure

Draka/Prysmian UC300 S24 Cat.5e F/UTP (48 pF/m) · Helukabel HELUKAT 200IND (47 pF/m) · Lapp ETHERLINE H Cat.5e (50 pF/m) · Leviton 21X-UTP5P (52 pF/m) · Belden 1583E (56 pF/m — the TIA ceiling restated) · Lapp UNITRONIC BUS CAN A (40 pF/m, 120 Ω) · Belden 3084A DeviceNet (39.4 pF/m, 120 Ω)

## Software

| Project | Licence | Use |
|---|---|---|
| **[CANopenNode](https://github.com/CANopenNode/CANopenNode)** | Apache 2.0 | Recommended stack. Implements CiA 301, 303 indicators, 304, **305 LSS** and 309-3; validated with the CiA Conformance Test Tool; vendored by Zephyr RTOS. Its [LSS documentation](https://canopennode.github.io/CANopenNode/group__CO__CANopen__305.html) is the best free substitute for the members-only CiA 305 text |
| Lely CANopen | Apache 2.0 | Mature C/C++ alternative; spec coverage not publicly enumerated |
| canopen (Python) | MIT | Good for tooling and test scripts; implements "the most common parts" of CiA 301 |
| CanFestival | LGPL | **Effectively unmaintained.** Legacy only |
