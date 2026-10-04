---
title: Deployment profiles
slug: deployment-profiles
sidebar_position: 30
sidebar_label: Deployment profiles
description: The DIN-rail backplane profile, the distributed profile for machines and buildings, and the mixed installation that most real systems use.
tags: [stepcan, installation, din-rail, building-automation]
---

# Deployment profiles

One electrical design serves two very different mechanical arrangements, and a mix of the two. This works because every module always presents the same thing to its neighbours — an upstream port and a downstream port carrying identical signals. Only the connector changes.

## Profile A — DIN-rail backplane

Modules clip onto DIN rail immediately adjacent to each other and mate directly, board-to-board. There is no backplane PCB: the right-hand connector of one module plugs into the left-hand connector of the next, the way Beckhoff and Phoenix Contact bus connectors work.

```
     DIN rail
 ════════════════════════════════════════════════════
   ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐
   │HEAD││ M1 ││ M2 ││ M3 ││ M4 ││ M5 ││ M6 ││ M7 │
   └────┘└────┘└────┘└────┘└────┘└────┘└────┘└────┘
      └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘
       board-to-board, 8-way, ~22 mm pitch
```

**Why no backplane PCB.** A passive backplane would have to route slot *n*'s chain OUT to slot *n+1*'s chain IN, which makes the backplane non-uniform and fixes the rack size. Direct module-to-module mating keeps every module identical and the rack arbitrarily extensible — the same property the distributed profile has.

**Electrical consequence, and it is the important one.** A 16-module backplane is about 400 mm long, a one-way propagation delay of roughly 2.7 ns. Against a slope-controlled CAN edge of order 100 ns, the structure is **electrically short** — it behaves as a lumped circuit, not a transmission line.

!!! note "This is why dense packing is safe here and would not be on cable"
    Reflections and the node-spacing rule do not apply within a backplane group; only total capacitance and DC loading do. A structure is electrically short when its one-way delay is under roughly one sixth of the signal rise time — 17 ns for a 100 ns edge. A 400 mm backplane is 2.7 ns. A 30 m cable is 150 ns and is firmly in the distributed regime.

Recommended limit: **16 modules per contiguous backplane group**, to keep the group electrically short and its supply current sane. Groups are then linked by cable.

## Profile B — distributed

Modules sit wherever the I/O is: at machine stations, in sub-distribution boards around a house, in separate enclosures along a line. They are linked by Cat5e FTP with RJ45 connectors, straight-through.

```
 ┌──────┐  3 m   ┌─────┐  6 m   ┌─────┐  4 m   ┌─────┐
 │ HEAD │────────│ M1  │────────│ M2  │────────│ M3  │  ...
 └──────┘  FTP   └─────┘  FTP   └─────┘  FTP   └─────┘
   cabinet        station 1      station 2      station 3
```

This is the profile where the chain earns its keep, because no slot-coding scheme can work here at all.

!!! warning "There is a minimum spacing between modules in this profile"
    On cable, each module is a lumped capacitance on a transmission line, and two modules closer together than the minimum spacing produce reflections the bus cannot absorb. With the recommended transceiver and TVS the figure is about **400 mm of cable between adjacent modules**, so a 64-module distributed bus must be at least **25 m long**. This is the opposite of the constraint people expect. It is derived in full under [bus capacitance and node spacing](../physical-layer/can.md#bus-capacitance-and-node-spacing).

    Modules that genuinely need to sit close together belong in a backplane group, which is exempt.

### Buildings and home automation

!!! tip "A whole house on one line: use stepCAN Home"
    The notes below are for building I/O on the **Industrial** level (500 kbit/s, 30 m, local 24 V). A line through a whole house — up to 150 m, modules in series, powered over the cable — is the **Home** level of revision 0.3: [Industrial and Home](./levels.md).

The distributed profile is directly usable for building I/O — lighting, blinds, heating zones, access control. Practical notes:

- **Sub-distribution boards are the natural module locations.** Put a small DIN group inside each board and run cable between boards. That is Profile C, and it sidesteps the minimum-spacing rule entirely.
- **Plan the cable as a single path through the house, not a star from the main board.** The bus is linear. 30 m covers a normal house comfortably if the head sits centrally rather than at one end.
- **Bond the FTP shield at both ends.** Building cable runs parallel to mains wiring and contactors, and the shield is doing real work. See [Protection](../physical-layer/protection.md#pe-earth-and-shield).
- **Power modules locally from each board's 24 V supply**, not over the cable. See [Power distribution](../implementation/power.md).

!!! danger "A run that leaves the building changes the compliance picture completely"
    Under EN IEC 61000-6-2 a line that leaves a building is a *long distance line* **regardless of its length**, and the ±1 kV line-to-earth surge test applies. A 12 m run to a garage needs surge protection; a 29 m run inside one building does not. The gas discharge tube stage is mandatory for any port that can be connected to a cable leaving the building.

## Profile C — mixed

The normal real installation. Groups of modules on DIN backplanes inside enclosures, the enclosures linked by cable.

```
 ┌─ cabinet A ──────────┐        ┌─ cabinet B ──────────┐
 │ HEAD│M1│M2│M3│M4     │══FTP══▶│ M5│M6│M7│M8│M9│M10   │══FTP══▶ ...
 └──────────────────────┘  12 m  └──────────────────────┘
```

Nothing in the protocol changes. The chain crosses a backplane gap and a cable gap identically; the modules cannot tell the difference and do not need to.

This is also the profile that resolves the minimum-spacing tension: modules that need to be close are in a group, and only the groups are spaced on cable. **For most installations Profile C is the right answer, not a compromise.**

The only design rule for mixed installations: **count the whole bus**, not each cabinet. The 64-module limit, the length window and the stub budget are properties of the segment, not of any enclosure.

## Profile comparison

| | Profile A — backplane | Profile B — distributed |
|---|---|---|
| Connector | 8-way board-to-board, ~22 mm pitch | RJ45, CiA 303-1 pinout |
| Inter-module distance | ~20 mm | 0.4 m minimum, 30 m total |
| Electrical behaviour | Lumped | Distributed transmission line |
| Node-spacing rule | Does not apply | **Applies — governs TVS and transceiver choice** |
| Stub budget | Negligible | Binding constraint |
| Protection | TVS only, on internal contacts | Full cascade on every port |
| Power | Shared across the group over the connector | Local, or injected per group |
| Typical group size | ≤16 modules | any |
