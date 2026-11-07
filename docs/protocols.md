---
layout: default
title: Network and control protocols
---
# Network and control protocols

[Home](../) · [Slicers](slicers.md) · [Hardware](hardware.md)

## Choose the correct layer

| Service | Endpoint | What it carries |
| --- | --- | --- |
| Stock Flashforge console | TCP 8899 on the printer | Vendor control/status commands and print-file transfer |
| Serial-over-TCP gateway | TCP 5038 in `ffa3_ssh_installer`; configurable in other implementations | Bytes between a Klipper host and the motion MCU |
| Moonraker | Configured HTTP/API endpoint on the Klipper host | Job upload, printer API and web-client integration |
| SSH | Configured SSH endpoint | Linux maintenance; not a slicer or MCU protocol |

These services are not interchangeable. In particular, do not send stock text
G-code to a bridge connected to an MCU running Klipper firmware.

## Stock Flashforge console

The [community Adventurer 3 protocol reference](https://github.com/Parallel-7/flashforge-api-docs/wiki/Adventurer-3-Series)
documents the TCP 8899 interface, with text commands prefixed by `~`. A typical
session requests control with `~M601 S1` and releases it with `~M602`.
Common information queries include:

| Command | Information reported |
| --- | --- |
| `~M115` | Printer/firmware information; responses can include device identity |
| `~M105` | Temperature/target values |
| `~M114` | Axis positions |
| `~M119` | Endstop and motion/status fields |

Consult the reference and an implementation for framing, response parsing and
file-transfer sequencing. Avoid competing control sessions while another
application owns the printer, and redact device identifiers before publishing
raw responses.

Orca's [Flashforge backend](https://github.com/OrcaSlicer/OrcaSlicer/blob/v2.5.0/src/slic3r/Utils/Flashforge.cpp)
implements a legacy TCP path, including control acquisition and binary file
upload. It also has a newer HTTP local-API path; the presence of that path does
not give older printers the HTTP API of newer Flashforge models. Slicing support,
connection success, upload and successful printing are separate checks.

The community reference also describes an optional camera MJPEG service on TCP
8080 where camera hardware and its service are available. This is distinct from
the printer-control console.

## Serial over TCP

An external Klipper host can reach the existing internal UART through the
printer's Linux board:

```text
Klipper host -> local pseudo-terminal -> TCP client
            -> Ethernet -> printer TCP server -> UART -> Klipper MCU
```

The [ffa3_ssh_installer boot script](https://github.com/vegardw/ffa3_ssh_installer/blob/9a6bba5550a56c7030b9495b290f2756684013e7/usb_stick/files/opt/auto_run.sh)
uses a socat listener on port 5038 and `/dev/ttyS1` at 230400 baud when its
Klipper mode is selected. These are settings of that implementation, not a
vendor-standard network service or a universal MCU baud rate.

The host-side PTY lets Klipper open a normal serial-device path. Configuring the
PTY baud does not negotiate the real UART speed through a raw TCP stream: the
printer-side UART and MCU build must agree. The stream uses the
[Klipper binary host/MCU protocol](https://www.klipper3d.org/Protocol.html).
It does not provide boot-pin control or flash MCU firmware by itself.

Keep a single owner of the MCU link. The documented socat server has neither
authentication nor encryption and can accept multiple clients; use a controlled
network path and avoid starting the vendor frontend alongside the Klipper host.

## Diagnosing connection failures

Follow the chain instead of treating every error as a firmware problem:

1. Check whether the web UI and Moonraker can reach the Klipper process.
2. Read Klipper's actual serial path and error. A missing PTY, permission denial
   and MCU identification timeout point to different layers.
3. Check the client bridge process and ownership/permissions of the local PTY.
4. Check the printer-side listener and real UART settings.
5. Compare the configured baud and responding MCU firmware with the host build.

A TCP connection alone proves neither UART communication nor MCU compatibility.
Before restarting a recovered link, inspect configured delayed startup macros;
they may initiate homing or other movement when Klipper becomes ready.

[Moonraker's API documentation](https://moonraker.readthedocs.io/en/latest/external_api/introduction/)
is the reference for host-side HTTP requests. Use the normal API endpoint for
Orca uploads, rather than the raw serial gateway or stock Flashforge console.
