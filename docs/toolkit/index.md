# Toolkit overview

Working tools (RE repo `toolkit/`, all tested live against base FW
0.3.41.0). Golden rules: **quit NayaFlow first** (it holds the ports),
halves awake, dry-run default everywhere, `--apply` writes.

| Tool | Purpose |
|---|---|
| `cdc-client.py` | Read/write keymaps, LED maps, aux telemetry. `left dump` / `left set KK spec` (`hid:`/`cons:`/`vend:`/`raw:`) / `left ledmap` / `left ledset KK H S` / `left dump100b` / `aux` / `raw TYPE C0C1 [params]` |
| `naya-backup.py` | Full read-only snapshot (keymap L0–L2 + LED L0–L2 + telemetry) → one JSON |
| `naya-restore.py` | Replay snapshot (`--snap f.json --apply`): per-record `30/1004`+`30/100e`, layer-echo ACK checks, readback report per section |
| `naya-undark.py` | One-shot LED revival: 1013 ceiling + winning ff-phase (`--port` default LEFT, `--apply` to write) |
| `naya-led-recover.py` | LED recovery phases (`--phase ff/bare/t00/t01`, `--apply`) |
| `naya-maxbrt.py` | Set the 1013 NVS ceiling (`--level 100 --apply`; no GET exists) |
| `naya-t10-spike.py` | T10 full-set writer probe (write + readback + restore). Dry-run green; live verdict recorded in findings |
| `naya-modules-spike.py` | `30/100b` c1=`0x0b` write hypothesis (+`0x0c` fallback), swap + restore |
| `naya-effect-spike.py` | `ed/1011` encoding probe (3 candidate forms, y/n visual) |
| `smp-unwedge.py` | MCUBoot recovery: canary echo + RESET (keep handy) |
| `smp-probe*.py`, `boot-trap.py` | SMP probe evolution; cold-boot log node-trap |
| `cdc-sniff.py`, `monitor.py` | Passive serial logger; live monitor |
| `extract_fw.py` | Carve MCUBoot images out of `app.asar`/NayaCore |
| `build-stock-backup.py` | Device dumps → NayaFlow-compatible stock zip (`--verify` compares) |
| `ble-scan.py` | BLE advertisement scan (bleak) |
| `verify-signing-key.py` | Zero-trust signing-key check (see [signing](../firmware/signing.md)) |
| `interposer.c` | macOS `DYLD_INSERT_LIBRARIES` tap logging NayaCore's serial I/O (needs a re-signed clone — hardened runtime strips `DYLD_*`) |

Recipes: [Python](python.md) · [JS](javascript.md).
DANGER list (never send): `ee/10be`, `ee/10ae`, `fa/1002`, `fa/1006`,
`clear_bonds`, `mcuboot_reset`, replayed `fe/100a` bytes.
