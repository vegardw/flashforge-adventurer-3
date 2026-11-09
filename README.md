# Flashforge Adventurer 3

A technical reference for experimenting with Adventurer 3 hardware, firmware,
slicers and software interfaces.

## Topics

| Topic | Contents |
| --- | --- |
| [Hardware and interfaces](docs/hardware.md) | Linux frontend, motion controller, board variants, UART and peripherals |
| [Linux GPIO and MCU connections](docs/gpio-and-mcu.md) | Reset, boot selection, UART signals and physical MCU pin assignments |
| [MCU variants and factory firmware](docs/mcu-variants.md) | Controller families, update paths and software-identification limits |
| [Firmware and embedded Linux](docs/firmware.md) | Boot scripts, USB updates, SSH access, firmware components and recovery material |
| [OpenWrt and package management](docs/openwrt.md) | Linux filesystem, opkg queries, compatible packages, feeds and offline installation |
| [Slicers and G-code](docs/slicers.md) | OrcaSlicer stock presets, vendor G-code and file-transfer compatibility |
| [Network and control protocols](docs/protocols.md) | Stock TCP console, camera stream, framing and diagnostic queries |
| [Development tools and projects](docs/development.md) | Cross-compilation, framebuffer graphics and useful reference implementations |

## Where to start

* **Printing with stock firmware:** start with OrcaSlicer's bundled Adventurer 3
  Series profile and the [slicer guide](docs/slicers.md#orcaslicer-with-stock-firmware).
* **Firmware and motion control:** the [Linux/MCU architecture](docs/hardware.md#system-architecture)
  explains the two processors and their responsibilities.
* **Writing software for the printer:** see the [development tools](docs/development.md)
  and [display format](docs/development.md#framebuffer-graphics).

The low-level Linux details cover the classic MIPS/OpenWrt platform.
