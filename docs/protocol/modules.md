# Modules on the wire

Module config lives on the **left half**: `30/100b` read (params
`[part, layer]`), content on L1 (L0/L2 all-zero 3 B records).
Records follow the universal rule: `[SLOT, FAMILY, LEN, payload]`.

## Dock matrix (live seat/remove/swap)

`de/1001` = `[00, PRESENT, TYPE|HALF, X]`:

| Setup | Payload | Read |
|---|---|---|
| L+Track | `00 01 20 30` | present, type `0x20` |
| L+Touch | `00 01 10 00` | present, type `0x10` |
| L+empty | `00 00 f0 e1` | absent (rest garbage) |
| R+Touch | `00 01 11 01` | present, `0x10\|1` |
| R+Track | `00 01 21 31` | present, `0x20\|1` |

TYPE: Touch=`0x10`, Track=`0x20`, bit0 = half (0 left, 1 right).
X: Track=`0x30`, Touch=`0x00` (+halfbit) — semantics open.

`de/1008` (left-only) = left dock info, 7 B:
`[00, TYPE, FLAG, 00, VER0, VER1, VER2]` — VER `02 03 03` = module FW
0.2.3.3 (zeroed when empty). **Framing trap:** the frame's last byte
is CRC, not an 8th payload byte — earlier “battery byte” readings
were CRCs.

`de/100b` = module-rail **millivolts** BE (`[00, HI, LO, 00]`) —
see [power](../device/power.md).

## Module config write (static-decoded, live PENDING)

Stock ritual chain (static): `… → 30/100A → 30/100B → 30/100C → 30/100D
→ 30/100E …` (read-before-write per section).

`30/100C` element format (`ModuleConfig::toByteArray`, static):
`[slot, 01, 01]` hardcoded immediates, then slots BELOW
`behaviourSlotStart` (the 9 gesture slots) → `[slot,01,01,X]` 4 B
(X = behavior byte or `0x00`); slots AT/ABOVE start →
`Slot::serializeSlot()` directly (`[SLOT, T, LEN, payload]`, same
T-table as keys; `[SLOT, 07, 00]` when empty), no prefix. All changed
slots concatenated into likely ONE frame. Guards: skip bit31-set and
≥ maxSlots indices.

Alternative live hypothesis under test: 30/1004-style frame with
**c1=`0x0b`**, params `[00, SLOT] + record`, ACK first byte `0x00`
(fallback `0x0c`). Same-length rule assumed. **OPEN until a live
module-gesture flash confirms the `01,01` bytes.**

## Layout-side discovery

Module enablement is per-half in the keymap profile: after re-seating,
a module shows DISABLED in NayaFlow layout until the profile is
re-flashed. A module on a disabled half shows only its indicator LED.
