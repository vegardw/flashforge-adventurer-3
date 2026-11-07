# Flashforge Adventurer 3

A technical reference for experimenting with Adventurer 3 hardware, firmware,
slicers and software interfaces.

## Topics

| Topic | Contents |
| --- | --- |
| [Hardware and interfaces](docs/hardware.md) | Linux frontend, motion controller, board variants, UART and peripherals |
| [Firmware and embedded Linux](docs/firmware.md) | Boot scripts, USB updates, SSH access, firmware components and recovery material |
| [Slicers and G-code](docs/slicers.md) | OrcaSlicer with stock firmware, Klipper profiles and vendor G-code differences |
| [Network and control protocols](docs/protocols.md) | Stock TCP console, serial forwarding, Moonraker and diagnostic queries |
| [Development tools and projects](docs/development.md) | Cross-compilation, framebuffer graphics and useful reference implementations |

## Where to start

* **Printing with stock firmware:** start with OrcaSlicer's bundled Adventurer 3
  Series profile and the [slicer guide](docs/slicers.md#orcaslicer-with-stock-firmware).
* **Changing firmware or controlling the MCU:** first identify the board and
  understand the [Linux/MCU split](docs/hardware.md#system-architecture).
* **Writing software for the printer:** check the target ABI and
  [display format](docs/development.md#framebuffer-graphics) before selecting tools.

Most low-level details here concern the classic MIPS/OpenWrt platform. Check
board revisions, MCU variants and firmware versions before applying them to a
Pro 2, a rebrand or another Adventurer model. Individual pages distinguish source
references and observed behaviour from capabilities that still need testing.
