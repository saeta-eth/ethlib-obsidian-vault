---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
rfc2104_version: "February 1997"
rfc5869_version: "May 2010"
tags:
  - ethlib
  - external-sources
  - hkdf
---

# HKDF extract, expand, and context

HKDF separates concentrating input keying material from expanding a pseudorandom key into output bytes.
Its salt, context, counter, and output length have different roles.
Swapping them can produce deterministic bytes that satisfy a length check while missing the protocol's intended derivation.
This note traces one published SHA-256 vector and identifies what its callers still need to establish.

## Sources and scope

The primary texts are [RFC 2104, HMAC: Keyed-Hashing for Message Authentication][hmac-rfc], February 1997, and [RFC 5869, HMAC-based Extract-and-Expand Key Derivation Function (HKDF)][hkdf-rfc], May 2010.
Both were published as Informational RFCs.
Reading covered RFC 2104 §§2–3 and §6, and RFC 5869 §§1–5 and Appendix A.1.
The fixed RFC numbers identify the published texts; this note does not assess later errata or the security of every protocol using them.

EthLib observations use commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`.
The selected functions implement HMAC-SHA-256 and HKDF-SHA-256, rather than a hash-parameterized interface.
The [validator-key note](Validator-key-derivation-and-keystores.md) describes additional application-specific derivation rules at its own recorded revisions.
Generic HKDF defaults do not replace such rules.

## Keep the HMAC arguments in order

For SHA-256, the hash output length, `HashLen`, is 32 bytes, while its compression block size is 64 bytes.
HMAC first hashes a key longer than one block; otherwise it keeps the key.
It then pads that result with zeros to a 64-byte block `K0`.
Its inner and outer computations use different repeated pad bytes:

```text
HMAC(K, message) = SHA256((K0 XOR opad) || SHA256((K0 XOR ipad) || message))
ipad = 0x36 repeated 64 times
opad = 0x5c repeated 64 times
```

RFC 5869's [Extract][extract-rfc] puts `salt` in HMAC's key position and input keying material, `IKM`, in its message position.
Expand instead keys HMAC with the pseudorandom key, `PRK`.
An extraction with a public salt is not a message-authentication claim about the sender of IKM.
The same HMAC construction serves a different role there.

## The published input and extraction

[RFC 5869 Appendix A.1][case1] publishes this SHA-256 case.
All hex below represents decoded bytes, not ASCII hex text:

```text
IKM  = 0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b  (22 bytes)
salt = 000102030405060708090a0b0c                    (13 bytes)
info = f0f1f2f3f4f5f6f7f8f9                          (10 bytes)
L    = 42 bytes

PRK = 077709362c2e32df0ddc3f0dc47bba6390b6c73bb50f9c3122ec844ad7c2b3e5
```

The PRK is the published 32-byte result of `HMAC-SHA-256(salt, IKM)`.
Here the 13-byte salt needs no preliminary hash: HMAC pads it with 51 zeros to obtain `K0`.
Its inner hash input has `64 + 22 = 86` bytes, and its outer input has `64 + 32 = 96` bytes.
Neither calculation treats the IKM as the HMAC key.
These are public test inputs, not usable secret material.

## Two blocks, then truncate

[Expand][expand-rfc] computes `N = ceil(L / HashLen)` blocks, so this case needs two.
The initial block `T(0)` is empty.
Each next block includes the previous block in full, the unchanged `info`, and one counter byte:

```text
T(1) = HMAC(PRK, info || 01)
T(2) = HMAC(PRK, T(1) || info || 02)
OKM  = first 42 bytes of (T(1) || T(2))
```

The counter is not ASCII `"1"` or a four-byte integer.
The first HMAC message has 11 bytes; the second has `32 + 10 + 1 = 43` bytes.
The following intermediate values were derived with Python's standard HMAC/SHA-256 functions from the RFC inputs:

```text
T(1) = 3cb25f25faacd57a90434f64d0362f2a2d2d0a90cf1a5a4c5db02d56ecc4c5bf
T(2) = 34007208d5b887185865b4b0a85a993b89b9b65683d60f0106d28fff039d0b6f
```

Appendix A.1 publishes PRK and OKM, not the full second block above.
The first block is also visible as the first 32 bytes of its published OKM.
Keep those 32 bytes and only the first 10 bytes of `T(2)`, `34007208d5b887185865`.
Their concatenation matches the published 42-byte result:

```text
OKM = 3cb25f25faacd57a90434f64d0362f2a2d2d0a90cf1a5a4c5db02d56ecc4c5bf
      34007208d5b887185865
```

Concatenate the lines before decoding.
Truncation keeps the leading bytes of the combined expansion; it does not shorten PRK or truncate every HMAC block separately.

## Missing salt differs from empty context

RFC 5869 sets an absent salt to `HashLen` zero bytes.
For this SHA-256 HMAC, an explicitly empty key and a 32-zero-byte key normalize to the same 64-zero-byte `K0`.
Thus they produce the same extraction result for any IKM.
This is key normalization, not a general statement that arbitrary salt values are interchangeable.

By contrast, absent `info` can be represented by a zero-length byte string.
With empty info, the first Expand message contains only `01`.
It does not contain 32 zero bytes followed by `01`.
EthLib's [`extract`][extract] and [`expand`][expand] take explicit byte strings, without an absence flag.
The theorem [`extract_empty`][empty-proof] proves the empty-salt equivalence above.
The API does not validate that a supplied salt has the independence or provenance needed by a particular protocol.

## Bounds and successful-call properties

The RFC limit is `255 * HashLen`, or 8160 bytes for SHA-256.
It bounds the number of blocks by the one-byte counter's range, 1 through 255.
The relevant length is 32, not the 64-byte HMAC block size.
The inspected EthLib branches give these outcomes, without running them as tests:

| Direct Expand input | Outcome in the definition |
| --- | --- |
| PRK shorter than 32 bytes, even with `L = 0` | `keyTooShort`. |
| PRK at least 32 bytes, `L = 0` | Success with empty output. |
| PRK at least 32 bytes, `L = 8160` | Success using 255 blocks. |
| PRK at least 32 bytes, `L = 8161` | `outputTooLong`. |

[`expand_ok_iff`][bounds-proof] states exactly the key-length and output-length acceptance conditions.
The full `hkdf` first derives a 32-byte PRK, so only the excessive output length can make that composition fail.
Neither acceptance nor a sufficiently long PRK establishes entropy or pseudorandomness.
There is no general 32-byte IKM minimum in this generic function.

## Context and security remain caller obligations

For successful expansions with one PRK and one info, [`expand_prefix`][prefix-proof] proves that the shorter output is a prefix of the longer.
In this case, choosing `L = 32` yields exactly `T(1)`.
Changing only L therefore does not select independent context bytes.
RFC 5869 §3.2 permits an application to bind L through info when that distinction is needed; it does not require every application to do so.

A caller also needs an unambiguous context encoding.
For example, concatenating ASCII fields `"ab" || "c"` and `"a" || "bc"` gives the same info bytes `61 62 63`.
With the same PRK and L, both derivations are identical.
Length prefixes or another agreed encoding can preserve field boundaries; the RFC does not prescribe one universal format.
Different context encodings are design inputs, not an absolute guarantee that different outputs can never collide.

RFC 5869 §3.2 says info should be independent of the IKM value.
Salt need not be secret and can be reused, but §3.4's analysis assumes independence from IKM and excludes attacker manipulation.
Its nonce example requires authenticating the nonces' source before using them as salt.
Section 3.3 says extraction SHOULD NOT be skipped for a Diffie-Hellman value, and discourages using PRK directly as short output instead of expanding with info.
Section 4 also explains that HKDF cannot create entropy or supply a password KDF's intentional work factor.
Security depends on input assumptions, the underlying functions, and the protocol; these algorithmic properties do not prove a whole key exchange secure.

The selected definitions and theorem statements were read at the recorded commit.
The author calculation reproduced the RFC PRK/OKM and derived the displayed intermediate blocks with separate Python tools.
The source passages, explanation, and cited definitions received a separate review.
An independent standard-library calculation reproduced PRK, OKM, both full blocks, message lengths, truncation, and the empty-salt equivalence for these inputs.
It also checked the counter-bound arithmetic and concrete context contrasts.
No Lean build, proof replay, EthLib vector execution, protocol-security proof audit, or constant-time assessment was performed.

[hmac-rfc]: https://www.rfc-editor.org/rfc/rfc2104
[hkdf-rfc]: https://www.rfc-editor.org/rfc/rfc5869
[extract-rfc]: https://www.rfc-editor.org/rfc/rfc5869#section-2.2
[expand-rfc]: https://www.rfc-editor.org/rfc/rfc5869#section-2.3
[case1]: https://www.rfc-editor.org/rfc/rfc5869#appendix-A.1
[extract]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Hkdf.lean#L50
[expand]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Hkdf.lean#L70
[empty-proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Hash/Hkdf.lean#L145
[bounds-proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Hash/Hkdf.lean#L106
[prefix-proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Hash/Hkdf.lean#L121
