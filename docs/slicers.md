---
layout: default
title: Slicers and G-code
---
# Slicers and G-code

[Home](../) · [Network protocols](protocols.md) · [Firmware](firmware.md)

## OrcaSlicer with stock firmware

Start with the bundled **Flashforge Adventurer 3 Series** preset matching the
physical nozzle. The [OrcaSlicer v2.5.0 profile family](https://github.com/OrcaSlicer/OrcaSlicer/tree/v2.5.0/resources/profiles/Flashforge/machine)
includes 0.4 and 0.6 mm variants. Its
[common stock profile](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/resources/profiles/Flashforge/machine/fdm_adventurer3_common.json)
provides:

| Setting | Stock preset value |
| --- | --- |
| Build area | X/Y from -75 to +75 mm; origin at the center |
| Build height | 150 mm |
| G-code flavor | `marlin` |
| Start/end code | Flashforge-specific sequences, including homing and heating commands |

The flavor selection does not make stock firmware a general Marlin controller.
Keep the vendor-specific start/end sequences from the selected preset unless a
change has been checked against the target firmware.

A practical workflow is:

1. Select the stock preset and correct nozzle, then review the generated start,
   purge and end moves in preview.
2. Check how the chosen profile controls nozzle/bed temperature and fans; these
   behaviours are not interchangeable across firmware dialects.
3. Configure and test the stock [Flashforge print host](protocols.md#stock-flashforge-console)
   if using network transfer. Exported G-code through a supported vendor/USB
   workflow is an alternative when direct transfer has not been verified.
4. Validate a small job before treating a changed profile as reusable. Record
   the Orca and firmware versions when reporting a problem or sharing a preset.

Current presets are a better starting point than old workarounds. For example,
[Orca PR #9726](https://github.com/OrcaSlicer/OrcaSlicer/pull/9726), merged in June
2025, adjusted the purge line after FlashPrint rejected files as outside the
build plate. Earlier discussions can describe bugs already addressed upstream.

The preset fields and uploader are confirmed from released source; that alone
is not a new hardware test of every stock firmware variant.

## OrcaSlicer with Klipper

A converted printer needs a separately reviewed profile matching its active
Klipper configuration:

| Item | What to use |
| --- | --- |
| G-code flavor | Klipper |
| Origin, printable area and travel | Values verified for the converted machine; do not inherit the stock centered origin blindly |
| Start/end code | The installed macro names, temperature parameters and extrusion-mode contract |
| Print host | Moonraker on the external host; this is separate from the raw MCU serial bridge |
| Device UI | Fluidd/Mainsail, where installed; a working web page does not prove that file upload is configured correctly |

Orca's [Moonraker integration](https://github.com/OrcaSlicer/OrcaSlicer/wiki/release_2_4_0_beta)
provides a native Klipper host option; older versions also use an Octo/Klipper
compatibility path. Follow the options available in the actual Orca release and
test upload independently of the embedded Device tab.

Do not copy a complete personal `printer.cfg` into a generic slicer profile.
Nozzle size, thermal limits, motion bounds and calibration need their own checks.
Klipper can also run configured delayed G-code after startup: inspect those
macros before service or firmware restarts, since they can initiate movement.

## Other slicers and vendor commands

The important boundary is the generated dialect and transfer protocol, rather
than the slicer name. [PrusaSlicer community experiments](https://forum.prusa3d.com/forum/prusaslicer/has-anybody-successfully-used-prusa-slicer-with-the-flashforge-adventurer-3/)
remain useful background, but imported profiles need version and firmware checks.

For specific stock-firmware behaviour, compare Orca's current start/end code,
the [stock protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
and upstream issue reports. The [temperature/fan discussion](https://github.com/OrcaSlicer/OrcaSlicer/issues/4514)
is a record of reported behaviour, not a universal instruction to replace
commands in every current preset.
