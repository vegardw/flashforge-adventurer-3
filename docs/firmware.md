---
layout: default
title: Firmware and embedded Linux
---
# Firmware and embedded Linux

[Home](../) · [Hardware](hardware.md) · [OpenWrt](openwrt.md) · [Development](development.md)

## Linux frontend and motion firmware

The MT7688 platform uses OpenWrt Chaos Calmer 15.05.1 and Linux `3.18.23`.
The [OpenWrt guide](openwrt.md) covers its filesystem, package management and
commands for inspecting the installed system.

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
Legacy component files ending in `.tar.xz` contain gzip-compressed tar archives;
the vendor script extracts them with `tar -xz`. Archive headers determine the
compression format, independently of the suffix.

## Controller image selection

Controller families use different images and programming protocols. The
Linux-side tool selects an update path from the controller's responses and
records its classification in marker files. The
[MCU reference](mcu-variants.md) explains the families, selection logic and marker
semantics. Application-only images run after a resident loader.

The programming tools are Linux executables. Their serial access, GPIO writes
and reset operations are active hardware operations even when their names or
output describe detection or connection checks. The
[GPIO guide](gpio-and-mcu.md#stock-programming-controls) documents their controls.

## Temperature sampling and conversion

Factory STM/HK application code samples the hotend and bed on separate ADC
inputs. Each conversion averages 20 samples from its interleaved ADC buffer:

| Sensor | Input in STM/HK code | Conversion path |
| --- | --- | --- |
| Hotend | PA5, ADC channel 5 | Derive resistance with `R = 4700 × A / (4095 − A)`, then interpolate a resistance/temperature table |
| Bed | PC4, ADC channel 14 | Interpolate a separate table of ADC counts and temperatures |

`A` is the averaged 12-bit ADC count. The hotend table describes a nominal
100 kΩ sensor at 25 °C. The 4700 Ω constant is the factory software's divider
assumption; it is not a measurement of a board resistor. The bed's separate
table does not independently establish a pull-up value or the same sensor curve.

The same table data and hotend calculation are present in the reviewed
STM/HK/NA/LDW application images; active input ordering was traced in STM/HK.
A table describes how software interprets a signal, not the independent
accuracy or exact manufacturer of the physical sensor. The
[sensor-interface overview](hardware.md#sensors-and-cooling) separates these
software findings from component markings and package pin definitions.

## USB update and script hook

The stock startup script, `/opt/auto_run.sh`, runs before the frontend
application. It tries to mount `/dev/sda1` to `/dev/sda4`, or an unpartitioned
`/dev/sda`, as FAT (`vfat`), and then checks the mounted stick:

1. If any `Adventurer3*.tgz` update bundle exists, the most recently modified
   one is copied to `/data`, extracted with `tar -xz`, and the bundle's own
   `flashforge_init.sh` is run. A stick containing a bundle never reaches the
   standalone hook below, even when the bundle fails.
2. Otherwise, a `flashforge_init.sh` in the stick's root directory is made
   executable and run.

Either script runs as root and receives the machine name and product ID
(`Adventurer3 0008`) as arguments. **If it exits with status 0, the startup
script then sleeps for 100000 seconds instead of starting the frontend**, so the
printer must be power-cycled without the stick. A non-zero exit unmounts the
stick and continues normal startup. This hook lets a USB stick install software
or modify files without using the touchscreen UI.

Two consequences matter when writing a hook script:

- A script that reports failure with a non-zero exit lets startup continue, and
  the frontend then draws over any error screen the script wrote to the
  [framebuffer](development.md#framebuffer-graphics). To keep an error visible,
  the script must hold it itself, for example by sleeping before exiting.
- Files copied from the FAT stick all arrive with the same owner and mode. Set
  owners and modes explicitly for anything that checks them, such as
  `authorized_keys` and other SSH files.

After the USB stage, the script drives GPIO 19 high, releasing the
[motion-MCU reset](gpio-and-mcu.md#stock-programming-controls), and restores the
Wi-Fi calibration file from the `factory` MTD partition if its header check fails.
The SSH installer's
[modified copy](https://github.com/vegardw/ffa3_ssh_installer/blob/28c19e754bc60e8d49eb3ad604b7854b029b1de9/usb_stick/files/opt/auto_run.sh)
keeps these stock branches and adds optional boot flags.

## SSH access

[ffa3_ssh_installer](https://github.com/vegardw/ffa3_ssh_installer) adds an SSH
server and copies configuration files through the USB startup hook. It requires
the user's public key, stops at the first failed step, verifies the result and
writes a step log to the stick. Its repository contains the installation
instructions and modified startup script.

The SSH server listens on every network the printer is on, including the
printer's own Wi-Fi access point. Use key-based login and keep that exposure in
mind when the access point is enabled.
The [OpenWrt page](openwrt.md#installing-a-package) also covers direct `opkg`
installation and USB-supplied package installation.

## Backups and recovery material

Choose backups to match the experiment. Configuration copies help undo file
edits; raw storage images preserve boot and filesystem structures that a file
archive cannot. Make the baseline while the printer is idle and no installation,
update or print-file transfer is changing the files being copied.

| Before changing | Preserve | Why |
| --- | --- | --- |
| Startup scripts, SSH or network configuration | `/etc`, root account files and relevant `/opt` application/startup files | Recover access and restore the original startup behavior |
| Packages or shared libraries | Installed package list, opkg metadata, configuration and matching package/dependency files; a consistent root-filesystem image for full rollback | A package list alone does not preserve executable files or their previous versions |
| Kernel, bootloader or storage layout | Partition/mount information, complete eMMC image and relevant MTD images | Preserve partition tables, boot components and filesystem contents |
| Device-specific settings | Factory/calibration partitions and device identity/configuration files | Retain the original unit's calibration and identifiers |
| Motion-MCU firmware | Exact MCU/board identity, current image or a supported flash readback, loader/offset information and matching programming tool | Linux/eMMC backups do not read the separate MCU's internal flash |

### File and configuration backup

Use a new destination directory on external storage, outside the trees being
archived. Replace `/path/to/external-storage` with its mounted location; `/tmp`
is RAM-backed and is not persistent backup storage.

```sh
backup_dir=/path/to/external-storage/adventurer3-backup
if mkdir "$backup_dir"; then
    {
        uname -a
        cat /etc/openwrt_release
        cat /proc/cmdline
        cat /proc/partitions
        cat /proc/mtd
        cat /proc/mounts
    } > "$backup_dir/system-info.txt"
    opkg list-installed > "$backup_dir/packages.txt"
    tar -czpf "$backup_dir/linux-files.tgz" -C / etc root opt usr/lib/opkg
fi
```

The archive retains configuration, root account material, vendor files and the
package database. Add needed print files or other `/data` content separately.
It does not include every Linux executable or library and is not a complete
package rollback. Preserve ownership, permissions and symlinks when restoring
individual files; copying an old opkg database alone does not restore packages.

Retain original vendor update bundles and component versions as recovery inputs.
A vendor bundle is an update source, not a copy of the printer's unique settings
or necessarily a complete bootable system image.

### Raw eMMC and MTD images

Raw filesystem images should be captured from an independent recovery environment
with their source filesystems unmounted. A live image of a writable mounted
filesystem can contain mutually inconsistent metadata and data. `sync` flushes
pending writes but does not freeze subsequent changes.

For a layout whose eMMC device is `/dev/mmcblk0`, this reads the entire device,
including its partition table and data partition, into a regular file:

```sh
dd if=/dev/mmcblk0 of="$backup_dir/emmc.img" bs=1M
```

For a **NOR** MTD partition identified as `mtd2` in the saved `/proc/mtd` table,
the read-only character-device interface, where provided, can be copied with:

```sh
dd if=/dev/mtd2ro of="$backup_dir/mtd2.img" bs=64k
```

On the documented MT7688 platform, `mtd2` is the 64 KiB `factory` partition. Its
start holds the Wi-Fi radio's calibration data: the stock startup script copies
its first 512 bytes to `/lib/firmware/mt7628.eeprom` when that file is damaged.
This data is unit-specific, so keep the image private and do not write another
unit's copy. Check the saved `/proc/mtd` labels before relying on a number, and
preserve the label, size and flash type alongside the image. NOR byte-image
examples are not NAND backup instructions: NAND adds bad-block, ECC and
out-of-band data considerations requiring suitable MTD tooling.

Keep `if=` as the source device and `of=` as a new regular backup file. A device
path used as `of=` would overwrite that device. Whole eMMC images may exceed
FAT32's approximately 4 GiB per-file limit; the destination filesystem must
support the image size. A short image or a command reporting read/write errors
is not a completed backup.

### Checking and retaining backups

Copy backups to another computer and inspect them there. For the file-backup
set above, run these commands in its copied directory using a Linux computer's
hash and archive tools:

```sh
sha256sum system-info.txt packages.txt linux-files.tgz > SHA256SUMS
sha256sum -c SHA256SUMS
tar -tzf linux-files.tgz > linux-files.contents.txt
```

Record checksums for raw images too, and compare image sizes with the saved
device/partition sizes. Listing an archive checks readability and contents;
matching hashes check copy integrity. Neither establishes that a restore has
been tested. Vendor MD5 manifests likewise check content integrity rather than
publisher authenticity.

Keep backups with their board identity, storage layout and recovery notes.
Network credentials, SSH keys and device identifiers make complete backups
private. Restoring another unit's factory partition or whole-device image can
also replace calibration and identity. Recovery therefore needs a matching board,
partition map and image purpose; no generic write-back command applies to all
Adventurer 3 variants.
