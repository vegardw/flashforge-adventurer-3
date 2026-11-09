---
layout: default
title: Slicers and G-code
---
# Slicers and G-code

[Home](../) · [Network protocols](protocols.md) · [Firmware](firmware.md)

## Choosing a slicer

Stock firmware needs an Adventurer 3 profile that supplies the build geometry
and Flashforge start/end commands. Slicing, file packaging and network upload
are separate parts of the workflow.

| Slicer | Adventurer 3 workflow |
| --- | --- |
| [OrcaSlicer](#orcaslicer-with-stock-firmware) | Bundled stock-firmware presets and a Flashforge print host |
| [UltiMaker Cura](#ultimaker-cura) | Bundled community profiles, with an earlier history of manual setup and plugins |
| [FlashPrint](#flashprint) | Flashforge's original standalone slicer, with Adventurer 3 setup and network transfer |
| [Orca-Flashforge / Flash Studio Desktop](#orca-flashforge-and-flash-studio-desktop) | Flashforge's newer Orca-based offering; its advertised model support focuses on newer printers |

## OrcaSlicer with stock firmware

OrcaSlicer includes a **Flashforge Adventurer 3 Series** preset family for stock
firmware. The [v2.5.0 profile family](https://github.com/OrcaSlicer/OrcaSlicer/tree/v2.5.0/resources/profiles/Flashforge/machine)
includes 0.4 and 0.6 mm variants. Its
[common stock profile](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/resources/profiles/Flashforge/machine/fdm_adventurer3_common.json)
provides:

| Setting | Stock preset value |
| --- | --- |
| Build area | X/Y from -75 to +75 mm; origin at the center |
| Build height | 150 mm |
| G-code flavor | `marlin` |
| Start/end code | Flashforge-specific sequences, including homing and heating commands |

The `marlin` flavor is the slicer's dialect setting. The stock preset supplies
Flashforge-specific start/end commands for homing, heating and print preparation.
Network uploads use the [Flashforge print host](protocols.md#stock-flashforge-console).
Automatically generated temperature and fan commands also need the
[dialect compatibility treatment](#comments-and-ignored-commands) described below.

[Orca PR #9726](https://github.com/OrcaSlicer/OrcaSlicer/pull/9726), merged in June
2025, adjusted the purge line after FlashPrint rejected files as outside the
build plate.

## UltiMaker Cura

Flashforge published an official
[Cura setup guide for the Adventurer 3 series](https://en.fss.flashforge.com/10000/software/59578ec5bff6b5671194cde932f0e99f.pdf),
and community projects supplied printer definitions and output plugins before
the profiles were bundled. Cura's
[5.7 changelog](https://github.com/Ultimaker/Cura/blob/5.7.0/resources/texts/change_log.txt)
records the addition of community-contributed Adventurer 3 family definitions.

Select **Flashforge → Adventurer 3** in Cura's printer list as the starting point
for a stock machine. The
[Cura 5.14.0 profile](https://github.com/Ultimaker/Cura/blob/5.14.0/resources/definitions/flashforge_adventurer3.def.json)
uses [shared Adventurer 3 machine settings](https://github.com/Ultimaker/Cura/blob/5.14.0/resources/definitions/flashforge_adventurer3c.def.json)
with [Marlin output and centered coordinates](https://github.com/Ultimaker/Cura/blob/5.14.0/resources/definitions/flashforge_adventurer_base.def.json).
The official manual setup guide specifies:

* A rectangular **150 × 150 × 150 mm** build volume, with the origin at the center.
* A heated bed, one extruder and **1.75 mm** filament.
* **Marlin** G-code flavor and a **0.4 mm** nozzle for the standard setup.
* Printer-specific start/end sequences, including `M132`, `M6`, `M7`, `M651`
  and `M652`.

The guide does not identify its Cura version. It is useful background for the
machine settings; the bundled definitions provide a versioned implementation.
Selecting a printer profile configures slicing; file output and upload still
need a compatible path. The
[FlashForge Cura Integration project](https://github.com/mtongnz/FlashforgeCuraIntegration)
documents a GX writer ported to Cura 5, but its recorded testing covers
**5.0.0-beta+1**. Use that as historical plugin documentation, rather than a
compatibility claim for current Cura releases. The file formats and transfer
options are explained [below](#g-code-files-and-transfer).

## Flashforge's own slicers

### FlashPrint

**FlashPrint** is the older standalone Flashforge application associated with
the Adventurer 3. The
[original Adventurer 3 manual](https://en.fss.flashforge.com/10000/software/e1bc9c105b1f9983ae67f1f94e47690b.pdf)
names it as the supplied slicer and describes selecting the printer, slicing,
and transferring jobs over Wi-Fi or Ethernet. It also documents printing `.g`
and `.gx` files from a USB stick.

FlashPrint therefore provides both a vendor slicing workflow and a way to send
compatible files prepared by another slicer. Orca's purge-line fix above is an
example of the latter workflow. Flashforge's
[official software download page](https://www.flashforge.com/blogs/download-center/software)
still lists **FlashPrint 5.8.7** alongside the newer applications, as of November
2026.

### Orca-Flashforge and Flash Studio Desktop

Flashforge later introduced **Orca-Flashforge**, a separate application based on
OrcaSlicer. Its [source repository](https://github.com/FlashForge/Orca-Flashforge)
identifies that lineage. Flashforge now calls this product
**[Flash Studio Desktop](https://www.flashforge.com/pages/flash-studio-desktop)**;
the current product page explicitly identifies it as the renamed Orca-Flashforge.
FlashPrint remains a separate application in the download catalogue.

The current Flash Studio Desktop page advertises support for newer machines,
including the Adventurer 5M series, AD5X, Creator 5 series and Guider 3 Ultra.
It does not list the Adventurer 3. These are separate distributions: the
Adventurer 3 presets and legacy Flashforge upload backend described above belong
to upstream OrcaSlicer. For the Adventurer 3, the documented vendor workflow is
FlashPrint.

## G-code, files and transfer

### Dialect and start/end sequences

The **Marlin** setting in Cura and Orca selects an output dialect; it does not
identify the printer's firmware as Marlin or establish support for every Marlin
command. Flashforge uses familiar movement commands alongside vendor-specific
heating, tool-selection and fan commands. The
[Flashforge command reference](https://github.com/Mrnt/OctoPrint-FlashForge/wiki/G-Code-Reference),
read alongside Flashforge's Cura guide, explains several commands used in the
stock profiles:

| Command | Role in the Flashforge dialect |
| --- | --- |
| `M104 S… T0` | Set the nozzle temperature target |
| `M140 S… T0` | Set the bed temperature target |
| `M6 T0` | Wait for the selected extruder to reach its target |
| `M7 T0` | Wait for the bed to reach its target; this is the form used in the official Cura start code |
| `M108 T0` | Select the active extruder; this differs from [Marlin's break/continue command](https://marlinfw.org/docs/gcode/M108.html) |
| `M132 X Y Z A B` | Load stored home offsets, as used in the vendor homing sequence |

Keep the complete homing/heating/shutdown sequences tied to their printer
profiles. A command's meaning on another Flashforge model can also differ:
the shared command reference describes `M651` as a position query, whereas
[Adventurer 3 observations](https://github.com/OrcaSlicer/OrcaSlicer/issues/4514#issuecomment-3086490955)
identify it as enclosure-fan control.

The [stock protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
documents the console commands and file-transfer exchange. The `~` prefix in
console examples is transport framing; it is separate from commands in a sliced
print file.

### Cooling fan control

The stock toolhead fan interface is **on/off**, rather than a general speed
control interface. The original
[Adventurer 3 fan-control investigation](https://github.com/RikshaDriver/adventurer3-orcaslicer-profile/issues/2)
found that the firmware ignored the `S` parameter in `M106`, including the
zero in `M106 S0`. This differs from
[Marlin's `M106`](https://marlinfw.org/docs/gcode/M106.html), where `S0` stops
the fan and intermediate values request a proportion of full output.

| Intended action | Stock Adventurer 3 command or limitation |
| --- | --- |
| Enable toolhead cooling | `M106` without parameters |
| Disable toolhead cooling | `M107` without parameters; replace a generic slicer's `M106 S0` with this |
| Request an intermediate toolhead speed | No equivalent speed setting in the documented stock interface; `M106 S128` does not provide 50% cooling |
| Control the enclosure exhaust fan | Separate `M651 S…` / `M652` commands, described below |

The toolhead has one fan shared by part cooling and heatbreak cooling; see
[sensors and cooling](hardware.md#sensors-and-cooling). An auxiliary fan channel
in a slicer profile does not represent an additional physical part-cooling fan.

The [2025 Adventurer 3 fan observations](https://github.com/OrcaSlicer/OrcaSlicer/issues/4514#issuecomment-3086490955)
describe `M651 S0`–`M651 S255` for the separate enclosure exhaust fan and `M652`
for off. Generic `M106 P3` / `M107 P3` exhaust-fan commands do not address it.
The same investigation found that toolhead fan commands took effect ahead of
queued movement: a fan change placed after a layer could happen before that
layer finished. Rapidly alternating on/off commands therefore cannot reproduce
reliable variable-speed control, and bridge-specific cooling changes may occur
too early.

### Comments and ignored commands

An inline semicolon comment can cause the **command itself to be skipped**.
The original [temperature-control test](https://github.com/RikshaDriver/adventurer3-orcaslicer-profile/issues/1)
found that a temperature-tower command with a trailing comment was ignored;
removing the comment restored the temperature change. For example, this form
can lose the heating command:

```gcode
M104 S200 T0 ; set nozzle temperature
```

Keep executable commands free of trailing comments. Put explanatory comments
on their own lines instead:

```gcode
; set nozzle temperature
M104 S200 T0
```

The [April 2025 parser observations](https://gist.github.com/rmeissn/f24294a20b57c870abf1c9e760e54954?permalink_comment_id=5526856#gistcomment-5526856)
describe this more broadly: a line containing `;` is discarded in full. When
normalizing a file, preserve the command before the semicolon; deleting every
line containing `;` would also delete movement, heating or shutdown commands.
Comments that already occupy whole lines can remain.

The [combined Orca report](https://github.com/OrcaSlicer/OrcaSlicer/issues/4514)
reproduced the fan/comment failures with **OrcaSlicer 2.0.0 Beta** and an
**Adventurer 3 Pro 2** in March 2024. The later observations cover an Adventurer
3, but these reports do not identify printer firmware build numbers.

These output patterns are also present in the
[OrcaSlicer v2.5.0 G-code writer](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/src/libslic3r/GCodeWriter.cpp):
its generic Marlin fan-off path emits `M106 S0`, and its nozzle/bed temperature
writers append inline comments. Those temperature comments are emitted
independently of the verbose-comment option. Consequently, editing start/end
code or disabling verbose comments alone does not address those generated
commands.

### Other parser differences

The [Adventurer 3 Cura investigation](https://gist.github.com/rmeissn/f24294a20b57c870abf1c9e760e54954)
also records integer-only temperature arguments, absolute extrusion, and
compatibility problems with `G0` and combined XYZ moves during mesh leveling.
Its conservative output uses integer targets such as `S200`, `G1` moves and
separate Z and XY moves. These are additional observations from that firmware
investigation, rather than a complete command-support list for every revision.

The [print-boundary investigation](https://gist.github.com/rmeissn/f24294a20b57c870abf1c9e760e54954?permalink_comment_id=5290040#gistcomment-5290040)
identifies `M118 X… Y… Z… T0` as a print-volume declaration used in file
validation. Purge lines and other generated moves must fit those declared
bounds as well as the physical build area. This is another vendor meaning:
Marlin documents [`M118`](https://marlinfw.org/docs/gcode/M118.html) as serial
text output.

### File formats and delivery

The original printer manual documents `.g` and `.gx` as print-file formats.
A `.g` file contains text G-code. A `.gx` file adds a binary header with a
thumbnail and print metadata before the G-code, as documented by the
[GX writer project](https://github.com/ronoaldo/FlashforgeFinderIntegration#about-the-support-for-gx-xgcode-files).
Renaming a file to `.gx` does not create that header or translate its commands.
For plain-text output saved as `.gcode`, using a `.g` extension addresses the
filename convention; the contents still need the Adventurer 3 dialect and
start/end sequences.

Delivery options are a USB stick, FlashPrint's network transfer, or a compatible
implementation of the stock **TCP 8899** upload protocol, such as OrcaSlicer's
Flashforge print host. A cloud account is unnecessary for the local USB and
console paths. See [network protocols](protocols.md#stock-flashforge-console)
for the transport details.

## Other slicers

The [PrusaSlicer community discussion](https://forum.prusa3d.com/forum/prusaslicer/has-anybody-successfully-used-prusa-slicer-with-the-flashforge-adventurer-3/)
covers profile experiments for the Adventurer 3. It is a useful reference for
adapting profiles, while the bundled Orca and Cura definitions provide more
direct starting points.
