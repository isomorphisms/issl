# Idriç TLS modules

The active rewrite is split by protocol responsibility rather than by OpenSSL object or subsystem names.

- `Record`: TLS record framing and TLS 1.3 inner plaintext.
- `Handshake`: handshake framing and exact transcript-message bytes.
- `Extension`: extension framing, duplicate refusal, and version negotiation.
- `Negotiation`: supported groups, signature schemes, key shares, and PSK modes.
- `PSK`: offered PSK identities and binder framing.
- `ServerNegotiation`: ServerHello and HelloRetryRequest extension shape.
- `Hello`: ClientHello/ServerHello parsing.
- `ConnectionNegotiation`: checks server selections against the actual client offer.
- `SuiteParameters`: AEAD/hash/key/IV/tag parameters selected by the cipher suite.
- `Transcript`: transcript accumulation boundary and HelloRetryRequest rewrite.
- `KdfLabel`: typed TLS 1.3 HKDF labels and HkdfLabel encoding.
- `Alert`: current TLS 1.3 alert semantics.
- `KeyUpdate`: post-handshake update obligations.
- `ClientState`: client handshake message ordering.

Cryptographic execution, certificate verification, sockets, resumption, early data, DTLS, and QUIC remain outside the implemented boundary.
