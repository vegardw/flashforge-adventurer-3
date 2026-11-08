---
layout: default
title: Hardware and interfaces
---
# Hardware and interfaces

[Home](../) · [Firmware](firmware.md) · [Protocols](protocols.md)

## System architecture

The Adventurer 3 separates a Linux frontend from the motion controller:

```text
Network / USB storage
        |
Linux frontend: display, files, network services and updates
        |
Internal UART
        |
Motion MCU: motors, heaters, fans and sensors
```

The Linux frontend runs the display application, network services and update
tools. The motion MCU runs separate firmware for motors, heaters, fans and
sensors. Linux applications and motion-controller images target different
processors.

## Processors and board variants

The classic Linux platform uses a little-endian MIPS userspace and the OpenWrt
`ramips/mt7688` target. The motion controller has several MCU variants, described
in [MCU variants and factory firmware](mcu-variants.md#controller-families-and-their-role).
The MCU families use different peripheral layouts and factory update paths.

## Interfaces

| Interface | Role |
| --- | --- |
| Ethernet / Wi-Fi | Network connection to the Linux frontend and its services |
| USB storage | Print-file transfer and boot/update hooks; see [firmware](firmware.md#usb-update-and-script-hook) |
| Internal UART | Linux-to-motion-controller communication through `/dev/ttyS1` |
| Linux framebuffer | `/dev/fb0` exposes display memory; see [display format](development.md#framebuffer-graphics) |
| Touch input and camera | Linux drivers and frontend application integration |

Disconnect mains power before opening the printer or changing wiring.

## Mechanics and nozzles

The printer is Cartesian: the bed moves on Y, and the toolhead moves on X and Z.
During homing, the toolhead moves to the top and rightmost position. The nominal
build volume is 150 × 150 × 150 mm.

The [manufacturer manual](https://en.fss.flashforge.com/10000/software/2ff1e48618315861cfee296866c0392a.pdf)
describes the mechanics and lists 240 °C and 265 °C nozzle variants.

## Sensors and cooling

The motion controller reads the hotend and bed temperature sensors and drives
the heater and fan outputs.

All Adventurer 3 printers use a single toolhead fan for both part cooling and
heatbreak cooling. The fan and air guide direct airflow to both functions.
