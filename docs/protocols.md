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
| Legacy printer discovery | UDP multicast `225.0.0.9:8899` | Probe/response discovery on the local network |
| Optional camera stream | TCP 8080 | MJPEG video |

The Adventurer 3 answers the legacy discovery protocol: a client sends any UDP
packet to the multicast group, and the printer replies. Newer Flashforge models
use a different discovery packet and an HTTP API that the Adventurer 3 lacks. The
community
[discovery reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Discovery-Protocol)
describes the 140-byte reply with the machine name, command port, product
identifiers and status, but no serial number. Retrieve the serial number over
TCP with `~M115`. The reference documents packet layouts and parsing details;
they are not duplicated here.

## Stock Flashforge console

The stock console uses text commands prefixed by `~`. A session requests control
with `~M601 S1` and releases it with `~M602`. The
[Adventurer 3 protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
documents command framing, responses and file-transfer sequencing.

| Command | Purpose |
| --- | --- |
| `~M115` | Printer and firmware information, including device identity |
| `~M105` | Temperature and target values |
| `~M114` | Axis positions |
| `~M119` | Endstop and machine/motion status |
| `~M27` | Print status and progress |
| `~M23 0:/user/<file>` | Select and start a stored print |
| `~M26` | Stop/cancel the current print |

`M119` response fields vary between firmware versions. Examples include
`MachineStatus`, `MoveMode`, and `Status`, with `LED` and `CurrentFile` also
present in a
[2021 response example](https://take4-blue.com/en/3d-printer/adventurer3-control-3/).
Clients should not assume every response has an identical set or order of
fields. The [community protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
also notes status-format variations.

### Firmware-management and upload variations

The control-session response and binary upload details are not identical across
all observed firmware generations. A
[2021 protocol analysis](https://take4-blue.com/en/3d-printer/adventurer3-control-3/)
describes an earlier `M28` upload as 4,112-byte blocks, padding the last block
with zeroes and reporting the transfer result at the end. A later
[analysis of software 1.3.1 with firmware-management version 2.1](https://take4-blue.com/en/3d-printer/adventurer3-control/adventurer3-wi-fi-connection-with-firmware-ver1-3-1/)
reports 4,096-byte data blocks, sending only the remaining bytes in the last
block and no confirmation from the printer that the data was received. These
are version-specific observations, not a universal block-size guarantee.

The later analysis also observed `V2.1` appended to the `M601` control
response. It reports that FlashPrint 4.6 treated that response as abnormal,
while FlashPrint 5.1 connected. This is historical compatibility evidence;
it does not establish behavior for current FlashPrint releases.

OrcaSlicer v2.5.0's
[Flashforge backend](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/src/slic3r/Utils/Flashforge.cpp)
implements this TCP interface, including control acquisition and binary file
upload. This is the stock network-transfer path described in the
[slicer guide](slicers.md#orcaslicer-with-stock-firmware).

For a fuller command catalog and state details, see the community
[Adventurer 3 reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series).
The project's [machine-readable command specification](https://github.com/Parallel-7/flashforge-api-docs/blob/main/endpoints/networkserver_commands_adventurer3.yaml)
identifies firmware v1.3.7 (2023-07-25); it is a useful structured companion,
not a guarantee that every firmware revision responds identically. The project's
HTTP API documentation concerns newer printer families and does not apply to
the Adventurer 3.

## Camera stream

The optional camera provides an MJPEG stream on TCP 8080, separate from the
printer-control console. Its endpoint is documented in the
[protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series).
