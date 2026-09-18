# naya-create-kb

Open knowledge base for the **Naya Create** split ergonomic keyboard
(Naya B.V. — company bankrupt, no vendor support): hardware, USB CDC
protocol, LittleFS/NVS storage, bootloader, host software internals,
disassembly findings, and a recovery cookbook. Written from live
device probing and static reverse engineering of NayaFlow/NayaCore.

## Read it as a website

```bash
pip install -r requirements.txt
mkdocs serve        # → http://127.0.0.1:8000
```

Publish to GitHub Pages (needs a push + Pages enabled on the repo):

```bash
mkdocs gh-deploy
```

Or just read the Markdown in [`docs/`](docs/) — every page stands alone.

## Layout

| Path | What |
|---|---|
| `docs/device/` | Specs, hardware deep dive, power architecture |
| `docs/protocol/` | Transport, framing, full command map, keymap/LED/settings/module records |
| `docs/storage/` | LittleFS & NVS, factory reset (`30/10ca`) |
| `docs/connectivity/` | Bluetooth, GATT services, custom pipe `0x1234` |
| `docs/firmware/` | MCUBoot serial recovery, RSA-2048 signing |
| `docs/software/` | NayaFlow/NayaCore, RPC↔ZMQ bridge, app data & backups |
| `docs/disassembly/` | Provenance, method, key functions, host-side maps |
| `docs/toolkit/` | CLI tools + copy-paste Python/JS recipes |
| `docs/recovery.md` | Symptom → recipe cookbook |
| `docs/glossary.md` | Terms (KK, T-record, NVS, wedge, ff-phase, …) |

## Ground truth & provenance

All wire facts were verified live against base FW **0.3.41.0** / module FW
**0.2.3.3** / NayaFlow **1.25.1** / NayaCore **v6.11.0** (macOS arm64),
or derived statically from the installed NayaCore binary
(md5 `84ded78022b6d312483aae0192584168`; archived RE copy
`b3dd3e14c255886d05b4a2728f61ffaa`).

Conventions used across pages:

- `xx/yyyy` = CDC frame family `xx`, command `yyyy` (e.g. `30/1004`).
- Bytes in hex, `KK` = key index, `T` = record type, `LEN` = payload length.
- “ACK `00`” = status byte zero. **ACK proves parse, not apply** — always
  verify with a readback.
- Every page marks open questions explicitly as **OPEN**.
