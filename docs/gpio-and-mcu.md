---
layout: default
title: Linux GPIO and MCU connections
---
# Linux GPIO and MCU connections

[Home](../) · [Hardware](hardware.md) · [MCU variants](mcu-variants.md)

## Numbering and serial signals

Linux GPIO numbers refer to the MediaTek MT7688 system. MCU names such as PA9
refer to motion-controller ports; package pin numbers refer to physical leads.
These are three different numbering systems.

On the classic MT7688 platform documented here, `/sys/class/tty/ttyS1/device`
resolves to `10000e00.uart2`. Therefore `/dev/ttyS1` is **hardware UART2**. The tty index
does not identify the hardware UART number. MT7688 UART2 uses GPIO-capable
pins 20 for TX and 21 for RX; they operate as serial signals in this mode.

| Printer Linux signal | STM32/HK32 MCU endpoint, LQFP64 | Function |
| --- | --- | --- |
| GPIO 19 | NRST, pin 7 | Active-low MCU reset |
| GPIO 39 | BOOT0, pin 60 | Serial ISP boot selection |
| UART2 TX / GPIO-capable pin 20 | PA10 / USART1 RX, pin 43 | Linux sends to the motion MCU |
| UART2 RX / GPIO-capable pin 21 | PA9 / USART1 TX, pin 42 | Linux receives from the motion MCU |
| GPIO 41 | Physical endpoint not established | Update flag used by the stock IAP tool |

The stock Linux programming tools use these reset, boot and serial controls.
GPIO 41's software role is known; it has not
been mapped to BOOT1 or any other physical MCU pin.

The GPIO controller exposes banks at bases 0, 32 and 64, with 32 lines each.
GPIO 19 is bank 0 bit 19; GPIO 39 and 41 are bank 1 bits 7 and 9. The archived
OpenWrt driver uses fixed bank bases, also exposed through sysfs metadata.

These read-only queries show controller numbering and the tty device link:

```sh
cat /sys/class/gpio/gpiochip*/base
cat /sys/class/gpio/gpiochip*/ngpio
cat /sys/class/gpio/gpiochip*/label
readlink -f /sys/class/tty/ttyS1/device
```

The [MRAA MT7688 pin reference](https://github.com/eclipse-mraa/mraa/blob/master/docs/linkit_7688.md)
also documents UART2 TX/RX on GPIO 20/21. Its development-board connector numbers
are separate from the Adventurer 3's board connections.

## Stock programming controls

**ISP** accesses the MCU's serial programming interface. **IAP** uses a resident
update loader while retaining the factory application's separate flash region.
The [factory selection flow](mcu-variants.md#controller-update-paths) chooses the
available path from the controller's responses.

| Tool or path | Linux control sequence |
| --- | --- |
| `ISPAdventurer3ISP` and newer `ISPCommand` entry | Set GPIO 39 high, then pulse GPIO 19 low/high before the serial bootloader handshake |
| Newer `ISPCommand` normal cleanup | Set GPIO 39 low, reset through GPIO 19, then unexport both lines |
| `IAPCommand`, NA and LDW paths | Drive GPIO 41 low, wait one second, then reset through GPIO 19; later cleanup/fallback paths drive 41 high |

The IAP function named `stm32UpdateFLag` wraps `gpio_out(41, value)`. IAPCommand
itself does not set GPIO 39; its ISP fallback child does. The binary `C` and
ASCII `4D` IAP handshakes share these controls, with different frame encodings.

Reset routines in the stock tools use a 200 ms low hold. A programmer can also
hold reset low while setting the boot mode, then release reset:

| Phase | Ordered GPIO levels |
| --- | --- |
| Bootloader entry | 19 low → 39 high → 19 high |
| Application reset | 19 low → 39 low → 19 high |

BOOT0 is set before reset release in both entry orderings. Timing and loader
compatibility belong to the relevant programming tool and MCU family.

Resetting the MCU interrupts its control of motors, heaters and fans. GPIO
export, direction and value writes are hardware operations; the sequences above
describe stock behavior rather than a command sequence to run for inspection.

## Package pins across MCU families

The reviewed LQFP64 definitions share the following endpoint positions:

| MCU family / package | NRST | BOOT0 | PB2 / BOOT1 | PA9 serial TX | PA10 serial RX |
| --- | ---: | ---: | ---: | ---: | ---: |
| STM32F103R | 7 | 60 | 28 | 42 | 43 |
| HK32F103R, including RBT6 | 7 | 60 | 28 | 42 | 43 |
| N32G455R, including REL7 | 7 | 60 | 28 | 42 | 43 |
| MM32F103R, older xCxE family | 7 | 60 | 28 | 42 | 43 |

Other packages have different lead numbers. Shared package positions do not
establish identical PCB routing, peripheral registers or programming protocols.
The [documented HK32 and N32 mainboards](hardware.md#documented-mainboards) have
similar layouts and the same revision suffix. Their full reset/boot networks,
BOOT1 bias and GPIO 41 endpoint are not yet documented for each variant. No exact
MM32 part or corresponding mainboard is identified by the available evidence.

An external [annotated wiring image](https://github.com/synthread/Klippventurer/blob/d4e83e06f4a8d0d83c3356930f899916e97d0570/images/adv3-diagram.png)
locates accessible reset, boot-selection and UART solder points on the Linux
module's edge rows. Those access points are not the MCU package leads themselves.
The image is evidence for its depicted board, not a complete all-variant schematic.

## Sources

- Factory ISP/IAP program analysis and GPIO-controller/tty-device definitions
  from the classic Linux platform.
- ST [STM32F103xC/D/E datasheet](https://www.st.com/resource/en/datasheet/stm32f103re.pdf),
  Figure 7 and pin definitions.
- Hangshun's HK32F103x8xB datasheet v1.1.5, Table 5-1, through a
  [third-party copy](https://unirec.ru/upload/datasheets/HK32F103x8xBDatasheetRev.1.1.5.pdf).
- Manufacturer-hosted [N32G455 datasheet](https://www.nationstech.com/uploads/Microcontrollers/N32G455xx_V3.5.0/2-Datasheet/EN_DS_N32G455_Series_Datasheet_V2.6.0.pdf),
  Figure 3-4 and Table 3-1.
- MindMotion's older MM32F103xx datasheet **1.13_o**, Table 3, available through
  a [third-party listing](https://www.alldatasheet.com/datasheet-pdf/pdf/2248422/MINDMOTION/MM32F103.html).
  This is the xCxE family; the `_n` datasheet covers a different family reference.

The two mirrored datasheets are manufacturer-authored; original manufacturer
downloads were not authenticated.
