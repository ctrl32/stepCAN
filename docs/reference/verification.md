---
title: Verification status
slug: verification
sidebar_position: 10
sidebar_label: Verification
description: The eight items revision 0.1 left open, all now closed, with what each one turned out to be and which design decisions changed.
tags: [stepcan, verification, standards, emc]
---

# Verification status

Revision 0.1 listed eight items that could not be established from public sources. All eight have been researched. Three changed the design, three confirmed what was assumed, and two turned out to be genuinely unpublished — which is itself a finding, and each has a derivation or a substitution in place of the missing figure.

## Summary

| # | Item | Outcome | Design impact |
|---|---|---|---|
| 1 | EN 61000-6-2 cable-length threshold | **Closed — and the common version of the claim is wrong twice** | GDT becomes the default for building installations |
| 2 | Cat5e mutual capacitance | **Closed — 48 pF/m** | None; conclusion was insensitive to it |
| 3 | SIT1050 ESD rating | **Closed — genuinely unpublished, and the part is NRND** | Transceiver changed to SIT1050Q |
| 4 | SIT1051A / SIT1042 figures | **Closed — and they fail the requirement** | Revision 0.1's upgrade recommendation withdrawn |
| 5 | CiA normative texts | **Closed — three of four are free** | Documentation only |
| 6 | Bosch bit-timing paper | **Closed — and it produced a new mandatory requirement** | **Crystal now mandatory (TR-5)** |
| 7 | Hybrid shield-bonding capacitor | **Closed — 3.3–10 nF, and a premise was wrong** | Guidance corrected |
| 8 | SP3490 supply current with load | **Closed — unpublished, derived instead** | None |

---

## 1. The cable-length threshold for surge testing

**Status: closed.** The widely repeated claim — *"EN 61000-6-2 requires the 61000-4-5 surge test on signal ports only above 30 m, in Table 3"* — is substantially correct in intent and wrong in three details, one of which matters a great deal.

| Claim as usually stated | Verdict |
|---|---|
| Surge on signal ports is gated by cable length | **Correct in substance** |
| It is in Table 3 | **Wrong.** Signal ports are **Table 2** in every edition since 1999. Table 3 is DC power ports. |
| The threshold is 30 m of cable | **Incomplete.** It is *30 m inside a building **or** any line that leaves the building, at any length*. |
| *(usually omitted)* The EFT test has the same gate | **Wrong — EFT's threshold is 3 m** |

**Editions 1 and 2** put the condition in a numbered table footnote on the surge row:

> *"Applicable only to ports interfacing with cables whose total length according to the manufacturer's functional specification may exceed 30 m."*

Note the construction: *"according to the manufacturer's functional specification"*. The trigger is what you declare, not what an installer actually pulls.

**Edition 3 (2016) and EN IEC 61000-6-2:2019** moved it into a defined term in clause 3:

> **3.5 long distance line** — *"line connected to a signal/control port and which inside a building is longer than 30 m, or which leaves the building (including a line installed outdoors)"*

**EN 61326-1:2006** carries the identical concept in freely citable form — definition 3.6 (*"lines within a building which are longer than 30 m, or which leave the building"*) and Table 2 footnote c) (*"Only in the case of long-distance lines (see 3.6)."*), with the EFT footnote d) (*"Only in the case of lines > 3 m."*) alongside it.

**EN/IEC 61131-2** does not maintain an independent rule; its edition 3 change note records *"cable length aligned to generic standards"*.

!!! danger "What changed in the design"
    - **EFT at ±1 kV applies to every stepCAN installation.** The 3 m threshold is not close.
    - **The building-exit clause is what catches stepCAN**, not the length. A 12 m run to a garage is a long distance line; a 29 m run inside one building is not.
    - A specification that says "up to 30 m" sits exactly *at*, not above, the boundary — a thin thread for a published product.
    - **The GDT stage is therefore the default for any port that can be connected to a cable leaving the building**, and revision 0.1's "populate-option" framing has been strengthened accordingly.

**Not fully settled:** the footnote letter and its printed position in EN IEC 61000-6-2:2019 Table 2. Two fetches of the same mirror contradicted each other. The substance is confirmed three independent ways. To settle the letter, read **Clause 3.5 and Table 2 of BS EN IEC 61000-6-2:2019** directly.

**Also unsettled:** whether EN 61000-6-2:2005 still holds a live citation in the Official Journal. No EMC standard has been cited in the OJEU since 9 June 2022, and the generic immunity standard does not appear in Commission Implementing Decision (EU) 2019/1326. The free, authoritative answer is the European Commission's *Summary list of references of harmonised standards — EMC* spreadsheet. **Do not publish a harmonisation claim without checking it.** The lab-standard hedge is to test and declare against EN IEC 61000-6-2:2019 and note that the 2005 requirements are also met.

---

## 2. Cat5e mutual capacitance

**Status: closed. Use 48 pF/m nominal, 45 pF/m worst case.**

The only tier-1 published figure for a 24 AWG solid F/UTP part matching this specification is **Draka/Prysmian UC300 S24 Cat.5e F/UTP at 48 pF/m**, confirmed identically across three jacket variants and the U/UTP sibling. Manufacturers publishing genuine nominals span 47–52 pF/m; theory for a 100 Ω line at NVP 0.64–0.72 predicts 46.3–52.1 pF/m. The two agree closely.

!!! warning "Two traps in this figure"
    **56 pF/m is the TIA/EIA-568-B.2 §4.3.4.3 advisory ceiling restated, not a measurement.** Several vendors publish exactly 5.6 nF/100 m because that is the limit, not because they measured it.

    **The conservative direction is the lowest capacitance, not the highest.** In `d > 0.98 × C_L / C`, required spacing is *inversely* proportional to capacitance. Substituting the TIA maximum is the least conservative thing you could do — it is easy to get backwards.

**Design impact: none.** The break-even capacitance, at which the required spacing would equal the available spacing, is 29.8 pF/m. No twisted-pair copper cable of any category comes near that; TI's own floor for any medium is 35 pF/m. **The conclusion holds for any cable above 30 pF/m**, which is a stronger statement than picking a number.

A related finding did emerge: Cat5e 24 AWG at 93.8 mΩ/m per conductor exceeds CiA 303-1's recommended ≤70 mΩ/m, and its 0.205 mm² is below the recommended 0.25 mm². The consequence is quantified under [connectors and cable](../physical-layer/connectors-and-cable.md#the-dc-resistance-deviation): the far node sees 95.5 % of the bus differential voltage, 1.43 V at the ISO minimum against a 0.9 V threshold. Acceptable, and declared.

---

## 3. SIT1050 ESD rating

**Status: closed. There is none, in any revision — and the part is NRND.**

Four datasheet revisions were checked across five independent mirrors plus two distributor parametric databases:

| Revision | Date | ESD section? |
|---|---|---|
| REC V1.4 | 2019.02 | **None** |
| REC V1.6 | 2019.05 | **None** |
| REC V1.14 | 2022.01 | **None** |
| V1.16 (current) | — | not readable; Sekorm's parametric extract carries no ESD attribute |

No HBM, no IEC 61000-4-2, no MM, no CDM. Note also that **V1.6 — the revision most commonly circulated — is several revisions stale**; the numbering runs V1.6 < V1.11 < … < V1.16, and DIP8 was dropped after V1.6.

!!! danger "Do not assume it inherits the TJA1050's ±4 kV HBM"
    ESD performance is a property of the silicon and the pad clamp design, not of the pinout or the functional specification. Three arguments against inheritance:

    1. **The parametric evidence proves the dies differ** — SIT1050 R_diff 30–80 kΩ against TJA1050's 25/50/75 kΩ; C_in 13 pF against 7.5 pF; bus fault ±40 V against ±36 V.
    2. **SITCORES publishes ESD for its other parts** — SIT1050Q ±8 kV HBM, SIT1042 ±8 kV HBM / ±4 kV IEC, SIT65HVD230 ±15 kV HBM. The omission on SIT1050 is a deliberate silence about *this* part, not a house style.
    3. Quoting a figure with a footnote saying "assumed from the TJA1050" will be read by a reviewer as a sourced figure, and it is not one.

**Separately: SIT1050T is marked NRND** — *不推荐用于新设计*, not recommended for new designs — in the manufacturer's own distributor listing.

**Design impact: the transceiver changed.** **SIT1050Q** is the replacement: identical pinout, R_diff 30 kΩ minimum (still meets TR-1), **±8 kV HBM published**, an AEC-Q100 compliance claim, and not NRND. Its capacitance is higher (10 pF differential against 5 pF), which tightens the node-spacing budget from 1.60× to 1.19× at 64 nodes over 30 m — still passing, and the reason TR-6 now states a minimum module spacing explicitly.

---

## 4. SIT1051A and SIT1042 — verified, and they fail

**Status: closed, and revision 0.1's recommendation is withdrawn.**

Revision 0.1 recommended **SIT1051AT/3** as the upgrade path from SIT1050, on a first-pass figure of 45 kΩ minimum differential input resistance. **That figure was wrong.**

| Part | R_diff min / typ / max | At 64 nodes | Verdict |
|---|---|---|---|
| SIT1051A, SIT1051AQ, SIT1042 | **19 / 30 / 52 kΩ** | 49.9 Ω | **Fails the 50 Ω mandatory bound** |
| SIT1050, SIT1050Q | **30 / 60 / 90 kΩ** | 53.2 Ω | Passes |

19/30/52 kΩ is exactly the NXP TJA1051 and TJA1042 figure, which these parts clone, and it was confirmed in three separate documents. The 45 kΩ first-pass reading was almost certainly a table mis-parse.

!!! warning "The counter-intuitive part"
    The **older** TJA1050 / SIT1050 generation guarantees a *higher* minimum input impedance (30 kΩ) than the **newer** CAN-FD generation (19 kΩ). Upgrading to a newer, faster, better-specified transceiver makes the node-count budget *worse*.

    This is exactly why TR-1 is written as a number rather than a part number, and why it says *minimum*, not *typical*.

**A second trap in the same family: SIT1051AT/3 is not a TJA1050 drop-in.** Its pin 5 is a **VIO supply input**, not a Vref output. A board that leaves pin 5 floating will not work. Use plain SIT1051AT (pin 5 = n.c.) if you need a drop-in, or SIT1042T if you need pin 5 to do something useful (SPLIT). SIT1042 additionally changes pin 8 from S (silent) to STB (standby).

**Also evaluated: SIT65HVD230**, SITCORES' 3.3 V-native SN65HVD230 clone — 120 nodes claimed, ±15 kV HBM on the bus pins, and the only 3.3 V part in the family. It publishes **no guaranteed minimum R_diff**, only 40 kΩ typical, so it cannot satisfy TR-1 as written. TI's own SN65HVD230 does specify 40 kΩ minimum and passes on impedance — but **fails the node-spacing rule at 64 nodes over 30 m** on capacitance, needing a 33 m minimum bus length.

---

## 5. Obtaining the CiA normative texts

**Status: closed, and the news is better than expected.**

CiA's access policy: *"The documents with status 'Public Available Specification' (PAS) or 'Technical Report' (TR) can be downloaded free of charge by everyone after registration… The documents classified as 'Draft Specification' (DS) are only accessible for members; 'Draft Specification Proposal' (DSP) documents are also limited to members."*

| Document | Title | Version | Date | Status | Free? |
|---|---|---|---|---|---|
| **CiA 301** | CANopen application layer and communication profile | 4.2.0 | 2011-02-21 | PAS | **Yes** |
| **CiA 303-1** | Device and network design — Part 1: CANopen physical layer | 2.0.1 | 2023-03-27 | TR | **Yes** |
| **CiA 303-3** | Device and network design — Part 3: CANopen indicators | 1.4.0 | 2012-04-27 | TR | **Yes** |
| **CiA 401-1** | CANopen profile for I/O devices — Part 1: Generic I/O modules | 3.1.0 | 2013-02-12 | PAS | **Yes** |
| **CiA 305** | CANopen layer setting services (LSS) and protocols | 3.0.0 | 2013-05-08 | DSP | **Members only** |
| CiA 401-B / -C / -F | I/O device profile, restructured | 1.0.0 | 2023-06-19 | DSP | Members only |

**CiA does not sell specifications individually.** There is no per-document price; a DS or DSP is obtained through membership or not at all. Membership fees are not published — they appear only on the application form.

**Design impact: documentation only.** Three of the four documents this specification depends on are free. For CiA 305, the free substitute is **CANopenNode's LSS documentation**, from an implementation validated against the CiA CANopen Conformance Test Tool.

Two related corrections:

- **EN 50325-4:2002** is the European standard for CANopen and is derived from CiA 301 — but it has not been revised since 2002 and therefore predates CiA 301 v4.2.0 (2011) by nine years. **Cite CiA 301 normatively; mention EN 50325-4 as context only.** EN 50325-5:2010 is CANopen Safety (CiA 304). No EN or ISO equivalent exists for CiA 303, 305 or 401.
- **CiA 303-2 is withdrawn**; its content moved to CiA 890.

---

## 6. The Bosch bit-timing paper — and the requirement it produced

**Status: closed. This is the item that changed the design most.**

The paper — Hartwich & Bassemir, *The Configuration of the CAN Bit Timing*, Robert Bosch GmbH, iCC 1999 — is freely available from a stable Kvaser mirror.

Its two oscillator tolerance conditions, as printed:

```
Condition I:   df ≤ min(Phase_Seg1, Phase_Seg2) / (2 · (13 · bit_time − Phase_Seg2))
Condition II:  df ≤ SJW / (20 · bit_time)
```

!!! note "Cite ISO 11898-1 for the conditions, not the conference paper"
    Both conditions are **normative in ISO 11898-1**. Hartwich & Bassemir is the informative, explanatory reference. Note also that the paper *states* the conditions without deriving the constants 13 and 10 — for the rationale, NXP EB181 gives each condition an explicit purpose: Condition I to *"sample correctly the first bit after sending an active error flag"*, Condition II for *"correct synchronization in stuffed part of the bit stream"*.

    And ISO 11898-1:2015 is **withdrawn** — the current edition is **ISO 11898-1:2024**.

Applied to this specification's 16 Tq / 87.5 % configuration:

| SJW | Condition I | Condition II | Binding | Budget |
|---|---|---|---|---|
| 1 | 0.4854 % | 0.3125 % | II | ±3125 ppm |
| **2** | **0.4854 %** | 0.6250 % | **I** | **±4854 ppm** |

!!! danger "New requirement TR-5 — a quartz crystal is mandatory"
    The STM32 internal HSI RC oscillator is **±1 % at ambient and ±3 % over temperature** per ST AN6363 — 2× to 6× outside the budget. Ceramic resonators at ±0.3 % to ±0.5 % either fail outright or leave no margin for ageing.

    **A quartz crystal of ±100 ppm or better is now a requirement**, giving roughly 48× margin. SJW is specified as 2, which buys 55 % more headroom than SJW = 1 at zero cost and is the maximum legal value with PHASE_SEG2 = 2.

    Revision 0.1 did not state a clock requirement at all. A resonator-based module would have passed every bench test and failed intermittently in the field over temperature — the worst possible failure mode to discover after production.

**Also confirmed:** the 87.5 % sample point this specification uses is exactly CiA-conformant, and the derived 16 Tq segmentation is not a CiA table — CiA 301 publishes nominal bit time and a sample-point range only, never register values.

---

## 7. Hybrid shield bonding

**Status: closed, with one premise of the original question refuted.**

**Capacitor values, with attribution:**

| Source | Value | Voltage rating | Context |
|---|---|---|---|
| ABB, *Grounding and cabling of drive systems* | *"a few nanofarads, for example, 3.3 nF / 630 V"* | 630 V | Far end of control cable shields |
| PROFIBUS/PROFINET guideline, rec. P4 | *"max. 10 nF"* | not stated | Where equipotential bonding is not assured |
| Analog Devices MT-095 | 0.01–0.1 µF | not stated | Instrumentation hybrid ground |
| Danfoss | 100 nF | not stated | Fallback when a PEC is not feasible |

The spread is a real trade-off: a larger capacitor is a better HF bond and passes more power-frequency leakage. **For 64 nodes, use 3.3–10 nF** — 100 nF each would build 6.4 µF of distributed leakage path to earth. Use a **Y2 safety-rated** part, which fails open; a standard ceramic that fails short silently re-creates the ground loop the capacitor was there to break.

!!! warning "The spark-gap premise was wrong"
    Revision 0.1 assumed a GDT or spark gap is normally fitted in parallel with a hybrid-bonding capacitor. **No mainstream EMC guide, manufacturer application note or standard says this.** ABB, Danfoss, Analog Devices, PI and Armstrong all describe the capacitor alone.

    The idea probably comes from the **isolating spark gap** (IEC 62561-3), which is a different technique for a different problem: lightning equipotential bonding across a joint that must stay DC-isolated, handling tens of kA. A capacitor handles µA–mA and blocks 50 Hz. Neither substitutes for the other, and a capacitor-only hybrid bond on an inter-building run is a design error — that case needs SPDs to IEC 61643-21 under IEC 62305-4.

**The decision criterion, settled.** The λ/20 rule puts a 30 m Cat5e cable into the electrically-long regime above about **330 kHz**, and the immunity tests run from 150 kHz to 1 GHz. **There is no frequency regime of interest in which one-end bonding is defensible for stepCAN.** Bond both ends, 360°, and use a PEC where the ends are in different equipotential areas.

**The ≥16 mm² Cu PEC figure is confirmed** — from IEC 62305-3 Tables 8 and 9 and EN 50310 clause 10.2. Cite it for the right reason: it is sized for **lightning current and telecoms bonding-bar practice**, not by an EMC data-cabling rule.

**Circulating shield current:** no standard specifies an acceptable value. The commonly cited 30 mA / 300 mA figures rest on a single secondary source. A measurement study injecting 5–10 A into cable shields found *"power frequency currents in metallised foil or braid shields do not inductively couple significant noise into their internal conductors even at current levels which cause the cables to warm up"* — so shield current is a thermal and safety limit, not a signal-integrity one. **Use a bond-impedance criterion instead**: ≤0.2 Ω at every bond, and EN 50310's ≤1.67 mΩ/m across the bonding network. Both are testable at commissioning and have a standard behind them.

---

## 8. SP3490 supply current with load

**Status: closed. It is genuinely unpublished, and always has been.**

Both existing revisions were obtained and compared — MaxLinear 1.0.3 (2017) and the Exar-branded 1.0.2 (2010, converted from a Sipex original of 2002). **Neither has a Typical Performance Characteristics section at all.** All thirteen figures are test circuits and switching waveforms; there is no supply-current curve. The older revision has *less* data, not more — it lacks even the thermal table. The archive route is exhausted.

The complete set of supply-current specifications is three rows, all unloaded: SP3490 1000 µA typ / 2000 µA max; SP3491 1000/2000 µA with the driver enabled and 800/1500 µA with it disabled.

**Derived instead**, which is defensible because the load is a resistor network this specification defines:

| | Quiescent | + Load (worst case) | **Total** |
|---|---|---|---|
| Typical | 1.00 mA | 1.68 mA | 2.68 mA |
| **Maximum** | 2.00 mA | 1.68 mA | **≈ 3.7 mA** |

Full arithmetic under [chain supply budget](../physical-layer/chain.md#supply-budget). Junction rise is 0.85 °C — thermally negligible.

**Design impact: none**, beyond being able to state a number. The rack total is about 171 mA typical across 64 modules.

---

## Declared deviations from standards

| Deviation | From | Justification |
|---|---|---|
| Pins 4–5 carry a manufacturer-specific signal | CiA 303-1 (reserved pins) | No CiA statement permits or prohibits it; declared as compatible-but-not-conformant |
| 100 Ω cable terminated into 120 Ω | CiA 303-1 recommends 120 Ω cable | 9 % reflection, fully decayed before the sample point at 500 kbit/s over 30 m |
| 93.8 mΩ/m conductor resistance | CiA 303-1 recommends ≤70 mΩ/m | Costs 4.5 % of far-node differential voltage; 1.6× margin remains |
| 0.205 mm² conductor | CiA 303-1 recommends ≥0.25 mm² | Same analysis |
| Node-ID not stored in non-volatile memory | Common CANopen practice | Deliberate — this is what makes module replacement configuration-free |
| LSS Fastscan not implemented | CiA 305 optional service | The chain makes it unnecessary and it is ~500× slower |
| Conformance to CiA 401-B/-C not claimed | Current CiA 401 series | Those texts are members-only and were not available to the authors; conformance to CiA 401-1 v3.1.0 is claimed |

Note that **CiA 303-1 is a Technical Report** — recommendations, not normative requirements. Where this document treats a CiA 303-1 figure as a requirement, that is stepCAN's choice.

## Corrections to widely repeated claims

Six things are stated confidently across the internet, in datasheets and in application notes, and all six are wrong in ways that matter to this design.

1. **"ISO 11898-2 defines unit load; 1 UL = 32 nodes."** It does not. ISO 11898-2 has no unit-load concept and states no maximum node count. The term is from TIA/EIA-485-A. Design to R_diff and the 50 Ω driver load.
2. **"The minimum CAN driver load is 45 Ω."** 45 Ω is the lower bound of the *optional extended* range, with a relaxed output window. The mandatory minimum is **50 Ω**. Designing to 45 Ω makes 19 kΩ transceivers look acceptable at 64 nodes when they are not.
3. **"CiA 301 specifies TSEG1 = 13, TSEG2 = 2 for 500 kbit/s."** CiA 301 specifies nominal bit time and a sample-point range. The segmentation is the implementer's and depends on the controller clock.
4. **"The surge cable-length condition is in Table 3 of EN 61000-6-2."** It is Table 2. Table 3 is DC power ports.
5. **"A GDT is fitted in parallel with a hybrid-bonding capacitor."** It is not. That conflates a capacitive HF bond with an IEC 62561-3 isolating spark gap.
6. **"Littelfuse CG3 is the 3-electrode data-line GDT family."** CG3 is a 2-electrode high-voltage series. The data-line family is SL1021A/B.

## What remains partially settled

Nothing blocks the design, but four documentary points would benefit from a copy of the primary text:

| Item | What is unsettled | How to settle it |
|---|---|---|
| EN IEC 61000-6-2:2019 Table 2 | The footnote letter and its printed position | Read Clause 3.5 and Table 2 of the standard |
| OJEU harmonisation status | Whether EN 61000-6-2:2005 holds a live citation | The European Commission's free *Summary list — EMC* spreadsheet |
| ISO 11898-1:2024 | Clause numbers for bit timing and tolerance | Cite the standard without clause numbers, or buy it |
| CiA 401-B/-C | Whether they are behaviourally equivalent to CiA 401-1 v3.1.0 | Members-only; this specification does not claim equivalence |
