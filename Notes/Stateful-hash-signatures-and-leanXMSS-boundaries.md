---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C12
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
leanxmss_source_commit: "4af2cdddd7cd41b9df2cc7e123e314a91b0b4462"
comparison_source: "RFC 8391, May 2018"
tags:
  - ethlib
  - external-sources
  - hash-signatures
  - stateful-signatures
---

# Stateful hash signatures and leanXMSS boundaries

A Merkle authentication path connects a one-time public key to a root; it does not make that key safe to reuse.
The pinned leanXMSS design separates a pure signing operation from the caller's obligation to allocate leaf indices without reuse.
This note explains that boundary with exact codeword positions and the full index range, without constructing a cryptographic forgery.

## Sources and comparison limits

The scheme source is the [leanXMSS specification draft][design] at leanVM commit `4af2cdddd7cd41b9df2cc7e123e314a91b0b4462`.
The complete self-contained `main.tex` and its referenced `refs.bib` were retrieved at that pin.
Reading covered Overview, Hashing and parameters, WOTS+C, Trees, The scheme, and the Security statement, Using a key and Quantum security paragraphs.
The tightness calculations, threshold-signing sketch and underlying security proofs were not audited.

[RFC 8391][rfc] is an Informational RFC from May 2018, used here for comparison, not authority for the leanXMSS variant.
Selected reading covered §1.1, WOTS+ chains/signing/verification in §§3.1.2–3.1.6, and XMSS state/signing/path verification in §§4.1.3 and 4.1.9–4.1.10.
The RFC's parameter sets, message hashing, addressing, WOTS+ checksum and encodings must not be silently substituted for the design's choices.

## What the chains and authentication paths do

In RFC WOTS+, a signature reveals one value from each chain according to message digits plus a checksum.
Verification completes the remaining chain steps, reconstructs the one-time public key and, in XMSS, authenticates its tree leaf to the public root.
RFC §3.1.3 requires a WOTS+ private key to sign at most one message.
Section 4.1.9 additionally says an implementation MUST NOT output a signature before updating the private-key state.
Returning a mathematically valid signature and enforcing that ordering are different obligations.

The leanXMSS draft instead uses WOTS+C with 58 chains, each having positions 0 through 3.
A 16-byte digest is read as 64 little-endian base-4 digits; its first 58 must sum to 106 and its final six must be zero.
This target-sum condition replaces the Winternitz checksum.
Signing searches deterministic randomizers for the first admissible digest, with a bound of `2^32` trials per one-time signature; failure remains an explicit possibility.

A disclosed chain value at position k permits forward computation, using the correct public addresses, to later positions.
The target sum makes two different admissible vectors incomparable: each needs at least one position earlier than the other exposes.
That is a restriction on deriving a second codeword from one disclosure, not permission to disclose two codewords under the same one-time key.

## Complete chain-position reuse example

Use the actual digit count and target, with positions numbered from zero.
Define these constructed vectors:

```text
x = [2 repeated 48 times, 1 repeated 10 times]
y = x, except y[0] = 3 and y[48] = 0
m = componentwise_min(x,y)
z = m, except z[1] = 3
```

Their differing entries and digit sums are:

| Vector | Digit 0 | Digit 1 | Digit 48 | Sum |
| --- | --- | --- | --- | --- |
| x | 2 | 2 | 1 | 106. |
| y | 3 | 2 | 0 | 106. |
| m | 2 | 2 | 0 | 105. |
| z | 2 | 3 | 0 | 106. |

Append six zero digits to each 58-digit vector and encode the resulting integer in 16 little-endian bytes:

```text
x: aaaaaaaaaaaaaaaaaaaaaaaa55550500
y: abaaaaaaaaaaaaaaaaaaaaaa54550500
m: aaaaaaaaaaaaaaaaaaaaaaaa54550500
z: aeaaaaaaaaaaaaaaaaaaaaaa54550500
```

The x, y and z digests satisfy the draft's codeword condition; m does not.
From x alone, reaching z would require moving chain 48 backwards from 1 to 0.
From y alone, it would require moving chain 0 backwards from 3 to 2.
If two signatures at the same one-time key instead disclose x and y, the available value on every chain includes its position in m.
One forward step on chain 1 then supplies all positions of z.
Authentication-path reuse does not restore the lost one-time restriction.

This is a conditional exposure calculation.
No actual chain values, signed messages, message/randomizer pairs producing these digests, authentication paths or valid signatures are supplied.
It demonstrates why reuse changes the chain argument; it is not a forgery vector, attack-cost estimate or security claim about a small hash function.

## Two layers give a finite index space

The draft's two Merkle layers each have height 16.
A global leaf index `idx < 2^32` selects bottom tree `b = floor(idx/65536)` and local leaf `idx mod 65536`.
The bottom one-time key signs the 32-byte message, while top key b signs that bottom tree's fixed 16-byte root.
Across different leaves of one bottom tree, the top key therefore signs the same root, not a succession of different messages.

| Global idx | Bottom tree b | Local leaf | Four index bytes, little-endian |
| --- | --- | --- | --- |
| 0 | 0 | 0 | 00000000. |
| 65535 | 0 | 65535 | ffff0000. |
| 65536 | 1 | 0 | 00000100. |
| 4294967295 | 65535 | 65535 | ffffffff. |

The next index, 4294967296, is exhaustion rather than a valid wrap to zero.
An allocator's next-index state must represent exhaustion separately from an ordinary four-byte leaf index.
For example, releasing a signature at 65535 while restoring allocator state to 65535 permits another call at an already allocated key.
A crash/recovery or replica design must preserve allocation across that boundary; the leaf arithmetic alone supplies no such persistence.
This is an operational consequence of the no-reuse rule, not a specified storage implementation.

One layer carries a 16-byte randomizer, 58 chain values and 16 siblings, each 16 bytes: `16*(1+58+16) = 1200` bytes.
The whole signature is `4+1200+1200 = 2404` bytes; its 32-byte public key contains a 16-byte root and a 16-byte public parameter.
These are design-specific layouts, not RFC XMSS serialization identifiers.

## Signing state and security scope

The draft's `Sig(sk,idx,m)` is deterministic and does not itself allocate idx.
Its stated classical random-oracle experiment allows each leaf index at most once and, for `q >= 1`, bounds success by `q/2^127`.
Here q counts every hash call in the experiment, including key generation, signing, adversarial queries and final verification, with repeated inputs counted.
That bound is not a claim of the same security for a fixed Keccak implementation or for an attacker with quantum queries.
The draft explicitly calls its quantum-security expectation conjectural.

Signing the same message again at the same index yields the same signature under the specified deterministic operation.
Nevertheless, the draft's security game excludes repeated leaf-index requests, so determinism does not extend that theorem's permitted experiment.
A verifier accepting a signature likewise does not establish that the signer preserved its allocation state.

At the recorded EthLib commit, [`signWith`][lean-sign] takes the seed, index and message, with reuse prevention assigned to the caller in its contract.
[`codeword?`][lean-code] checks the digit condition; the selected [`codeword_incomparable` theorem][lean-incomparable] assumes successful decoding of both vectors and that the vectors differ.
[`verifyWith`][lean-verify] recovers a root through each layer and compares the top root with the public key.
These selected definitions and the incomparability proof were read; no general unforgeability-proof audit or execution claim follows.

Python checked all vectors, digest encodings, sums, forward-position relations, index boundaries and serialized-size arithmetic.
No Keccak chain, key generation, randomizer search, signing/verification, persistent allocator, EthLib binding, Lean build or proof replay ran.
Applying the signature scheme requires a chosen scheme revision, a non-reusing state contract and explicit hash/security assumptions.

[design]: https://github.com/leanEthereum/leanVM/blob/4af2cdddd7cd41b9df2cc7e123e314a91b0b4462/doc/lean_xmss/main.tex
[rfc]: https://www.rfc-editor.org/rfc/rfc8391.html
[lean-sign]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Xmss/Sign.lean#L81-L87
[lean-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Xmss/Encoding.lean#L51-L60
[lean-incomparable]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Xmss/Encoding.lean#L128-L134
[lean-verify]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Xmss/Verify.lean#L106-L119
