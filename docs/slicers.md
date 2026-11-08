---
layout: default
title: Slicers and G-code
---
# Slicers and G-code

[Home](../) · [Network protocols](protocols.md) · [Firmware](firmware.md)

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

[Orca PR #9726](https://github.com/OrcaSlicer/OrcaSlicer/pull/9726), merged in June
2025, adjusted the purge line after FlashPrint rejected files as outside the
build plate.

## Other slicers and vendor commands

The [PrusaSlicer community discussion](https://forum.prusa3d.com/forum/prusaslicer/has-anybody-successfully-used-prusa-slicer-with-the-flashforge-adventurer-3/)
covers profile experiments for the Adventurer 3. The
[stock protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
documents vendor commands, while Orca's
[temperature/fan issue](https://github.com/OrcaSlicer/OrcaSlicer/issues/4514)
collects firmware-specific temperature and fan-control reports.
