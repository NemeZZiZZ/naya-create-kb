# Recovery cookbook

Symptom → recipe. All recipes assume: NayaFlow quit, halves awake,
correct port (`1101`-style = left, `21101`-style = right).

## Dark board, keys dark, modules lit, typing OK

The key render is parked (usually zeroed NVS: 1013 ceiling /
scanmode). Module LEDs are on a separate channel — this split is the
tell.

1. Run the one-shot revival (left default):
   `python3 toolkit/naya-undark.py --apply`
   (1013 ceiling 100 + ff-phase: RESUME/ON/MAXBRT=100/SCANMODE=1/
   RGB-white/BRT=100/SOLID/ON).
2. If still dark → [factory format](storage/factory-reset.md)
   `30/10ca [01]` on the left, then restore the snapshot, then
   ff-phase again.
3. Right half is CDC-deaf by design (ED ACKs, no effect; `1014`
   never answers): use the hardware Layer2 LED combos on the right
   half itself (firmware-internal path), after a true cold boot.

## Dark right half after an ED burst

ED to the right (especially RESUME/RGB phases) can park its render
with modules going dark too. Right still types. Try minimal ON+BRT
on the right port (`dst 0x51`); else cold USB reset of that half.

2026-09-22 relapse data: an ff-phase ladder fired **via the right port**
(explicit `dst 0x51`, 8/9 ACKs from src aa51) parked the right half AND
its module fully dark; it did not relight after a stock NayaFlow flash.
Earlier the same ff-ladder fired **via the left port** had relit the
right half once (mechanism unclear — the script is `dst 0x50`-shaped).
Treat right-side ff writes as risky; left-port ff first, cold boot next,
`30/10ca` as the last resort.

## `30/1001` / `30/1003` answer error `16 00`

Empty store — expected right after a `30/10ca` format. Restore the
snapshot (`naya-restore.py --snap … --apply`), verify `IDENTICAL`
per section.

## Device parked in MCUBoot recovery (extra USB node, app dead)

Idle the ports 20–50 s (no holders) → console revives → canary echo →
RESET (group 0, id 5, op 1) → verify `fa/1001`. Details:
[bootloader](firmware/bootloader.md). (`smp-unwedge.py` automates it.)

## Reflashing didn't fix it

Expected: MCUBoot reflash (verified uploaded to both halves) does NOT
touch the LittleFS data partition. `repair_flash` doesn't either
(mount-only unless the self-test fails). Only `30/10ca` formats it.

## True cold boot (the only real reset besides `ee/10ce`)

USB out **+** modules undocked **+** on/off switches. Switch flips
with USB plugged in are fake resets. `ee/10ce` reboots both halves
over software (USB drops ~0.13 s).

## Brightness weirdness (OPEN, observed 2026-09-18)

After an ff-phase recovery the left showed full white → slight dim →
further dim with no further writes; `SCANMODE=0`+`BRT=100`+`ON` then
had no visible effect. Brightness keys inverted (max→very low) was
seen earlier. If you reproduce a trigger (idle minutes? PC sleep?
dock/undock?), record it — 2–3 correlations pin the wake-path suspect.

2026-09-22 wire probe (`naya-brt-wrap-spike.py`): DEC steps clamp
honestly — 50→…→10→**0**, and 0 is an OFF-park: `ADJ_BRT` upward does
NOT relight. So the brightness **wrap** seen in stock NayaFlow
(…10→100→…→0 / 0→100→10…) is host-side value arithmetic interacting
with the OFF-park, not a device render wrap. The park itself survives
`ee/10ce`; relight needs the ff-phase ladder (or a full dark-saga
recovery).

## Never send

`ee/10be`, `ee/10ae` (DFU/MCUBoot resets), `fa/1002`, `fa/1006`
(format/erase chip), text `clear_bonds` / `mcuboot_reset`, 1-byte ED payloads (they write
zeros to NVS), oversized single `30/100e` frames (wedge the parser).

> `fe/100a` replay downgraded 2026-10-01 from the danger list: it is a
> plain timeouts write (params echo the payload verbatim), and an
> independent two-board measurement on 3.41.0 (2026-09-16..21,
> Create-knowledge-base) observed no wedge. The residual hazard is
> *semantic*: replayed bytes silently restore stale timeout values.
> Send a fresh `fe/100a` from live state instead of replaying captures.
