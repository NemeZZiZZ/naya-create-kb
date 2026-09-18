# Known problems & FAQ

Symptom → cause → fix, all verified live on base FW 0.3.41.0. Companion
page: [Recovery cookbook](recovery.md).

## Board is dark but keys still type

**Cause:** the LED runtime state is wedged (LittleFS/NVS side), not the
firmware. ED writes parse-ACK but no longer apply to the live state.

**Fix ladder** (stop at the first step that works):

1. `python3 toolkit/naya-undark.py --apply` (left half default).
2. Full winning phase: `naya-led-recover.py --phase ff --apply`.
3. Factory format: raw `30/10ca [01]` to the left port, then
   `naya-restore.py --snap <backup> --apply` to bring the profile back
   (see [Factory reset](storage/factory-reset.md)).
4. True cold boot: USB out **+** modules undocked **+** power switches.
   Flipping the switch with USB plugged in is NOT a reset.

**Do not bother:** NayaFlow reflash / clear keymap / repair SPI flash —
the wedged state lives in the LittleFS **data** partition, which
survives MCUBoot reflash. `repair_flash` only formats when the flash
self-test *fails* (healthy flash → mount + patch, ~1.3 s, no effect).
A logged `Status=success` in NayaFlow means the command ran, not that
it changed anything.

## Static light is dim, animation is bright / brightness keys inverted

**Cause:** the 1013 NVS brightness ceiling vs. the live brightness are
out of sync (same wedged-state family as above).

**Fix:** `python3 toolkit/naya-maxbrt.py --level 100 --apply`, then the
dark-board ladder if it persists. Note there is **no GET** for 1013 —
an empty read returns a bare `00` ACK; the value is set-only.

## Right half goes dark on USB, module stays lit, sync key blinks red

**Cause:** split-link/pairing state confusion on the right half.

**Fix:** unplug USB (it lights back up), then re-seat via true cold
boot. If halves unlink, re-pair from NayaFlow
(`create_pairing_start`). Clearing BLE bonds on the PC does not help.

## `Resource busy` / port won't open

**Cause:** NayaFlow (or its `flow-bg-server` child) holds the serial
port.

**Fix:** quit NayaFlow fully before any toolkit script. Golden rule —
check nothing else owns `/dev/cu.usbmodem*` first.

## Writes NACK on layers 1–2 but layer 0 works

**Cause:** the device **echoes the layer** in the write ACK
(`00 00` on L0, `00 01` on L1, `00 02` on L2). A strict `00 00`
matcher false-NACKs every L1/L2 write.

**Fix:** accept `00 00` (any layer) or `00 <layer>` — see
`isWriteAck(ack, layer)` in [JS recipes](toolkit/javascript.md).

## LED writes land on the wrong layer

**Cause:** `30/100e` LED params are `[00, layer, KK, H, S]` — the layer
byte is mandatory. Without it every write silently hits L0's map.

## Right half never answers

- The right half speaks **only** as dst `0x51` and never answers the
  `30/1001` handshake — use raw transact with explicit dst, not
  `Session()`.
- It never answers `ed/1014` (3× confirmed) — don't wait on it.
- To reach the right half from the left USB port, send dst `0x51`:
  the left half proxies the linked right half.

## ACK bytes don't mean what you think

- The second ACK byte is an **internal slot id**, not the written
  value (proven: constant `0x18` across BRT=50/100). Never parse it as
  data.
- Map observed live: `1013→03, 1010→00, 1003→13, 1012→02, 1014→04,
  1050→40, 1008→18, 1011→01`.

## Short ED payloads can zero NVS fields

**Suspected mechanism** (fits all observations): short payloads are
zero-filled on write — a 1-byte send can **write zeros** into NVS.
Always send the full documented form (`[target, value]`, see
[Settings](protocol/settings.md)).

## Battery % looks wrong / `de/100b` reads zero

- % is **host-computed** from `de/100b` millivolts, not a device value
  (≈4222 mV→100 %, ~9.3 mV/% in the mid region).
- With the dock empty, right reports `de/100b` zeros while left floats
  (~`0x1060`, unloaded charger rail). Use `de/1001` byte 1 as the true
  module-presence flag.

## Never send (DANGER list)

`ee/10be` (DFU reset), `ee/10ae` (MCUBoot reset), `fa/1002`,
`fa/1006`, `clear_bonds`, `mcuboot_reset`, replayed `fe/100a` bytes.
Background: the `fe/100a` commit is **unnecessary** — our writer is
handshake → `30/1004` → readback, no commit. Replaying sniffed
`fe/100a` frames wedged the state machine (proven live). `ee/10ce`
(NORMAL RESET) is the safe one: ACK, USB drops ~0.13 s, node back
~0.9 s.

## `clear_all_data` over ZMQ fails with `invalid_command_event`

Expected: the installed NayaCore's ZMQ whitelist has 15 events and
`clear_all_data` is not one of them. Send the wire op directly:
`30/10ca [01]` (see [Factory reset](storage/factory-reset.md)).
