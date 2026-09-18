# Key functions (NayaCore)

Mangled names as in the binary; wire effects verified live where noted.

## Clear-all-data chain

`naya_zmq::ZMQHandler::si_clearAllData_req_source` →
`Naya_ThreadManager::si_clearAllData_req_router2` →
`Naya_DeviceManager::sl_clearAllData_req_receiver` →
`_startClearAllDataOperations` → `doClearAllDataOperations`
(`Naya_DeviceManager_ClearAllData.cpp`) →
`naya_serial::ProtocolCDCProcessWorker::_remapClearFlash` +
`_handleSpiflashFormatPartition` +
`_handleRemapClearAllDataResponse`.

- `_remapClearFlash` builds `QByteArray(0x01)` → `_constructRemapMessages(w1=0x10ca, …)`
  = wire [`30/10ca [01]`](../storage/factory-reset.md).
- `doClearAllDataOperations` also enqueues `naya_remap::Profile(ADD_DEFAULT_DATA)`.
- “Reconnected to device %1 after clear all data” is its completion log.
- ZMQ slot exists but the event string is **not dispatched** (15-event
  whitelist) — dead code from the bridge's point of view.
- `naya_fw::create::ClearAllData_MinVersion` (+ `naya_fw::ClearAllData_MinVersion`):
  FW-version gate constants in the create-FW update flow; values not
  statically extractable (zero in file `__DATA`).

## Remap write path

`_remapWriteLayerList` → `_remapReadLayerData` (30/1003) →
`_remapWriteLayerData` (30/1004) → `_remapWriteModuleConfigList`
(30/100A) → `_remapReadModuleData` (30/100B) → `_remapWriteModuleData`
(30/100C) → READ color (30/100D) → WRITE color (30/100E).
Read-before-write per section — matches the captured stock ritual.

`Key::serializeBindingData(offset)` emits primary (offset 0) and the
`@KK+0x52` shadow (offset 1); `Key::wrapDblTapRecord(h1,h2,inner)` emits
`[h1, 0x10, LEN, term_lo, term_hi, h2, inner…]`, LEN = inner.size()+3;
`operator==` compares both offsets; `hasDoubleTapBindings()` = map
lookup @ `Key+0x18`. `ModuleConfig::toByteArray(QList<int>)` builds
30/100C elements (branch on `behaviourSlotStart`).

## LED construction

`_constructLEDMessages`: `ldrsb params[0][0]` → static payload, codes
`0x1003–0x1050` via jump table. Host validators: brightness 0–100
(`Invalid brightness…`), hue 0–360, sat 0–100, `effect_id: uint8_t
(0-3)`. `Binding::serializeBindingData` is the shared key/module slot
emitter behind the same T-table.

## SPI-flash interactive flow

`_handleSpiflashTestFlash` → `_handleConfirmation` →
`_handleSpiflashFormatPartition` (dispatch compare chain). Format runs
only when the flash self-test FAILS — the `repair_flash` mount-only
behavior falls out of this chain.

## Version gating

`operationMinFWVersion(DeviceType, Operation)` — switch over operation
id selecting among `naya_fw::create::{Keymap,UpdateModule,
ClearAllData,ClearAllSplitLinks,ActivityTimeouts,…}_MinVersion` and
`naya_fw::dongle::ProtocolCDC_MinVersion`. All thresholds dynamic
(see [method](index.md)) — treat gate behavior as OPEN.

## Misc singletons/maps

`+0xb8` = mutex-guarded lazy singleton pointer, `+0xc0` = its mutex
(double-checked locking; destructor via `___cxa_atexit`) — not data
maps despite sitting in the map census page.
