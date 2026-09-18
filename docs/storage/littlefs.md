# LittleFS & NVS

Each half has an **external SPI flash** carrying a **LittleFS**
filesystem (`LFS_ERR_CORRUPT`, `FORMAT_PARTITION`, `SPIFLASH_TEST`,
`VERIFY FLASH` in the binary). It stores:

- keymap profiles (layers, the `30/10xx` store),
- NVS settings keys — proven: `scanmode_pwm`, `led_layer_override`,
  `max LED Brightness` (1013 ceiling; the binary logs
  `No max LED Brightness found in the settings table. Resorting to
  default.`),
- module `*_UserApp.sfb` images + HASH files (flash staging for
  module updates via MCUBoot through the base).

## What survives what (all proven live)

| Operation | Clears NVS wedges? |
|---|---|
| Switch flip with USB plugged | No (not even an MCU reset) |
| `ee/10ce` reboot / USB replug | No (NVS persists by design) |
| NayaFlow “full reset” / clear keymap | No (doesn't touch NVS LED settings) |
| `repair_flash` (“Repair SPI flash”) | **No** — it is mount+patch: LittleFS mount, and a format happens ONLY if the flash self-test FAILS (`TestSPIFlash` strings). Log proof: 1318 ms “success” with zero effect |
| MCUBoot firmware reflash (both halves, verified uploaded) | **No** — the wedged LED state survived; the data partition is independent of the FW slots |
| [`30/10ca` clear-all-data](factory-reset.md) | **Yes** — device-side format of the data partition |

Mental model: there are (at least) two domains — the FW image slots
(MCUBoot swaps these) and the LittleFS data partition (profiles,
NVS). Nothing in the stock UI formats the data partition; the only
exposed path is the hidden `30/10ca` op.

## NVS gotchas

- ED writes with wrong arity **zero-fill**: a 1-byte send is
  `[target=X, value=0]` and WRITES ZEROS into NVS (this is the
  dark-saga mechanism — not random garbage).
- `fe/100b` (timeouts token) is **stable across reboots and factory
  restore** — it is not a revision counter.
- Tapping-term `c8 00` in T10 records mirrors the profile header
  (200 ms); tail `4b 02 00` is a sticky dirty flag once any multi-key
  exists.
