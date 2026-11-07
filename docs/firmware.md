---
layout: default
title: Firmware and embedded Linux
---
# Firmware and embedded Linux

[Home](../) · [Hardware](hardware.md) · [Development](development.md)

## Linux frontend and motion firmware

Classic firmware inspected for this platform uses OpenWrt Chaos Calmer 15.05.1,
Linux `3.18.23` and a MIPS userspace. These are version-specific observations;
read the running system before choosing packages or a cross-toolchain:

```sh
uname -m
uname -r
cat /etc/openwrt_release
cat /proc/partitions
cat /proc/mtd
```

The vendor startup path launches `ffstartup-mipsle` from `/opt/PROGRAM/` with
`ffstartup.cfg`. Firmware packages contain separate components:

| Component | Role in the inspected updater layout |
| --- | --- |
| `control` | Motion-MCU images and IAP/ISP flashing tools |
| `kernel` | Linux boot image |
| `software` | Frontend application/startup files |
| `library` | Userspace runtime dependencies |

Inspect each component's `run.sh` and manifest. A matching package filename is
not sufficient evidence that its controller image or storage layout matches a
particular board. Some updaters check architecture, kernel and model; those
checks should be understood before attempting a modification.

## USB update and script hook

The [boot script used by ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer/blob/9a6bba5550a56c7030b9495b290f2756684013e7/usb_stick/files/opt/auto_run.sh)
looks for FAT-formatted USB storage and matching `Adventurer3*.tgz` update bundles.
It also supports a standalone `flashforge_init.sh` hook. The update bundle is
considered before the standalone script, so a USB stick containing both can
select an unintended operation.

The hook can run installation code as root. Treat it as an execution mechanism,
not ordinary print-file loading; inspect scripts and their file-copy/update
paths before booting with a maintenance stick attached.

## SSH access and boot controls

[ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) packages an SSH
server, filesystem overlay and modified `/opt/auto_run.sh` for USB installation.
Use its instructions and check the bundled package/configuration versions before
reusing them. The actual key file is `authorized_keys`; one preparation step in
the historical README misspells the name.

Its additional boot flags are implementation-specific:

| File | Effect in that script |
| --- | --- |
| `/opt/DONT_START_SOFTWARE` | Removed on use; restart networking and skip the vendor application for one boot |
| `/opt/KLIPPER` | Persistent; start a TCP-to-UART bridge and skip the vendor application |

The one-shot flag is checked first. Creating the Klipper flag does not flash the
MCU or install a Klipper host. The byte transport is described in
[serial over TCP](protocols.md#serial-over-tcp).

## Klippventurer

[Klippventurer](https://gitlab.com/synthread/proj/Klippventurer) supplies Flashforge
Klipper board/configuration work. Its `main` documentation describes a manual
hardware-assisted installation path. The separate
[USB installation preview](https://gitlab.com/synthread/proj/Klippventurer/-/tree/next/firmware-package)
is developing a packaged flow. Check the selected branch, hardware support and
release status instead of treating preview plans as a supported installer.

## Backups and recovery material

Preserve the original firmware identity, component versions, checksums and
storage layout alongside any backup. Filesystem copies, raw MTD images and a
whole eMMC image preserve different things; none is a tested restore procedure
merely because the file exists. Inspect the actual updater and target layout
before attempting a restore.

Vendor bundles may use MD5 manifests to compare component contents. A matching
manifest detects a content mismatch; it does not establish an authenticated
publisher. Backups can contain network settings, device identity and keys, so
publish sanitized research rather than raw device images.
