---
layout: default
title: Firmware and embedded Linux
---
# Firmware and embedded Linux

[Home](../) · [Hardware](hardware.md) · [Development](development.md)

## Linux frontend and motion firmware

The classic MIPS platform uses OpenWrt Chaos Calmer 15.05.1 and Linux `3.18.23`.
Linux exposes architecture, release and storage-layout information through:

```sh
uname -m
uname -r
cat /etc/openwrt_release
cat /proc/partitions
cat /proc/mtd
```

The vendor startup path launches `ffstartup-mipsle` from `/opt/PROGRAM/` with
`ffstartup.cfg`. Firmware packages contain separate components:

| Component | Role |
| --- | --- |
| `control` | Motion-MCU images and IAP/ISP flashing tools |
| `kernel` | Linux boot image |
| `software` | Frontend application/startup files |
| `library` | Userspace runtime dependencies |

Component `run.sh` scripts define the installation steps, while manifests list
content checksums. The updater includes architecture, kernel and model checks.

## Controller image selection

Controller families use different images and programming protocols. The
Linux-side tool selects an update path from the controller's responses and
records its classification in marker files. The
[MCU reference](mcu-variants.md) explains the families, selection logic and marker
semantics. Application-only images run after a resident loader.

## USB update and script hook

The [USB startup script](https://github.com/vegardw/ffa3_ssh_installer/blob/9a6bba5550a56c7030b9495b290f2756684013e7/usb_stick/files/opt/auto_run.sh)
looks for FAT-formatted USB storage and matching `Adventurer3*.tgz` update bundles.
It also supports a standalone `flashforge_init.sh` hook. The update bundle takes
priority over the standalone script when both are present.

The hook executes installation code as root during startup. This allows a USB
stick to install software or modify files without using the touchscreen UI.

## SSH access

[ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) adds an SSH
server and filesystem overlay through the USB startup hook. Its repository
contains the installation instructions and modified startup script.

## Backups and recovery material

Backup material includes firmware and component versions, checksums and storage
layout. Filesystem copies preserve files, raw MTD images preserve flash
partitions, and whole eMMC images preserve the eMMC device layout and contents.

Vendor bundles use MD5 manifests to compare component contents. These checksums
provide content-integrity checks rather than publisher authentication. Device
backups also contain private material such as network settings, device identity
and SSH keys.
