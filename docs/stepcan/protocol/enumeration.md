---
title: Enumeration
slug: enumeration
sidebar_position: 10
sidebar_label: Enumeration
description: How the chain turns 64 modules into ordered CANopen node-IDs in under a second, with the LSS command sequence, state machines, timing and error handling.
tags: [stepcan, lss, cia-305, enumeration]
---

# Enumeration — the core of stepCAN

## Principle

At power-up every module is unconfigured: CANopen node-ID **0xFF**, chain OUT **not asserted**. An unconfigured CANopen node sends no boot-up message, no heartbeat, no EMCY and no PDO, and has no SDO server — it answers only LSS. So the bus is silent, and exactly one module, the first, has its chain IN asserted by the head.

The head then walks the rack. Because the chain guarantees that **exactly one module is addressable at any instant**, the head never has to distinguish between candidates — no Fastscan, no serial-number search, no collision handling.

## Sequence

```
 HEAD                                        MODULE n
   │
   │  assert CHAIN_OUT ──────────────────────▶ CHAIN_IN goes high
   │                                           (debounce 1 ms)
   │
   │  LSS Switch State Global → CONFIGURATION
   │  0x7E5: 04 01 00 00 00 00 00 00 ────────▶ enters config state
   │                                           (only this module is listening)
   │
   │  LSS Inquire Identity — vendor-ID
   │  0x7E5: 5A .. ──────────────────────────▶
   │  ◀────────────────────────── 0x7E4: 5A <vendor-ID>
   │      … repeat for 5B product, 5C revision, 5D serial
   │
   │  LSS Configure Node-ID = n
   │  0x7E5: 11 <n> 00 00 00 00 00 00 ───────▶
   │  ◀────────────────────────── 0x7E4: 11 00 (success)
   │
   │  ── NOTE: do NOT send Store Configuration (0x17) ──
   │
   │  LSS Switch State Global → WAITING
   │  0x7E5: 04 00 00 00 00 00 00 00 ────────▶ leaves config state
   │
   │  NMT Reset Communication
   │  0x000: 82 <n> ─────────────────────────▶ node-ID becomes active
   │  ◀──────────── 0x700+n: 00 (boot-up)
   │
   │  SDO write 0x5FFF:00 = 1  (assert chain)
   │  0x600+n: 2F FF 5F 00 01 .. ────────────▶ drives CHAIN_OUT high
   │  ◀────────────────────────── 0x580+n: 60 ..
   │
   │  ── repeat for module n+1 ──
   │
   │  … no response after 3 attempts → end of rack
   │
   │  SDO write 0x5FFE:00 = 1  (enable termination) to the last module
   │
   │  NMT Start Remote Node (broadcast)
   │  0x000: 01 00 ──────────────────────────▶ all modules → Operational
```

!!! danger "Two details that break naive implementations"
    **A node-ID assigned by LSS is *pending* until an NMT Reset Communication.** It does not take effect immediately. Omitting the reset is the most common LSS implementation bug.

    **Do not send LSS Store Configuration (0x17).** The node-ID must not survive a power cycle. Re-deriving it from the chain on every boot is exactly what makes a module swap configuration-free.

## LSS commands used

All LSS traffic is 8 data bytes on two COB-IDs: **0x7E5 manager → server**, **0x7E4 server → manager**. Byte 0 is the command specifier.

| CS | Service | Direction | Data | Used by stepCAN |
|---|---|---|---|---|
| 0x04 | Switch state global | M→S | b1 = 0 waiting, 1 configuration | **Yes** |
| 0x11 | Configure node-ID | M→S | b1 = new node-ID (1–127) | **Yes** |
| 0x11 | …response | S→M | b1 = error code, b2 = specific | **Yes** |
| 0x17 | Store configuration | M→S | — | **Deliberately not used** |
| 0x5A | Inquire identity — vendor-ID | both | resp. b1–4 | **Yes** |
| 0x5B | Inquire identity — product code | both | resp. b1–4 | **Yes** |
| 0x5C | Inquire identity — revision | both | resp. b1–4 | **Yes** |
| 0x5D | Inquire identity — serial number | both | resp. b1–4 | **Yes** |
| 0x5E | Inquire node-ID | both | resp. b1 | Diagnostics only |
| 0x40–0x44 | Switch state selective | both | 32-bit LSS address parts | Not needed — the chain selects |
| 0x51 | Fastscan | M→S | binary search | **Not used** |
| 0x13 / 0x15 | Configure / activate bit timing | M→S | table index (500 kbit/s = index 2) | Optional |

**Configure node-ID error codes** (byte 1 of the 0x11 response):

| Code | Meaning |
|---|---|
| 0x00 | Success |
| 0x01 | Node-ID out of range |
| 0xFF | Manufacturer-specific, detail in byte 2 |

The four Inquire Identity values map directly onto object **0x1018** sub-indices 1–4, which is what the head stores for configuration verification.

!!! note "CiA 305 is the one document you cannot download free"
    CiA 301, CiA 303-1, CiA 303-3 and CiA 401-1 are Publicly Available Specifications or Technical Reports, free from can-cia.org after registering an account. **CiA 305 v3.0.0 is a Draft Specification Proposal, available to CiA members only, and CiA does not sell it individually.**

    The best free substitute is CANopenNode's LSS module documentation, from an implementation validated against the CiA CANopen Conformance Test Tool: <https://canopennode.github.io/CANopenNode/group__CO__CANopen__305.html>

## Module state machine

```
        power-up
           │
           ▼
   ┌───────────────┐
   │  UNCONFIGURED │  node-ID 0xFF, CHAIN_OUT = 0, termination OFF
   │               │  CAN RX enabled, LSS only
   └───────┬───────┘
           │ CHAIN_IN asserted (debounced 1 ms)
           ▼
   ┌───────────────┐
   │  ADDRESSABLE  │  responds to LSS Switch State Global → configuration
   └───────┬───────┘
           │ Configure Node-ID + NMT Reset Communication
           ▼
   ┌───────────────┐
   │  CONFIGURED   │  boot-up sent, node-ID active, Pre-operational
   └───────┬───────┘
           │ SDO 0x5FFF:00 = 1
           ▼
   ┌───────────────┐
   │   CHAINED     │  CHAIN_OUT = 1, successor now addressable
   └───────┬───────┘
           │ NMT Start Remote Node
           ▼
   ┌───────────────┐
   │  OPERATIONAL  │  PDOs active, heartbeat running
   └───────┬───────┘
           │ CHAIN_IN deasserted (debounced)
           ▼
       back to UNCONFIGURED  ── and CHAIN_OUT = 0, cascading downstream
```

The last transition is the whole hot-swap story, and it is free: a module that loses its upstream enable drops its own enable, so everything downstream of a removed module returns to UNCONFIGURED without the head having to tell them anything.

## Manufacturer-specific objects

Two objects in the manufacturer-specific area (0x2000–0x5FFF) drive the chain and the terminator.

| Index | Sub | Name | Type | Access | Meaning |
|---|---|---|---|---|---|
| **0x5FFF** | 00 | Chain output control | UNSIGNED8 | RW | 0 = CHAIN_OUT low, 1 = CHAIN_OUT high |
| **0x5FFE** | 00 | Termination control | UNSIGNED8 | RW | 0 = termination off, 1 = 120 Ω enabled |
| 0x5FFD | 00 | Chain input state | UNSIGNED8 | RO | Debounced CHAIN_IN level — diagnostics |
| 0x5FFC | 00 | Assigned position | UNSIGNED8 | RO | Position in the chain as enumerated |

## Timing budget

Per module, at 500 kbit/s:

| Step | Frames | Time |
|---|---|---|
| LSS Switch State Global → configuration | 1 | 0.11 ms |
| Inquire identity ×4 (request + response) | 8 | 0.88 ms |
| Configure node-ID + response | 2 | 0.22 ms |
| Switch State Global → waiting | 1 | 0.11 ms |
| NMT Reset Communication | 1 | 0.11 ms |
| Module reset and boot-up | 1 | ~3 ms (firmware dependent) |
| SDO write 0x5FFF + response | 2 | 0.22 ms |
| Chain settle + debounce | — | 3 ms |
| **Total per module** | **17** | **≈8 ms** |

**64 modules ≈ 0.5 s**, plus one end-of-rack timeout.

Against LSS Fastscan's roughly 4 s per device — about 4 minutes for 64 modules, with no physical order at the end of it — the chain is about 500× faster and answers a better question.

## Configuration verification

After enumeration the head holds a table:

| Position | Vendor-ID | Product code | Revision | Serial number |
|---|---|---|---|---|
| 1 | 0x00000AB1 | 0x00010004 | 0x00010000 | 0x0004C11E |
| 2 | 0x00000AB1 | 0x00020008 | 0x00010000 | 0x0004C2A7 |
| … | | | | |

It compares this against a stored reference and reports precisely:

| Condition | Report |
|---|---|
| Identical | Configuration OK |
| Same vendor + product, different serial | **Module replaced in position n** — informational, accept |
| Different product code | **Wrong module type in position n** — fault, do not go Operational |
| Fewer modules than reference | **Module missing from position n** — fault |
| More modules | **Unexpected module at position n** — warning, ask the operator |

This is what a real PLC does at startup, and the chain gives it to you without any addressing infrastructure.

## Optional — automatic bit-rate setting

Because LSS runs before any bit-rate-dependent traffic, the head can also set the bit rate with **Configure Bit Timing (0x13)** using standard table selector 0, index **2 = 500 kbit/s**, followed by **Activate Bit Timing (0x15)**.

| Index | Bit rate | Index | Bit rate |
|---|---|---|---|
| 0 | 1000 kbit/s | 5 | reserved |
| 1 | 800 kbit/s | 6 | 50 kbit/s |
| **2** | **500 kbit/s** | 7 | 20 kbit/s |
| 3 | 250 kbit/s | 8 | 10 kbit/s |
| 4 | 125 kbit/s | 9 | automatic detection |

For stepCAN this is unnecessary — the bit rate is fixed by this specification — but it is worth implementing, because it makes a module that somehow came up at the wrong rate recoverable without a programmer.

## Error handling and hot-swap

| Event | Detection | Response |
|---|---|---|
| **Chain break during enumeration** | No LSS response after 3 attempts at position *n* | Head reports "rack ends at position *n*". If the reference configuration expects more, this is a **fault localised to the link between *n* and *n+1***. |
| **Module removed while running** | Downstream modules see CHAIN_IN drop; head loses heartbeat | Downstream modules cascade to UNCONFIGURED. Head marks positions *n*…end as lost, drives outputs to their fail-safe values, re-enumerates from *n*. |
| **Module inserted while running** | Head sees no change until it probes | Head re-probes the chain end every 5 s and enumerates any new module. |
| **Duplicate node-ID** | Structurally impossible | The head assigns IDs sequentially and nothing is stored in the module. |
| **Module resets alone** | Boot-up message from a configured node-ID | Head re-sends configuration SDOs and returns it to Operational. It keeps its position because its CHAIN_IN is still asserted upstream. |
| **Head resets** | — | Full re-enumeration, with identical results. |

Hot-swap mechanics: staggered connector contacts (ground and supply first, signals last) and a termination switch that opens when unpowered.

## What is deliberately not stored

| Item | Stored in module? | Why |
|---|---|---|
| Node-ID | **No** | Re-derived from the chain every boot; this is what makes module swaps configuration-free |
| Bit rate | Optional | Fixed by this specification; storing it only helps recovery |
| I/O configuration (filters, polarity, fail-safe values) | **Yes**, via 0x1010 | Belongs to the module's function, not its position |
| Identity (0x1018) | Yes, in production | Read-only, factory-set |
| Position → identity map | **Head only** | It is the head's job to know what the rack should look like |
