---
title: Firmware architecture
slug: firmware
sidebar_position: 30
sidebar_label: Firmware
description: The module's chain task, the head's enumeration routine, and the startup sequence.
tags: [stepcan, firmware, c, canopen]
---

# Firmware architecture

## Module — chain handling

```c
typedef enum {
    ST_UNCONFIGURED,   /* node-ID 0xFF, CHAIN_OUT low          */
    ST_ADDRESSABLE,    /* CHAIN_IN asserted, LSS may configure  */
    ST_CONFIGURED,     /* node-ID active, Pre-operational       */
    ST_CHAINED,        /* CHAIN_OUT asserted                    */
    ST_OPERATIONAL     /* PDOs running                          */
} chain_state_t;

/* called every 1 ms from a timer */
void chain_task(void)
{
    static uint8_t  cnt;
    static bool     stable;
    bool raw = gpio_read(CHAIN_IN);          /* TR-3: idle reads as 0 */

    /* --- 1 ms debounce ------------------------------------------- */
    if (raw == stable) { cnt = 0; }
    else if (++cnt >= CHAIN_DEBOUNCE_MS) { stable = raw; cnt = 0; }

    /* --- rising edge: we have become addressable ----------------- */
    if (stable && state == ST_UNCONFIGURED)
        state = ST_ADDRESSABLE;

    /* --- falling edge: upstream is gone. Cascade. ---------------- */
    if (!stable && state != ST_UNCONFIGURED) {
        gpio_write(CHAIN_OUT, 0);            /* drop our successor   */
        term_enable(false);
        io_failsafe();                       /* 0x6206 / 0x6207      */
        emcy_send(0xFF01);                   /* chain input lost     */
        canopen_set_node_id(0xFF);
        canopen_reset_communication();
        state = ST_UNCONFIGURED;
    }
}

/* SDO write handler for 0x5FFF:00 */
void od_write_chain_out(uint8_t v)
{
    gpio_write(CHAIN_OUT, v ? 1 : 0);
    if (v) state = ST_CHAINED;
}

/* SDO write handler for 0x5FFE:00 */
void od_write_termination(uint8_t v)
{
    term_enable(v != 0);
}
```

!!! note "Note what the module firmware does not contain"
    No addressing logic, no position awareness, no negotiation. It reacts to one input level and two SDO writes. All the intelligence is in the head — which is why a module is cheap to build and cheap to get right.

## Head — enumeration

```c
int stepcan_enumerate(node_info_t *rack, int max_nodes)
{
    int n = 0;
    chain_out_assert();                       /* enable position 1   */

    while (n < max_nodes) {
        delay_ms(CHAIN_SETTLE_MS);            /* 3 ms                */

        if (!lss_switch_global(LSS_CONFIGURATION))
            break;

        node_info_t id;
        if (!lss_inquire_identity(&id))       /* CS 0x5A..0x5D       */
            break;                            /* nobody there: done  */

        if (!lss_configure_node_id(n + 1))    /* CS 0x11             */
            return -ERR_LSS_CONFIG;

        /* Deliberately NOT lss_store_configuration() */

        lss_switch_global(LSS_WAITING);
        nmt_reset_communication(n + 1);       /* makes the ID active */

        if (!wait_bootup(n + 1, BOOTUP_TIMEOUT_MS))
            return -ERR_NO_BOOTUP;

        rack[n] = id;
        rack[n].position = n + 1;

        sdo_write_u8(n + 1, 0x5FFF, 0x00, 1); /* assert chain out    */
        n++;
    }

    if (n > 0)
        sdo_write_u8(n, 0x5FFE, 0x00, 1);     /* last node terminates */

    return n;
}
```

## Head — startup sequence

```
 1. Assert chain, enumerate            → rack[] and node count
 2. Compare rack[] against stored reference
       mismatch in product code  → fault, stop here
       serial number differs     → log "module replaced", continue
 3. Configure every module by SDO (PDO mapping, inhibit, event timer,
    0x6206/0x6207 fail-safe values, heartbeat 0x1017 = 500 ms)
 4. Set up the head's own 0x1016 consumer heartbeat table
 5. NMT Start Remote Node, broadcast   → all modules Operational
 6. Start the SYNC producer
 7. Enter the cyclic loop; monitor heartbeat and re-enumerate on loss
```

## Stack selection

For the CANopen stack itself, **CANopenNode** is the recommended starting point: Apache 2.0 licensed, implements CiA 301, CiA 303 indicators, CiA 304 and **CiA 305 LSS**, validated with the CiA CANopen Conformance Test Tool, and vendored by Zephyr RTOS.

Its LSS documentation is also the best free substitute for the members-only CiA 305 text: <https://canopennode.github.io/CANopenNode/group__CO__CANopen__305.html>

!!! note "Cite the repository, not a release tag"
    CANopenNode's GitHub Releases page is stale — the newest tag shown is v1.3, while the v4 line is developed directly on `master`.
