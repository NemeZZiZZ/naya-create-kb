# Disassembly — provenance & method

Target: `NayaCore` (Qt host service inside `NayaFlow.app`,
NayaCore v6.11.0, app v1.25.1, macOS arm64).

## Provenance

| Copy | MD5 | Notes |
|---|---|---|
| Archived RE copy (`NayaSniff.app`, ad-hoc re-signed for the interposer rig) | `b3dd3e14c255886d05b4a2728f61ffaa` | basis of the archived `nayacore-dis.txt` |
| Installed copy (`/Applications/NayaFlow.app/Contents/core/…/MacOS/NayaCore`) | `84ded78022b6d312483aae0192584168` | differs (app updated in place); same disassembly line layout, signature only |

**Re-derive from your own install.** Line numbers cited in findings
refer to `otool -tV` output of `(__TEXT,__text)` (~1.7 M lines).

## Method (hands-free, reproducible)

1. `otool -tV <binary> > nayacore-dis.txt`, `strings <binary> > nayacore-str.txt`.
2. Logged wire constants == code constants: every `construct*Commands`
   table was cross-checked against ≥1 live frame before trusting it
   (20+ cross-checks; the tables below are the ones that survived).
3. Jump-table + `ldrsb`/`adrp` + literal-pool-symbol reads resolve
   handler maps (`ED 0x1003–0x1050` via jump table on `w22-0x1003 <=
   0x4d`; `operationMinFWVersion` switch on operation id; ZMQ dispatch
   as a string-compare chain).
4. Extractor scripts (run against the disassembly text): single-insert
   census → key-batch v1–v4 with base-mutation + w21-register tracking
   (282/282 pairs final). See [host maps](host-maps.md).
5. Reader hunts: full `adrp→page` census per member offset; a map with
   14 fill-cluster users and zero readers is declared write-only
   (the `+0x70` verdict).

## What static analysis can and cannot do here

- CAN: command tables, record layouts, insert values, caller censuses,
  whitelist/blacklist verdicts (e.g. the 15-event ZMQ list).
- CANNOT: dynamically-initialized static values (all
  `naya_fw::*_MinVersion` gates read zero from file `__DATA` — guard
  vars `__ZGV` present; version-gate thresholds are not statically
  extractable), runtime-base+offset reads, anything behind a
  `runtime base + offset` pattern (declared OPEN, not guessed).
