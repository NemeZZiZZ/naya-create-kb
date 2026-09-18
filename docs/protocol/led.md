# LED subsystem

Two stores + one render engine:

- **LED map** (`30/100d` read / `30/100e` write): per-key color store,
  136 entries × `[KK, H_lo, H_hi, S]` per layer.
- **ED engine** (`ED/10xx`): ON/OFF, brightness, effects, overrides —
  some NVS-persisted.
- **Module LEDs** sit on a separate channel outside the key gate
  (lit modules + dark keys = the tell of a parked key render).

## LED map format

Multipart read like the keymap, **544 B/layer = 136 × `[KK, B1, B2, B3]`**,
KK `0x00..0x87` sequential. Entry = `[KK, Hue_lo, Hue_hi, Sat]`:
**H = (B2<<8)|B1 (9-bit hue, degrees), S = B3.**

| Color | H | S | Entry tail |
|---|---|---|---|
| Red | 0 | 70 | `00,00,46` |
| Orange | 30 | 100 | — |
| Yellow | 60 | 70 | — |
| Green | 120 | 70 | `78,00,46` |
| Cyan | 180 | 100 | `b4,00,64` |
| Teal | 171 | 100 | `ab,00,64` |
| Blue | 240 | 100 | `F0,00,64` |
| Purple | 266 | 70 | `0a,01,46` |
| Magenta | 300 | 100 | `2c,01,64` (H300 = 256+44) |
| Pink | 300 | 100 | — |
| White | 0 | 0 | `00,00,00` |
| Factory amber | 38 | 100 | `26,00,64` |

No Value component — brightness is global (ED commands). There is no
Value channel: white is (H0,S0); sentinel S=150 means unset.

**136 positions**: keys 0–73, edge strips 74–80/81–87, module bays
88–111 (left) / 112–135 (right); Touch lights the first index of its bay.

## Writes

- **Single-entry** `30/100e` (readback-proven, incl. L1/L2):
  req `AA 00 50 id 30 08 10 0e [00, layer, KK, H_lo, H_hi, S] CRC 04`,
  ACK `00 00`. Ritual per key: READ-ALL → 100e → READ-ALL; no commit;
  persists without commit.
- **Full-map form** (nayactl claim): `[layer] + 136 entries`, chunked
  ≤242 B with the layer byte re-prefixed per continuation.
- Oversized single frames wedge the parser — keep single-entry or
  correct chunking.
- Bulk `100e` writes drop the 1050 RGB override back to follow-map
  render (override state is NOT part of snapshots).

## ED engine notes

- All ED commands take `[target, value…]`, `0xFF` = all (device
  ignores the target value; **length matters** — 1-byte sends are
  `[target=X, value=0]` and WRITE ZEROS into NVS).
- Empty params = self-target. **1012/1013/1014 have no GET path**
  (empty-param reads return bare ACK `00`).
- `1050 RGB_BRT` (global color override `[target, R, G, B, brightness]`)
  **survives reboot** and beats the map (white override over a zeroed
  map still lights keys).
- `10d1/10d2` FORCE ON/OFF: consistent NO-REPLY, no handler.
- `1014` on the **right half: NO-REPLY** (left ACKs). Right render is
  CDC-deaf in general: ED ACKs without effect; right recovery path is
  firmware-internal (hardware Layer2 LED combos).
- **Cold-boot saturation-drop firmware bug**: white keys come back RED
  after a power cycle with a byte-identical map — boot render ignores
  saturation. Amber maps (S=100) are unaffected.
- **OPEN (2026-09-18):** `SCANMODE=0` + `BRT=100` + `ON` produced no
  visible change on a dim-lit left half; progressive dimming after an
  ff-phase recovery is unexplained. Brightness keys inverted symptom
  also observed (max→very low). See [recovery](../recovery.md).

## The dark-saga (what we learned)

Malformed (wrong-arity) ED writes parked the LED render for hours,
surviving reboots and NayaFlow resets. Mechanism (reframed by
nayactl PR #6): short payloads are **zero-filled** — our 1-byte sends
wrote ZEROS into NVS (`1013` ceiling, scanmode). **1013 = persistent
NVS ceiling** (`max LED Brightness` settings-table key): at 0 the board
stays dark with normal ACKs on every LED command; module LEDs sit
outside the gate.

Recovery recipe (the “ff-phase”, left half, all `[0xFF, value]`):
`1013 maxbrt=100 → 1012 scanmode=1 → 1014 override=0 → 1050 white/100 →
1008 brt=100 → 1011 SOLID → 1003 ON` (+RESUME). Idempotent; re-runnable.
When NVS is truly wedged (survives even MCUBoot reflash — the state
lives in the LittleFS data partition), use [factory reset
`30/10ca`](../storage/factory-reset.md).
