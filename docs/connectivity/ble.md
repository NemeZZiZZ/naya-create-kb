# Bluetooth & GATT pipe

Each half is an independent **BLE peripheral**: HID over GATT + a
custom service. The host merges the two streams; there is no
inter-half radio.

## Stack facts

- SoC: Nordic **nRF52811** (coded-PHY evidence), Bluetooth 5.4,
  1M/2M/125k, up to +9 dBm. No proprietary radio.
- Separate **BLE FW v0.2.29** (`be/100f` → `00 02 1d` — not battery).
- BLE-side commands: see `BE` in the [command map](../protocol/commands.md)
  (pair addr, unpair, slots, dongle addr, CLEAR ALL SPLIT LINKS).
- USB serials double as device IDs (per-device values, private —
  not reproduced here).

## GATT map (confirmed, bonded via nRF Connect)

- `0x1800` Generic Access, `0x1801` Attribute,
  `0x180F` Battery (`0x2A19` NOTIFY — ticks 89–92%),
- `0x180A` Device Info (Model/Manufacturer/PnP),
- `0x1812` HID over GATT (standard keyboard/mouse/consumer reports),
- **`0x1234` custom service, single characteristic `0x5678`
  [Notify, Read, Write] + CCCD**.

## Pipe `0x1234/0x5678` verdict: NOT an input stream

- Unbonded READ → static byte `0x65`. Unbonded WRITEs accepted, no effect.
- Bonded: CCCD subscribe **succeeds**, but **zero notifications arrive
  during trackball/module activity** (~16 s window) while battery
  notifications keep flowing in the same window (subscription path OK).
- Trackball/touch input reaches the host exclusively via standard HID
  reports (Report ID 3 = mouse).

The pipe is most likely a host-initiated command/control channel
(pairing, config, module DFU?) — phone→keyboard, not keyboard→phone.
Bonded READ stays static `0x65` (`"e"`); value never changes, nothing
notifies — the pipe is idle in normal use.

## Open micro-steps

1. Bonded READ of DIS `0x180A` strings (free info).
2. Careful bonded WRITE probes (small, observed) — deferred, needs a
   wedge-recovery plan first.
