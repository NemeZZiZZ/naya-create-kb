# Python recipes

Needs `pyserial`. Conventions: `DST_LEFT = 0x50`, `DST_RIGHT = 0x51`;
`transact(port, dst, type_, c0, c1, params, timeout)` (dst is
positional); `Session(port, dst, timeout)` performs the `30/1001`
handshake — `30/10xx` needs it on the same open handle.

## Minimal frame (illustrative sketch)

```python
def build_frame(dst, type_, c0, c1, params=b""):
    body = bytes([type_, len(params) + 2, c0, c1]) + bytes(params)
    crc = 0
    for b in body[1:]:      # XOR from C0 through end of payload
        crc ^= b
    return b"\xaa\x00" + bytes([dst, 0]) + body + bytes([crc, 0x04])

def parse_frame(f):
    assert f[0] == 0xAA and f[-1] == 0x04
    crc = 0
    for b in f[6:-2]:
        crc ^= b
    assert crc == f[-2], "CRC mismatch"
    return {"src": f[1], "type": f[4], "c0": f[6], "c1": f[7],
            "payload": f[8:-2]}
```

## Session + reads

```python
from cdc_client import Session, DST_LEFT  # names illustrative

ses = Session("/dev/cu.usbmodem1101", DST_LEFT)  # handshake inside
info = ses.cmd(0xFA, 0x10, 0x01, b"")            # fa/1001 device info
fw   = ses.cmd(0xFE, 0x10, 0x02, b"")            # fe/1002 → 00 00 03 29 00 38 = 0.3.41.0
layer0 = ses.read_layer(0)                        # 30/1003 multipart, both parts
```

## Writes (same-length only!)

```python
# key write, layer-aware ACK (accept 00 00 OR 00 <layer>)
ack = ses.write_key(record_bytes, layer=1)
assert len(ack) == 2 and ack[0] == 0x00 and ack[1] in (0x00, layer)

# LED single-entry write: [00, layer] + [KK, H_lo, H_hi, S]
ses.cmd(0x30, 0x10, 0x0E, bytes([0x00, layer, kk, h & 0xFF, h >> 8, s]))

# ED: ALWAYS [target, value] (0xFF = all). 1-byte sends write ZEROS to NVS.
ses.cmd(0xED, 0x10, 0x13, bytes([0xFF, 100]))  # 1013 ceiling = 100
```

## Snapshot & restore (CLI)

```bash
python3 toolkit/naya-backup.py                      # → research/dumps/*.json
python3 toolkit/naya-restore.py --snap snap.json    # dry run
python3 toolkit/naya-restore.py --snap snap.json --apply   # writes
python3 toolkit/naya-undark.py --apply              # LED revival (left)
python3 toolkit/naya-maxbrt.py --level 100 --apply  # 1013 ceiling
```

## Factory format (destructive — read the risks first)

```python
# left port, dst 0x50: type 0x30 c0 0x10 c1 0xCA params [01]
resp = transact(port, 0x50, 0x30, 0x10, 0xCA, b"\x01")
# expect status 00; afterwards 30/1001 + 30/1003 answer error 16 00 (empty store)
```

See [factory reset](../storage/factory-reset.md) and the
[recovery cookbook](../recovery.md).
