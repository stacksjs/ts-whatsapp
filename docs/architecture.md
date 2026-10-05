# Architecture

`ts-whatsapp` is a WhatsApp *linked device* (a companion, like WhatsApp Web)
written from scratch in TypeScript on Bun's standard crypto. It never depends
on another WhatsApp or Signal implementation at runtime; reference
implementations are used only in tests, to check outputs bit for bit.

Every layer works on `Uint8Array` and has no I/O of its own except `client/`.

| Layer | Directory | What it is |
|---|---|---|
| Crypto | `src/crypto/` | Curve25519 key agreement, XEdDSA signatures, HKDF, HMAC, SHA, AES-GCM/CBC/CTR |
| Protobuf | `src/proto/` | A small protobuf runtime and the message schemas WhatsApp uses |
| WABinary | `src/binary/` | WhatsApp's binary node format (an XML-like tree in compact bytes) and JIDs |
| Noise | `src/noise/` | The Noise_XX_25519_AESGCM_SHA256 handshake and framed transport |
| Signal | `src/signal/` | X3DH sessions, the Double Ratchet, sender keys for groups, and key generation |
| Client | `src/client/` | Connection, pairing, login, sending and receiving, receipts, history and app-state sync, media |

## Conventions shared by every layer

- Keys are raw 32-byte `Uint8Array`s. A Signal *public* key travels with a
  leading type byte `0x05` (33 bytes); `crypto/curve.ts` converts both ways.
- No classes hold global state. Stores (Signal sessions, keys, credentials)
  are interfaces the caller implements; `src/store/` has in-memory and
  file-backed versions.
- Every public function has a test. Where a reference exists (libsignal,
  RFC test vectors, captured frames), the test compares against it exactly.
- Names follow the protocol's own terms (`pkmsg`, `skmsg`, `usync`) so the code
  can be read next to a protocol description.
