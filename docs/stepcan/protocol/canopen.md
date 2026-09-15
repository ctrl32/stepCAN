---
title: CANopen application layer
slug: canopen
sidebar_position: 20
sidebar_label: CANopen layer
description: Node-ID allocation and its priority consequence, the object dictionary, PDO strategy for 64 nodes, heartbeat, NMT, fail-safe outputs and bus load.
tags: [stepcan, canopen, pdo, cia-401]
---

# CANopen application layer

## Node-ID allocation and the priority consequence

Node-IDs are assigned **1…64 in chain order**, position 1 nearest the head. This has a consequence that must be a deliberate decision, not an accident.

In the predefined connection set a PDO's COB-ID is `base + node-ID`, and CAN arbitration is won by the *lowest* COB-ID. So **node 1's TPDO1 (0x181) always beats node 64's TPDO1 (0x1C0)**. Priority is strictly physical position.

!!! tip "Put latency-critical modules nearest the head"
    Safety-adjacent inputs, fast counters, anything feeding a control loop. If that conflicts with the mechanical layout, remap COB-IDs explicitly via object 0x1800:01 rather than accepting the default.

## Predefined connection set

| Object | COB-ID | Range at 64 nodes | Configured by |
|---|---|---|---|
| NMT node control | **0x000** (broadcast) | — | not configurable |
| SYNC | **0x080** | — | 0x1005 |
| EMCY | **0x080 + node-ID** | 0x081–0x0C0 | 0x1014 |
| TIME stamp | 0x100 | — | 0x1012 |
| **TPDO1** (module → head) | **0x180 + node-ID** | 0x181–0x1C0 | 0x1800:01 |
| **RPDO1** (head → module) | **0x200 + node-ID** | 0x201–0x240 | 0x1400:01 |
| TPDO2 | 0x280 + node-ID | 0x281–0x2C0 | 0x1801:01 |
| RPDO2 | 0x300 + node-ID | 0x301–0x340 | 0x1401:01 |
| TPDO3 / RPDO3 | 0x380 / 0x400 + node-ID | — | 0x1802 / 0x1402 |
| TPDO4 / RPDO4 | 0x480 / 0x500 + node-ID | — | 0x1803 / 0x1403 |
| SDO server → client | 0x580 + node-ID | 0x581–0x5C0 | 0x1200:02 |
| SDO client → server | 0x600 + node-ID | 0x601–0x640 | 0x1200:01 |
| Heartbeat / boot-up | **0x700 + node-ID** | 0x701–0x740 | not configurable |
| LSS | 0x7E4 / 0x7E5 | — | not configurable |

With 64 nodes using TPDO1 and RPDO1 only, the used ranges are 0x181–0x1C0 and 0x201–0x240 — comfortably inside the standard allocation, with TPDO2–4 free for high-density modules.

## Object dictionary

### Mandatory communication objects

| Index | Name | Type | Notes |
|---|---|---|---|
| 0x1000 | Device type | UNSIGNED32 | 0x000F0191 for a CiA 401 digital I/O device |
| 0x1001 | Error register | UNSIGNED8 | Mandatory |
| 0x1005 | COB-ID SYNC | UNSIGNED32 | 0x80 |
| 0x1014 | COB-ID EMCY | UNSIGNED32 | 0x80 + node-ID |
| **0x1016** | **Consumer heartbeat time** | ARRAY U32 | Bits 23–16 = monitored node-ID, bits 15–0 = time in ms |
| **0x1017** | **Producer heartbeat time** | UNSIGNED16 | ms, 0 = disabled |
| **0x1018** | **Identity** | RECORD | sub1 vendor-ID, sub2 product code, sub3 revision, sub4 serial — **this is the LSS address and the basis of configuration verification** |
| 0x1019 | SYNC counter overflow | UNSIGNED8 | Required for SYNC staggering |
| 0x1010 | Store parameters | ARRAY U32 | I/O configuration only, never the node-ID |
| 0x1400… / 0x1600… | RPDO communication / mapping | RECORD / ARRAY U32 | |
| 0x1800… / 0x1A00… | TPDO communication / mapping | RECORD / ARRAY U32 | |

### CiA 401 device profile objects

| Index | Name | Type | Access |
|---|---|---|---|
| **0x6000** | **Read digital input 8-bit** | ARRAY of UNSIGNED8 | RO |
| 0x6002 | Polarity digital input 8-bit | UNSIGNED8 | RW |
| 0x6003 | Filter constant digital input 8-bit | UNSIGNED8 | RW |
| 0x6005 | Global interrupt enable digital | BOOLEAN | RW |
| 0x6006 | Interrupt mask, any change | UNSIGNED8 | RW |
| 0x6007 / 0x6008 | Interrupt mask low→high / high→low | UNSIGNED8 | RW |
| **0x6200** | **Write digital output 8-bit** | ARRAY of UNSIGNED8 | RW |
| 0x6202 | Change polarity digital output | UNSIGNED8 | RW |
| **0x6206** | **Error mode digital output 8-bit** | UNSIGNED8 | RW |
| **0x6207** | **Error value digital output 8-bit** | UNSIGNED8 | RW |
| **0x6401** | **Read analogue input 16-bit** | ARRAY of INTEGER16 | RO |
| **0x6411** | **Write analogue output 16-bit** | ARRAY of INTEGER16 | RW |
| 0x6421 | Analogue input interrupt trigger selection | UNSIGNED8 | RW |
| 0x6424 / 0x6425 | Interrupt upper / lower limit | INTEGER32 | RW |
| 0x6426 | Interrupt delta | UNSIGNED32 | RW |
| 0x6443 / 0x6444 | Analogue output error mode / value | — | RW |

0x6000 and 0x6200 are **arrays indexed by sub-index**: sub 1 = channels 1–8, sub 2 = channels 9–16, and so on. A 32-channel module uses sub-indices 1–4.

!!! note "Which CiA 401 to reference"
    This specification references **CiA 401-1 v3.1.0** (2013-02-12), a Publicly Available Specification, free from can-cia.org after registration.

    CiA has since restructured the profile into **CiA 401-B** (functional behaviour and parameters), **CiA 401-C** (mapping to classic CANopen) and **CiA 401-F** (mapping to CANopen FD), all v1.0.0, 2023-06-19 — all Draft Specification Proposals available to CiA members only. Devices conforming to this specification conform to CiA 401-1; alignment with 401-B/-C is intended but is not claimed, since those texts were not available.

### Manufacturer-specific

0x5FFF, 0x5FFE, 0x5FFD and 0x5FFC, defined under [Enumeration](./enumeration.md#manufacturer-specific-objects).

## PDO strategy — this decides whether the bus works

A frame-time calculation first. At 500 kbit/s, a standard 11-bit-ID frame:

| Data bytes | Nominal | Worst case with bit stuffing |
|---|---|---|
| 0 | 47 bit = 94 µs | 55 bit = 110 µs |
| 2 | 63 bit = 126 µs | 75 bit = 150 µs |
| 8 | 111 bit = 222 µs | 135 bit = **270 µs** |

The naive design — every module sends one 8-byte TPDO on every SYNC:

| Modules | SYNC | Inputs | Packed outputs | **Cycle** |
|---|---|---|---|---|
| 8 | 0.11 ms | 2.16 ms | 0.54 ms | **2.8 ms** |
| 16 | 0.11 ms | 4.32 ms | 1.08 ms | **5.5 ms** |
| 32 | 0.11 ms | 8.64 ms | 2.16 ms | **10.9 ms** |
| **64** | 0.11 ms | **17.28 ms** | 4.32 ms | **21.7 ms** |

!!! warning "21.7 ms at 100 % bus load is not acceptable for most machine I/O"
    Three mechanisms fix it, and a real stepCAN system uses all three. The 21.7 ms figure then becomes what it should be — a worst case, seen only when everything changes at once.

### (a) Event-driven inputs

Transmission type **255** (device-profile-specific event — for digital inputs, a change of state) with:

| Parameter | Object | Value | Effect |
|---|---|---|---|
| Transmission type | 0x1800:02 | 255 | Transmit on change |
| **Inhibit time** | 0x1800:03 | **10** (= 1 ms; units of 100 µs) | Rate-limits a chattering input so one broken sensor cannot flood the bus |
| **Event timer** | 0x1800:05 | **200** (ms) | Periodic refresh even without a change; acts as a data watchdog |

Digital inputs in a real machine do not all change at once. Typical bus load collapses to a few percent and change-to-frame latency drops below 1 ms.

### (b) Packed outputs

A 16-channel digital output module consumes 2 bytes. Four modules' output images fit in one 8-byte frame, so the head can drive 64 modules with 16 frames instead of 64 — but only if the RPDO COB-IDs are remapped so four modules share one frame at different byte offsets. Standard CANopen PDO mapping; just not the default.

### (c) SYNC staggering

Where deterministic sampling is needed, use synchronous transmission types with **0x1800:06 (SYNC start value)** together with **0x1019 (SYNC counter overflow)**:

| Group | Modules | SYNC start value | Transmission type |
|---|---|---|---|
| A | 1–16 | 1 | 4 (every 4th SYNC) |
| B | 17–32 | 2 | 4 |
| C | 33–48 | 3 | 4 |
| D | 49–64 | 4 | 4 |

With a 2 ms SYNC period each group transmits every 8 ms, but only 16 modules burst at once (4.3 ms), so the bus never saturates and jitter is bounded.

!!! danger "Do not use transmission types 252 or 253"
    They are RTR-based, deprecated, unsupported by many modern controllers, and absent from CAN FD entirely.

### Recommended default configuration

| | Inputs (TPDO1) | Outputs (RPDO1) |
|---|---|---|
| Transmission type | 255 (event) | 255 (apply immediately) |
| Inhibit time | 1 ms | — |
| Event timer | 200 ms | 200 ms (as RPDO watchdog) |
| Mapping | 0x60000108 … | 0x62000108 … |

## Heartbeat

Use **heartbeat (0x1016 / 0x1017)**, not the legacy node guarding (0x100C / 0x100D). Node guarding is RTR-based and costs two frames per node per period — 128 frames per period at 64 nodes. Heartbeat costs one.

| Parameter | Value | Justification |
|---|---|---|
| Producer time (0x1017), all modules | **500 ms** | 64 frames / 500 ms = 128 frames/s ≈ **1.4 % bus load** |
| Consumer time (0x1016) at the head | **1000 ms** | 2× producer, tolerates one lost frame |

Heartbeat frame: COB-ID `0x700 + node-ID`, one data byte = NMT state (0x00 boot-up, 0x04 Stopped, 0x05 Operational, 0x7F Pre-operational).

## NMT

| CS | Command | Resulting state |
|---|---|---|
| 0x01 | Start remote node | Operational |
| 0x02 | Stop remote node | Stopped |
| 0x80 | Enter pre-operational | Pre-operational |
| 0x81 | Reset node | Initialisation (application + communication) |
| **0x82** | **Reset communication** | Initialisation (communication only) — **required after LSS Configure Node-ID** |

NMT frame: COB-ID **0x000**, two bytes — command specifier, then node-ID (0 = broadcast). It is the highest-priority ID on the bus, which is exactly what you want for an emergency stop.

| State | Services available |
|---|---|
| Initialisation | none; sends boot-up on exit |
| Pre-operational | SDO, EMCY, SYNC, heartbeat, LSS — **no PDOs** |
| Operational | everything |
| Stopped | NMT and heartbeat only |

## Fail-safe outputs

This matters more in a distributed installation than in a backplane, because a cable can be cut.

Every output module shall be configured with **0x6206 (error mode)** and **0x6207 (error value)** so that on loss of communication — RPDO event-timer expiry, NMT Stopped, or bus-off — outputs go to a defined state rather than freezing at their last value.

| 0x6206 bit | Meaning |
|---|---|
| 0 | Output holds its last value on error |
| 1 | Output takes the corresponding bit of 0x6207 |

!!! info "stepCAN default: 0x6206 = 0xFF, 0x6207 = 0x00"
    Every output de-energises on communication loss. Change this per channel only with a documented reason.

## EMCY

Emergency objects on `0x080 + node-ID` report module-level faults. stepCAN-specific codes in the manufacturer area:

| Code | Meaning |
|---|---|
| 0xFF01 | Chain input lost while Operational |
| 0xFF02 | Chain output driver fault |
| 0xFF03 | I/O supply out of range |
| 0xFF04 | Output short circuit / overload |
| 0xFF05 | Module over-temperature |

## Bus load summary

| Traffic | Frames/s at 64 nodes | Load at 500 kbit/s |
|---|---|---|
| Heartbeat, 500 ms | 128 | 1.4 % |
| Event-driven inputs, typical machine | ~200 | ~2.2 % |
| Packed outputs, 20 ms refresh | 800 | ~8 % |
| SYNC, 2 ms | 500 | ~2.8 % |
| **Typical total** | | **~15 %** |
| Worst case, everything changes at once | | 100 % for ~22 ms |

A healthy CAN bus sits below 50 % average load; 15 % leaves room for SDO traffic during configuration and for burst events.
