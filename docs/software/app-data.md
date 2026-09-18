# App data & backups

Base dir (macOS): `~/Library/Application Support/NayaFlow/`.

## Logs (the ground-truth tap)

- `logs/core/nayacore_*.log` — CDC traffic + ZMQ command verdicts.
  Profile-interpreter dumps log per-key `tap: (ACTION)` + `wire: <hex>`
  pairs (8 full dumps) — the source the whole T-table was decoded from.
  Forensics examples: `repair_flash` 1318 ms “success” (mount-only,
  see [LittleFS](../storage/littlefs.md)); `FW Right/Left file
  uploaded successfully` (MCUBoot reflash really happened yet the
  wedge survived).
- `logs/flow/NayaFlow-bg-*.log` — every ZMQ message the bridge sends
  (`Sending ZMQ message to NayaCore: %+v`) — how the
  [bridge format](rpc-zmq.md) was recovered.

## SQLite (`user-data.db`)

Tables that matter: `key_bindings` (190 rows stock, ALL
`behavior='press'`; `behavior ∈ {press, hold, double_tap, tap_hold}`
per key), `module_settings` (MS-3=10 pointer speed, MS-4=20 accel,
MS-2=50 scroll), `module_config_bindings` (`binding_location` =
`track:/touch:/tune:` × `keyboard_left/right`; `state='disabled'`
vs NULL = enabled), `settings` (2× numboxSlider 6000/6000 idle/sleep
ms), macro step tables (1 BASIC macro stock, never flashed),
`keys.color_hex` (host-side UI state only).

Layers: QWERTY(order0) / Keypad+Arrow(order1) / System(order2).
`No data to write for layer list` is always logged (names/count are
host-only). Positions 74–96 (23/layer) are host-only spares, never
read (`Read 74 keys`).

## Backup format

`backups/1.25.1/*.zip` (auto-backup every ~30 min) =
`backup_meta.json{software_version}` + `user-data.db`. A compatible
backup/restore path for third-party clients: copy the live DB as a
skeleton and rewrite `key_bindings` only (round-trip verified:
222/222 keys byte-equal to the live DB).

Our own snapshot format (simpler, device-first): one JSON with keymap
L0–L2 + LED maps L0–L2 + reference telemetry (FW versions, timeouts,
module presence/FW/rail). Restore = per-record `30/1004` + `30/100e`
with readback verify (`IDENTICAL`/`DIFFERS` per section).
