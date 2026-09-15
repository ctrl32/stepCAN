---
title: Module reference design
slug: module-design
sidebar_position: 20
sidebar_label: Module design
description: Block diagram, MCU requirements including the mandatory crystal, KiCad sheet structure, net list and layout rules.
tags: [stepcan, hardware, kicad, stm32]
---

# Module reference design

## Block diagram

```
   IN port                                                   OUT port
 ┌────────┐                                               ┌────────┐
 │ RJ45   │                                               │ RJ45   │
 │  or    │                                               │  or    │
 │  B2B   │                                               │  B2B   │
 └───┬────┘                                               └───┬────┘
     │ 1,2  CAN_H/L ═══════════╤═══════════════════════════════╡ 1,2
     │ 3,6  GND/SHLD ══════════╪═══════════════════════════════╡ 3,6
     │ 7,8  0V/+24V ═══════════╪═══════════════════════════════╡ 7,8
     │ 4,5  CHAIN ──┐          │                          ┌────╡ 4,5
     │              │          │                          │
 [GDT][CHOKE][TVS]  │     [GDT][CHOKE][TVS]          [GDT][CHOKE][TVS]
                    │          │                          │
                 A/B│      CAN_H/L                     Y/Z│
                 ┌──┴──────────┴──────────────────────────┴──┐
                 │   SP3490                  SIT1050Q         │
                 │   R  D                    TXD  RXD  S      │
                 └───┬──┬───────────────┬────┬────┬───────────┘
                     │  │               │    │    │
                 ┌───┴──┴───────────────┴────┴────┴───────────┐
                 │              MCU  (STM32)                  │
                 │   CHAIN_IN / CHAIN_OUT / CAN_TX / CAN_RX   │
                 │   TERM_EN ──▶ [120R + MOSFET across CAN]   │
                 │   ◀── XTAL, +/-100 ppm  (MANDATORY)        │
                 │   LEDs: RUN (grn), ERR (red), CHAIN (yel)  │
                 └────────────────────┬───────────────────────┘
                                      │
                          ┌───────────┴───────────┐
                          │  Isolated I/O section │
                          │  (DI / DO / AI / AO)  │
                          └───────────┬───────────┘
                                      │
                               field terminals

   Power:  +24V ──[fuse]──[reverse prot]──[DC/DC]──▶ +5V ──[LDO]──▶ +3.3V
```

## MCU requirements

| Requirement | Reason |
|---|---|
| **Quartz crystal, ±100 ppm or better, as the CAN clock** | **TR-5.** The internal RC oscillator and ceramic resonators are prohibited — see [clock accuracy](../physical-layer/can.md#clock-accuracy-a-crystal-is-mandatory) |
| CAN clock a multiple of 8 MHz | Exact 125 ns Tq. **36 MHz is unusable** |
| CAN / FDCAN peripheral with ≥3 filter banks | Node-ID-dependent filtering for RPDO, NMT, SYNC, LSS |
| **5 V-tolerant pin for CAN_RX** | SIT1050Q's RXD swings to VCC. Most STM32 CAN_RX pins are FT — verify for your exact part and package |
| ≥32 kB flash, ≥8 kB RAM | A CANopen stack with object dictionary and LSS |
| Internal 96-bit UID or a serial-number EEPROM | Populates 0x1018:04 |
| Non-volatile storage for 0x1010 parameters | I/O configuration only — **never the node-ID** |

STM32G0 and STM32G4 both fit comfortably and have FDCAN; the G0 is the cost-optimised choice for a simple digital I/O module.

!!! danger "The crystal is not optional and not a cost-reduction target"
    A ceramic resonator will pass every bench test at room temperature and then fail intermittently in the field over temperature. Anyone proposing to remove the crystal should be shown the arithmetic in TR-5 first: the budget is ±4854 ppm total, and a resonator is ±3000 to ±5000 ppm before ageing.

## KiCad sheet structure

A clean hierarchical split, one sheet per concern:

| Sheet | Contents |
|---|---|
| `01_power` | Fuse, reverse protection, DC/DC to 5 V, LDO to 3.3 V, bulk and decoupling |
| `02_port_in` | IN connector, GDT, series R, choke, TVS, chain receiver side |
| `03_port_out` | OUT connector, GDT, series R, choke, TVS, chain driver side |
| `04_bus` | SP3490, SIT1050Q, termination switch, bias network |
| `05_mcu` | MCU, crystal and load capacitors, SWD, LEDs, reset |
| `06_io` | I/O section — the only sheet that differs between module variants |

Sheets 01–05 are identical across the whole product family. That is the point of the architecture: only sheet 06 is ever redrawn.

## Net summary for schematic capture

| Net | From | To |
|---|---|---|
| `CAN_H` | IN pin 1 | OUT pin 1, transceiver CANH, termination switch |
| `CAN_L` | IN pin 2 | OUT pin 2, transceiver CANL, termination switch |
| `CAN_GND` | IN pin 3 | OUT pin 3, module 0 V (single point) |
| `CHAIN_IN_A` | IN pin 4 | SP3490 pin 8 (A), bias R1 |
| `CHAIN_IN_B` | IN pin 5 | SP3490 pin 7 (B), bias R2 |
| `CHAIN_OUT_Y` | SP3490 pin 5 (Y) | OUT pin 4 |
| `CHAIN_OUT_Z` | SP3490 pin 6 (Z) | OUT pin 5 |
| `MCU_CHAIN_IN` | SP3490 pin 2 (R) | MCU GPIO input |
| `MCU_CHAIN_OUT` | MCU GPIO output | SP3490 pin 3 (D) |
| `MCU_TERM_EN` | MCU GPIO output | Termination MOSFET gate |
| `PE` | GDT centre electrodes | Chassis earth stud, ≥2 vias, short and wide |

## Layout rules

1. **Protection components as close to the connector as possible** — high-frequency surge current follows least inductance, not least resistance.
2. **GDT centre electrode to the chassis earth stud: short, wide, at least two vias.** Do not route it through the ground plane fill.
3. **CAN stub ≤100 mm** from the through-path to the transceiver, including the protection network (TR-2).
4. **Keep the CAN pair as a coupled differential pair** through the module, with the transceiver tapped off as a short stub, not broken and re-driven.
5. **Chain pairs need no impedance control** — static levels.
6. **Crystal close to the MCU**, with its load capacitors, a local ground pour and no switching traces underneath. The whole bus depends on it.
7. **Separate the field-side I/O ground from logic ground** and bridge only at the designated point.
8. **Termination MOSFET open when unpowered.** A dead module must not leave a terminator on the bus.
9. **Three status LEDs** — see [Indicators](../protocol/indicators.md).
