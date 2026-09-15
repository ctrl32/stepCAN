---
title: Commissioning and test
slug: commissioning
sidebar_position: 40
sidebar_label: Commissioning
description: The bring-up checklist, the measurements that validate the node-count analysis, and the diagnostics the head should expose.
tags: [stepcan, test, commissioning]
---

# Commissioning and test

## Bring-up checklist

| # | Step | Expected | If it fails |
|---|---|---|---|
| 1 | Power one module alone, no cable | UNCONFIGURED, CHAIN LED off, bus silent | If the bus is not silent, the module is booting with a stored node-ID — it must not |
| 2 | Measure CHAIN_IN receiver output with no cable | Reads **not enabled** | Bias network wrong or reversed (TR-3) |
| 3 | Check the CAN clock frequency at a test point | Within ±100 ppm of nominal | A resonator or the internal RC has been fitted — see TR-5 |
| 4 | Head + one module | Enumerates to 1 node, CHAIN LED on | If the chain never asserts, check Y/Z–A/B polarity |
| 5 | Head + two modules | Enumerates to 2 in the right order | If it enumerates in reverse, the OUT pair polarity is swapped |
| 6 | Swap the two modules physically | Same node-IDs, different serial numbers per position | Confirms position-based addressing works |
| 7 | Measure termination | 60 Ω across CAN_H/CAN_L, powered and enumerated | If 120 Ω, auto-termination did not fire; if 40 Ω, a stray terminator is populated |
| 8 | Populate the full rack | Enumeration completes in under 1 s | If it stops at position *n*, the fault is in link *n*→*n+1* |
| 9 | Pull a middle module while running | Downstream modules go UNCONFIGURED; head re-enumerates | Confirms the hot-swap cascade |

## Measurements worth taking on the prototype

| Measurement | Instrument | Target |
|---|---|---|
| **Dominant differential voltage at the far node, full rack** | Scope, differential probe | **≥1.5 V** — this directly validates the node-count analysis |
| CAN eye diagram at the far node | Scope, persistence | Clean crossing at 87.5 % of the bit |
| **Edge rise time, full rack vs two nodes** | Scope | Degradation quantifies the capacitance budget |
| **Actual module spacing on cable** | Tape measure | ≥0.5 m between adjacent modules (TR-6) |
| CAN clock accuracy over the operating temperature range | Frequency counter, thermal chamber | Within ±100 ppm including drift |
| Cable mutual capacitance | LCR meter, 1 kHz, on a 30 m sample | Confirms the 48 pF/m assumption for your actual cable |
| CAN error counters at each node after 24 h | Firmware readout | 0 |
| **Bond impedance at each shield termination** | Milliohm meter | ≤0.2 Ω |
| Shield circulating current | Wide-bandwidth true-RMS clamp | Current predominantly in the PEC, not the shield |
| Enumeration time, full rack | Firmware timestamp | <1 s |

!!! tip "The first row is the one that matters most"
    With 64 nodes the calculation predicts a 54.3 Ω driver load and a far-node differential of 95.5 % of the bus voltage. Measuring it on a fully populated rack is the single test that validates the whole node-count analysis. If it comes in below 1.5 V, something in the transceiver selection or the protection network is not what the BOM says it is.

## Diagnostics the head should expose

- **Rack map**: position, node-ID, product code, serial number, NMT state, heartbeat age
- **Enumeration result** and the position where it stopped, if it stopped early
- **Per-node CAN error counters** and last EMCY
- **Configuration comparison** result against the stored reference
- **Chain state per node** (0x5FFD) — invaluable when a link is intermittent
