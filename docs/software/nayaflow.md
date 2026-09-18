# NayaFlow / NayaCore / flow-bg-server

Stock host stack (macOS, app v1.25.1), three processes:

| Process | Binary | Role |
|---|---|---|
| NayaFlow | Electron app (`/Applications/NayaFlow.app`) | renderer UI + main |
| flow-bg-server | Go helper (`Contents/flow/flow-bg-server -port 56486`) | HTTP bridge: renderer ↔ NayaCore |
| NayaCore | Qt service (`Contents/core/NayaCore.app/…/MacOS/NayaCore`, v6.11.0) | owns USB CDC + BLE, SQLite, ZMQ REP on `127.0.0.1:56500` |

CI paths leaked in the binary: `/Users/runner/work/NayaCore/NayaCore/…`
(GitHub Actions macOS runner). Full serial-side source map (embedded
paths) spans `Naya_SerialPort/{Naya_SerialWorker, …,
Naya_DeviceManager{,_Operation,_Enqueue,_FWUpdate,_ModuleFwUpdate,
_Pairing,_ClearAllData,_ClearBLEDevices,_TestSPIFlash},
MCUBootWorker/{,_CreateLeft{,_Modules},_CreateRight},
ProtocolCDCWorker/{…, Message_Worker/{Integration_Worker,
Process_Worker/{…, System, BLE, Firmware, Flash, LED, META, Module,
Remap, SysPower}}}}}`.

## What stock flashing actually does

Remap flash = **per-key `30/1004` writes** (+ `30/100e` for colors),
each verified by a full readback: READ-ALL → WRITE → READ-ALL.
Stock ritual chain (static): `_remapWriteLayerList` → `_remapReadLayerData`
(30/1003) → `_remapWriteLayerData` (30/1004) → `_remapWriteModuleConfigList`
(30/100A) → `_remapReadModuleData` (30/100B) → `_remapWriteModuleData`
(30/100C) → READ color (30/100D) → WRITE color (30/100E).

Single-key color flash = lone `30/100e` on the LEFT port only; the
right port gets polls only, forever (NayaFlow's path to right-half
colors stays unobserved).

## Dead / host-only features (no wire path)

- Settings UI for scan-mode / LED-max / LED-override-mode: ZERO ED
  frames on flash (only `fe/100a` timeouts + reads).
- Per-layer LED animations: registry
  `{SOLID, SWIRL, BREATHE, SPECTRUM}` + icons exist, `animation_id`
  NULL in DB, no renderer→backend calls.
- Macros: device store answers but is always empty; zero macro lines
  in NayaCore logs — NayaFlow never syncs them.
- Naya-type palette actions (MAC_OS/WINDOWS_OS/…): zero `tap:` hits —
  the device gets DISABLE/plain-HID for those slots.
- “Factory Reset” string in the UI = app-settings reset (language/
  theme group), not a device wipe. There is no device-factory-reset
  button in stock UI.

## Extracting the renderer

`app.asar` unpacks with a tiny header formula
(`jsize=u32@12, json@16, base=align4(16+jsize)`, offsets relative to
base) or `npx --yes @electron/asar extract`. Interesting trees:
`dist/main/index.js`, `dist/renderer/assets/`, `assets/icons/action/`
(860 keycap glyphs — reused for the open web client's icon set).
