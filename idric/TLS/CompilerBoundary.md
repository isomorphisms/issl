# Compiler and cryptographic boundary

The Idriç rewrite deliberately separates TLS protocol decisions from cryptographic execution.

A protocol module may select an algorithm, a key length, an IV length, a transcript-hash family, a KDF purpose, a key-share group, or a handshake transition. It must not obtain those distinctions by passing arbitrary strings or integers to a generic native context.

The first executable cryptographic layer should therefore expose purpose-specific operations whose inputs already carry the protocol constraints established by the pure modules. Examples are transcript hashing over exact handshake bytes, HKDF derivation with typed TLS labels, AEAD protection under suite-selected parameters, and shared-secret derivation for a negotiated group.

Native/compiler primitives, if temporarily necessary, belong below those operations and must not leak through the public TLS state machine. Replacing a primitive implementation later should not require changing the protocol types.

This file does not claim that any cryptographic primitive is implemented or verified yet.
