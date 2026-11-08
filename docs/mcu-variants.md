---
layout: default
title: MCU variants and factory firmware
---
# MCU variants and factory firmware

[Home](../) · [Hardware](hardware.md) · [Firmware](firmware.md)

The Linux frontend and motion controller are separate computers. The motion
controller uses a **microcontroller (MCU)**: a chip combining a processor, program
flash, working memory and hardware interfaces. Its firmware handles timed motor
control, temperature sampling, heaters, fans and communication with the Linux
frontend. The Linux processor handles the display, files and network services.

## Controller families and their role

Names such as N32G45x and MM32F103xCxE identify chip families, not firmware formats
or communication protocols. The full part number specifies the memory, package
and available peripherals within a family.

| Name | What it is | Relevance to factory firmware |
| --- | --- | --- |
| STM / STM32 | STMicroelectronics' microcontroller range; STM32F103 belongs to its Arm Cortex-M3-based STM32F1 family | The factory updater has an STM image branch, shared with the programming tool's STM/GD classification. |
| HK / HK32 | Hangshun's microcontroller range, including the HK32F103 family | The factory updater has a separate HK image branch and records that classification in `/root/HK32`. |
| NA / N32G45x | [Nations Technologies](https://www.nationstech.com/product/general/n32g/n32g45x/) 32-bit Arm Cortex-M4F microcontrollers, with hardware floating-point support; the family includes N32G451/452/455/457 | Motion and peripheral control using timers, analog inputs and serial interfaces. NA firmware uses the N32G45x peripheral map and floating-point instructions. |
| LDW / MM32F103xCxE | [MindMotion](https://www.keil.arm.com/packs/mm32f103xcxe_dfp-mindmotion/devices/) 32-bit Arm Cortex-M3 microcontrollers; the name groups xC and xE subfamilies | Motion and peripheral control with a register layout specific to this family. LDW firmware uses this family's GPIO, ADC, UART and clock definitions, and the updater calls its writer MM32. |
| GD32 | [GigaDevice's](https://www.gigadevice.com/product/mcu) range of 32-bit microcontrollers, spanning multiple processor families | The fallback programming tool labels its shared image branch GD32/STM32. The exact GD32 part is not identified in the available factory material. |

A mapping from MCU families to production years, serial ranges or printer
submodels is not documented.

## Controller update paths

Factory updates separate Linux-side update programs from the MCU images they
send. `IAPCommand` and `ISPCommand` run on the MIPS Linux frontend and implement
the following selection flow:

| Stage | Selection behavior |
| --- | --- |
| Linux update script | Checks the platform and supplies the available controller image names to the programming tool |
| First IAP handshake | A byte `C` followed by acknowledgement `0x06` selects the NA path |
| Second IAP handshake | If the first fails, ASCII `4D` followed by `0x06` selects the LDW/MM32 path |
| ISP fallback | If neither IAP handshake succeeds after retries, the tool uses the controller's serial programming interface and Get ID response to choose its STM/GD or HK path |

The first accepted IAP protocol wins; this is protocol detection rather than a
manufacturer-ID comparison. The NA writer sends binary frames, while the MM32
writer encodes frame bytes as hexadecimal ASCII.

### Marker files and software classification

Both IAP paths create `/root/N32`, including after the LDW/MM32 writer. The
marker cannot distinguish NA from LDW, and the tool does not check the writer
result before creating it.

The ISP path selects HK for Get ID response bytes `04 10`, and otherwise takes
the STM/GD branch. It creates `/root/HK32` or `/root/STM32` before programming and
verification. This is a tool-specific heuristic: ST's
[AN2606](https://www.st.com.cn/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf)
also lists `0x410` for genuine medium-density STM32F10xxx devices.

The selection path chooses an image from controller responses, without reading
the marker files. Markers record the selected programming branch independently
of the programming result. The IAP fallback also reports success when the child
command was invoked, without decoding its exit status.

## Firmware compatibility and flash layout

Controller firmware depends on the chip's peripheral layout as well as its
processor instruction set. GPIO controls digital inputs and outputs;
ADC converts analog sensor signals; UART/USART carries serial traffic. Timers,
interrupt assignments, clock setup and DMA data-transfer behavior can also vary
between chip families.

STM/HK firmware shares startup code, with differences in timer selection,
secondary serial hardware and receive-event handling. NA/LDW also differ in
GPIO/ADC/UART register layouts, clock programming and floating-point instructions.

The IAP images contain application code linked above the start of flash. The
MCU-side update protocol and resident-loader layout remain undocumented.

## Sources

Family descriptions use manufacturer information and Arm's device-pack catalogue:
[STM32F103](https://www.st.com/en/microcontrollers-microprocessors/stm32f103.html),
[HK32F103 documentation](https://en.hsxp-hk.com/companyfile/1870362968418979840.html),
the Nations and MindMotion family links above, and
[MindMotion's device header](https://pack-content.cmsis.io/MindMotion/MM32F103xCxE_DFP/1.0.6/Device/MM32F103xCxE_o/Include/MM32F103xCxE_o.h).
The N32 peripheral comparison uses Nations-copyrighted source in a
[third-party SDK mirror](https://github.com/NationsTechCoreLib/N32G455xx/tree/5c43c149bfcde5bc37b90f711071883d3f57b0c5).

The update-path and firmware comparisons come from static analysis of factory
scripts, controller programming tools and MCU images.
