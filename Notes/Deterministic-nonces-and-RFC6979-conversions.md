---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C07
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
rfc6979_version: "August 2013, Independent Submission, Informational"
rfc6979_sha256: "456e8f17558fdbd206f968b96fc6f1b4a71ea331ab30ad17f711ab3adaa7d701"
tags:
  - ethlib
  - external-sources
  - deterministic-nonces
  - rfc6979
---

# Deterministic nonces and RFC 6979 conversions

RFC 6979 derives a signing nonce from a private key and a message hash without fresh randomness for each signature.
Its hash-to-seed conversion reduces modulo the group order; its nonce-candidate check must not do that.
This note follows both boundaries through a published P-256 control, then identifies EthLib's narrower profile.

## Source and scope

[RFC 6979, Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA)][6979], August 2013, is an Independent Submission with category Informational.
It is not an IETF Standards Track specification.
Reading covered §§2.1–2.4, §§3.1–3.6, §4, and the NIST P-256/SHA-256 `sample` case in §A.2.5.
The recorded SHA-256 identifies the downloaded RFC text.
This is a nonce-derivation check, not a reproduction of the appendix's complete signature or a security proof.

The [curve-validation note](Curve-validation-ECDSA-and-recovery-boundaries.md) supplies the separate ECDSA, recovery-parity and transaction low-s distinctions.
RFC 6979 calls the group order q; that is the quantity called n there, rather than the coordinate-field modulus p.

## Three conversions with different jobs

Let `qlen` be q's bit length and `rlen = 8*ceil(qlen/8)`.
The private key x must lie in `[1,q-1]`.
RFC 6979 §§2.3.1–2.3.4 define:

| Conversion | Exact operation |
| --- | --- |
| `bits2int(b)` | Retain the leftmost `qlen` bits if b is longer, otherwise add leading zero bits; interpret big-endian. No reduction modulo q. |
| `int2octets(x)` | Encode an integer below q in exactly `rlen/8` big-endian bytes, retaining leading zero bytes. |
| `bits2octets(b)` | Compute `bits2int(b) mod q`, then apply `int2octets`. |

Discarding excess right-hand bits differs from keeping the low bits of an oversized integer.
Encoding x as variable-length hexadecimal text also differs from encoding it as fixed-width octets.
For 256-bit q and a 32-byte hash h1, `bits2octets(h1)` is the hash integer modulo q, encoded in 32 bytes.
That simplification does not cover arbitrary order sizes or hash lengths.

## Initialize HMAC state, then generate candidates

The base procedure uses the same hash H for `h1=H(message)` and HMAC.
For SHA-256 with a 256-bit order, define the 64-byte seed material:

```text
seed = int2octets(x) || bits2octets(h1)
V = 32 bytes of 01
K = 32 bytes of 00
K = HMAC-SHA256(K, V || 00 || seed)
V = HMAC-SHA256(K, V)
K = HMAC-SHA256(K, V || 01 || seed)
V = HMAC-SHA256(K, V)
```

Each assignment uses the state produced by the preceding assignment.
The HMAC key is K, not simply the private-key bytes; x enters the HMAC message through seed material.
The two seed-bearing HMAC messages each contain `32+1+32+32=97` bytes.

RFC 6979 §3.2 then concatenates successive `V=HMAC_K(V)` outputs until T contains at least `qlen` bits.
Here a single 32-byte output suffices.
Compute `k=bits2int(T)` and accept only `1 <= k < q` and a suitable resulting ECDSA r.
An unsuitable candidate updates `K=HMAC_K(V || 00)`, then `V=HMAC_K(V)`, before trying a new T.
The rejection update uses the current V, including the candidate-generation update.

In particular, a hypothetical 256-bit candidate integer `q+1` is rejected even though `(q+1) mod q = 1` would be in range.
Zero is rejected too.
These are derived boundary cases, not published candidate outputs or executed forced-rejection tests.
Reducing candidates modulo q would change the distribution; RFC 6979 explicitly warns against that substitution.
Its hash-to-seed reduction has a different role.

## Complete published nonce control

RFC 6979 §A.2.5 specifies this NIST P-256 case:

```text
curve = NIST P-256
q = ffffffff00000000ffffffffffffffffbce6faada7179e84f3b9cac2fc632551
x = c9afa9d845ba75166b5c215767b1d6934e50c3db36e89b127b8a622b120f6721
message = ASCII "sample" = 73616d706c65
H = SHA-256
published k =
a6e3c57dd01abe90086538398355dd4c3b17aa873382b0f24d6129493d8aad60
```

The order and hash lengths are both 256 bits, and the encoded private key occupies exactly 32 bytes.
Hashing those six message bytes gives the following calculated h1:

```text
af2bdbe1aa9b6ec1e2ade1d694f41fc71a831d0268e9891562113d8a62add1bf
```

It is below q, so `bits2octets(h1)` retains these same bytes.
The calculated initialization state after step g is:

```text
K = b6d4f98ebae70aa15a2238ade4e20ab323fc1e777d22f0c582d8ef2e6ba73569
V = bae57fe256de2de806b10635497237e7bae96754582566384c47c6c3416494d1
```

The next HMAC output equals the published k and lies strictly between zero and q.
Thus the control reaches its first in-range nonce candidate after five HMAC evaluations: four initialization calls and one candidate call.
The displayed h1 and intermediate state were calculated from the procedure; §A.2.5 publishes k, not these intermediate state values.
This is a P-256 control, not a secp256k1 vector, and no curve multiplication or r/s verification ran here.

## Determinism does not make the nonce public

RFC 6979 §3.3 creates a fresh generator instance for each signature process.
Do not treat K/V as one persistent stream shared among unrelated signing requests.
Under a fixed profile, the same private key and converted hash seed reproduce the same nonce stream.
That reproducibility does not imply that different messages always produce different seeds.
Hash collisions or conversion collisions remain possible.

The nonce and its derivation state remain secret.
As the curve note illustrates, knowing a signing nonce can reveal the private key through the ECDSA equation.
RFC 6979 §4 still requires strong randomness for private-key generation and leaves key storage and access control outside its scope.
It discusses HMAC/derandomization assumptions, the double use of x, protocols that may depend on randomization, and side-channel risks.
Its recommendation is that implementations SHOULD use defensive measures against side-channel leakage.
One matching control does not establish those security conditions.

## Read the selected EthLib profile literally

At the pinned commit, [`Rfc6979.nonceSeed`][seed] assumes a 256-bit order, fixes HMAC-SHA-256, and consumes a `Vector UInt8 32` hash.
Its q and x arguments are `Nat`; the helper's type alone does not prove the order-size or private-key hypotheses.
[`nonceNext` and `nonceReject`][next] keep candidate generation and rejection updates separate.

The [`secp256k1 sign`][sign] entry point takes a scalar-field key, rejects zero, and checks candidates against that curve's order.
Its `signWith` also rejects zero r/s, while `MAX_NONCES=16` bounds retries; the RFC loop has no such fixed bound.
The implementation can separately return `unrecoverableNonce` for its parity-only recovery profile.
Low-s normalization and its parity flip happen after signing and do not alter RFC 6979's seed conversion.

The caller supplies the digest; this function does not hash an Ethereum message itself.
If that digest uses Keccak while the nonce generator uses HMAC-SHA-256, it is a distinct-hash variant discussed in §3.6, rather than the base same-H procedure checked above.
The caller must still select the message bytes and signing domain.

Python `hashlib` and `hmac` reproduced the published k, the displayed SHA-256 and initialization values, and the stated byte widths and range arithmetic.
No full signature, EthLib binding, Lean build, proof replay, side-channel test, or general rejection suite ran.
The selected EthLib definitions explicitly make no constant-time claim.

[6979]: https://www.rfc-editor.org/rfc/rfc6979.html
[seed]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Rfc6979.lean#L45
[next]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Rfc6979.lean#L60
[sign]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Sign.lean#L73
