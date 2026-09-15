---
title: Physical layer
slug: index
sidebar_position: 3
sidebar_label: Physical layer
description: Connectors, cable, the CAN segment, the stepCAN chain, and the surge protection cascade.
tags: [stepcan, hardware]
---

# Physical layer

Everything below the protocol. This is where the 64-node node count, the 500 kbit/s bit rate and the choice of Cat5e stop being assumptions and start being constraints with arithmetic behind them.

- [Connectors and cable](./connectors-and-cable.md) — RJ45 pinout, the board-to-board variant, and what Cat5e costs you against dedicated CAN cable.
- [CAN physical layer](./can.md) — transceiver selection for 64 nodes, bus loading, capacitance and node spacing, stub budget, bit timing and clock accuracy.
- [stepCAN chain](./chain.md) — the SP3490, fail-safe biasing, polarity, and why the RS-485 node limit does not apply.
- [Protection](./protection.md) — the GDT → choke → TVS cascade, coordination, earthing and shield bonding.
