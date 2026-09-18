# RPC & ZMQ bridge

Renderer → `flow-bg-server` (`http://127.0.0.1:56486`, routes under
`/api/*` and `/rpc/*`) → NayaCore ZMQ REP (`127.0.0.1:56500`, JSON
payloads) → device.

## Generic ZMQ bridge (live-proven)

```http
POST /rpc/send-nayacore-zmq-message
{"messages": ["command", "<event>", "", "<json-string>"]}
```

i.e. `messages = [topic, event, ...frames]`. Server replies
`{"status": "message sent"}` — that only means the bridge forwarded
it; watch the NayaCore log for the real verdict.

Observed live messages (from `logs/flow/NayaFlow-bg-*.log`,
`Sending ZMQ message to NayaCore: %+v`):

```
[command update_create_fw  ]
[command create_pairing_start  {"target_device_left":"<usb-serial-left>","target_device_right":"<usb-serial-right>"}]
[command clear_data  {"targetDevices":[]}]
[command repair_flash  {"target_devices":[],"target_partitions":[]}]
[command clear_ble_devices  {...}]
```

Note the casing drift (`targetDevices` vs `target_devices`) — match
the exact form per command.

## Dispatch whitelist (15 events — static, installed binary)

`flash_keymap, update_keymap, start_device_manager,
close_device_manager, force_touch_start, force_tune_start,
force_track_start, create_pairing_start, update_module_fw,
update_create_fw, update_fw_files, repair_flash, clear_data,
clear_ble_devices, set_handshake_frequency`.

Anything else → `Unknown command event: invalid_command_event`
(Error 4). In particular **`clear_all_data` is NOT dispatchable**:
the full `si_clearAllData_req_source → … →
doClearAllDataOperations` chain exists in the binary (17 ZMQ slots
incl. `clearAllData`), but the event string never routes to it —
the `'clear_all_data'` string is a log/ProcessID tag inside the
unreachable function. The device-side op is reachable directly on
the wire instead: [`30/10ca`](../storage/factory-reset.md).

Other bridge routes (binary strings): `/factory-reset` (present in
binary, **404 on the live mux** as bare `GET/POST/OPTIONS` —
registration unknown), `/naya-core-zmq`, `/naya-core-device-info`,
`/naya-core-naya-devices`, `/export-user-data/`,
`/import-user-data/`, `/import-userdata-beta/`,
`/list-userdata-backups/`, `/module-settings/`, `/actions/:symbol`,
`/open-log-folder`, plus `sse:` streams (`device_list`,
`message-stream`, `flash-keymap-state`, `device-info-stream`,
`naya-devices-stream`).
