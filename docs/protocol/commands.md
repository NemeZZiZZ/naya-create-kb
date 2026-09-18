# Command map

Notation `xx/yyyy`: family byte `xx`, command `C0 C1 = yyyy`.
“Left-only” = answered only via the left port (merge host).
All tables cross-checked between the installed-NayaCore jump tables
and live probes (20+ empirical cross-checks).

## `30` — remap (LEFT ONLY)

| Cmd | Name | Notes |
|---|---|---|
| 1001 | READ LAYER LIST (= handshake/inventory) | params `00 00` → 72 B. Required first, same handle |
| 1002 | WRITE LAYER LIST | never observed on wire |
| 1003 | READ LAYER DATA | params `[part, layer]`, multipart (more-flag + layer echo) |
| 1004 | WRITE LAYER DATA | **per-key**: params `[00, layer, KK] + record`; ACK echoes layer |
| 1005–1008 | MACRO LIST/DATA read/write | device answers, **store always empty** — host-only feature |
| 1009 | MODULE CONFIG LIST read | 41 B profile header/checksum |
| 100A | MODULE CONFIG LIST write(?) | stock ritual only |
| 100B | MODULE CONFIG DATA read | params `[part, layer]`; content on L1 |
| 100C | WRITE MODULE CONFIG DATA | static-decoded, never observed (see [modules](modules.md)) |
| 100D | READ LED MAP | 136×4 B per layer |
| 100E | WRITE LED MAP | single-entry `[00, layer, KK, H_lo, H_hi, S]`; ACK `00 00` |
| 10CA | **CLEAR ALL DATA** | params `[01]` → device ACKs, **formats the data partition** (see [factory reset](../storage/factory-reset.md)) |

Writer recipe: handshake → `30/1004` per key → readback via `30/1003`.
`30/1004` applies instantly to RAM **and persists across reboot** — no
commit needed. (`fe/100a` is NOT a commit; see [settings](settings.md).)

## `ED` — LED engine

| Cmd | Name | Params `[target, value…]` (`FF` = all) |
|---|---|---|
| 1003/1004/1005 | ON / OFF / TOGGLE | `[target]` |
| 1006/1007 | INCREMENT / DECREMENT | `[target, amount 0–100]` |
| 1008 | ADJUST BRIGHTNESS | `[target, brightness 0–100]` (nayactl's 0–255 is wrong) |
| 100D | EFFECT CYCLE | `[target]` — steps SOLID→BREATHE→SWIRL→SPECTRUM |
| 100E | HUE SATURATION | `[target, hue u16, sat 0–100]` |
| 100F / 1010 | HALT / RESUME | `[target]` (freeze/continue animation) |
| 1011 | SELECT EFFECT | `[target, effect 0–3]` = SOLID/BREATHE/SWIRL/SPECTRUM |
| 1012 | SET SCANMODE PWM | `[target, value]` |
| 1013 | SET LED MAX BRIGHTNESS | `[target, 0–100]` — **persistent NVS ceiling**; at 0 the board stays dark with normal ACKs |
| 1014 | SET LED LAYER OVERRIDE | `[target, value]` — **left ACKs, right never answers** |
| 1050 | RGB BRIGHTNESS (global color override) | `[target, R, G, B, brightness]` — survives reboot; bulk `100e` writes drop it back to follow-map |
| 10D1/10D2 | FORCE ON/OFF | **NO-REPLY on both halves** (normal — no device handler) |

Notes: empty params = self-target. Empty-param 1012/1013/1014 return
bare ACK `00` — **no GET path exists**. ON/OFF state and brightness
level are separate axes (ON does not relight from brightness-0;
INCREMENTs do). Details + the dark-saga: [LED subsystem](led.md).

## `FE` — system

| Cmd | Notes |
|---|---|
| 1001 | MEDIA ID REQUEST |
| 1002 | GET FW VERSION → `00 00 03 29 00 38` = base **0.3.41.0** |
| 1003 | MODULE BATTERY RECOVERY |
| 1004 | GET HW ID NUMBER |
| 1005 | SET HOST OS |
| 1006 | GET KB BATTERY LEVEL (base mV; bytes[1..2] BE) |
| 1007 | SET RELEASE MODE |
| 1008 | TOGGLE KEYSCAN MODE (1 param byte; live key events) |
| 1009 | KEYSCAN EVENT (`[row, col, state]`, 0 = PRESS) |
| 100A | **SET ACTIVITY TIMEOUTS** — 13 B payload = status `00` + 3×u32LE ms (idle, sleep, deep). **Not a commit.** Sample: `(6000000, 6000000, 30000)` |
| 100B | GET ACTIVITY TIMEOUTS (the `fe/100a` params echo this payload verbatim; it is stable across reboots) |

## `BE` — BLE

1001 SET PAIR ADDR · **1002 GET PAIR ADDRESS** (8 B) · 1003 UNPAIR PAIR ·
1004 UNPAIR ALL · 1005 GET ALL PAIRS · 1006 GET BLE NAME (`0b` +
ASCII) · 1007 SET BLE NAME · 1008 GET BLE ADDRESS · 1009 SELECT BLE
PROFILE · 100A CLEAR BLE PROFILE · 100B SELECT BLE OUT · 100C GET BLE
STATUS (250 B live blob) · 100D GET DONGLE ADDR · 100E GET SLOTX ADDR ·
100F GET BLE FW VERSION (`00 02 1d` = v0.2.29 — **not battery**) ·
1010 CLEAR ALL SPLIT LINKS.

## `DE` — module

1001 SEND HANDSHAKE · 1002 MODULE DETECT · 1003 CHECK HANDSHAKE ·
1004 UNKNOWN · 1005 MODULE FWUP · 1006 RESET MODULE · 1007 GET ADDRESS ·
1008 GET MODULE FW VERSION (`…02 03 03` = 0.2.3.3; slice `p[3..6]`, the
leading zero is a status byte) · 1009 GET BATTERY (6 B:
`[00][batt u16BE 0.1 mV][usb u16BE 0.1 mV]`) · 100A MODULE FILE FW
VERSION · **100B GET PRECISE BATTERY LEVEL** (module-rail mV).
Decoded dock matrix: [modules on the wire](modules.md).

## `FA` / `EE` / `FF` / `CA` / `F1`

- **FA** (exactly 3): 1001 TEST FLASH · 1002 FORMAT PARTITION ·
  1006 ERASE CHIP. **Danger zone** — see [recovery](../recovery.md).
- **EE**: 10CE NORMAL RESET · 10BE DFU RESET · 10AE MCU BOOT RESET.
  `ee/10ce` = the only software true-reboot.
- **FF** (host-side): 1000 WAIT · 1001 VERIFY FLASH · 1002 ENQUEUE READ
  LAYERS · 1003 GET MODULE INFO. `ff/1000` + `ff/1003` get NO-REPLY
  from the device — no device handler (FF/1000 WAIT is a host
  META-script WAIT).
- **CA, F1**: all-UNKNOWN groups, no known commands.
