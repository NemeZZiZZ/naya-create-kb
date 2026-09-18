# Firmware signing keys: what the fundraiser must buy

## One key signs everything

All 15 stock images — every base epoch (left/right, including the
`_64` second-revision boards) **and** the module image — carry the
same MCUboot KEYHASH TLV (type `0x01`):

```
KEYHASH = de8b07187913e6e788306618e4166e38a8c2eda99b68970d17fd00e75fd5b972
```

So there is a single RSA-2048 signing keypair for the whole product
line (base halves + Track/Tune/Touch modules). Whoever holds **that
one private key** can sign firmware for every component. TLV layout
per image: `0x10` SHA256 (32 B) + `0x01` KEYHASH (32 B) +
`0x20` RSA2048-PSS sig (256 B) + `0x30` ENCRSA2048 wrapped key (256 B).

## Verify before money changes hands

A verification script checks a claimed private key with zero trust:
derive the public key, hash it in all common encodings, compare
against the KEYHASH above, plus an RSA-PSS roundtrip:

```bash
pip install cryptography
python3 verify-signing-key.py claimed-key.pem   # exit 0 = genuine
```

The fundraiser should publish (or escrow-verify) this check: anyone
offering “the Naya keys” proves possession by running the script.
No key, no deal.

## What exactly to ask for

1. **RSA-2048 MCUboot signing PRIVATE key** matching the KEYHASH
   above. This alone unlocks custom base + module firmware via the
   stock update path.
2. Nice-to-have: the image-encryption RSA keypair (TLV `0x30` shows
   images are encrypted). Probably NOT required: MCUboot boots
   signed-but-unencrypted images (encryption is a per-image flag), so
   the first experiment after key recovery is a signed-plain test
   image. If the bootloader rejects plain images, the encryption
   public key must be recovered from a bootloader dump (SWD).
3. Nice-to-have: NayaCore/build scripts (convenience only — the update
   path through the app CDC protocol is already reverse-engineered).

## Why no donor board is needed for firmware trials

MCUboot swap semantics make experiments safe on a daily driver: a new
image uploads to the secondary slot and boots **on trial**; if it
fails verification (or crashes before confirming), the bootloader
reverts automatically. Worst realistic case of a bad first build: one
extra reboot back into stock — not a brick. SWD pads stay the last
resort, not the first step.
