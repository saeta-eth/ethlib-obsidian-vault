---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 2
delivery_batch: "follow-up-8"
catalogue_family: C06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
sec1_version: "2.0, May 21, 2009"
sec1_sha256: "54687189af3de645756a30a801b596fd14c6566992c13bfd58eede399f324a89"
sec2_version: "2.0, January 27, 2010; running headers say Draft"
sec2_sha256: "87b8f3703364ed5b21ba8582e411cc0cbf477bcaa3f4f45e0d6580d1c00d9952"
tags:
  - ethlib
  - external-sources
  - secp256k1
  - ecdsa
---

# Curve validation, ECDSA, and recovery boundaries

Decoding a curve point, validating a public key, verifying a signature, and recovering a candidate key answer different questions.
Ethereum adds transaction rules that its ECRECOVER precompile does not share.
Preserve those boundaries when a contribution connects bytes, curve arithmetic, and transaction admission.

## Sources and reading scope

The primary curve texts are [Standards for Efficient Cryptography 1 (SEC 1), version 2.0][sec1], May 21, 2009, and [Standards for Efficient Cryptography 2 (SEC 2), version 2.0][sec2], January 27, 2010.
The recorded PDF hashes identify the bytes read because the versioned URLs are not content-addressed.
SEC 2's cover says version 2.0, while its running headers retain “Draft”; this note preserves that identity without assigning it an EIP status.
Reading covered SEC 1 §2.2.1, §§2.3.3–2.3.8, §§3.2.1–3.2.3, and §§4.1.1–4.1.6, plus SEC 2 §2.4.1.

[EIP-2][2] and [EIP-155][155] both have status Final at the recorded EIPs commit.
Reading covered their signature rules, relevant rationale, and EIP-155's signing/recovery specification.
They supply historical fork conditions, rather than a complete account of every transaction format.
The [typed-transaction note](Typed-transaction-envelopes-and-signing.md) covers the separate envelope and signing-preimage question.

## Coordinates and scalars use different bounds

SEC 2 specifies secp256k1 as `y^2 = x^3 + 7` over the prime field with modulus p:

```text
p = fffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f
n = fffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141
h = 1
```

Coordinates are integers in `[0,p-1]`; n is the generator's order, and h is the cofactor.
Private keys and ECDSA signature integers have bounds `[1,n-1]`.
Both moduli occupy 32 bytes, but `n < p`.
An integer can therefore fit the coordinate range while exceeding a signature-scalar bound.
Parsing a coordinate must reject an out-of-range integer rather than reduce it modulo p and accept different bytes as the same encoding. [SEC 1 §2.3.6][sec1]

SEC 2 publishes these complete generator encodings, written here as continuous hex:

```text
compressed, 33 bytes:
0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798

uncompressed, 65 bytes:
0479be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8
```

The compressed prefix `02` selects even y; `03` would select the opposite point's odd y.
The uncompressed prefix `04` precedes x and y, each as 32 big-endian bytes.
Compression still requires recovering a square root of the curve equation and rejecting an x with no such root.

## A point is not automatically a usable public key

SEC 1 §2.3.4 decodes the single byte `00` as the point at infinity O.
Its public-key validation primitive in §3.2.2.1 rejects O, checks coordinate ranges and the curve equation, and checks `n*Q = O`.
The primitive assumes valid domain parameters; this note does not validate arbitrary caller-supplied curves.

Partial public-key validation omits the subgroup test.
For valid secp256k1 parameters, cofactor 1 makes `n*Q = O` follow for points on the curve, as SEC 1 explains after the full validation primitive.
That conclusion cannot be transferred to an arbitrary curve with a different cofactor.
Neither arithmetic validity nor successful decoding identifies the person who controls a key.
SEC 1's ECDSA key-deployment procedure says the verifier should obtain the public key authentically.
It separately requires assurance that the public key is valid.

## Verification takes a key; recovery needs a key-selection policy

Given a validated key Q and digest-derived integer z, ECDSA verification first checks `1 <= r,s < n`.
It computes `R = (z/s)*G + (r/s)*Q`, with scalar arithmetic modulo n, rejects infinity, and compares `x_R mod n` with r.
SEC 1 derives z by retaining at most the order's bit length from the hash; for secp256k1 and a 32-byte digest, all 256 bits remain before scalar arithmetic.
The algorithm does not define which Ethereum message domain or transaction bytes were hashed.

Recovery instead considers possible R and solves `Q = r^(-1)*(s*R-z*G)`.
SEC 1 §4.1.6 considers `x_R = r+j*n` and both signs of R, then requires authenticating the candidate Q.
A parity bit alone does not specify every possible x candidate in generic recovery.
The pinned [`Ecdsa.recover`][recover] interface is narrower: it accepts parity 0 or 1 and uses x=r, as its documented Ethereum profile states.
Returning a candidate key is not an application authorization decision.

## A constructed signature pair with complete scalar bytes

Choose public teaching values `d=1`, `k=1`, and a 32-byte digest whose integer is `z=1`.
Then `Q=d*G=G`, `R=k*G=G`, `r=x_G`, and `s=k^(-1)*(z+r*d)=r+1 mod n`:

```text
digest = 0000000000000000000000000000000000000000000000000000000000000001
r      = 79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
s      = 79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81799
n-s    = 8641998106234453aa5f9d6a3178f4f7b812e00b817a776265dfdd31b93e29a8
```

This is an author-constructed arithmetic control, not a published signature vector or a claimed hash of a particular message.
It is deliberately insecure: knowing k exposes `d=(s*k-z)/r mod n`, which gives 1 here.
It provides no key-generation or nonce-selection advice.

For `(r,s)`, the verifier's scalar sum is `(1+r)/s = 1 mod n`, so it rebuilds G.
For `(r,n-s)`, it rebuilds `-G`, whose x is unchanged, so verification under the same Q also succeeds.
The first s is low and the second is high.
To recover the same Q after changing s, change R's parity from even to odd too: `(-s)*(-R)=s*R`.
Keeping the original parity recovers a different candidate in this example.

## Ethereum normalizes a pair, not every signature

From its Homestead condition, EIP-2 invalidates transaction signatures with `s > n/2`, while explicitly leaving ECRECOVER's high-s acceptance unchanged.
Low-s selects one member of the sign-related pair; it does not make signatures from different signing nonces identical.
EIP-2's rationale changes legacy v between 27 and 28 together with s.

EIP-155's chain-protected legacy form instead uses `v = 35 + 2*chainId + parity` and a chain-dependent signing hash.
Its signing change is SHOULD under its activation conditions, and the resulting v formula is MUST if that form is chosen.
It explicitly retains legacy 27/28 signatures.
Do not pass an encoded chain-protected transaction v directly as a two-valued recovery parity.

## Code observations and checks

At the pinned EthLib commit, [`Point.decode`][decode] rejects infinity, bad sizes/prefixes, oversized coordinates, and off-curve uncompressed points.
[`Ecdsa.verify`][verify] checks key and scalar validity; [`isLowS`][verify] is a separate bound predicate and alone does not reject zero.
[`validS` and `recoverSender`][sender] impose transaction bounds by fork, while [`ECRECOVER.output`][precompile] accepts v=27/28 and calls recovery without imposing low-s.
These are observations of the selected definitions, not proof-coverage findings.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Standalone Python integer and affine-group calculations matched the published generator bytes, its curve equation and `n*G=O`, both signature equations, low/high classification, and the two parity-adjusted recoveries.
They also confirmed the different-key result when parity is unchanged.
No Ethereum transaction, precompile, EthLib binding, Lean build, or proof replay ran.
Nonce generation, constant-time behaviour, general malformed-input coverage, and an unforgeability argument remain outside this reading.

[sec1]: https://www.secg.org/sec1-v2.pdf
[sec2]: https://www.secg.org/sec2-v2.pdf
[2]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2.md
[155]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-155.md#specification
[decode]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Encoding.lean#L71
[verify]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Ecdsa.lean#L54
[recover]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Recovery.lean#L45
[sender]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Transactions/Signature.lean#L173
[precompile]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Precompiles/Ecrecover.lean#L46
