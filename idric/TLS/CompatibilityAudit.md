# TLS 1.3 static compatibility audit

This note records protocol and compiler-boundary checks found while the exact-head Idriç acceptance run is queued.

## Protocol checks already represented

- TLS 1.3 record content and length boundaries are typed separately from handshake messages.
- Handshake transcript bytes exclude record headers and retain exact handshake header plus body bytes.
- HelloRetryRequest uses the synthetic `message_hash` transcript form.
- Duplicate extension types are rejected while unknown extension codes remain representable for registry evolution.
- `supported_versions`, `supported_groups`, `signature_algorithms`, `key_share`, `psk_key_exchange_modes`, and `pre_shared_key` have semantic parsers rather than raw integer/string interpretation.
- Client key shares are checked as an ordered subset of `supported_groups`.
- Server selections are checked against the client's version, cipher-suite, extension, key-share, and PSK offers.
- `psk_ke` and `psk_dhe_ke` remain distinct because ServerHello `key_share` presence identifies the selected mode.
- A second HelloRetryRequest is rejected.
- KeyUpdate response obligations are represented as state and block subsequent Application Data until discharged.
- Current RFC 9846 alert behavior includes warning `close_notify`, `general_error`, and fatal treatment of unknown/error alerts.

## Deliberately unresolved until the compiler receipt is green

The current branch does not yet execute cryptographic primitives. The next implementation boundary should keep these separate from protocol state:

- SHA-256 / SHA-384 transcript hashing;
- HMAC and HKDF Extract/Expand;
- AEAD seal/open for the five base TLS 1.3 suites;
- X25519/X448, NIST-curve, and FFDHE shared-secret operations;
- signature verification and CertificateVerify context construction;
- secure random generation and ephemeral-key lifetime/erasure;
- X.509 path and hostname verification.

Protocol types should select these operations and constrain their lengths/purposes. They should not collapse them into generic strings, integers, or an OpenSSL-style opaque context.

## Compiler receipt

An earlier run failed before any TLS source compiled because the pinned Idriç snapshot's wrapper delegates `bootstrap` to a missing Make target. The current receipt explicitly bootstraps upstream Idris2 0.8.0 through Chez, then uses that stage-zero compiler to build the pinned Idriç fork and bootstrap libraries.

No compiler PASS is claimed by this note.
