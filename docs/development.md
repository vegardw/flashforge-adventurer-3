---
layout: default
title: Development tools and projects
---
# Development tools and projects

[Home](../) · [Hardware](hardware.md) · [Firmware](firmware.md)

## Cross-compiling Linux applications

Match the target's CPU, endianness, libc and dynamic loader. The classic
OpenWrt-based platform uses a little-endian MIPS userspace. A reference SDK
identified for it is OpenWrt 15.05.1 with GCC 4.8 Linaro, uClibc 0.9.33.2 and a
`mipsel_24kec+dsp` toolchain, supplied as Linux-host binaries.

Inspect the actual SDK and running firmware before choosing a compiler or
package feed. A modern OpenWrt package or an x86-64 build of a utility is not
necessarily runnable on this target. Standard binary-inspection tools help
separate host and target artifacts:

```sh
file application
readelf -h application
readelf -l application
```

Check linked libraries and the requested interpreter as well as the architecture.
An executable requesting `/lib/ld-uClibc.so.0` needs the corresponding target
runtime. A completed build does not establish that display, input or printer
control works on hardware.

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
to avoid this overflow. Check the tool's current implementation and dependency
versions before generating display images; a correctly sized file can still
have wrong colours. NumPy explains the promotion change in its
[migration guide](https://numpy.org/doc/2.0/numpy_2_0_migration_guide.html#changes-to-numpy-data-type-promotion).

For a general renderer, query `FBIOGET_FSCREENINFO` and `FBIOGET_VSCREENINFO`
rather than assuming resolution, row stride, offsets or channel layout. The
[Linux framebuffer API](https://www.kernel.org/doc/html/latest/fb/api.html)
describes these fields. Display-memory writes also need coordination with any
running frontend that draws to the same device.

[LVGL's Linux framebuffer port](https://github.com/lvgl/lv_port_linux_frame_buffer)
is a starting point for a custom UI. Check the selected driver's colour format,
rotation and input initialization: some fbdev implementations assume RGB565 for
16-bit displays. Display output and touchscreen input require separate checks.
[fbgrab](https://github.com/GunnarMonell/fbgrab) is a screenshot-tool reference;
verify its target build and pixel-format handling before relying on it.

## Reference implementations

| Project | Use it for |
| --- | --- |
| [ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) | USB-installed SSH access, vendor boot-hook implementation and optional UART gateway |
| [png2ffa3fb](https://github.com/vegardw/png2ffa3fb) | PNG/raw-screen conversion and display-format reference |
| [Klippventurer](https://gitlab.com/synthread/proj/Klippventurer) | Flashforge Klipper support, board configurations and installation research |
| [OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer) | Maintained slicing presets and print-host implementations |
| [Flashforge protocol research](https://github.com/Parallel-7/flashforge-api-docs) | Community firmware analysis and network protocol specifications |

Inspect each project's supported versions, licence and current implementation.
Keep reusable fixes in the project that owns the implementation, and link its
source from the relevant topic instead of maintaining another copy here.
