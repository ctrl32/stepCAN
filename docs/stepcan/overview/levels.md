---
title: Industrial and Home
slug: levels
sidebar_position: 40
sidebar_label: Industrial and Home
description: The two levels of stepCAN — Industrial at 500 kbit/s over 30 m, Home at 50 kbit/s over 150 m with power over the cable and optional sleep — and what differs between them.
tags: [stepcan, home-automation, building-automation, power, sleep]
---

# Industrial and Home

Revision 0.3 splits stepCAN into two **levels**. The protocol, the chain, the enumeration, the connector and the cable are the same; the physical envelope, the power and the module electronics differ.

| | **stepCAN Industrial** | **stepCAN Home** |
|---|---|---|
| Intended for | machines, cabinets, DIN-rail racks | a house: one line through the building, modules in series |
| CAN bit rate | 500 kbit/s | **50 kbit/s** |
| Segment length | 25–30 m distributed (TR-6) | **up to 150 m** |
| Modules per segment | 64 | 64 |
| Topology | single linear segment, daisy-chained | single linear segment, daisy-chained — **no star** |
| Cable | Cat5e F/UTP, 24 AWG | Cat5e F/UTP, 24 AWG |
| Termination | 120 Ω at both ends | 120 Ω at both ends |
| Module power | local 24 V | **over the cable from one end**, within the budget of TR-H3 |
| Reference MCU | STM32G0 / G4 | **ESP32-C6** |
| CAN transceiver | SIT1050Q (TR-1) | **TCAN1042HV**: TR-1 plus standby with bus wake-up and a 3.3 V logic supply (TR-H2) |
| Sleep | **never** — modules run continuously | optional, per module (TR-H4) |
| Enumeration, 64 modules | ≈0.5 s | ≈1.6 s |

Everything this specification says elsewhere applies to both levels unless this page says otherwise. The pages written before revision 0.3 state the Industrial figures.

!!! info "TR-H1 — one level per segment"
    A segment runs one level. The bit rate is a property of the segment, not of a module: a Home module and an Industrial module never share a segment at different rates. A module's default bit rate is its level's rate (500 kbit/s Industrial, 50 kbit/s Home), so it hears LSS at power-up; LSS *Configure Bit Timing* (table index 6 = 50 kbit/s, index 2 = 500 kbit/s) remains the recovery path for a module that came up at the wrong rate.

## Why a second level

A house is not a machine. The cable runs once through the whole building — 150 m is a realistic line through a family house with its garage — the modules sit one per room or per sub-distribution box, there is no 24 V supply in most of those places, and the traffic is events (a button, a relay, a temperature that changed), not a fast cyclic process image. Industrial's 30 m and its local 24 V per enclosure do not fit, and its 500 kbit/s buys nothing a house needs.

## Physical layer at 150 m

### Far-node voltage — the cable resistance

*Derived*, with the formula of [Connectors and cable](../physical-layer/connectors-and-cable.md#the-dc-resistance-deviation):

```
cable loop resistance, 150 m = 2 × 150 × 0.0938  = 28.1 Ω
far terminator branch        = 120 + 28.1         = 148.1 Ω
far-node differential        = V_bus × 120 / 148.1 = 0.810 × V_bus
```

| Driver | Far node sees | Receiver threshold | Margin |
|---|---|---|---|
| ISO minimum 1.5 V | **1.22 V** | 0.9 V | 1.35× |
| Typical 2.0 V | 1.62 V | 0.9 V | 1.8× |

1.35× is less than Industrial's 1.6× at 30 m and still a working margin. **150 m is the limit of this level for this reason**, not because of the bit rate: the resistance does not care how fast the bits are. A longer line needs a CAN repeater (two segments, each terminated) or a thicker cable — not a lower termination, see below.

!!! warning "Termination stays at 120 Ω"
    The [connectors page](../physical-layer/connectors-and-cable.md#the-100-impedance-deviation) explains why lowering it breaks a 64-node bus. At 150 m the cable resistance helps the driver load: 120 Ω ∥ 148.1 Ω ∥ (30 kΩ / 64) = **58.1 Ω** — *Derived*, above the 50 Ω mandatory bound. Both levels fit the same 120 Ω switched terminator.

### Bit timing at 50 kbit/s

*Derived*, the same method as [Bit timing](../physical-layer/can.md#bit-timing):

```
T_PROP = 2 × (255 ns + 150 m × 4.98 ns/m) = 2 × (255 + 747) = 2004 ns  ≈ 10 % of the 20 µs bit
```

With the ESP32-C6's 40 MHz crystal as the TWAI clock: prescaler 50 → Tq = 1.25 µs, **16 Tq per bit**, SYNC 1 + PROP 2 + PHASE_SEG1 11 + PHASE_SEG2 2, **sample point 87.5 %**, SJW 2. PROP needs 2004 ns = 1.6 Tq and gets 2. The clock budget is the same ±4854 ppm as Industrial (it depends on the segmentation, not on the rate), so **TR-5 stands unchanged** and the C6's crystal meets it.

### Stubs and spacing

TR-2 (100 mm stub, daisy-chain only) and TR-6 (0.5 m minimum between modules) apply unchanged. Both are conservative at 50 kbit/s; they are kept so that one module design serves both levels. 64 modules over 150 m average 2.3 m apart.

### Bus load

A frame at 50 kbit/s takes ten times as long as at 500 kbit/s — an 8-byte frame ≈ 2.2 ms, a 1-byte heartbeat ≈ 1.1 ms. Practical capacity is a few hundred frames per second, ample for events and far too little for Industrial's 2 ms SYNC cycle. Home therefore runs **event-driven**: TPDOs on change with an inhibit time, no SYNC producer (see TR-H4).

## Module electronics

### TR-H2 — the CAN transceiver

!!! info "TR-H2 — Home transceiver"
    The Home CAN transceiver shall meet **TR-1** (R_diff ≥ 30 kΩ minimum), shall have a **standby mode with bus wake-up** reported on RXD, and shall have a **logic supply input (VIO) at 3.3 V**, because the ESP32-C6's pins are not 5 V tolerant.

!!! danger "The obvious standby transceivers fail TR-1"
    TJA1042, TJA1043, SIT1042 and SIT1051A all have standby and wake-up, and all of them are **19 kΩ minimum** — they miss the 64-node load bound ([CAN physical layer](../physical-layer/can.md#the-loading-calculation)). A Home design that reaches for TJA1042T/3 because it has VIO and standby is the TR-1 trap again.

**Selected: TCAN1042HV** (TI, SOIC-8 `TCAN1042HVDR`, or VSON-8 `TCAN1042HVDRBR`). *Quoted* from the TI datasheet, beside the runner-up:

| | **TCAN1042HV** | TCAN1044V |
|---|---|---|
| R_diff (TR-1) | **30 / 80 kΩ** min / max — meets TR-1, N_max 100 | 40 / 90 kΩ — N_max 133 |
| C_diff | 12 typ / **15 max** pF | 10 pF typ, no max published |
| Standby supply | I_CC 0.5 / 5 µA (V variant: **VCC not needed in standby**) + I_IO 12 / 17 µA | I_CC 0.2 / 1 µA + I_IO 8.5 / 13.5 µA |
| Bus wake-up | WUP, t_WK_FILTER 0.5–1.85 µs, reported on RXD | WUP, t_WK_FILTER 0.5–1.8 µs, t_WK_TIMEOUT 0.8–6 ms, on RXD |
| VCC / VIO | 5 V / 3.3 V supported | 4.5–5.5 V / 1.7–5.5 V |
| Bus fault protection | **±70 V** | ±58 V |
| Receiver common mode | **±30 V** | ±12 V |
| IEC 61000-4-2 on CANH/CANL | **±8 kV powered, ±15 kV unpowered** (contact) | not found in the datasheet |
| Loop delay | 100 typ / 160 max ns | 125 / 210 ns |
| Package | **SOIC-8**, VSON-8 | SOT-23-THIN-8 |

Why the HV and not the 1044V, although the 1044V has more TR-1 margin:

- **Footprint.** The SOIC-8 pinout is the SIT1050Q's — TXD 1, GND 2, VCC 3, RXD 4, CANL 6, CANH 7 — with pin 5 VIO instead of Vref and pin 8 STB instead of S. One CAN footprint serves the Industrial and the Home board.
- **A house is not a cabinet.** 24 V travels in the same RJ45 as the CAN pair; ±70 V bus-fault tolerance survives a miswired crimp, ±30 V common mode covers a ground offset on a 150 m line with room to spare, and the IEC system-level ESD figure is published.
- **The 5 V rail can be switched off in sleep.** The V variant needs only VIO in standby, so a sleeping Home module keeps 3.3 V for the C6 and drops the transceiver's 5 V supply.

*Derived* — the spacing budget with the HV's 15 pF maximum, the same TVS and parasitics as the [spacing table](../physical-layer/can.md#bus-capacitance-and-node-spacing): C_L ≈ 24.4 pF, minimum spacing 0.98 × 24.4 / 48 ≈ **0.50 m**. Exactly TR-6's 0.5 m; on a Home line 64 modules over 150 m average 2.3 m. On an Industrial 30 m segment with 64 modules it would leave no margin — the Industrial board keeps the SIT1050Q.

Every frame wakes a standby transceiver: at 50 kbit/s a bit is 20 µs, far longer than t_WK_FILTER, so a single frame is a valid wake-up pattern. This is what TR-H4 builds on.

### ESP32-C6 against the module requirements

| Requirement ([Module design](../implementation/module-design.md#mcu-requirements)) | ESP32-C6 | Note |
|---|---|---|
| Quartz crystal, TR-5 | 40 MHz crystal, mandatory for the chip | Meets TR-5; 40 MHz is a multiple of 8 MHz |
| 5 V-tolerant CAN_RX | **No** | Replaced by the VIO requirement of TR-H2 |
| ≥3 CAN filter banks | TWAI has **one** acceptance filter (single or dual mode) | Filter the remaining COB-IDs in software; at 50 kbit/s that is at most a few hundred frames/s — *Derived* |
| ≥32 kB flash, ≥8 kB RAM | far more | — |
| 0x1018:04 serial number | the factory MAC in eFuse | — |
| Non-volatile 0x1010 parameters | NVS in flash | **Never the node-ID** — unchanged |
| Deep-sleep wake inputs | LP_IO 0…7 only | CHAIN_IN, the transceiver's RXD and any wake button go on these pins |

The C6 has two TWAI controllers; a Home module uses one.

### Outputs and the power budget

Relays on a cable-powered module are **bistable** (a pulse to set, a pulse to reset, no holding current). A monostable relay with a 24 V coil draws 30–70 mA for as long as it is on; twenty of them are an ampere and more than 10 V of drop over 150 m. A module that needs a holding load is powered locally.

## TR-H3 — power over the cable

Home modules are powered over the cable from one end. Pin 8 carries +24 V, pin 7 the 0 V return, and **pin 6 — the optional shield contact of the Industrial pinout — becomes a second +24 V conductor** in Home (the F/UTP foil is bonded through the connector shell).

*Derived:* a C6 module with its transceivers and a DC/DC converter draws about 0.12–0.15 W, ≈ 6 mA at 24 V.

| Load, 64 modules | Current at 24 V | +24 V conductors | Drop at the far end (uniform load) |
|---|---|---|---|
| all running, 6 mA each | 0.38 A | pin 8 only | ≈ 4.0 V |
| all running, 6 mA each | 0.38 A | **pins 8 + 6** | **≈ 2.7 V** |
| at the budget limit, 0.5 A | 0.50 A | pins 8 + 6 | ≈ 3.5 V |

!!! info "TR-H3 — power budget"
    A Home segment fed from one end shall draw **≤ 0.5 A at 24 V** in total (the injection limit of [Power distribution](../implementation/power.md)), which is **≤ 7.5 mA average per module** at 64 modules. The module's DC/DC converter shall accept at least 18–30 V. A module that needs more, or a segment that sums to more, takes local power or a second injection point.

The module's 0 V is tied to CAN_GND at its single point, so part of the return current flows in pin 3 as well. The resulting ground offset between the ends — about half the drop above — sits well inside the CAN common-mode range of −2 V … +7 V.

## TR-H4 — sleep (optional, Home only)

Industrial modules never sleep. A Home module may, and at 150 m it does not have to: the cable carries 64 running modules within TR-H3. Sleep is for the standby consumption of the house.

!!! info "TR-H4 — sleep rules"
    1. A module declares its **power class** in object 0x5FFB: 0 = always running, 1 = light sleep, 2 = deep sleep. A dimmer or anything with PWM is class 0 or 1.
    2. A module enters sleep only when the head has allowed it (0x5FFB written by the head) and the bus has been idle for the module's idle time.
    3. **No SYNC producer** on a segment that has sleeping modules, and heartbeat (0x1017) **off** in sleeping modules. Instead a sleeping module wakes every **report interval** (0x5FFA, seconds), sends its state as a TPDO and sleeps again; the head's liveness check for that module is the report, not the heartbeat.
    4. The head wakes a module by sending any frame (the transceiver's bus wake-up, ISO 11898-2:2016), waits the wake time, then sends the real request and repeats it until it is answered. **The frame that woke a module is lost** by construction.
    5. While asleep a module **holds CHAIN_OUT** (GPIO hold) and keeps CHAIN_IN on an LP_IO wake input: losing the chain still cascades, exactly as when awake.
    6. The node-ID survives deep sleep in **LP RAM** — memory that a power cycle clears. This keeps the rule that the node-ID is never stored in non-volatile memory, and it prevents what would otherwise happen: several modules waking from deep sleep at once with CHAIN_IN asserted would all be ADDRESSABLE together and LSS would collide.

**The consequence of bus wake-up.** A standby transceiver wakes on *any* traffic, so every frame on the segment wakes every sleeping module. Sleep pays off only on a quiet segment: events only, inputs with an inhibit time, sensors reporting on change and no more often than the report interval. A segment with chatty modules simply keeps everyone awake — it still works, it just saves nothing. Selective wake-up (ISO 11898-2:2016 partial networking, a transceiver that wakes only on its own identifier) would remove this limit; *Open* — no such part has been checked against TR-1.

**The chain transceiver.** The SP3490 has no enable and draws 1–2 mA plus the bias current whenever the module is powered — more than a sleeping C6 and the standby CAN transceiver together. *Open:* a full-duplex RS-485 part with a low quiescent current for sleeping Home modules, or larger bias resistors; the asserted chain level has the same polarity as the bias, so the load in that state is already the small one (≈ 0.28 mA, [stepCAN chain](../physical-layer/chain.md#supply-budget)).

### Manufacturer-specific objects added in revision 0.3

| Index | Sub | Name | Type | Access | Meaning |
|---|---|---|---|---|---|
| 0x5FFB | 00 | Power class | UNSIGNED8 | RW | 0 always running, 1 light sleep, 2 deep sleep; the head writes the class it allows, the module refuses a class its hardware cannot do |
| 0x5FFA | 00 | Report interval | UNSIGNED16 | RW | seconds between wake-and-report while asleep; 0 = report on events only |

## Enumeration at 50 kbit/s

*Derived* from the [timing budget](../protocol/enumeration.md#timing-budget): the 17 frames per module take ≈ 19 ms instead of ≈ 2 ms; with the module's reset and the chain settle (≈ 6 ms, rate-independent) that is **≈ 25 ms per module, ≈ 1.6 s for 64 modules**.

## Open in this revision

| Item | What is needed |
|---|---|
| Home CAN transceiver | **Selected: TCAN1042HV** (figures quoted above). Open only: its normal-mode recessive I_CC at 5 V for the TR-H3 budget, to be measured on the first board |
| Low-power chain transceiver | a full-duplex RS-485 part with low quiescent current for sleeping modules |
| Selective wake-up | a partial-networking transceiver that also meets TR-1 |
| 150 m on real cable | measure the far-node differential and the error counters on a 150 m Cat5e line with 64 modules (or their equivalent load) before the level is called proven |
| Module power figure | measure a real C6 Home module at 24 V; TR-H3's 6 mA is derived, not measured |
