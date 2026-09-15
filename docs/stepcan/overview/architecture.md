---
title: System architecture
slug: architecture
sidebar_position: 20
sidebar_label: Architecture
description: Roles, the two channels, why a hardware chain beats every alternative addressing method, and the topology rules.
tags: [stepcan, architecture, canopen]
---

# System architecture

## Roles

| Role | Count | Function |
|---|---|---|
| **Head** | 1 | CANopen NMT master, LSS manager, chain originator, process-image owner. Sits at one physical end of the bus. |
| **Module** | 1…64 | CANopen slave (CiA 401-1 device profile), chain repeater. Every module is identical in its bus interface; only the I/O section differs. |
| **Terminator** | 1 | Physically the last module, which enables its own CAN termination on command from the head. |

There is no separate backplane controller, no bus coupler and no address plug. A module is a module wherever it sits.

## The two channels

```
                 ┌──────────────────────────────────────────────┐
                 │  CAN  —  common, multi-master, all nodes      │
                 │         process data, configuration, NMT      │
                 └──────────────────────────────────────────────┘
   ┌──────┐         ┌────────┐      ┌────────┐      ┌────────┐
   │ HEAD │═════════│ MOD 1  │══════│ MOD 2  │══════│ MOD n  │
   └──────┘         └────────┘      └────────┘      └────────┘
        │  chain ───────▶ │  chain ─────▶ │  chain ────▶ │
        └─────────────────┴───────────────┴──────────────┘
          RS-485, point-to-point, one direction, static level
```

**CAN** is the real bus: one electrical segment, every node on it, arbitration sorting out who talks. It carries everything that is actually data.

**The chain** is not a bus. It is a series of independent point-to-point RS-485 links, each spanning exactly one gap between physical neighbours, each carrying a single static logic level in one direction. There is no protocol on it, no framing, no baud rate — just a level that means *enabled* or *not enabled*.

That asymmetry is the whole design. The chain is topological information the CAN bus cannot possibly have, delivered in the simplest form that survives 30 m of cable in a machine.

## Why a chain, and not one of the alternatives

Every modular I/O system has to solve the same problem: a node must learn which physical position it occupies. There are five known approaches and they are not equivalent.

| Method | Gives physical order? | Extra HW per node | Works distributed? | Commissioning step? | Verdict |
|---|---|---|---|---|---|
| **DIP / rotary address switches** | No — gives whatever the installer set | Switch | Yes | Yes, manual and error-prone | Rejected. The failure mode — two nodes at one address — is silent and awful. |
| **Slot-coded backplane pins** | Yes | None | **No** | No | Rejected. Requires a non-uniform backplane and a fixed rack size; impossible in the distributed profile. |
| **Resistor ladder per slot** | Yes | ADC input | No | No | Rejected. Tolerance stack limits it to ~8–16 positions. |
| **CANopen LSS Fastscan** | **No** — a node-ID per serial number, in arbitrary order | None | Yes | Yes, mapping serial → position | Rejected as the primary mechanism. See the timing below. |
| **stepCAN chain** | **Yes** | 1 transceiver + 2 resistors | **Yes** | **No** | Adopted. |

### The LSS Fastscan comparison, with numbers

Fastscan deserves a number, because it is the option a standards-first design reaches for. It is a binary search over the 128-bit LSS address — vendor-ID, product code, revision, serial number. CiA's own published figure is roughly 128 requests per device, and with a 30 ms timeout per request that is **approximately 4 seconds per device — about 4 minutes for 64 modules** — and it still does not tell you which module is physically where.

With the chain, exactly one module is ever in LSS configuration state, so the search collapses to nothing: the head sends *Switch State Global → Configuration*, *Configure Node-ID*, and moves on. On the same 500 kbit/s bus that is **about 8 ms per module, under 1 second for 64 modules**, and the result is ordered by construction.

!!! tip "The chain is roughly 500× faster than Fastscan, and answers a question Fastscan cannot"
    That ratio is the entire justification for the extra wire pair. It also buys three things Fastscan has no mechanism for.

- **Break localisation.** If enumeration stops at module 17, the fault is in the link between 17 and 18. No other method tells you that.
- **Free auto-termination.** The module that finds nothing downstream is by definition the last one.
- **Hot-swap awareness.** Pulling a module drops the chain for everything downstream, which those modules detect themselves without being told.

## Topology rules

1. The bus is a **single linear segment**. No stars, no spurs, no rings.
2. The head is at one end. The chain originates there.
3. Every module has exactly one upstream port (IN) and one downstream port (OUT).
4. The CAN pair passes straight through every module; the chain does not.
5. Termination is at the head and at the last module, nowhere else.
6. An empty position breaks the chain, and this is intended: the head reports the rack as ending there.

!!! warning "A star or hub topology fails the stub budget immediately"
    At 500 kbit/s CiA allows 27.5 m of *accumulated* stub across the whole bus. Divided across 64 nodes that is 430 mm each. Drop cables from a trunk spend that budget in the first few nodes. Rule 1 is not a stylistic preference.
