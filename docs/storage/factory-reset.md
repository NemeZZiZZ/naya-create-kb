# Factory reset — `30/10ca` clear-all-data

Hidden device-side format of the LittleFS data partition. Found by
disassembling the installed NayaCore: `_remapClearFlash` builds
`QByteArray(0x01)` and calls
`ProtocolCDCProcessWorker::_constructRemapMessages(w1=0x10ca, …)` —
i.e. CDC frame **type `0x30`, c0=`0x10`, c1=`0xCA`, payload `[01]`**.
The `_constructRemapMessages` caller census is otherwise exactly the
known family (`0x1001–0x100E`) + `0x10ca`.

The full host-side chain exists in the binary
(`si_clearAllData_req_source → … → sl_clearAllData_req_receiver →
_startClearAllDataOperations → doClearAllDataOperations →
_remapClearFlash + _handleSpiflashFormatPartition +
_handleRemapClearAllDataResponse`) but the ZMQ event whitelist (15
events) has **no `clear_all_data` slot** — sending it over the
RPC↔ZMQ bridge returns `Unknown command event: invalid_command_event`
(Error 4). Direct-wire is the only path in this build.

## Firing it

Target the **left** port, `dst 0x50`:

```
type 0x30, c0 0x10, c1 0xca, params [01]
```

Expected reply: ACK with status `00` (observed:
`aa 50 00 00 30 04 10 ca 00 80 …` — status 00 + flag `80`).
No reboot needed for CDC.

## Effects (observed 2026-09-18)

- `30/1001` and `30/1003` answer error **`16 00`** (empty store).
  `ED` still ACKs, `fe/1002` alive, FW still 0.3.41.0.
- Keymap + LED maps + settings are factory-default/empty.
  **Restore a snapshot immediately after** (per-record `30/1004` +
  `30/100e` + readback verify — a restore script that reports
  `IDENTICAL` per section; ours restored 468 keys + 408 LEDs clean).

## Risks (explain before firing)

1. Keymaps/settings reset — recoverable from a snapshot (keep one:
   full JSON of keymap L0–L2 + LED L0–L2 + telemetry).
2. Split-link pairing: most likely survives (`ClearAllSplitLinks` is a
   separate op), but if halves unlink, re-pair via NayaFlow.
3. ED defaults must be re-set after (idle timeouts persist in the
   snapshot; max-brightness ceiling has no GET — re-send 100).

After success: re-dump the device, restore the profile, verify LED
revival, re-apply the [ff-phase](../protocol/led.md#the-dark-saga-what-we-learned).
