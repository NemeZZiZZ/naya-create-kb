# Keymap records

Keymaps live on the **left half** (merge host), 3 layers × **74 keys**.
Read via `30/1003` (`[part, layer]`, 2 parts/layer); write per-key via
`30/1004` (`[00, layer, KK] + record`). L0 starts Esc (`0x29`),
Grave (`0x35`), `1` (`0x1E`), `2` (`0x1F`)… = top row.

## Universal rule

Every keymap record is **`[KK, T, LEN, payload × LEN]`** — record
length = byte2 + 3. Proven over all dumps (1776-record log census
closes exactly). `KK` = key index; `position == KK` proven.

## Type table

| T | Size | Meaning | Layout |
|---|---|---|---|
| 01 | 7 B | Plain key | `[KK,01,04,HID,PAGE_BE,MOD]` where u32 param1 = `(mod<<24)\|(page<<16)\|hid`. Consumer keys: page `0x000c` (e.g. C_MUTE `20 01 04 b6 00 0c 00`). Example: `KK 01 04 CC 00 07 00` (CC = USB HID, page `0x0007`) |
| 05 | 7 B | Special | `[KK,05,04,ID,00,00,00]`. Family `04`: `01` = momentary-layer-1 hold MO(1) (thumb keys KK67/68); `02` = hold-layer-2 (bottom corners KK62/73 — NOT the Naya action; keycap print ≠ action) |
| 07 | 3 B | DISABLE | `[KK,07,00]` |
| 0E | 3 B | TRANSPARENT | `[KK,0e,00]` (unbound positions). The 07-vs-0e choice is one flag byte (`[Key+8]+0x28 == 0` → `07`) |
| 08 | 7 B | Output select | `[KK,08,04,ID,00,00,00]`: 1 = USB_DEVICE, 2 = BT_OUT |
| 00/09/0F | 11 B | Vendor action | `[KK,A,08,X u32LE,Y u32LE]`; full action ID = `(A<<16)\|(X<<8)\|Y`. (0,3)=BT (`BT_DEVICE_1` = X=3,Y=1), (15,3)=mouse, (9,13)=LED. Full table: [host maps](../disassembly/host-maps.md) |
| 06 | 7 B | naya-type | `[KK,06,04,p1 u32LE]` (values 150/151/200/201/300/301/400/401). **No genuine wire sample exists** — the palette MAC_OS resolves host-side to a plain HID LGUI key |
| 03 | 24 B | Hold-only | `[KK,03,15, 01,01,00, c8,00, HOLD…, TAP…]` — the two-behavior shape (see below) |
| 10 | 27 B | Multi-behavior primary / full shadow | Format below |
| 78 | 3 B | Empty filler | `[KK,78,00]` |
| 02,03(macro),04,0A | — | Unmapped anywhere | |
| 0B,0C,0D | — | Dead binding types, never on wire | |

MODMASK is the TOP BYTE of the u32 param1 (`(mod<<24)|(page<<16)|hid`,
standard HID boot-modifier bits); the only non-zero sample in 911+
records is `02` (Shift) on the parenthesis keys.

## Multi-behavior keys (T03/T10 — live-solved)

Shape depends on behavior count (NayaFlow UI enforces the chain
Tap→Hold→DoubleTap→Tap&Hold, so these are the only stock states):

| Behaviors | Shape |
|---|---|
| 2 (press+hold) | **T03 single, 24 B, no shadow**: `[22,03,15, 01,01,00, c8,00, 09,00,07,00, pad4, 07,00,07,00, pad4]` (KK22: Tap=D / Hold=F) |
| 3 (+double) | **T10 primary 27 B + MINI shadow 10 B @KK+0x52**: mini = `74 10 07 c8 00 01 05 00 07 00` (header + term + `01` + double triple) |
| 4 (+taphold) | **T10 primary 27 B + FULL shadow 27 B @KK+0x52** |

T10 format:
`[KK, 10, 18, c8,00, 03, 01,01,00, c8,00, A_HID,00,07,00, 00×4, B_HID,00,07,00, 00×4]`

- `c8 00` ×2 = tapping-term 200 ms u16LE (hold threshold + double-tap window).
- A/B = two behaviors as bare `[HID,00,07,00]` triples (no MODMASK).
- Primary @real KK: A=hold, B=tap. Full shadow: A=tap_hold, B=double_tap.
- **Shadow slot = KK+0x52** (proven static in
  `Key::serializeBindingData` AND live 3×: KK30→82, KK32→84, KK22→74).
- Tail index triplet `4b 00 00` → `4b 02 00` once ANY T10 exists — a
  layout-version/dirty flag (00 factory → 02 modified), not a counter.

A writer must emit primary + shadow and set `4b=02`, exactly like
stock (see `toolkit/naya-t10-spike.py` in the RE repo; live verdict
pending).

## Write constraints

- **Same-length writes apply; length-changing rewrites (7 B↔11 B) are
  silently ignored** — the report lists diverged KKs. Plan writes accordingly.
- Writes persist with `30/1004` alone (RAM + NVS). Never replay
  `fe/100a` bytes.
- Layer sizes (healthy board): L0 ≈ 789 B, L1 ≈ 636 B, L2 ≈ 660 B
  (exact sizes shift with customs; byte math must close every time).
