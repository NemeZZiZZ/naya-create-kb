# Device overview

The Naya Create is a **split keyboard**: a left half and a right half,
each an independent device with its own USB port, BLE radio, battery
buffer and firmware. Input **modules** (trackball / touchpad / dial)
dock onto either half over pogo pins. A USB **dongle** (“Speedlink”)
covers the desktop-receiver case.

| Piece | Name / ID | Firmware (observed) |
|---|---|---|
| Left half | per-device stock name + USB serial (private — redacted) | base **0.3.41.0** |
| Right half | per-device stock name + USB serial (private — redacted) | base **0.3.41.0** |
| Modules | Track / Touch / Tune (+ Float, Query unreleased) | module **0.2.3.3** |
| Dongle | NAYA-100-1 “Speedlink” | — |
| Host app | NayaFlow 1.25.1 / NayaCore v6.11.0 (macOS arm64) | — |
| BLE stack | — | BLE FW v0.2.29 |

## Split architecture (matters for everything else)

- **The left half is the merge host.** Keymaps, LED maps and module
  config live on the left and are reachable **only** via the left USB
  port (`30/10xx` family). The right half answers telemetry
  (`fa/be/de/fe`) but never `30/1001` — by design, not a defect.
- Each half is an independent **BLE peripheral** (HID over GATT + a
  custom `0x1234` service). The host merges the two streams.
  **There is no radio link between halves.**
- When the halves are USB-linked, **left proxies the right**: the left
  port answers `dst 0x50` (left) *and* `dst 0x51` (right); the right
  port answers only `dst 0x51`. Replies to `dst 0x51` arrive with SRC
  byte `0x50` — parse the SRC nibble, not the port.
- Module enablement is **per-half in the keymap profile**: a module on
  a half whose profile disables it shows only its indicator LED.

## Operating modes

- **USB mode** — plugged in: CDC serial + HID. All research in this KB
  was done here.
- **BT mode** — up to several bonded hosts (BT slot keys), HID over GATT.
- **Asleep halves answer nothing** on CDC. Wake the board (press keys)
  before talking to it; the first frame after wake is often lost —
  retry once.

Further reading:

- [Hardware deep dive](hardware.md) — chips, boards, radios, FCC.
- [Power architecture](power.md) — who powers whom, batteries, rails.
- [Transport & framing](../protocol/transport.md) — how to open a session.
