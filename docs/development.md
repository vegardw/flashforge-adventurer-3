---
layout: default
title: Development tools and projects
---
# Development tools and projects

[Home](../) · [Hardware](hardware.md) · [Firmware](firmware.md)

## Cross-compiling Linux applications

The classic OpenWrt platform uses a little-endian MIPS userspace. Its reference
SDK is OpenWrt 15.05.1 with GCC 4.8 Linaro, uClibc 0.9.33.2 and a
`mipsel_24kec+dsp` toolchain, supplied as Linux-host binaries.

The executable ABI includes the CPU architecture, endianness, libc and dynamic
loader. These commands display a binary's format, ELF header and program headers:

```sh
file application
readelf -h application
readelf -l application
```

An executable requesting `/lib/ld-uClibc.so.0` requires that target dynamic loader
and compatible libraries.

## Framebuffer graphics

[png2ffa3fb](https://github.com/vegardw/png2ffa3fb) documents and implements the
raw image layout used for the classic Adventurer 3 display:

| Property | Converter format |
| --- | --- |
| Dimensions | 240 × 320 pixels |
| Storage | 16 bits per pixel; 153600 bytes without a file header |
| Pixel fields | RGBA5551: R bits 15–11, G 10–6, B 5–1, alpha bit 0 |
| Byte order | Little-endian |
| Orientation | Converter rotates 180° in both directions |

The decoder expands each five-bit colour channel by shifting left three, giving
values from 0 to 248 in steps of eight. Encoding composites RGBA input onto white
and sets the one-bit alpha field. This format differs from RGB565 despite using
the same number of bytes per pixel.

In [converter revision 78f82dc](https://github.com/vegardw/png2ffa3fb/blob/78f82dc085bddfbcedf031ac0c34ff088c0ec6d1/png2ffa3fb.py),
encoding with NumPy 2.2.4 loses high colour bits because channel shifts retain
eight-bit scalar types. Convert channels to Python integers before bit shifting
to avoid this overflow. NumPy explains the promotion change in its
[migration guide](https://numpy.org/doc/2.0/numpy_2_0_migration_guide.html#changes-to-numpy-data-type-promotion).

The [Linux framebuffer API](https://www.kernel.org/doc/html/latest/fb/api.html)
exposes runtime dimensions, row stride, offsets and channel layout through
`FBIOGET_FSCREENINFO` and `FBIOGET_VSCREENINFO`. The vendor frontend also draws to
framebuffer memory, so a replacement renderer shares the same display resource.

[LVGL's Linux framebuffer port](https://github.com/lvgl/lv_port_linux_frame_buffer)
provides a custom-UI framework with a Linux display backend. Its display driver
and input initialization are separate components.
[fbgrab](https://github.com/GunnarMonell/fbgrab) is a framebuffer screenshot tool.

## Reference implementations

| Project | Use it for |
| --- | --- |
| [ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) | USB-installed SSH access and startup-hook implementation |
| [png2ffa3fb](https://github.com/vegardw/png2ffa3fb) | PNG/raw-screen conversion and display-format reference |
| [Klippventurer](https://gitlab.com/synthread/proj/Klippventurer) | Alternative motion firmware for Flashforge printers; follow that project for its porting and setup details |
| [OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer) | Maintained slicing presets and print-host implementations |
| [Flashforge protocol research](https://github.com/Parallel-7/flashforge-api-docs) | Community firmware analysis and network protocol specifications |
