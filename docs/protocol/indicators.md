---
title: Indicators
slug: indicators
sidebar_position: 30
sidebar_label: Indicators
description: The RUN, ERR and CHAIN LED patterns, following CiA 303-3, with exact flash timings.
tags: [stepcan, cia-303, diagnostics, led]
---

# Status indicators

Every stepCAN module carries three LEDs. Two follow **CiA 303-3** (*Device and network design — Part 3: CANopen indicators*, v1.4.0, a free Technical Report); the third is stepCAN's own.

## Flash pattern definitions

CiA 303-3's definitions, with exact timings:

| Pattern | Definition | Timing |
|---|---|---|
| On | constantly on | continuous |
| Off | constantly off | continuous |
| **Flickering** | iso-phase on and off at approximately 10 Hz | 50 ms on / 50 ms off |
| **Blinking** | iso-phase on and off at approximately 2,5 Hz | 200 ms on / 200 ms off |
| **Single flash** | one short flash followed by a long off phase | 200 ms on / 1000 ms off |
| **Double flash** | two short flashes separated by an off phase | 200 on / 200 off / 200 on / 1000 off |
| **Triple flash** | three short flashes | 3 × (200 on / 200 off), then 1000 off |
| **Quadruple flash** | four short flashes | 4 × (200 on / 200 off), then 1000 off |

## RUN LED — green

| Pattern | Meaning | stepCAN |
|---|---|---|
| Flickering | LSS / auto-bitrate in progress | **Used — this is enumeration** |
| Blinking | PRE-OPERATIONAL | Mandatory |
| Single flash | STOPPED | Mandatory |
| Double flash | reserved | not used |
| Triple flash | firmware download running | optional |
| On | OPERATIONAL | Mandatory |

The flickering state is worth implementing: during enumeration you can watch the RUN LEDs light up in sequence down the rack, which makes a chain fault visible from across the room before you have opened any diagnostic tool.

## ERR LED — red

| Pattern | Meaning | stepCAN |
|---|---|---|
| Off | no error, device in working condition | Mandatory |
| Flickering | LSS / auto-bitrate in progress | optional |
| Blinking | invalid configuration | **Used — configuration mismatch against the stored reference** |
| Single flash | CAN error counter warning limit reached | Mandatory |
| Double flash | guard or heartbeat event | Mandatory |
| Triple flash | SYNC message not received | Mandatory if 0x1006 is supported |
| Quadruple flash | expected PDO not received | optional |
| On | bus off | Mandatory |

## CHAIN LED — yellow, stepCAN-specific

| Pattern | Meaning |
|---|---|
| Off | CHAIN_OUT not asserted — this module has not been enumerated, or is the last in the rack before termination is assigned |
| On | CHAIN_OUT asserted — a successor is enabled |
| Blinking | CHAIN_IN lost while Operational (EMCY 0xFF01) |

## Colour and the conflict rule

CiA 303-3 permits a single bicolour LED as a combined STATUS indicator, and states that *"in case of a conflict between turning the LED on green versus red, the LED may be turned on red."*

!!! warning "CiA 303-3 is a Technical Report, so that is a 'may', not a 'shall'"
    If you need deterministic indicator behaviour across a mixed rack, restate it as a requirement in your own product specification. stepCAN does:

!!! info "TR-7 — indicator determinism"
    Modules shall fit **two separate LEDs**, red for ERR and green for RUN, plus a yellow CHAIN LED. A single bicolour STATUS LED shall not be used. Where a module nevertheless implements a bicolour indicator, red **shall** take precedence over green.
