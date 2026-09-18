# Naya Create — Open Knowledge Base

The Naya Create is a split ergonomic keyboard (two BLE/USB halves +
docking input modules) by Naya B.V. (Groningen, NL). The company is
**bankrupt and there is no vendor support**, no published protocol, no
open firmware. This knowledge base is the community replacement for
the missing developer docs: everything here was recovered by probing
a live device over USB CDC and by disassembling the stock host
software.

## Start here

1. [Device overview](device/index.md) — what the product is, both halves,
   modules, dongle, firmware versions on the wire.
2. [Transport & framing](protocol/transport.md) — the one page you need to
   talk to the keyboard: frame layout, XOR checksum, sessions.
3. [Command map](protocol/commands.md) — every known `TYPE/C0C1` command.
4. [Recovery cookbook](recovery.md) — dark board? wedged after a bad write?
   Start here, not in the protocol pages.

## What is covered

- **Hardware** — FCC filings (grantee `2BQ4V`), SoCs, boards, radios,
  antennas, power path, debug pads.
- **Wire protocol** — full CDC frame format, keymap records (`T01…T10`),
  LED maps, ED LED engine, timeouts, module config, multi-part reads.
- **Storage** — external SPI flash + LittleFS, NVS settings keys,
  what survives reflash, the `30/10ca` factory format.
- **Firmware** — MCUBoot serial recovery (SMP), RSA-2048 signing story.
- **Host software** — NayaFlow/NayaCore/flow-bg-server, the
  RPC↔ZMQ bridge, SQLite app data, backup format.
- **Disassembly** — provenance, method, key functions, host-side maps.
- **Toolkit** — copy-paste [Python](toolkit/python.md) and
  [JS](toolkit/javascript.md) recipes.

## What is NOT here (yet)

- The identity of the USB MCU in the halves (**biggest open hardware
  question** — see [hardware](device/hardware.md)).
- A custom-firmware path (gated by the RSA-2048 signing key —
  see [signing](firmware/signing.md)).
- Macros on the wire (device store answers, always empty; host-only).
- Per-layer LED animations from stock software (registry exists, no sender).

Every page labels unverified items **OPEN** instead of guessing.
