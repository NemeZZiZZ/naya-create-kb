# Bootloader & MCUBoot serial recovery

The halves run **MCUBoot** (`*** Booting MCUboot 9ddeffa8169c ***`,
Zephyr OS `v3.7.0-5411-g31fea97e05fd`). App CDC `ee/10ce`
(NORMAL RESET) reboots into it; during recovery an **extra USB node**
appears (e.g. `1103` alongside `1101`), window ~2.2 s, then the app
boots and the node disappears.

Boot banner (361 B, captured on the extra node):

```
*** Booting MCUboot 9ddeffa8169c ***
*** Using Zephyr OS build v3.7.0-5411-g31fea97e05fd ***
I: Starting bootloader / Primary image: magic=good, swap_type=0x3, copy_done=0x1, image_ok=0x1 / Scratch: magic=unset / Boot source: none / Image index: 0, Swap type: none / I: Enter the serial recovery mode
```

Console roles: ACM1 (`1101`-style) = serial-recovery COMMAND channel
(echo/reset answer here). ACM2 = boot LOG only (never answers).

## Protocol (classic mcuboot-serial ASCII framing)

Request: `06 09` + base64(`u16 totlen BE ‖ nmgr_hdr` 8 B
(op,flags,lenBE,groupBE,seq,id) ‖ CBOR ‖ crc16) + `\n`.
Multi-line responses continue with `04 00` + base64 chunks (~124 B/line).

- CRC16 = XMODEM poly `0x1021`, **seed `0x0000`** (proven: seed-0 echo
  answered instantly, `0xFFFF` echo ignored; response CRC verifies).
- DEFAULT group (0) WORKS: echo id 0 (op 0, body `{"d":"naya"}` →
  `{"r":"naya"}`); RESET id 5 (op 1 write, empty body) → bootloader
  reboots to app (port dies).
- IMAGE group (1): states read (id 0) **NEVER responds** — even on a
  stable parked console while echo answers ⇒ **IMAGE group compiled
  out** (no serial upload/list/slot-info).

## Parking, wedge, recovery (recipe proven 3×)

- Any VALID frame resets the serial inactivity timeout ⇒ the device
  PARKS in recovery indefinitely (extra node persists, app protocol
  dead, session wake hangs — classic symptom).
- Flooding frames into the window can additionally wedge the USB CDC
  driver (console mute on both nodes).
- Recovery recipe (no hands, all software):
  1. idle ports 20–50 s (no holders) → USB re-enumerates, console revives;
  2. canary: echo seed0 on the command node;
  3. RESET (group 0, id 5, op 1) → device boots app. Verify with `fa/1001`.

## Implications

- Serial image upload unavailable ⇒ stock base-FW updates flow through
  the APP CDC protocol (download → secondary slot → reboot → MCUBoot
  swap), matching NayaCore's `MCUBootWorker_*` classes.
- Custom FW is gated by RSA-2048 signature + encrypted bodies on every
  DFU path ⇒ SWD pads (labeled on PCB) remain the only custom-FW
  entry — see [signing](signing.md).
- Zephyr 3.7 base: a ZMK port is architecturally a board-def + driver
  exercise on the same nRF Connect SDK generation.

Reference tooling (RE repo `toolkit/`): `smp-unwedge.py` (canary +
RESET — keep handy), `smp-probe*.py` (probe evolution), `boot-trap.py`
(node-trap for cold-boot log capture).
