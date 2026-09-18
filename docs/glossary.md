# Glossary

| Term | Meaning |
|---|---|
| KK | Key index byte (also `position`; `position == KK` proven) |
| T-record | Keymap record `[KK, T, LEN, payload]`; T = type byte |
| LEN | Payload length; record length = LEN + 3 |
| MODMASK | Top byte of the u32 param1 (`(mod<<24)\|(page<<16)\|hid`); HID boot-modifier bits |
| Merge host | Left half — owns keymaps/LED maps/module config; only it answers `30/10xx` |
| Proxy | Left port answering `dst 0x51` for the linked right half (check reply SRC) |
| ff-phase | Winning LED recovery sequence, all `[0xFF, value]` ED writes (see [LED](protocol/led.md)) |
| NVS | Non-volatile settings in LittleFS (`scanmode_pwm`, `led_layer_override`, max brightness) |
| Wedge | Device state machine stuck by off-protocol input (stale commit replay, oversized frame, parked recovery) |
| Dark-saga | The 2026-09-17 LED-render parking incident and its zero-fill mechanism |
| Dirty flag | Tail triplet `4b 02 00` once any multi-behavior key exists (00 = factory) |
| Shadow slot | `KK+0x52` — T10 mini/full shadow record position |
| Slot-id | ED ACK byte2: per-command constant, NOT a value echo |
| `16 00` | Empty-store error (post-format, pre-restore) |
| True cold boot | USB out + modules undocked + switches (only real reset besides `ee/10ce`) |
| Dead control | Stock UI control with no wire path (scan-mode toggle, LED-max slider, animations, macros) |
| Write-only map | Static map with fill code but no reader (`+0x70` verdict) |
| `.sfb` | Module firmware image (+HASH), staged in base SPI flash |
