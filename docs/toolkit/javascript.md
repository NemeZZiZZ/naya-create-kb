# Web/JS recipes

Sketches adapted from the open web client (`src/lib/naya.ts`):
Web Serial / Web BLE transport, same wire bytes as Python.

## Frame build + parse (TypeScript sketch)

```ts
function buildFrame(dst: number, type: number, c0: number, c1: number, params: number[]): Uint8Array {
  const body = [type, params.length + 2, c0, c1, ...params];
  let crc = 0;
  for (const b of body.slice(1)) crc ^= b;   // XOR from C0
  return new Uint8Array([0xaa, 0x00, dst, 0x00, ...body, crc, 0x04]);
}

interface Frame { src: number; type: number; c0: number; c1: number; payload: Uint8Array }

function parseFrame(f: Uint8Array): Frame {
  let crc = 0;
  for (let i = 6; i < f.length - 2; i++) crc ^= f[i];
  if (crc !== f[f.length - 2]) throw new Error("CRC mismatch");
  return { src: f[1], type: f[4], c0: f[6], c1: f[7], payload: f.slice(8, -2) };
}

// Layer-echo ACK: accept 00 00 OR 00 <layer>
function isWriteAck(ack: Uint8Array, layer: number): boolean {
  return ack.length === 2 && ack[0] === 0x00 && (ack[1] === 0x00 || ack[1] === layer);
}
```

## Session calls (client API shape)

```ts
await ses.cmd(0xfa, 0x10, 0x01, []);        // fa/1001 device info
await ses.cmd(0xfe, 0x10, 0x02, []);        // fe/1002 firmware version
await ses.writeKey(record, layer);           // 30/1004 + layer-aware ACK check inside
await ses.writeLed(kk, h, s, layer);        // 30/100e single-entry
await ses.readModuleConfig(layer);          // 30/100b multipart
const t = await ses.getTimeouts();           // fe/100b → { idleMs, sleepMs, deepMs }
await ses.setTimeouts(idleMs, sleepMs, deepMs); // fe/100a 13 B payload
```

Notes: `cmd()` returns the parsed frame; settings ACKs carry no
layer echo (status byte `0x00` check, not `isWriteAck`); never send
`fe/100a` as a “commit”; ED payloads are always `[target, value…]`;
right-half `30/10xx` and `ed/1014` never answer — don't await them,
fail fast.
