---
title: Protection
slug: protection
sidebar_position: 40
sidebar_label: Protection
description: The GDT to choke to TVS cascade, why the coordinating impedance is mandatory, earthing, shield bonding, and which stages to populate for each profile.
tags: [stepcan, emc, esd, surge, iec-61000]
---

# Protection design

Every stepCAN port is a connectorised, human-touchable interface on a cable that runs through a machine or a building past contactors, drives and mains wiring. The protection is a three-stage cascade, and the stages are not interchangeable.

## Threat model and the thresholds that decide applicability

| Threat | Basic standard | Level on signal ports | Criterion | **Applicability threshold** |
|---|---|---|---|---|
| RF conducted, common mode | IEC 61000-4-6 | 10 V, 150 kHz–80 MHz | A | applies |
| **EFT / burst** | IEC 61000-4-4 | **±1 kV**, 5/50 ns, capacitive clamp | B | **cables longer than 3 m** |
| **Surge, line-to-earth only** | IEC 61000-4-5 | **±1 kV**, 1.2/50 (8/20) | B | **long distance lines** — see below |

!!! danger "Two corrections to the claim everyone repeats"
    The common statement is *"EN 61000-6-2 requires the surge test on signal ports only above 30 m, in Table 3."* Both details are wrong and the second one will catch you.

    **It is Table 2, not Table 3.** Signal ports have been Table 2 in every edition since 1999; Table 3 is DC power ports.

    **The threshold is not simply 30 m.** EN IEC 61000-6-2:2019 defines the term in clause 3.5:

    > **long distance line** — *"line connected to a signal/control port and which inside a building is longer than 30 m, or which leaves the building (including a line installed outdoors)"*

    So a 12 m run to a garage is a long distance line and needs surge protection. A 29 m run inside one building does not. **Length is only half the test.**

    The identical concept appears, in freely citable form, in EN 61326-1 definition 3.6 and its Table 2 footnote c) — *"Only in the case of long-distance lines (see 3.6)."* If you need to quote the rule in a document, quote 61326-1.

!!! warning "The EFT threshold is 3 m, and it applies to every stepCAN installation"
    EFT arrives as common mode through a capacitive clamp, not by direct injection. That is why the **common-mode choke — not the TVS — is the effective EFT defence**, and why the choke is never optional on a cable port.

### What this means for stepCAN

- **EFT at ±1 kV applies unambiguously.** Every stepCAN bus is far over 3 m.
- **Surge is a knife-edge you control by what you declare.** A specification that says "up to 30 m" sits exactly at, not above, the boundary. That is a thin thread to hang a published product on, and a reviewer or notified body may not accept it.
- **The building-exit clause is what will actually catch you.** stepCAN is intended for machines *and buildings*; the moment a segment leaves a building it is a long distance line regardless of length.

A useful calibration: ABB's AC500 CI502 communication interface, declaring to EN 61131-2, states **1 kV line-to-ground surge on communication lines with no length caveat printed at all**. The market expectation for PLC-class hardware with a network port is that it survives 1 kV L-E, whatever the standard's applicability gate technically permits.

### Surge current, and why the GDT exists

The surge generator for data lines uses a **42 Ω source impedance**, so:

| Test level | Open-circuit voltage | Peak current |
|---|---|---|
| Level 2 | 1 kV | ~24 A |
| Level 3 | 2 kV | ~48 A |
| Level 4 | 4 kV | ~96 A |

**No SOT-23 TVS survives 48 A.** That is the entire reason the GDT stage exists.

## Cascade architecture

```
              ┌──── CM CHOKE ────┐
  RJ45  ──────┤                  ├──── TVS ────▶ transceiver
   pin        │  ACT45B-510-2P   │      │
              └──────────────────┘      │
     │              ▲                  ─┴─
     │              │                   ▼  logic 0V
     │         [Rs series]
     │
   ──┴──  GDT 3-electrode
     ▼    GDT35-09-S1-RP
    PE (chassis)   ← short, wide, ≥2 vias
```

Order from the connector inward: **GDT → series impedance → common-mode choke → TVS → transceiver.**

### Why the series element is not negotiable

A GDT is slow, and its firing voltage rises steeply with dV/dt: a 90 V GDT fires at 250 V at 100 V/µs but at **550 V at 1 kV/µs**. A TVS is fast. So in a surge the TVS clamps *first*, and during the microsecond before the GDT ionises the TVS alone stands between the generator and the transceiver — where it is destroyed.

The series impedance develops the extra drop that raises the node voltage to the GDT's impulse spark-over and hands the energy over. It also lets the GDT extinguish afterwards: a fired GDT has an arc voltage around 10 V and will otherwise sustain follow-current.

!!! danger "The common-mode choke cannot be the series element"
    Its leakage inductance (0.15–0.20 µH) develops only about **0.6 V** at an 8/20 µs surge's di/dt — three orders of magnitude short of what is needed. Its DC resistance is 1–2 Ω per line, a fifth of the required value. And its 150–200 mA current rating means it is deeply saturated at 24 A, so the inductance you were counting on is not there.

    **A separate pulse-proof resistor or a TBU is mandatory.**

    The choke stays in the design regardless: at EFT's 5 ns rise time the same 0.2 µH develops roughly **800 V**, which is precisely the stress it is there to block.

## Gas discharge tube

| Parameter | Recommended: **Bourns GDT35-09-S1-RP** | Alternative: Bourns 2036-09-SMLF |
|---|---|---|
| Electrodes | 3, single shared gas chamber | 3 |
| DC spark-over | 90 V ±20 % | 90 V ±20 % |
| Impulse spark-over @1 kV/µs | up to ~1200 V | 550 V |
| Surge current, 8/20 µs | 14 kA, 10 operations | 20 kA (1 op) / 10 kA (>10 ops) |
| **Capacitance** | **<0.7 pF** | <2 pF |
| Package | 5 × 7.2 mm SMD | 5 mm dia × 7.3 mm SMD |
| Price @1k | ~$0.88 | ~$1.00 |

**Why 90 V.** The criterion is not the signalling voltage but the highest legitimate voltage that may appear on the wire, plus the GDT's −20 % tolerance. ISO 11898-2 permits CAN common mode to ±12 V; RS-485 permits −7 V to +12 V. A 90 V part has a worst-case spark-over of 72 V, clear of every legitimate bus voltage and of the ±40 V bus-fault rating of the transceivers, while giving the lowest impulse let-through of the available range. **The same 90 V GDT serves both the CAN pair and the chain pair.**

!!! warning "A 3-electrode part is mandatory, not a convenience"
    Two separate 2-electrode GDTs fire independently, so one line can strike while the other has not, leaving a large residual **differential** voltage across the pair — exactly the transverse stress the transceiver cannot take. A 3-electrode device shares one gas chamber, so both electrodes strike together.

    Note also that Littelfuse's 3-electrode data-line family is **SL1021A/B**, not CG3 — CG3 is a 2-electrode high-voltage series and is the wrong part.

**Two practical consequences:**

- **Board area.** One GDT protects one pair. Both pairs means two GDTs at 5 × 7.2 mm each, plus keepout, next to the connector. On a small module this is the dominant reason to make the GDT a populate-option.
- **Hi-pot conflict.** A 90 V GDT fires during any insulation test above 72 V. A 500 V DC insulation test **will** arc it. Provide a test link, specify a reduced test voltage, or document the exclusion.

## Common-mode choke

| Part | L_CM | Z @10 MHz | I_rated | DCR/line | Stray L | Price @1k |
|---|---|---|---|---|---|---|
| **TDK ACT45B-510-2P** (CAN) | 51 µH | 2800 Ω typ | 200 mA | **1.0 Ω** | 0.15 µH | ~$0.74 |
| TDK ACT45B-101-2P (chain) | 100 µH | 5800 Ω typ | 150 mA | 2.0 Ω | 0.20 µH | ~$0.89 |
| Würth 744232101 | 105 µH | 6000 Ω | 90 mA | 5.5 Ω max | not specified | ~$1.04 |

**Use 51 µH on CAN, 100 µH on the chain.** The reasoning is asymmetric on purpose:

- On **CAN**, the choke's DCR is in series with a 60 Ω bus. The 51 µH part's 1.0 Ω/line adds 2 Ω differential, costing about 3 % of the dominant differential output; the 100 µH part costs 6 %. At 64 nodes the driver already works into 53 Ω against a 50 Ω floor, so 3 % is worth having. The 51 µH part also has the higher current rating.
- On the **chain**, the signal is a static level into a 21 kΩ bias load. A few ohms of DCR is irrelevant, so take the extra common-mode attenuation. A slow pair is the *best* candidate for a big choke, not the worst.

There is no bit-rate-dependent standard inductance for CAN. TI's own survey of sixteen chokes recommends no value and no part; 51–100 µH is simply the band the industry uses, and TDK markets the ACT45B specifically for CAN.

## TVS — capacitance is the deciding parameter

### CAN pair

| Part | Standoff | Peak 8/20 µs | **C per line** | Differential contribution | Verdict at 64 nodes |
|---|---|---|---|---|---|
| **Nexperia PESD2CANFD24V-T** | 24 V | 2.6 A | **5.2–6 pF** | ~3 pF | **Recommended** |
| ST ESDCAN03-2BWY | 24 V | 3.7 A | 3 pF | ~1.5 pF | Best capacitance |
| Nexperia PESD2CANFD24L-T | 24 V | 4 A | 9–10 pF | ~5 pF | Acceptable |
| Bourns CDSOT23-T24CAN | 24 V | 8 A | 11 pF line-line | 11 pF | Marginal |
| **onsemi NUP2105L** | 24 V | 350 W | **26–30 pF** | ~13 pF | **Do not use** |

### Chain pair

| Part | Standoff | Peak 8/20 µs | C per line | Verdict |
|---|---|---|---|---|
| **Bourns CDSOT23-SM712** | **−7 V / +12 V asymmetric** | 17 A, 400 W | 75 pF | **Recommended** |
| Littelfuse SM712-02HTG | −7 / +12 V | 19 A, 600 W | 75 pF | Equivalent, higher rating |
| TI TPD2E007 | ±13 V symmetric | 4.5 A | 10–15 pF | If the chain is ever made fast |

!!! warning "SM712 is right for the chain and wrong for CAN"
    The SM712's −7 V / +12 V window mirrors the TIA/EIA-485 common-mode range exactly — that is what it was built for. On CAN it fails twice over: ISO 11898-2 permits CAN_L down to −12 V, which an SM712 would clamp at −7 V, loading the bus and in a sustained fault cooking the TVS; and its 75 pF is twelve times the capacitance of a proper CAN part.

    The industry standoff for CAN is 24 V, and every credible CAN TVS uses it.

The 75 pF on the chain is affordable only because the chain is static — each protected node presents roughly seven RS-485 unit loads, but with two devices per segment that is 14 of 32, which is fine. If the chain is ever repurposed as a fast data channel, switch to TPD2E007 and accept its symmetric ±13 V window.

## Coordinating impedance

| Bus | Series resistance per line | Rationale |
|---|---|---|
| **Chain (RS-485)** | **10 Ω, 0.33 W pulse-proof** | The published recommendation is 10–20 Ω *"to minimise bus voltage attenuation during normal operation"*. A receiver needs 200 mV of the driver's 1.5 V; 20 Ω of added series resistance is invisible. |
| **CAN** | **≤4.7 Ω, ideally 2–3.3 Ω pulse-proof** | CAN has almost no headroom — see below |

**Why CAN cannot take 10 Ω.** ISO 11898-2 requires a dominant differential output of 1.5–3.0 V into 50–65 Ω. Two series resistors of R each add 2R to the load:

| Series R per line | Driver sees | Fraction of nominal V_diff | Verdict |
|---|---|---|---|
| 0 Ω | 60 Ω | 1.00× | — |
| 2 Ω (or choke DCR alone) | 64 Ω | 0.94× | Safe |
| 4.7 Ω | 69.4 Ω | 0.86× | Acceptable |
| **10 Ω** | 80 Ω | **0.75×** | **A 2.0 V driver falls to 1.5 V — at the limit** |
| 20 Ω | 100 Ω | 0.60× | Non-compliant |

With 64 nodes already loading the driver to 53 Ω, stack no more than 2–3.3 Ω on CAN. TI's own CAN protection reference design uses **zero** series resistance on the bus lines and a **Bourns TBU** overcurrent protector instead; if you need Level 4 surge on CAN, that is the route.

!!! note "Use pulse-proof resistors, not ordinary chip resistors"
    An ordinary 0603 arcs across its own body at 550 V. Specify anti-surge thick-film or wirewound.

## PE, earth and shield

!!! danger "TR-4 — GDT earth reference"
    The 3-electrode GDT's centre electrode shall connect to **PE / chassis earth**, never to the module's logic 0 V.

    A GDT referenced to logic ground turns a surge event into *"dump 10 kA through my own ground plane"*, which is materially worse than fitting no GDT at all. The path from the centre electrode to the chassis earth point must be **short and wide, with at least two vias** — high-frequency surge current follows the path of least inductance, not least resistance.

For a Class II or floating module with no PE, the GDT reference must be a bonded functional-earth point tied to the cable shield. A GDT with nowhere to discharge to is decoration.

### Shield bonding — bond both ends

The λ/20 criterion settles this unambiguously for stepCAN. A cable is electrically short when it is under 1/20 of a wavelength at the highest frequency of concern. At Cat5e's velocity factor, a 30 m cable equals λ/20 at about **330 kHz** — and the immunity tests run from 150 kHz to 1 GHz. **The cable is electrically long across virtually the entire test range**, so there is no frequency regime in which one-end bonding is defensible.

| Situation | Correct treatment |
|---|---|
| Both ends in the same equipotential bonding area | **Both ends, 360° through the connector. No capacitor.** |
| Same building, different equipotential areas | Both ends **plus a parallel earthing conductor (PEC)** run alongside the data cable |
| Ends in different buildings | **Fibre, or PEC plus SPDs to IEC 61643-21 / IEC 62305-4.** Not a capacitor. |
| Hazardous area, bonding not assured | One end solid plus **≤10 nF** at the other |

One-end-only bonding suppresses electric fields only and leaves you exposed to the magnetic coupling that actually causes EFT failures. Schneider's guidance, drawing on IEC 61000-5-2, calls it *"dangerous"* — an insulation fault can put a fatal voltage on the shield.

!!! note "If you must use a hybrid bond, 3.3–10 nF, Y2-rated"
    Published values: ABB specifies *"a high-frequency capacitor with a few nanofarads, for example, 3.3 nF / 630 V"*; the PROFIBUS/PROFINET guideline says *"max. 10 nF"*; Analog Devices suggests 10–100 nF for instrumentation.

    The spread is a real trade-off, not sloppiness: a larger capacitor is a better HF bond and passes more power-frequency leakage. For 64 nodes, 100 nF each would build 6.4 µF of distributed leakage path to earth. **Use 3.3–10 nF.**

    Use a **Y2 safety-rated** part, which fails open. A standard ceramic that fails short silently re-creates the ground loop the capacitor was there to break — and does so while a fault may be present.

!!! danger "A spark gap is not normally fitted in parallel with a hybrid-bonding capacitor"
    This is a common misconception, and it conflates two different techniques. A capacitor handles µA–mA and blocks 50 Hz; an isolating spark gap to IEC 62561-3 handles tens of kA for lightning equipotential bonding across a joint that must stay DC-isolated. Neither substitutes for the other.

    If the bus runs between buildings you have a lightning problem as well as an EMC problem, and the treatment is **SPDs to IEC 61643-21** under IEC 62305-4 — not a capacitor and not a bare spark gap. A capacitor-only hybrid bond on an inter-building run is a design error.

### Parallel earthing conductor

Where a PEC is required, **≥16 mm² Cu**, routed parallel to and touching the data cable to minimise the enclosed loop area.

!!! note "Cite the right reason for the 16 mm²"
    The figure comes from **lightning-current capability (IEC 62305-3, Tables 8 and 9)** and from **telecommunications bonding-bar sizing (EN 50310 clause 10.2)** — not from an EMC data-cabling rule. Writing "≥16 mm² because EMC" will be marked down by anyone who knows the source.

!!! danger "The cable shield shall never be used as the equipotential bonding conductor"
    Both the PROFIBUS manual (*"The shield of a bus cable should NEVER be used for potential equalization!"*, their capitals) and the EN 50310/EN 50174-2 practice guidance (*"the cable screen shall not be used as a parallel bonding conductor"*) are explicit.

### Circulating shield current

With 64 modules all bonding the shield at both ends you have 64 parallel earth paths. That is not 64 problems — adding parallel bonds *lowers* the impedance of the bonding network, which is what you want. The failure mode is not "too many bonds", it is "bonds into a poor common bonding network".

**Commissioning criterion.** Rather than publish a shield-current limit — the commonly cited 30 mA / 300 mA figures rest on a single secondary source with no standard behind them — use a **bond impedance** criterion, which is testable and has a standard behind it:

| Criterion | Source |
|---|---|
| ≤0.2 Ω at every bond | structured-cabling practice |
| DC resistance between any two points of the bonding network ≤1.67 mΩ/m | EN 50310:2016 |

Measurement is with a wide-bandwidth true-RMS clamp on the whole cable including the shield, with the bus running and idle. The diagnostic that matters is the **current division between the PEC and the shield**: if most of the current is in the shield rather than the PEC, the bonding network is the problem.

Remedies, in the order the PROFIBUS/PROFINET guideline gives them: improve the loop impedance of the bonding system; use a meshed network; add intermediate shield connections; verify the supply is TN-S, not TN-C; check for unintended multiple connections. **Fitting capacitors is not on that list.**

### Isolation

With CAN_GND carried in the cable and a 30 m run inside one building, the ±7 V RS-485 and ±12 V CAN common-mode ranges are ample and **galvanic isolation of the bus is not required**. Isolate at the I/O terminals, where the field-side energy is.

If a run crosses between separately-earthed buildings, isolate — and use fast digital isolators, not optocouplers, for the bit-timing reason in [CAN physical layer](./can.md#clock-accuracy-a-crystal-is-mandatory).

## Population options by profile

| Port type | GDT | Choke | TVS | Series R |
|---|---|---|---|---|
| Backplane board-to-board, internal | No | No | **Yes** | No |
| Backplane group's external RJ45 ports | **Yes** | **Yes** | **Yes** | **Yes** |
| Distributed RJ45, entirely inside one building | **Recommended** | **Yes** | **Yes** | **Yes** |
| **Distributed RJ45, cable can leave the building** | **Mandatory** | **Yes** | **Yes** | **Yes** |

- **Internal backplane contacts are never touched and never leave the enclosure.** A TVS is enough, and it is there for handling during assembly.
- **In-building, up to 30 m:** at 1 kV / 42 Ω the surge is ~24 A. A PESD2CANFD24L (4 A) or CDSOT23-T24CAN (8 A) plus the series resistor is marginal on its own; the GDT is cheap insurance against installer error and against someone extending the bus outdoors later without reading this document.
- **Leaving the building:** ±2 to ±4 kV means 48–96 A. No SOT-23 TVS exists with that rating. Consider the 10/700 µs telecom waveform too — it carries far more energy than 8/20 µs.

!!! tip "Design the PCB for the full cascade and make the GDT a do-not-populate option"
    A footprint costs nothing. A respin costs a month. Given that revision 0.1's "is the GDT mandatory?" question resolved to *"yes, for any port that can reach outside a building"*, the safe default for a product intended for buildings is to **populate it**.
