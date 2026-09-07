# Idriç rewrite

This branch starts a clean Idriç implementation of TLS alongside the inherited OpenSSL tree.
It is not an Idriç binding to OpenSSL.

The inherited C source is useful as a behavior inventory and compatibility reference, but new Idriç modules should express protocol concepts directly and should not reproduce OpenSSL's BIO, provider, allocator, error-stack, or object-model machinery unless a protocol requirement actually needs an equivalent concept.

## Normative TLS reference

Use RFC 9846 as the TLS 1.3 specification. RFC 9846 obsoleted RFC 8446 in July 2026 while retaining TLS version 1.3 and wire compatibility. The first framing slice below uses wire values and record limits that are unchanged by that update.

Future handshake work must incorporate RFC 9846's tightened requirements rather than copying an RFC 8446-era implementation mechanically. In particular: KeyShare values must not be reused between connections, TLS 1.0/1.1 negotiation is forbidden, KeyUpdate requirements are stricter, and alert `general_error` is now defined.

## Implemented first slice

`idric/TLS/Record.idric` implements pure TLS 1.3 record semantics:

- the four TLS record content types used by TLS 1.3;
- parsing the five-byte TLS record header;
- consuming exactly the payload length declared by that header while returning bytes belonging to the next record;
- refusal of a truncated record payload;
- the TLS 1.3 ciphertext legacy-version rule (`0x0303`);
- the TLS 1.3 ciphertext size limit (`2^14 + 256` bytes);
- a distinct type for valid TLS 1.3 inner content;
- removal and counting of TLS 1.3 zero padding;
- refusal of all-zero inner plaintext, `change_cipher_spec` as inner content, unknown inner content types, and oversized plaintext content.

`idric/TLS/Handshake.idric` implements pure TLS 1.3 handshake framing:

- named TLS 1.3 handshake message kinds instead of a raw integer;
- parsing the one-byte kind plus three-byte length header;
- refusal of unsupported handshake kinds.

OpenSSL's inherited `ssl/record` code remains present for comparison, but the Idriç implementation does not call it.

## Deliberately not claimed

This slice does **not** yet implement a usable TLS connection. In particular it does not claim:

- AEAD encryption or decryption;
- HKDF or transcript hashing;
- X25519, P-256, signatures, or other key operations;
- secure random generation;
- ClientHello or ServerHello bodies;
- the TLS 1.3 handshake state machine;
- certificate parsing or verification;
- hostname verification;
- socket I/O;
- alerts, shutdown, session tickets, resumption, PSK, 0-RTT, DTLS, QUIC, or legacy TLS versions.

Those should be added as separately testable Idriç modules. Native primitives should be introduced only where the compiler or operating system genuinely owns an operation; protocol logic itself should remain ordinary Idriç.

## Build

The focused rewrite does not use OpenSSL's build system:

```sh
make -f Makefile.idric idric-test IDRIC=/path/to/edric
```

CI pins an exact Idriç compiler commit and runs the same acceptance target.
