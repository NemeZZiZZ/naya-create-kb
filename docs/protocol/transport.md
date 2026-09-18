# Transport & framing

USB CDC-ACM. Each half exposes CDC-Control + CDC-Data + HID
(VID `0x37D1`, PID 100 left / 200 right). On macOS the data ports show
up as `/dev/cu.usbmodem*` (one per half; during MCUBoot recovery an
extra node appears — see [bootloader](../firmware/bootloader.md)).

!!! warning "One owner at a time"
    NayaFlow holds the ports while running. **Quit NayaFlow first.**
    Baud rate is irrelevant (CDC-ACM ignores it); set DTR+RTS like the
    stock client does.

## Session rules

1. Halves must be **awake and in USB mode** — asleep halves answer
   nothing. Press keys to wake.
2. Open the port, assert DTR+RTS, then send commands.
3. The `30/10xx` (remap) family needs a **handshake first, on the same
   open handle**: `30/1001` with params `00 00`, else NO-REPLY.
4. `fa/be/de/fe` answer standalone. `30/10xx` answers **only on the
   left port** (left = merge host; right has no keymap store).
5. When halves are USB-linked, **left proxies right**: left port
   answers `dst 0x50` and `dst 0x51`; right port answers only
   `dst 0x51`. Check the reply **SRC byte**: `0x50` = left,
   `0x51` = right — do not trust which port you asked.
6. The **first frame after wake is often lost** — retry once
   (`sync lost` is the normal symptom, not a bug).

## Frame format

Request (11–12 B typical):

```
AA | 00 | DST | 00 | TYPE | LEN | C0 C1 | PARAMS... | CRC | 04
```

Response:

```
AA | SRC | 00 | 00 | TYPE | LEN | C0 C1 | PAYLOAD... | CRC | 04
```

| Field | Meaning |
|---|---|
| `AA` | header |
| `DST` / `SRC` | `0x50` left, `0x51` right (SRC echoes the request DST) |
| `TYPE` | family byte (`fa be de ed fe ff 30 …`) |
| `LEN` | request: `len(C0 C1 PARAMS)`; response: `2 + len(PAYLOAD)` |
| `C0 C1` | command id, e.g. `10 04` |
| `CRC` | **XOR of all bytes from C0 through end of payload** (`frame[6:-2]`) — verified on all 269 frames of the reference capture |
| `04` | constant footer |

Examples (left half):

- Device info: `AA 00 50 00 fa 03 10 01 00 CRC 04` → 43 B payload.
- Handshake: `30/1001` params `00 00` → 72 B (`00 00 00 10` + three 16-byte UUIDs).
- Key write: `30/1004` params `[00, layer, KK] + record` → ACK payload
  `00 00` (L0) — see [ACK semantics](#ack-semantics).

## Multi-part reads

`30/1003` (keymap) and `30/100d`/`30/100b` read with params
`[part, layer]` (2 parts per layer for keymaps). Response header byte3
= **remaining-parts counter** (`02, 01, 00…`); payload byte0 =
more-flag (`01` = more parts, `00` = last), payload byte1 = **layer echo**.

## ACK semantics

- Status byte `0x00` = parsed OK. **ACK proves parse, not apply** —
  always confirm with a readback.
- **Write ACK echoes the layer** (live-proven): `30/1004` ACK payload =
  `00 00` on L0, `00 01` on L1, `00 02` on L2. A strict `== 00 00`
  matcher false-NACKs L1/L2 writes that actually applied — accept
  `00 00` **or** `00 <layer>`.
- ED ACK byte2 is an **internal slot id, not a value echo**: it is
  constant per command (`1013→03, 1010→00, 1003→13, 1012→02, 1014→04,
  1050→40, 1008→18, 1011→01`) and does not change with the value sent.
- Error status `16 00` = empty store (seen after a data-partition
  format: `30/1001` and `30/1003` answer `16 00` until a profile is
  restored).
- Response status vocabulary (from binary strings): `Invalid command /
  Invalid format / Invalid ID / Memory full / Save failed / Load failed
  / No data / NVS`.

## What never to send

`ee/10be` + `ee/10ae` (DFU/MCUBoot resets), `fa/1002`, `fa/1006`,
text commands `clear_bonds` / `mcuboot_reset`. And **never replay
captured `fe/100a` bytes across sessions** — stale commits wedge the
state machine (a power reboot recovers, but don't).

See also: [command map](commands.md) · [recovery cookbook](../recovery.md).
