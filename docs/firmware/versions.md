# Firmware version inventory

Recovered from the NayaFlow install tree (`backup/firmware/README.md` in
the companion repo). Packages are MCUBoot containers; payloads are
**RSA-2048 signed and encrypted** — no custom firmware without Naya's
private key (see [Bootloader](bootloader.md)).

## Base firmware (nRF52811 halves)

| NayaFlow | Left `sz` | Right `sz` | Notes |
|---|---|---|---|
| v0.1.1 | 313 392 | 235 296 | Oldest archived |
| v1.3.11 (= 1.6.10) | 352 256 | 239 184 | Renamed release |
| v1.11.11 | 358 496 | 243 840 | |
| v1.15.1 | 361 632 | 244 336 | Desktop core moves out of asar → standalone |
| v1.21.0 | 312 272 | 220 688 | Shrinks (feature split or compression change) |
| v1.25.1 | 328 880 ×2 (`sz` + `fwr_64`) | 226 000 ×2 (`sz` + `fwl_64`) | **Base FW 0.3.41.0** (both halves, `fe/1002` = `00 00 03 29 00`); dual-slot entries |

## Module firmware

- Present only in packages **≤ 1.6.10** (v0.1.1 ships one 175 136-byte
  module blob).
- Packages **≥ 1.15** carry **no** module firmware — modules update
  through a different channel (`update_module_fw` exists in the ZMQ
  command set but was never exercised live).

## Notes

- Filenames `sz` = size-prefixed MCUBoot image; `fwr_64`/`fwl_64` =
  dual-bank slots (left/right) introduced in v1.25.1.
- The bench boards run 0.3.41.0 — current as of the last Naya release
  (company defunct; no further updates expected).
- Full carve table with hashes: `backup/firmware/README.md`.
