---
layout: default
title: Hardware and interfaces
---
# Hardware and interfaces

[Home](../) · [Firmware](firmware.md) · [Protocols](protocols.md)

## System architecture

The classic Adventurer 3 platform separates a Linux frontend from the motion
controller:

```text
Network / USB storage
        |
Linux frontend: display, files, network services and updates
        |
Internal UART
        |
Motion MCU: motors, heaters, fans and sensors
```

Installing a Linux application, replacing the display UI and flashing the MCU
are different operations. A program built for the Linux CPU cannot be flashed
as motion-controller firmware.

## Identifying a board

Record the printer variant, board revision, MCU marking and installed firmware
before selecting a pin map or image. The [Klippventurer installation guide](https://gitlab.com/synthread/proj/Klippventurer/-/blob/main/docs/installation.md)
distinguishes MCU vendors and installation paths; its pin maps and support notes
are a better starting point than assuming that all Adventurer 3-branded boards
are identical. Treat Pro 2 and rebrands separately until their hardware is known.

The classic Linux environment reports `ramips/mt7688` and has a little-endian
MIPS userspace. That target name is software metadata, not a complete physical
board identification. Likewise, a compiled Klipper MCU target does not identify
all compatible chip variants fitted to production boards.

## Interfaces relevant to modifications

| Interface | Role and useful distinction |
| --- | --- |
| Ethernet / Wi-Fi | Reach Linux services; the available protocol depends on the running firmware/application |
| USB storage | Vendor file transfer and boot/update hooks; see [firmware](firmware.md#usb-update-and-script-hook) |
| Internal UART | Linux-to-motion-controller communication; the documented serial bridge uses `/dev/ttyS1` |
| Linux framebuffer | `/dev/fb0` exposes display memory; image encoding is described under [development](development.md#framebuffer-graphics) |
| Touch / camera / other peripherals | Separate drivers and application integration; drawing a framebuffer does not establish working touch or camera support |

For a direct UART or programmer connection, check the actual header pinout,
logic levels and power arrangement against the board revision. A Linux device
name or GPIO number is not a connector pin number. Power down and disconnect
mains before opening the printer or changing wiring.

The [modified vendor boot script](https://github.com/vegardw/ffa3_ssh_installer/blob/9a6bba5550a56c7030b9495b290f2756684013e7/usb_stick/files/opt/auto_run.sh)
drives GPIO 19 during startup. Its electrical purpose is not established by the
script alone; it should not be used as an inferred reset/boot pin map.

## Alternative control arrangements

Klippventurer documents a direct-wiring approach to an external Klipper host.
A serial-over-TCP arrangement can instead retain the Linux board as a gateway to
its existing internal UART. This changes the transport, while the MCU still
needs suitable firmware. See [the protocol layers](protocols.md#serial-over-tcp).

Per-printer configurations and complete deployment instructions belong with the
implementation that supplies them. Use this reference to understand interfaces
and select a compatible approach.
