---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C01
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
fips202_version: "August 2015"
fips202_sha256: "1592607831ff0908cc590632ce371c6c95e94025bb1a0c8ae90a4d0ec1ed025e"
keccak_reference_version: "3.0, 2011-01-14"
keccak_reference_sha256: "834e74ed4b917b43a5f9cee932166161e09246a0cb90544539a0c81a04d748a5"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - keccak
  - sha3
---

# Keccak, SHA-3 padding, and domains

Ethereum's Keccak-256 and standardized SHA3-256 use the same permutation, rate, capacity, and output length.
They give the sponge different input bits before absorption.
A hash function chosen only by its 256-bit output width can therefore produce the wrong Ethereum hash.

## Sources and scope

| Primary source | Version and selected reading |
| --- | --- |
| [FIPS 202][fips] | August 2015: §3.4, sponge Algorithm 8 in §4, padding Algorithm 9 in §5.1, §5.2, §6.1, and Appendix B.1/B.2. |
| [The Keccak reference][reference] | Version 3.0, 14 January 2011: §§1.1.1–1.4, including multi-rate padding and sponge Algorithm 1. |
| [Ethereum Yellow Paper][yp] | Commit `efc5f9a1…`, Shanghai text: the Conventions paragraph defining `KEC` and `KEC512`. |

The original reference defines Keccak's construction; FIPS 202 defines the standardized SHA-3 functions and its bit-to-hexadecimal convention.
The pinned Yellow Paper explicitly selects the contest-entry version of Keccak-256 rather than final SHA-3 for `KEC`.
That Ethereum choice comes from its primary text, not from guessing what an API named `sha3` computes.
This note covers byte-aligned inputs and one 256-bit output; other capacities, extendable-output functions, and bit-oriented API contracts need separate treatment.

## One permutation, two sponge inputs

The permutation has a 1600-bit state and 24 rounds.
FIPS 202 §3.4 identifies `Keccak-f[1600]` with `Keccak-p[1600,24]`.
For these two hash functions, capacity `c = 512` bits leaves rate `r = 1088` bits, or 136 bytes.
The digest has 256 bits, or 32 bytes.
Here `M` is the message bit string:

```text
Ethereum Keccak-256(M) = Keccak[512](M,       256)
SHA3-256(M)            = Keccak[512](M || 01, 256)

Keccak[512](N,d) = SPONGE[Keccak-f[1600], pad10*1, 1088](N,d)
```

The `01` in this formula is a two-bit string: first a zero bit, then a one bit.
It is not the hexadecimal byte `0x01`.
FIPS 202 §6.1 adds that suffix for domain separation before applying the same `pad10*1` rule.
Original Keccak receives `M` directly; it does not add SHA-3's two-bit suffix.

Algorithm 8 starts with an all-zero state.
It splits the padded input into rate-sized blocks, XORs each block into the rate part, and applies the permutation after each absorption.
The capacity part receives zeros in that XOR, but remains part of the permuted state.
The first 32 output bytes fit within the first 136-byte output block, so this output length needs no additional squeezing permutation.

## Convert suffix and padding bits to bytes

For an input of `m` bits to Keccak, Algorithm 9 appends:

```text
1 || zero_bits(j) || 1
j = (-m - 2) mod 1088
```

For SHA3-256, `m` includes the two suffix bits already appended to the original message.
The first and last `1` above belong to padding; neither is SHA-3's domain suffix.
At least two padding bits are appended, including when the original message fills a rate block.

Appendix B.1 places successive bits of a byte at bit positions 0 through 7, from least significant to most significant.
Thus raw Keccak's first padding bit sets bit 0: `0x01`.
For SHA-3, suffix bits `0,1` followed by the first padding bit `1` set positions 1 and 2: `0x06`.
The final padding bit sets bit 7 of the last byte: `0x80`.
If both ends occupy one byte, combine them by OR, giving `0x81` or `0x86`.
These are byte representations of suffix-plus-padding, not replacement permutations.

For a byte-aligned message of length `L`, both constructions append `q = 136 - (L mod 136)` bytes.
FIPS 202 Appendix B.2 explicitly gives SHA-3's one-byte `86`, two-byte `06 80`, and longer `06 … 80` cases.
Raw Keccak's corresponding bytes follow from the original reference's padding definition and the byte conversion above.
The displayed message bytes remain in their original order; least-significant-bit-first conversion does not mean reversing the byte string.

## Empty input and the bytes of `abc`

The empty message contains zero bytes.
The string `abc` here means its three ASCII/UTF-8 bytes `61 62 63`, without a terminator or newline.
Each case below specifies every byte of its complete 136-byte absorption block:

| Message and function | Complete block, with explicit repetition counts |
| --- | --- |
| Empty, Keccak-256 | `01` followed by 134 zero bytes, then `80`. |
| Empty, SHA3-256 | `06` followed by 134 zero bytes, then `80`. |
| `abc`, Keccak-256 | `61 62 63 01` followed by 131 zero bytes, then `80`. |
| `abc`, SHA3-256 | `61 62 63 06` followed by 131 zero bytes, then `80`. |

For `abc`, indices 0–2 contain the message, index 3 contains `01` or `06`, indices 4–134 are zero, and index 135 is `80`.
For empty input, index 0 is `01` or `06`, indices 1–134 are zero, and index 135 is `80`.
One permutation follows absorption of each block.

The outputs below were independently calculated for these exact messages, rather than attributed as published vectors to either PDF:

| Input bytes | Keccak-256 | SHA3-256 |
| --- | --- | --- |
| Empty | `c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470` | `a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a` |
| `616263` | `4e03657aea45a94fc7d47ba826c8d667c0d1e6e33a64a036ec44f58fa12d6c45` | `3a985da74fe225b2045c172d6bd390bd855f086e3e9d525b46bfe24511431532` |

## The 135/136-byte boundary

For 135 message bytes, only one byte remains in the block.
Raw Keccak appends `81`; SHA3-256 appends `86`.
Appending separate `01` and `80` bytes would exceed that block and implement the wrong padding.

For 136 message bytes, the message occupies an entire block, and padding requires another complete block.
That second block is the empty-input layout above, with `01` or `06` first and `80` last.
The padded input has 272 bytes, and absorption applies the permutation twice.
No padding-free exception exists for an exact block multiple.

## Read the EthLib boundary precisely

At the pinned EthLib commit, [`keccakF`][permutation] applies the 24 rounds.
[`paddingByte` and `padding`][padding] use `01` and the final `80`; they implement the raw-Keccak byte case described above.
[`laneAt` and `bytesOfLane`][lanes] read and emit little-endian lane bytes.
[`spongeAbsorb` and `keccak256`][entry] connect padding, block absorption, and the 32-byte result.
These are observations from the definitions, not a replay of a Lean proof or a claim about every binding that exposes hashing.

[ABI selectors and event topics](ABI-encoding-and-event-topics.md) and [ENS namehash](ENS-namehash-and-resolver-boundaries.md) depend on the named hash function and exact preimage bytes together.
Replacing a primitive on the basis of a similar name changes those derived identifiers.

## Checks and limits

A standalone Python calculation compared bit-oriented suffix-plus-`pad10*1` construction with these byte layouts for empty input, `abc`, 135 zero bytes, and 136 zero bytes.
All eight comparisons agreed, including padded lengths 136, 136, 136, and 272 bytes in each domain.
Python's `hashlib.sha3_256` also produced the displayed SHA3-256 outputs.
An independent PyCryptodome 3.23.0 Keccak calculation and standard-library SHA3 calculation reproduced both columns, with separate checks of the byte-padding boundaries.

The selected source sections and pinned EthLib definitions were read; the complete cryptanalysis in the original reference was not reviewed.
No Lean build, binding execution, exhaustive bit-length test, or security reduction ran for this note.
Different displayed digests establish these cases, not a proof that cross-function collisions cannot occur or a proof of collision resistance.

[fips]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf
[reference]: https://keccak.team/files/Keccak-reference-3.0.pdf
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex#L219-L239
[permutation]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Keccak.lean#L275-L288
[padding]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Keccak.lean#L324-L356
[lanes]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Keccak.lean#L367-L381
[entry]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Keccak.lean#L447-L471
