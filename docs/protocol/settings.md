# Settings & behavior

## Activity timeouts (`fe/100a` / `fe/100b`)

`fe/100a` is **SET ACTIVITY TIMEOUTS, not a commit**. 13 B payload =
status `00` + 3×u32LE milliseconds: **(idle, sleep, deep)**.
User sample: `(6000000, 6000000, 30000)`. `fe/100b` reads the same
payload back (stable across reboots — it doubles as the “commit token”
seen in captures, but the token never changes, so there is nothing to
refresh).

NayaFlow's Behavior Settings page maps onto this directly:
Idle/Sleep Timeout sliders (0–6000 s) → the first two u32s.
The v1.25.1 controls for **scan-mode toggle, LED max slider and
LED-override mode are dead host-side controls** — a settings flash
produced ZERO ED frames, only `fe/100a` (timeouts) + reads
(capture5, sniff clone).

## Typing behavior (host-side profile)

From the stock profile header + UI: tapping term **200 ms**
(`c8 00` u16LE, also embedded twice in every T10 record),
tap-hold flavour 0, transparent-as-default 1. UI-only “Interrupt
Flavor” presets (Balanced/Fast/Deliberate ≈ 200/150/280 ms) have no
wire counterpart — they re-emit the same T10 terms.

## LED-adjacent settings

`1013` max-brightness ceiling (NVS, no GET), `1012` scanmode PWM
(NVS: `scanmode_pwm`), `1014` layer override (NVS:
`led_layer_override`), `1011` effect 0–3. See [LED](led.md).
`1050` RGB override: no known “clear to follow-map” value
(cosmetic OPEN).

## Module behavior slots

9 host gesture slots (static map): 0 MOUSE_HORIZONTAL,
1 MOUSE_VERTICAL, 2 MOUSE_STATIC, 3 MOUSE_BUTTONS,
4 MOUSE_SCROLL_VERTICAL, 5 STATIC_SCROLL_VERTICAL,
6 MOUSE_SCROLL_HORIZONTAL, 7 STATIC_SCROLL_HORIZONTAL, 8 STATIC_ZOOM.
Track buttons/gestures and Touch/Track vocabularies in
[hardware](../device/hardware.md); wire form in
[modules on the wire](modules.md).
