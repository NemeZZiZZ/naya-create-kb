# Official user manual (claims)

Source: `CRR-users-manual-1.pdf` archived in `research/fcc/` of the
companion repo (right-half filing). Everything below is an **official
claim**, not a live-verified fact — where the bench contradicts it, the
field note wins (marked ⚠️).

## Box contents

- Travel case
- Split keyboard (left + right halves)
- USB Y-cable
- Keycap/switch puller
- 3 spare switches + 2 spacers

## Official spec sheet

| Item | Claim |
|---|---|
| Model | NAYA-800-1 (NAYA-CREATE) |
| Size | 212 × 118 × 18 mm |
| Weight | 1.4 kg |
| Body | Aluminum + polycarbonate |
| Switches | Kailh low-profile clicky, CPG-1232 (0.45 × 0.42 mm pins) |
| Keycaps | Low-profile, backlit, transparent characters |
| Battery | Internal ("45mA" in the sheet — read as small buffer cells, see ⚠️) |
| Radio | Bluetooth 5.4 |
| Power input | 4.2 V |

⚠️ **Field note:** the bench measures two FH301217 50 mAh pouch cells
(one per half) plus ~700–1000 mAh module packs carrying the real runtime
(labeled cells: Tune 1000 mAh, Track 700 mAh — see [Exhibits](exhibits.md)
and [Power](power.md)). "45mA" is consistent with the buffer cells.

## Module indicator LEDs (official table)

The small LEDs on the modules report charge/connect state:

| Pattern | Meaning |
|---|---|
| Blinking red | Battery low |
| Solid red | Charging problem |
| Blinking orange | Charging |
| Blinking green | Near full |
| Solid green | Fully charged |
| Blinking white | Connecting |
| Solid white (2 s) | Connected |

## Sleep defaults (official)

- **Sleep:** 1.5 min idle (configurable; see `fe/100a`)
- **Deep sleep:** 10 min (configurable; see `fe/100a`)

## Pairing and output modes

- **Bluetooth slots:** `Layer2 + 1…5` selects the paired device slot.
- **Enter pairing mode:** `Layer2 + Esc`.
- **USB mode:** `Layer2 + Z` switches a half to wired output.
- **Cable:** the supplied Y-cable powers/links both halves over USB.

## Charging

- Modules charge over Qi — docked on a half **or** standalone on any Qi pad.
- Halves charge over USB; docked modules charge from the half.
- OS support claimed: Windows 10+, macOS Ventura+, Ubuntu 24.04.

## Official best-practices (verbatim guidance)

- Do **not** use a wall outlet — charge from a computer USB port (5 V / 1 A).
- Power draw is rated 5 V ⎓ 1.5 A (matches the FCC label).
- Keep firmware updated via NayaFlow.
