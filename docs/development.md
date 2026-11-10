---
layout: default
title: Development tools and projects
---
# Development tools and projects

[Home](../) · [Hardware](hardware.md) · [Firmware](firmware.md) · [OpenWrt](openwrt.md)

## Cross-compiling Linux applications

The MT7688 platform uses a little-endian MIPS userspace. Its reference
SDK is the OpenWrt 15.05.1 `ramips/mt7688` SDK with GCC 4.8 Linaro, uClibc
0.9.33.2 and a `mipsel_24kec+dsp` toolchain, supplied as x86-64 Linux-host
binaries. The
[archived SDK](https://archive.openwrt.org/chaos_calmer/15.05.1/ramips/mt7688/OpenWrt-SDK-15.05.1-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2)
matches the target named in the printer's package feeds.

The executable ABI includes the CPU architecture, endianness, libc and dynamic
loader. These commands display a binary's format, ELF header and program headers:

```sh
file application
readelf -h application
readelf -l application
readelf -d application
```

An executable requesting `/lib/ld-uClibc.so.0` requires that target dynamic loader
and compatible libraries.
An “existing file not found” error can therefore refer to its missing interpreter.
The dynamic section lists needed shared libraries; it does not by itself prove
that the versions installed on the printer are ABI-compatible.

### SDK packages

The OpenWrt SDK builds Linux userspace applications and `.ipk` packages for this
ABI. It does not build the motion MCU's program merely because both processors
are on one board. Its package recipes describe source, dependencies, licence
and installation paths. `CONFIG_PACKAGE_name=m` selects a package to build as
a separate installable artifact; it does not include it in a base image or prove
that a build completed. Installation is handled by
[opkg or the USB package stage](openwrt.md#installing-a-package).

## Inspecting native programs

`strings` and `readelf` inspect executables as data. Disassembly can recover
constants and call sequences when source is unavailable. Symbols, relocations
and architecture-specific details matter: MIPS instructions after a branch or
call can execute in a delay slot, and stripped binaries may lack function names.
Adjacent code is not necessarily one function. A named but uncalled routine is
not evidence that the normal program path executes it.

`strace` records system calls by **running** or attaching to a program. Its `-f`
option follows children; `-o` saves output, and `-s` controls string truncation.
Without child following, shell commands launched through `system()` can be
absent even though the parent shows process creation and waits. Options vary
with the installed strace version.

Tracing a vendor programming tool can reset or program the MCU. Such a trace
is not a read-only substitute for static analysis. Saved traces can be inspected
without replaying their commands. Exit status zero or a success message is also
insufficient when a tool ignores an underlying error; the
[stock updater's classification and result handling](mcu-variants.md#marker-files-and-software-classification)
provides concrete examples.

The [strace manual](https://man7.org/linux/man-pages/man1/strace.1.html) and
[GNU Binutils documentation](https://sourceware.org/binutils/docs/binutils/)
describe these development tools.

## Framebuffer graphics

The Adventurer 3's display (`fb_ili9341`) is a 240 × 320 framebuffer at
`/dev/fb0` with this layout:

| Property | Value |
| --- | --- |
| Dimensions | 240 × 320 pixels |
| Storage | 16 bits per pixel, 480-byte rows; 153600 bytes, no header |
| Pixel fields | RGB565: R bits 15–11, G 10–5, B 4–0, no alpha |
| Byte order | Little-endian |
| Orientation | Mounted upside down: rotate images 180° before writing |

The driver reports these fields at runtime (`R11/5 G5/6 B0/5 A0/0`), and RGB565
colour and orientation test charts display correctly on the panel. The
[Linux framebuffer API](https://www.kernel.org/doc/html/latest/fb/api.html)
exposes dimensions, row stride, offsets and channel layout through
`FBIOGET_FSCREENINFO` and `FBIOGET_VSCREENINFO`; the same values appear under
`/sys/class/graphics/fb0/`. Check them before writing raw bytes on other firmware.

A raw screen can be saved, replaced and restored from a shell:

```sh
dd if=/dev/fb0 of=/tmp/saved.img bs=153600 count=1
cat image.img > /dev/fb0
cat /tmp/saved.img > /dev/fb0
```

The vendor frontend also draws to framebuffer memory and can overwrite the image
while it runs, so a replacement renderer shares the same display resource.

[png2ffa3fb](https://github.com/vegardw/png2ffa3fb) converts PNG images to and
from this format, including the rotation; its repository has the usage details
and device test charts. Revisions before
[`f9c62e3`](https://github.com/vegardw/png2ffa3fb/commit/f9c62e3c53bf943e455b80be2c0783ad18c9051a)
wrote RGBA5551 (green in bits 10–6, an alpha bit in bit 0), which shows shifted
colours on this display, and lost high colour bits under NumPy 2 because channel
shifts kept eight-bit scalar types (see NumPy's
[promotion change](https://numpy.org/doc/2.0/numpy_2_0_migration_guide.html#changes-to-numpy-data-type-promotion)).
Regenerate images made with those revisions.

[LVGL's Linux framebuffer port](https://github.com/lvgl/lv_port_linux_frame_buffer)
provides a custom-UI framework with a Linux display backend. Its
[fbdev driver](https://github.com/lvgl/lvgl/blob/master/src/drivers/display/fb/lv_linux_fbdev.c)
selects RGB565 for any 16-bit framebuffer, which matches this display. Display
output and touch input are initialized separately.
[fbgrab](https://github.com/GunnarMonell/fbgrab) is a framebuffer screenshot tool.

## Reference implementations

| Project | Use it for |
| --- | --- |
| [ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) | USB-installed SSH access and startup-hook implementation |
| [png2ffa3fb](https://github.com/vegardw/png2ffa3fb) | PNG/raw-screen conversion and display-format reference |
| [Klippventurer](https://gitlab.com/synthread/proj/Klippventurer) | Alternative motion firmware for Flashforge printers; follow that project for its porting and setup details |
| [OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer) | Maintained slicing presets and print-host implementations |
| [Flashforge protocol research](https://github.com/Parallel-7/flashforge-api-docs) | Community firmware analysis and network protocol specifications |
