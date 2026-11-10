---
layout: default
title: OpenWrt and package management
---
# OpenWrt and package management

[Home](../) · [Firmware](firmware.md) · [Development](development.md)

## Platform and filesystem

The Adventurer 3 Linux frontend runs OpenWrt Chaos Calmer 15.05.1,
revision `r49389`, with Linux `3.18.23`. The MediaTek MT7688AN executes
little-endian MIPS code. OpenWrt package architecture `ramips_24kec` describes
this Linux userspace; it says nothing about the separate motion MCU's brand.

The shell and many basic commands come from BusyBox. Services use OpenWrt's
`/etc/init.d/` scripts and procd service manager. Instructions written for a
systemd-based desktop or a current OpenWrt release target a different environment.
The [OpenWrt init-script documentation](https://openwrt.org/docs/guide-developer/procd-init-scripts)
describes the service model; vendor application startup is covered in the
[firmware guide](firmware.md#linux-frontend-and-motion-firmware).

Useful commands for inspecting the installed system are:

```sh
uname -m
uname -r
cat /etc/openwrt_release
cat /proc/mounts
cat /proc/partitions
cat /proc/mtd
df -h
```

The eMMC layout documented here has writable ext4 system and data filesystems:
the kernel command line mounts `/dev/mmcblk0p1` as `/`, and `/dev/mmcblk0p2` is
mounted at `/data`. It does not use the read-only SquashFS plus writable overlay
arrangement common on OpenWrt routers. The `/rom` directory alone is not a copy
of the original system.

| Location | Purpose |
| --- | --- |
| `/` | Linux system files, libraries and installed packages |
| `/data` | Separate eMMC data partition |
| `/opt/PROGRAM` | Vendor startup and versioned update components |
| `/opt/finder_rush` | Frontend application files |
| `/opt/stm32` | Controller images and Linux-side programming tools |
| `/root` | Root account's home; also used for controller-classification markers |
| `/tmp` | RAM-backed temporary storage, lost at reboot |

MTD flash partitions and eMMC block devices are distinct storage resources. The
separate NOR flash reports `u-boot`, `u-boot-env`, `factory`, `firmware`, `kernel`,
`rootfs` and `rootfs_data` partitions in `/proc/mtd`; the running root filesystem
is nevertheless the eMMC partition above. The
[backup overview](firmware.md#backups-and-recovery-material) explains their
different contents. System-file and package changes on the ext4 root survive
reboots. USB installers can separately copy additional files into that hierarchy.

## Inspecting packages with opkg

`opkg` maintains the installed package database, resolves dependencies and
installs OpenWrt `.ipk` packages. These queries do not install or upgrade software:

| Command | Information |
| --- | --- |
| `opkg print-architecture` | Accepted package architecture names and priorities |
| `opkg list-installed` | Installed package names and versions |
| `opkg status strace` | Installed status and metadata for a package |
| `opkg info strace` | Package information from available metadata |
| `opkg files strace` | Paths registered to an installed package |
| `opkg search /usr/bin/strace` | Installed package owning a path |

`/usr/lib/opkg/status` holds installation records; `/usr/lib/opkg/info/` contains
per-package metadata, file lists and maintainer scripts. These records can
distinguish a package-managed tool from an unregistered copied executable.
They do not establish that a tool was supplied by Flashforge, and an
`Installed-Time` value depends on the system clock at installation.

## Feeds and package compatibility

`/etc/opkg.conf` supplies package-manager settings. Feed URLs are also stored in
`/etc/opkg/distfeeds.conf` and `/etc/opkg/customfeeds.conf` on this platform.
A feed is an index of packages available for a particular OpenWrt build.

Use the release, architecture and userspace ABI together when selecting an
archive. The printer's own `distfeeds.conf` names the `ramips/mt7688` target,
whose historical packages are in the
[OpenWrt Chaos Calmer archive](https://archive.openwrt.org/chaos_calmer/15.05.1/ramips/mt7688/packages/).
Its `ramips_24kec` package labels identify the userspace architecture. The
matching architecture label is necessary but does not establish every library
or kernel dependency.

The stock feed lines use `http://downloads.openwrt.org/chaos_calmer/...` URLs.
Those addresses now answer with a permanent redirect to the HTTPS archive, so
`opkg update` depends on a download tool that can follow the redirect and
complete a modern TLS connection. If the printer cannot fetch over HTTPS,
download the packages on another computer and install them offline as below.

Linux applications require compatible uClibc libraries and the expected dynamic
loader. Kernel modules additionally require the vendor kernel's matching build
and ABI. Current-release or snapshot feeds are not replacements for these legacy
feeds. See [ELF inspection and cross-compilation](development.md#cross-compiling-linux-applications).

## Installing a package

With compatible feeds configured and reachable, a typical root-shell operation is:

```sh
opkg update
opkg install strace
```

`update` refreshes package indexes; it does not upgrade the operating system.
For offline installation, transfer the selected `.ipk` and its required
dependencies to the printer, then install the local file:

```sh
opkg install /tmp/package.ipk
```

Here `package.ipk` stands for the actual compatible package. Local installation
does not supply missing dependencies automatically when their feeds are
unreachable. The [SSH installer](https://github.com/vegardw/ffa3_ssh_installer)
also supports USB-supplied packages in `usb_stick/packages`; its package stage
precedes file copying and scripts.

Examples of useful package roles are an SSH server for shell access, `strace`
for syscall tracing, and `stm32flash` for supported MCU serial bootloaders.
`stm32flash` is an OpenWrt package, not an assumed stock component. Installing it
does not flash or identify the motion MCU; running a programming command is a
separate operation. MCU compatibility is described in the
[controller reference](mcu-variants.md).

Package installation writes system files and can execute maintainer scripts as
root. Retain trusted package sources and dependency metadata. A bulk
`opkg upgrade` is unsuitable as a general printer update procedure: it can
replace the libraries and services expected by the vendor system. A generic
router `sysupgrade` image is likewise not an Adventurer 3 recovery image.

## Common package and executable problems

| Symptom | Cause or useful diagnostic |
| --- | --- |
| Package cannot be found | Missing/stale indexes, a disabled feed or a feed for another release; inspect feed configuration and `opkg update` errors |
| Dependency or architecture rejection | Package ABI, libraries or kernel requirements do not match; forcing the install does not repair that mismatch |
| HTTPS download fails | Old download tools, CA stores, an incorrect clock or server TLS compatibility can prevent access; a trusted download on another computer can be transferred for local installation |
| Existing executable gives “not found” | Its ELF interpreter can be absent; inspect `file` and `readelf -l` output on a development computer |
| Program fails to load a shared library | Its runtime dependencies are missing or incompatible; inspect ELF dependencies and package metadata |

OpenWrt's [opkg reference](https://openwrt.org/docs/guide-user/additional-software/opkg)
documents the commands. Its general
[package-upgrade guidance](https://openwrt.org/docs/guide-user/additional-software/show_upgradable_packages_after_ssh_login)
explains the risks of replacing packages independently of a firmware image;
the printer's ext4 layout differs from the router overlay examples there.
