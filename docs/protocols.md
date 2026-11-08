---
layout: default
title: Network and control protocols
---
# Network and control protocols

[Home](../) · [Slicers](slicers.md) · [Hardware](hardware.md)

## Stock services

| Service | Endpoint | What it carries |
| --- | --- | --- |
| Flashforge console | TCP 8899 | Vendor control/status commands and print-file transfer |
| Optional camera stream | TCP 8080 | MJPEG video |

## Stock Flashforge console

The stock console uses text commands prefixed by `~`. A session requests control
with `~M601 S1` and releases it with `~M602`. The
[Adventurer 3 protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
documents command framing, responses and file-transfer sequencing.

| Command | Information reported |
| --- | --- |
| `~M115` | Printer and firmware information, including device identity |
| `~M105` | Temperature and target values |
| `~M114` | Axis positions |
| `~M119` | Endstop and motion/status fields |

OrcaSlicer v2.5.0's
[Flashforge backend](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/src/slic3r/Utils/Flashforge.cpp)
implements this TCP interface, including control acquisition and binary file
upload. This is the stock network-transfer path described in the
[slicer guide](slicers.md#orcaslicer-with-stock-firmware).

## Camera stream

The optional camera provides an MJPEG stream on TCP 8080, separate from the
printer-control console. Its endpoint is documented in the
[protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series).
