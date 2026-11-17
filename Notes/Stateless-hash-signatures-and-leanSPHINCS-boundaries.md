---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C13
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
leansphincs_source_commit: "1538d0a97845c3b771088f09561ddf7607b560a1"
comparison_source: "FIPS 205, final, 2024-08-13"
tags:
  - ethlib
  - external-sources
  - hash-signatures
  - stateless-signatures
---

# Stateless hash signatures and leanSPHINCS boundaries

Stateless signing selects a few-time key from a message digest instead of allocating a fresh leaf with a persistent counter.
That change removes an index-allocation obligation, but does not give every hash-signature variant the same parameters, encoding or security budget.
The useful contribution question is which exact scheme a proposed definition or test is meant to describe.

## Sources and their roles

[FIPS 205][fips-record], published and effective 13 August 2024, standardizes SLH-DSA based on SPHINCS+.
The [final standard][fips] is the authority for its approved SLH-DSA parameter sets, not for leanSPHINCS.
Selected reading covered the construction in §3, addressing and conversions in §§4.1–4.4, hypertree structure and verification in §7, FORS in §8, internal signing and verification in §§9.2–9.3, and §11/Table 2.

The separate [leanSPHINCS design][design] is pinned to `1538d0a97845c3b771088f09561ddf7607b560a1`.
Its title is *Exploring leanSphincs*, and its opening warns that it is mostly AI-written.
Reading covered §1, the selected Lifetimes statement, and the FORS comparison in *The few-time signature: a two-level WOTS forest*.
The threshold, hardware-wallet and underlying security-proof material was not audited.
This exploratory document and the observed Lean profile require explicit reconciliation before their claims can be combined.

## What FORS and the hypertree authenticate

In FIPS §8, each FORS signature discloses one secret and its authentication path per tree.
The digest selects the leaves, whose recovered roots are compressed into one candidate FORS public key.
The hypertree then authenticates that key through successive XMSS signatures to `PK.root`.
FORS tolerates limited reuse with degrading security, while its surrounding WOTS+ keys sign the fixed public material assigned to them.
This differs from manually allocating fresh indices in [stateful hash signing](Stateful-hash-signatures-and-leanXMSS-boundaries.md).

FIPS addresses identify the layer, tree, function type and relevant key or node, with 32 bytes and big-endian words.
Changing the address type clears the final 12 bytes before setting that type's fields.
These are hash-input separation rules, not network addresses.
The standard's approved sets use SHA-2 or SHAKE, WOTS+ and equal tree heights within each hypertree.

## Preserve the leanSPHINCS version differences

The design's §1 offers multiple lifetimes, tree shapes, hashes and few-time signatures.
Its layered FORS choice at lifetime `2^30` has 24 trees of height 10, hash values of 16 bytes, and layer heights `10+8+8` from the top.
Its WOTS+C uses 64 chains with positions 0 through 3 and target digit sum 128 instead of the standard WOTS+ checksum.
These choices alone do not identify all hash inputs or wire fields.

At the pinned EthLib commit, the observed profile also selects those numerical parameters, with layers numbered from the bottom as `8,8,10`.
However, selected source passages differ:

| Boundary | Pinned design text | Pinned EthLib definitions |
| --- | --- | --- |
| Address | A 16-byte address after the 16-byte public parameter. | `encodeAdrs` emits a 32-byte address and adds WOTS+C encoding type 7. |
| Digest input | The Hash inputs paragraph says the digest hashes `m || R`. | `hMsg` concatenates `R || PK.seed || PK.root || M || BE32(i)` for counters 0 and 1, then retains 34 bytes. |
| FORS material | The FORS comparison omits root compression and has WOTS+C sign the 48 nodes below the roots directly. | `forsPkFromRoots` compresses 24 roots into one 16-byte digest before the bottom OTS signs it. |

The design's table and the Lean profile both give 7,740 signature bytes for this layered FORS choice.
Matching that length does not resolve the differing inputs or establish interoperability.
The source-selection task is to identify and document the intended profile, not silently repair one description using the other.

## Complete constructed digest and index trace

Use the actual observed Lean profile, with this deliberately constructed 34-byte digest:

```text
000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f2021
```

This is a parser input, not a published test vector or a claimed output of `H_msg` on some message.
The first 30 bytes contain 240 bits, read most significant bit first in consecutive groups of 10.
The 24 FORS leaf indices, in tree order, are:

```text
0,16,128,772,20,96,450,9,40,176,771,270,
60,257,68,531,80,337,389,792,100,417,711,29
```

The remaining bytes split as `tree=1e1f20` and `leaf=21`, with hexadecimal byte notation.
Reading the tree field big-endian gives 1,974,048, whose low 18 bits are 139,040.
The leaf is 33, so `idx = 139040*256+33 = 35594273 = 0x21f2021`, below `2^26`.
Six high bits of the three-byte tree field are ignored.
Replacing byte 30 from `1e` with `fe` therefore changes the digest while preserving every selected leaf and the index.
This demonstrates a projection of the digest, not a collision in a cryptographic hash.

The same index locates the three one-time keys:

| Layer | Tree | Leaf | Index operation |
| --- | --- | --- | --- |
| Bottom | 139040 | 33 | Drop 8 bits for the tree and retain the low 8 for the leaf. |
| Middle | 543 | 32 | Drop 16 bits for the tree and retain bits 8–15 for the leaf. |
| Top | 0 | 543 | The remaining 10 bits select a leaf of the single top tree. |

Nothing in this arithmetic supplies the leaf secrets, path nodes or chain values needed for a signature.

## Fixed length is separate from verification

For this observed profile, each FORS opening contains `16+10*16=176` bytes.
A layer contains its four-byte counter, 64 chain values and its authentication path.
The complete half-open byte ranges are:

| Field | Bytes | Range |
| --- | --- | --- |
| Randomizer | 16 | `[0,16)` |
| 24 FORS openings | 4224 | `[16,4240)` |
| Bottom layer | 1156 | `[4240,5396)` |
| Middle layer | 1156 | `[5396,6552)` |
| Top layer | 1188 | `[6552,7740)` |

The public key is 32 bytes, containing two 16-byte fields.
`decodeSignature` checks exactly 7,740 bytes and splits these fields, while `verifyWith` separately reconstructs the FORS key and the three layer roots.
Verification rejects if a layer's counter fails its target-sum encoding or if the final root differs from `PK.root`.
A decoded byte string is therefore not evidence of a valid signature.

## EthLib entry points and review limits

Selected definitions are [`digestLeaves` through `digestIndex`][digest-code], [`OtsKey.ofIndex`][keys-code], [`encodeAdrs`][address-code], [`forsPkFromRoots`][fors-code], [`decodeSignature`][decode-code] and [`verifyWith`][verify-code].
The complete selected proofs of [`digestIndex_lt` and `digest_fields`][digest-proof] establish the index bound and exact field decomposition for an arbitrary typed 34-byte digest.
They do not establish that an actual hash output is uniform or prove unforgeability.
The selected [`SIGNATURE_LIMIT`][limit-code] is `2^30`, and [`signWith`][sign-code] has no persistent usage counter.
Its caller's budget is distinct from stateful leaf allocation and must not be replaced with FIPS's different parameter-set budgets.

Standalone integer and byte calculations checked the shown indices, ignored-bit control, all-zero/all-one boundaries and field sizes.
No cryptographic signature, key generation, hash-tree evaluation, Lean build or proof replay ran.
The remaining contribution work includes resolving the design/profile differences and selecting source-specific interoperability evidence before making conformance or security claims.

[fips-record]: https://csrc.nist.gov/pubs/fips/205/final
[fips]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.205.pdf
[design]: https://github.com/TomWambsgans/sphincs.art/blob/1538d0a97845c3b771088f09561ddf7607b560a1/leanSPHINCS.tex
[digest-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Digest.lean#L112-L132
[keys-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Types.lean#L129-L140
[address-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Address.lean#L36-L54
[fors-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Fors.lean#L98-L104
[decode-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Serialization.lean#L137-L157
[verify-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Verify.lean#L95-L118
[digest-proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Sphincs/Digest.lean#L39-L220
[limit-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Constants.lean#L144-L152
[sign-code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Sphincs/Sign.lean#L96-L120
