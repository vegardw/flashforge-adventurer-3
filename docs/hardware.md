---
layout: default
title: Hardware and interfaces
---
# Hardware and interfaces

[Home](../) · [MCU connections](gpio-and-mcu.md) · [Firmware](firmware.md) · [Protocols](protocols.md)

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

The Linux platform uses a MediaTek **MT7688AN** system-on-chip on a raised
module. It runs little-endian MIPS userspace and the OpenWrt `ramips/mt7688`
target. This Linux processor is separate from the motion MCU. Its operating
system and package ABI are described in the [OpenWrt guide](openwrt.md).
The motion controller has several MCU variants, described
in [MCU variants and factory firmware](mcu-variants.md#controller-families-and-their-role).
The MCU families use different peripheral layouts and factory update paths.

### Documented mainboards

Two documented mainboard populations share the `180515` revision suffix and a similar
component layout, while carrying different motion MCUs:

| Board marking | Motion MCU at U6 | Stepper drivers and sense resistors |
| --- | --- | --- |
| Visible revision `0.6_180515` | Hangshun **HK32F103RBT6**, LQFP64 | Four TMC2208-LA drivers; eight R220 resistors |
| **SZ16-MAIN BOARD V1.0.6_180515** | Nations **N32G455REL7**, LQFP64 | Four TMC2208-LA drivers; eight R220 resistors |

The N32 board also carries the MT7688AN Linux module. The shared revision suffix
does not identify the MCU manufacturer or establish a production cutover.
The documented markings and component positions do not establish every signal
route; a complete schematic for both assemblies is not documented here.

`TMC2208-LA` identifies Trinamic's UART-configurable stepper-driver IC. The
external resistors marked `R220` have a nominal resistance of 0.22 Ω and belong
to the motor-current sensing circuitry. Their value alone is not the selected
motor current. See the [TMC2208 datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2202_TMC2208_TMC2224_datasheet_rev1.14.pdf).

## Interfaces

| Interface | Role |
| --- | --- |
| Ethernet / Wi-Fi | Network connection to the Linux frontend and its services |
| USB storage | Print-file transfer and boot/update hooks; see [firmware](firmware.md#usb-update-and-script-hook) |
| Internal UART | Linux-to-motion-controller communication through `/dev/ttyS1`, mapped to hardware UART2 on the documented MT7688 platform |
| Reset and boot controls | Linux GPIOs select MCU reset/update states; see [connections and sequences](gpio-and-mcu.md) |
| Linux framebuffer | `/dev/fb0` exposes display memory; see [display format](development.md#framebuffer-graphics) |
| Touch input and camera | Linux drivers and frontend application integration |

Disconnect mains power before opening the printer or changing wiring.

## Mechanics and nozzles

The printer is Cartesian: the bed moves on Y, and the toolhead moves on X and Z.
During homing, the toolhead moves to the top and rightmost position. The nominal
build volume is 150 × 150 × 150 mm.

The [manufacturer manual](https://en.fss.flashforge.com/10000/software/2ff1e48618315861cfee296866c0392a.pdf)
describes the mechanics and lists 240 °C and 265 °C nozzle variants.
The replaceable 0.4 mm high-temperature cartridge is marked `0.4–265`; Flashforge's
[archived product listing](https://web.archive.org/web/20230401164747/https://www.flashforgeshop.com/product/nozzle-assembly-265-degree-for-flashforge-adventurer-3-series?cID=31)
identifies it as the 265 °C nozzle assembly for the Adventurer 3 series. Its
temperature designation does not specify its heater resistance or thermistor
manufacturer.

## Sensors and cooling

The motion controller reads the hotend and bed temperature sensors and drives
the heater and fan outputs.

The cartridge exposes separate heater and NTC sensor circuits through four
contacts. The small toolhead connector PCB labels the pairs **HOT** and **NTC**;
one documented larger toolhead PCB revision is **PRINT_HEAD V1.0.3**. A contact
order drawn from one face is reversed when viewed from the opposite face.
NTC means negative temperature coefficient: resistance falls as temperature rises.

Factory STM/HK firmware uses PA5 for the hotend temperature input and PC4 for
the bed. The same package positions have different ADC channel numbers on N32:

| MCU family, LQFP64 | PA5, package pin 21 | PC4, package pin 24 |
| --- | --- | --- |
| HK32F103x8/xB | ADC12_IN5 | ADC12_IN14 |
| N32G455 | ADC2_IN2 | ADC2_IN5 |

These channel assignments come from the Hangshun and Nations pin definitions
linked in the [MCU connection sources](gpio-and-mcu.md#sources). The N32 entries
describe chip capabilities, not a separately traced N32 sensor netlist. The
[factory temperature conversions](firmware.md#temperature-sampling-and-conversion)
also distinguish a software divider assumption from an actual resistor value.

All Adventurer 3 printers use a single toolhead fan for both part cooling and
heatbreak cooling. The fan and air guide direct airflow to both functions.
