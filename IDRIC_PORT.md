# Idriç rewrite

This branch starts a clean Idriç implementation of TLS alongside the inherited OpenSSL tree.
It is not an Idriç binding to OpenSSL.

The inherited C source is useful as a behavior inventory and compatibility reference, but new Idriç modules should express protocol concepts directly and should not reproduce OpenSSL's BIO, provider, allocator, error-stack, or object-model machinery unless a protocol requirement actually needs an equivalent concept.

## Normative TLS reference

Use RFC 9846 as the TLS 1.3 specification. RFC 9846 obsoleted RFC 8446 in July 2026 while retaining TLS version 1.3 and wire compatibility.

The implementation should follow RFC 9846's tightened requirements rather than copy an RFC 8446-era implementation mechanically. Current code already accounts for the second-HelloRetryRequest refusal, KeyUpdate request/response restrictions, warning-level `close_notify`, fatal treatment of error and unknown alerts, and the new `general_error` alert.

## Implemented protocol slice

`idric/TLS/Record.idric` implements TLS record framing:

- named TLS record content types;
- five-byte record-header parsing;
- exact payload boundary consumption with truncated-record refusal;
- TLS 1.3 ciphertext legacy-version and size rules;
- a distinct TLS 1.3 inner-plaintext type;
- zero-padding removal and inner-content validation.

`idric/TLS/Handshake.idric` implements handshake framing:

- named TLS 1.3 handshake message kinds;
- one-byte kind plus three-byte length parsing;
- refusal of unsupported handshake kinds.

`idric/TLS/Extension.idric` implements bounded extension vectors:

- length-delimited extension framing;
- preservation of unknown extensions as typed code plus bytes;
- duplicate-aware lookup for extensions that must occur at most once;
- client and server `supported_versions` parsing;
- structurally decreasing parser fuel rather than disabling totality checking.

`idric/TLS/Hello.idric` implements a first TLS 1.3 hello boundary:

- ClientHello legacy version, 32-byte random, session-id bound, cipher-suite vector, exact legacy compression `[0]`, and required TLS 1.3 `supported_versions`;
- ServerHello legacy version, 32-byte random, session-id echo bound, TLS 1.3 cipher-suite selection, zero legacy compression, and selected TLS 1.3 version;
- all five cipher suites defined by the base TLS 1.3 specification;
- recognition of HelloRetryRequest by its fixed random value;
- explicit refusal cases for malformed vectors, duplicate/missing version negotiation, bad compression, and unsupported selected suites.

`idric/TLS/Alert.idric` models current TLS 1.3 alert semantics, including `general_error`, warning `close_notify`, and fatal treatment of error or unknown alerts.

`idric/TLS/KeyUpdate.idric` models the post-handshake update obligations: a peer update cannot be requested repeatedly without receiving the peer's next KeyUpdate, and a requested response must be sent before subsequent Application Data.

`idric/TLS/ClientState.idric` models the client-side handshake ordering for certificate, optional client-authentication, and PSK paths. Unexpected transitions and a second HelloRetryRequest are refused rather than represented as arbitrary integer states.

The acceptance executable exercises both successful paths and refusal cases across these modules.

## Deliberately not claimed

This still does **not** implement a usable TLS connection. In particular it does not yet claim:

- AEAD encryption or decryption;
- HKDF, HMAC, or transcript hashing;
- X25519, P-256, signatures, or other key operations;
- secure random generation or KeyShare generation/reuse prevention;
- detailed parsing/validation of key_share, supported_groups, signature_algorithms, PSK, certificate, CertificateVerify, Finished, or EncryptedExtensions bodies;
- certificate-chain or hostname verification;
- a server-side handshake state machine;
- socket I/O;
- traffic-secret/key derivation or record encryption state;
- shutdown transport behavior beyond alert semantics;
- session tickets, resumption, 0-RTT, DTLS, QUIC, or legacy TLS implementations.

Those should remain separately testable Idriç modules. Native primitives should appear only where the compiler, operating system, or cryptographic primitive genuinely owns an operation; protocol logic itself should remain ordinary Idriç.

## Build and receipt

The focused rewrite does not use OpenSSL's build system:

```sh
make -f Makefile.idric idric-test IDRIC=/path/to/edric
```

CI pins Idriç commit `d2463ec8a3a0dd4ac167029927452f3e83805dc3`, whose active compiler tree lives under `_`. The rewrite workflow therefore bootstraps through `.tools/Idric/_/edric` and compiles with `.tools/Idric/_/build/exec/idris2`.

An earlier dedicated run (`34104452866`) failed before compiling any TLS source because the workflow incorrectly invoked `make bootstrap` at the Idriç repository root. That run is not a TLS failure or a compiler acceptance receipt. The workflow now checks out the exact PR head, uses the relocated Idriç build entrypoint, and cancels superseded dedicated runs. A current-head PASS is still required before this draft should claim compiler acceptance.
