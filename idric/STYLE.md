# Idriç source style in `issl`

The repository root `STYLE.md` belongs to the inherited OpenSSL C tree. Idriç code under `idric/` follows the current Idriç source conventions instead.

## Read the protocol first

A file near the top of `idric/TLS/` should primarily name TLS concepts, rules, states, and refusals. Representation and compiler machinery should be pushed into lower-level modules.

Prefer source that reads in purpose order:

1. name the protocol choices and records;
2. state the accepted transitions or parsing rules;
3. refuse invalid protocol states explicitly;
4. leave byte slicing, integer widths, primitives, and host machinery below that layer.

## Names

Use ordinary domain words and `snake_case`.

Do not introduce inherited names such as `Nat`, generic `Int`, `Vect`, `Ptr`, or backend-specific vocabulary into protocol code when the program is describing a TLS concept.

Use `ℕ` where a count or nonnegative length is actually the mathematical quantity being modeled.

## Wire representation

`Byte` means one uninterpreted raw 8-bit wire value. `bytes` means a sequence of such values. Those names are defined in `TLS.Wire`, which owns bit-width conversion and exact byte slicing.

Protocol modules may necessarily accept or return `bytes` at serialization boundaries, but should not duplicate `Bits8`/`Bits16` arithmetic or byte-vector mechanics internally.

If a number has protocol meaning, give it a protocol type or name rather than letting the raw encoded integer become the definition of the concept.

## Constructors and semantic helpers

Positional `Mk...` constructors are tolerated where the currently pinned compiler requires the inherited Idris record form. New high-level code should prefer semantic helper operations when they make call sites read more clearly.

Do not replace a meaningful TLS object with a tuple merely to shorten a definition.

## Equality and newer notation

The wider Idriç language is converging on `=` for equality and `≝` for definition. This branch is pinned to an older compiler snapshot whose accepted surface still contains inherited equality/definition forms. Do not make a style-only change that knowingly stops the pinned acceptance build from parsing.

When compiler support catches up, update the source rather than treating the compatibility spelling as canonical.

## Cryptographic boundary

Protocol types decide what operation is required. Cryptographic or native implementations execute that decision below the protocol layer. Raw FFI, provider contexts, allocator details, and backend representation must not become the public TLS ontology.

See `TLS/CompilerBoundary.md` for the existing architectural rule.
