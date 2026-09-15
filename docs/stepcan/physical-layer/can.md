---
title: CAN physical layer
slug: can
sidebar_position: 20
sidebar_label: CAN physical layer
description: Transceiver requirements for 64 nodes, bus loading arithmetic, capacitance and minimum node spacing, stub budget, bit timing and the mandatory crystal.
tags: [stepcan, can, iso-11898, transceiver, bit-timing]
---

# CAN physical layer

## Transceiver requirement — specify it, do not assume it

The single most important electrical decision in a 64-node design is a **minimum differential input resistance**, because every node's receiver hangs permanently across the bus and they add in parallel.

!!! info "TR-1 — differential input resistance"
    Every node's CAN transceiver shall have a differential input resistance **R_diff ≥ 30 kΩ (minimum, not typical)** over the ISO 11898-2 common-mode range.

Write this into your module datasheet. Without it, a future third-party or cost-reduced module with a 12 kΩ transceiver can silently degrade a bus that was working fine, and the symptom is intermittent errors at the far end rather than an obvious failure.

The word *minimum* is load-bearing. Several transceivers marketed as 30 kΩ parts specify 30 kΩ as the **typical** and 19 kΩ as the minimum, and a 64-node bus is decided by the minimum.

## The loading calculation

With two 120 Ω terminators the network presents 60 Ω; the *N* receivers add R_diff/*N* in parallel. The driver must still meet ISO 11898-2 Table 2:

- **Normal load range: R_L = 50 Ω … 65 Ω**, dominant differential output 1.5 V … 3.0 V — *mandatory*.
- Optional extended range: R_L = 45 Ω … 70 Ω, output 1.4 V … 3.3 V.

Design to the **50 Ω mandatory bound**. Treat 45 Ω as contingency only.

```
R_load = (R_diff / N)  ∥  60 Ω      must be ≥ 50 Ω

  ⇒   N_max = R_diff(min) / 300 Ω
```

| Transceiver class | R_diff min | Load at 64 nodes | N_max at 50 Ω | Suitable? |
|---|---|---|---|---|
| ISO 11898-2 bare minimum | 12 kΩ | **45.5 Ω** | 40 | **No** |
| TJA1042 / TJA1057 / TJA1043 / **SIT1042 / SIT1051A** | 19 kΩ | **49.9 Ω** | 63 | **No — fails 64 by 0.1 Ω** |
| NCV7351 | 25 kΩ | 52.0 Ω | 83 | Marginal |
| **SIT1050 / SIT1050Q** / TCAN1042 | **30 kΩ** | **53.2 Ω** | 100 | **Yes** |
| TI SN65HVD230 / TCAN1044 | 40 kΩ | 54.7 Ω | 133 | Yes on loading — but see capacitance below |

!!! danger "The 19 kΩ row is the trap this requirement exists to catch"
    TJA1042 is an extremely common, well-regarded automotive CAN transceiver. At 64 nodes it misses the mandatory load bound by **0.1 Ω**. So do SIT1042 and SIT1051A, which are clones of that generation.

    Note the direction of the trend: the older TJA1050 / SIT1050 generation guarantees a **higher** input impedance (25–30 kΩ minimum) than the newer CAN-FD generation (19 kΩ minimum). Upgrading to a "newer, better" transceiver makes the node-count budget *worse*. This is genuinely counter-intuitive and it is why TR-1 is written as a number rather than a part number.

!!! warning "Terminology correction"
    Transceiver datasheets and application notes routinely speak of "unit load" and "1 UL = 32 nodes" for CAN. **ISO 11898-2 contains no such concept and states no maximum node count.** "Unit load" is defined in TIA/EIA-485-A for RS-485 — a 15 kΩ load, 32 unit loads per driver — and has been borrowed into CAN marketing. What ISO 11898-2 actually specifies is R_diff = 12 kΩ … 100 kΩ per node and the driver load ranges above. Cite those, not "unit load".

## Transceiver selection

### Recommended: SIT1050Q

| Parameter | SIT1050Q | Assessment |
|---|---|---|
| R_diff | **30 / 60 / 90 kΩ** | **Meets TR-1** on the guaranteed minimum |
| ESD, HBM | **±8 kV, all pins** | Published |
| ESD, IEC 61000-4-2 | not specified | External TVS still required |
| AEC-Q100 | *"Comply with AEC-Q100 requirements"* (grade not stated) | Good sign for a production part |
| Supply | 4.5–5.5 V, no VIO pin | Requires a 5 V rail on every module |
| TXD input threshold | V_IH min 2.0 V | A 3.3 V push-pull GPIO drives it |
| **RXD output** | swings to VCC | **Requires a 5 V-tolerant MCU pin** |
| C_in / C_diff | 18 pF / **10 pF** | Higher than SIT1050 — see the spacing budget below |
| Pinout | 1 TXD · 2 GND · 3 VCC · 4 RXD · 5 Vref · 6 CANL · 7 CANH · 8 S | Identical to SIT1050 and TJA1050 |
| Packages | SOP8 (QT), HVSON8/DFN3×3 (QTK) | |

### Verdict on SIT1050

!!! danger "SIT1050T is NRND and publishes no ESD rating at all"
    Two findings, either of which is enough to keep it out of a new design.

    **It is marked "不推荐用于新设计 (NRND/NFND)"** — not recommended for new designs — in the manufacturer's own distributor listing.

    **No datasheet revision contains an ESD section.** Revisions V1.4 (2019.02), V1.6 (2019.05) and V1.14 (2022.01) were each checked across five independent mirrors: no HBM, no IEC 61000-4-2, no MM, no CDM. Distributor parametric databases carry no ESD attribute either.

    **Do not assume it inherits the TJA1050's ±4 kV.** ESD performance is a property of the silicon and the pad clamp design, not of the pinout. The parametric evidence proves the dies differ — SIT1050 R_diff is 30–80 kΩ against TJA1050's 25/50/75 kΩ, C_in 13 pF against 7.5 pF, bus fault ±40 V against ±36 V. And SITCORES publishes ESD figures for its *other* parts (SIT1050Q ±8 kV, SIT1042 ±8 kV HBM / ±4 kV IEC, SIT65HVD230 ±15 kV), so the omission on SIT1050 is a deliberate silence about this part, not a house style.

On loading, SIT1050 is fine — R_diff 30 kΩ minimum meets TR-1, and it has the **lowest bus capacitance of any candidate** at 5 pF differential, which buys real margin in the spacing budget below. If you already have SIT1050 in stock, it works. For a new design, SIT1050Q is the same part with the two problems fixed.

### Parts to avoid, and why

| Part | Problem |
|---|---|
| **SIT1051A, SIT1051AQ, SIT1042** | R_diff **19 kΩ minimum** (30 kΩ is only typical). Fails TR-1 and fails 64 nodes. Also **SIT1051AT/3 is not a TJA1050 drop-in** — its pin 5 is a VIO supply *input*, not a Vref *output*, so a board that leaves pin 5 floating will not work. |
| **TJA1042 / TJA1057 / TJA1043** | Same 19 kΩ minimum. |
| **MCP2542FD** | 12 kΩ — exactly one ISO unit of loading. Caps out around 40 nodes. |
| **TI SN65HVD230 / SIT65HVD230** | Passes loading comfortably at 40 kΩ, but **16–20 pF of differential capacitance fails the node-spacing rule at 64 nodes over 30 m** (see below). The SITCORES clone additionally publishes no guaranteed *minimum* R_diff, only 40 kΩ typical, so it cannot satisfy TR-1 as written. |

!!! note "The 3.3 V temptation"
    SN65HVD230 is attractive because it removes the 5 V rail and the 5 V-tolerant-pin problem entirely. At fewer nodes, or on a longer bus, it is the better part. At 64 nodes over 30 m its capacitance is what rules it out, not its impedance — and that is a constraint almost nobody checks.

## Termination

| Item | Value |
|---|---|
| Termination resistance | **120 Ω, at the head and at the last module only** |
| Tolerance | 1 %, ≥0.25 W |
| Split termination (optional) | 2 × 60 Ω with 4.7 nF to CAN_GND at the midpoint — measurably better common-mode emissions for almost nothing |

## Bus capacitance and node spacing

Applies to the **distributed profile only**. A backplane group is electrically short and is exempt.

TI's node-spacing criterion (SLLA279) sets the **minimum** distance between lumped loads on a transmission line:

```
d  >  0.98 × C_L / C_cable
```

where `C_L` is the total differential capacitance a node presents and `C_cable` is the cable's mutual capacitance per metre.

!!! warning "This is a minimum spacing, which implies a minimum bus length"
    For *N* modules distributed on cable, the bus must be at least `N × d` long. A 64-module distributed bus that is only 15 m long violates the criterion just as surely as a 64-module bus on the wrong transceiver. Modules that must sit close together belong in a backplane group.

**Budget**, with `C_L` = transceiver C_diff + TVS differential contribution + GDT 0.35 pF + choke 1 pF + PCB and connector 5 pF, cable at 48 pF/m:

| Transceiver | TVS | C_L | Minimum spacing | Minimum bus length, 64 nodes | Margin at 30 m |
|---|---|---|---|---|---|
| SIT1050 (5 pF) | PESD2CANFD24V | 14.3 pF | 293 mm | 18.8 m | 1.60× |
| SIT1050 (5 pF) | ESDCAN03-2BWY | 12.8 pF | 262 mm | 16.8 m | 1.79× |
| **SIT1050Q (10 pF)** | **PESD2CANFD24V** | **19.4 pF** | **395 mm** | **25.3 m** | **1.19×** |
| SIT1050Q (10 pF) | ESDCAN03-2BWY | 17.9 pF | 364 mm | 23.3 m | 1.29× |
| SIT1042 / SIT1051A (12 pF) | PESD2CANFD24V | 21.4 pF | 436 mm | 27.9 m | 1.08× |
| SN65HVD230 (16 pF) | PESD2CANFD24V | 25.4 pF | 518 mm | **33.1 m** | **0.91× — fails** |
| SIT1050Q (10 pF) | **NUP2105L** (13 pF) | 29.4 pF | 599 mm | **38.3 m** | **0.78× — fails** |

!!! info "TR-6 — minimum module spacing, distributed profile"
    In the distributed profile, adjacent modules shall be separated by at least **0.5 m of cable**. A 64-module distributed segment shall therefore be at least **30 m** long, or the modules shall be grouped onto backplanes.

    0.5 m is 395 mm rounded up with margin for the tolerance on the cable's capacitance.

**Two component decisions fall directly out of this table.**

- **The TVS must be a low-capacitance part.** NUP2105L is the default CAN TVS in most reference designs and is entirely correct on a ten-node automotive bus. At 64 nodes it consumes the whole spacing budget on its own and the failure looks like a mysterious distance limit rather than a component choice.
- **SN65HVD230 cannot be used at 64 nodes over 30 m** despite passing every impedance test. It needs a 33 m minimum bus length at that node count.

The conclusion is insensitive to the exact cable capacitance. The break-even figure — where the required spacing would equal the 469 mm available — is **29.8 pF/m**, and no twisted-pair copper cable of any category comes near that. TI's own quoted floor for any medium is 35 pF/m.

## Stub budget

CiA publishes, for 500 kbit/s:

| Bit rate | Bus length | Max single stub | **Accumulated stub** |
|---|---|---|---|
| 1 Mbit/s | 25 m | 1.5 m | 7.5 m |
| **500 kbit/s** | **100 m** | **5.5 m** | **27.5 m** |
| 250 kbit/s | 250 m | 11 m | 55 m |

The 100 m bus-length figure is not what limits stepCAN. The **27.5 m accumulated stub allowance divided across 64 nodes gives 430 mm per node**, and that is the number the mechanical design must respect.

!!! info "TR-2 — stub length"
    The stub from the through-path to the CAN transceiver — connector pin, protection components, PCB trace — shall not exceed **100 mm** per module. Drop cables from a trunk are **not permitted**; modules are daisy-chained in and out.

100 mm × 64 = 6.4 m, comfortably inside 27.5 m and leaving room for the occasional longer run. This is achievable because a stepCAN module is wired in-and-out rather than tapped off a trunk: the bus passes *through* the module and the only stub is the internal tap.

## Automatic termination

The chain gives termination for free, which removes the most common field error in CAN installations — a missing or doubled terminator.

1. Every module powers up with its termination **disabled**.
2. During enumeration, after a module has been given its node-ID, the head asks it to assert its chain OUT and then probes for a successor.
3. If no successor answers within the timeout, the head sends that module a **"you are last"** command and it enables its 120 Ω across CAN_H/CAN_L through a MOSFET or analogue switch.
4. The head's own termination is permanent.

On re-enumeration the assignment recalculates, so extending the rack needs no intervention.

!!! warning "The termination switch must be open when unpowered"
    A dead module must not leave a stray terminator on the bus. Choose the switch topology accordingly — a normally-open MOSFET with a gate pull-down, not a normally-closed relay.

## Bit timing

CiA 301 specifies for 500 kbit/s a nominal bit time of **2 µs**, a valid sample-point range of **85 % to 90 %**, and a recommended sample point of **87.5 %**. It does **not** publish register values; the segmentation below is derived here.

### Propagation requirement

From ISO 11898-1, and stated in Hartwich & Bassemir as `Prop_Seg ≥ 2 × [max(node output delay + bus line delay + node input delay)]`:

```
T_PROP  =  2 × (t_transceiver_loop  +  t_cable)
        =  2 × (255 ns  +  30 m × 4.98 ns/m)
        =  2 × (255 + 149)
        =  808 ns   ≈  40 % of the bit time
```

The factor of two covers the arbitration worst case: a bit from the far node must reach the near node *and* the near node's response must return before the far node samples. 255 ns is the ISO 11898-2 Table 12 maximum loop delay; SIT1050's actual 190 ns maximum gives additional real margin.

### Derived segmentation — 16 Tq, sample point 87.5 %

| Segment | Tq | ns |
|---|---|---|
| SYNC_SEG | 1 | 125 |
| PROP_SEG | 7 | 875 |
| PHASE_SEG1 | 6 | 750 |
| **TSEG1 = PROP_SEG + PHASE_SEG1** | **13** | **1625** |
| **TSEG2 = PHASE_SEG2** | **2** | **250** |
| **Total** | **16** | **2000** |
| **SJW** | **2** | **250** |
| **Sample point** | (1+13)/16 | **87.5 %** |

Required PROP_SEG is 808 ns = 6.5 Tq, rounded up to 7 Tq. Available in TSEG1 is 13 Tq, so **6 Tq remain as resynchronisation buffer — roughly 2× margin on propagation.**

The 87.5 % sample point is exactly CiA-conformant; that is a cheap conformance claim worth making.

### Prescaler

Target Tq = 125 ns, i.e. 8 MHz into the bit-time counter:

| CAN clock | BRP | Usable |
|---|---|---|
| 8 MHz | 1 | Yes |
| 16 MHz | 2 | Yes |
| 24 MHz | 3 | Yes |
| 32 MHz | 4 | Yes |
| **36 MHz** | **4.5** | **No — pick a different clock** |
| 40 MHz | 5 | Yes |
| 48 MHz | 6 | Yes |
| 64 MHz | 8 | Yes |
| 80 MHz | 10 | Yes |

## Clock accuracy — a crystal is mandatory

This is the requirement revision 0.1 was missing, and it is the one most likely to be value-engineered out by someone who has not read the arithmetic.

ISO 11898-1 imposes two oscillator tolerance conditions, derived in Hartwich & Bassemir and stated with their purposes in NXP EB181. Both must hold simultaneously. `df` is the deviation of **one node's** oscillator; the factor of 2 in each denominator accounts for the second node drifting the other way.

**Condition I — error-flag localisation.** After an active error flag a node must still sample the following bit correctly, and in that window there is no resynchronisation edge for up to 13 bit times and no SJW correction available. The only reserve is the Phase Buffer Segments.

```
df ≤ min(PHASE_SEG1, PHASE_SEG2) / (2 × (13 × bit_time − PHASE_SEG2))
   = min(6, 2) / (2 × (13 × 16 − 2))            [in Tq]
   = 2 / 412
   = 0.4854 %  =  4854 ppm
```

**Condition II — resynchronisation in the stuffed bit stream.** Because of the 5-bit stuffing rule, the longest run without a usable recessive-to-dominant edge is 10 bit times. SJW is the per-edge correction budget.

```
df ≤ SJW / (2 × 10 × bit_time)

  SJW = 1:   1 / 320  =  0.3125 %  =  3125 ppm
  SJW = 2:   2 / 320  =  0.6250 %  =  6250 ppm
```

| SJW | Condition I | Condition II | Binding | **Budget** |
|---|---|---|---|---|
| 1 | 0.4854 % | 0.3125 % | II | **±3125 ppm** |
| **2** | **0.4854 %** | 0.6250 % | **I** | **±4854 ppm** |

SJW = 2 is the maximum legal value here (SJW may not exceed the smaller Phase Buffer Segment, which is 2). It buys 55 % more headroom than SJW = 1 at zero cost, so this specification uses **SJW = 2** and the budget is **±4854 ppm**.

That budget covers each node's **total** frequency error: initial tolerance, temperature drift across the operating range, ageing, and load-capacitance error — summed, not each individually.

| Clock source | Total error | Verdict |
|---|---|---|
| STM32 HSI, ambient only | ±1 % = 10 000 ppm | **FAIL — 2.1× over** |
| STM32 HSI, full temperature range | ±3 % = 30 000 ppm | **FAIL — 6.2× over** |
| Ceramic resonator, ±0.5 % | 5 000 ppm | **FAIL** |
| Ceramic resonator, ±0.3 % | 3 000 ppm | Passes with 1.6× — no margin for ageing |
| **Quartz crystal, ±100 ppm** | 100 ppm | **PASS — 48× margin** |
| Quartz crystal, ±50 ppm | 50 ppm | PASS — 97× margin |

!!! danger "TR-5 — CAN clock source"
    The CAN clock source shall be a **quartz crystal or crystal oscillator of ±100 ppm or better**, covering initial tolerance, temperature, ageing and load-capacitance error combined.

    The STM32 internal HSI RC oscillator — ±1 % at ambient, ±3 % over temperature per ST AN6363 — **shall not** be used as the CAN clock source. **Ceramic resonators are not permitted**: at ±0.3 % to ±0.5 % they either fail outright or leave insufficient margin for ageing.

    A ceramic resonator will appear to work on the bench at room temperature and fail intermittently in the field over temperature. That is the classic failure mode, and it is the reason this requirement is written as a prohibition rather than a recommendation.

The ±100 ppm figure gives roughly 48× margin, which is deliberate: it means a badly matched load capacitance or a wide-temperature AEC-Q200 crystal still passes without re-deriving anything.

!!! note "If you galvanically isolate the CAN interface, re-derive the propagation budget"
    An optocoupler adds 40–100 ns each way, doubled by the factor of two: T_PROP rises to about 1210 ns = 10 Tq, leaving PHASE_SEG1 = 3 Tq. Still valid, but the margin is much thinner and resynchronisation suffers. **Use fast digital isolators (10–20 ns), not optocouplers**, if isolation is required. For a 30 m in-building bus with CAN_GND carried in the cable, isolation is generally unnecessary.
