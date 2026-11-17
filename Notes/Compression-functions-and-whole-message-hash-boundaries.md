---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
sha_source: "FIPS 180-4, August 2015 final"
blake_source: "RFC 7693, Informational, November 2015"
tags:
  - ethlib
  - external-sources
  - hash-functions
  - precompiles
---

# Compression functions and whole-message hash boundaries

A compression function updates a chaining state from one block and its other inputs.
A whole-message hash additionally specifies initialization, block formation, finalization and output encoding.
Confusing these interfaces changes the function being computed, even when the same round operations appear inside both.
EIP-152's configurable BLAKE2 compression call makes that distinction visible at the byte boundary.

## Exact sources and scope

[FIPS 180-4][fips-record] is the August 2015 final *Secure Hash Standard*, superseding its March 2012 edition.
The publication record includes a March 2023 planning note that NIST decided to revise it.
This note reads that [2015 text][fips], without substituting a later design or treating a revision plan as withdrawal.
Selected reading covered word interpretation in §3.1, SHA-256 functions/constants in §§4.1.2 and 4.2.2, padding/parsing/initialization in §§5.1.1, 5.2.1 and 5.3.3, and §6.2's computation.

[RFC 7693][rfc] is an Informational RFC from November 2015.
Reading covered §§2–3's BLAKE2 parameters, byte order, compression and whole-message procedure, Appendix A's BLAKE2b trace, and the IV constants in Appendix C.
The RFC's omitted personalization/tree-hashing options and full cryptanalytic rationale were not reviewed.
[EIP-152][eip] is Final at the recorded EIPs commit.
Its Specification, gas formula and selected test cases define the separate precompile interface; document status alone does not identify a network's activation rules.

## SHA-256: padding belongs outside compression

SHA-256 uses 64-byte blocks, eight 32-bit state words and 64 rounds per block.
FIPS §6.2.2 expands each block's sixteen big-endian words into a 64-word schedule.
The working variables start from the incoming state, undergo the rounds and are added back to that state modulo `2^32`.
Whole-message hashing starts from §5.3.3's initial state and processes the padded blocks in order.
An arbitrary supplied state and block therefore do not automatically describe the hash of some caller-provided message.

Section 5.1.1 admits message lengths below `2^64` bits.
For a whole-byte message of n bytes, append `80`, then enough zero bytes to reach byte position 56 modulo 64, then the original length `8*n` in eight big-endian bytes.
The appended length counts the original message bits, not its padded size.
An implementation that wraps an out-of-range length is outside that admitted SHA-256 domain.

The standard's `abc` padding example has 24 message bits.
Its complete 64-byte padded block, in four consecutive rows, is:

```text
61626380000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000018
```

Consequently `W_0=0x61626380`, `W_1` through `W_14` are zero, and `W_15=0x00000018` before schedule expansion.
The marker byte already includes the initial one bit and its following seven zero bits.
A separate `hashlib.sha256` computation of the original three bytes returned:

```text
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

This computed digest is distinct evidence from reading the published padding example.
The same padding arithmetic gives one block for 55-byte input and two blocks for 56-byte input.
The [Keccak/SHA3 note](Keccak-SHA3-padding-and-domains.md) treats a different padding and domain-separation construction.

## BLAKE2b: counters and final flags belong inside compression

RFC §3.2's F consumes an eight-word state h, sixteen-word block m, a 128-bit byte counter t and final-block flag f.
BLAKE2b words are 64 bits and byte encodings are little-endian.
The counter's low and high words are XORed into working words 12 and 13.
On the final block, all bits of working word 14 are inverted.
Twelve rounds mix the state and message, after which both halves of the working vector are XORed into h.

Whole-message BLAKE2b in §3.3 also initializes h using the parameter block, zero-pads a partial final block, supplies the correct counters and selects the output length.
For unkeyed hashing, the final counter is the original byte length rather than the padded length.
An unkeyed empty message still processes one all-zero block, with counter zero and f true.
A full final 128-byte block is marked final itself; the zero-padding rule does not require another block.

Digest size is an initialization parameter, not merely a final slice.
For unkeyed 64-byte output, `p_0=0x01010040` and `h_0=IV_0 XOR p_0`.
Requesting 32-byte output instead changes that parameter to `0x01010020` before any compression.
Treating BLAKE2b-256 as the first half of BLAKE2b-512 silently changes the construction.

## Complete EIP-152 `abc` control

EIP-152 calls F at address `0x09` with exactly 213 tightly packed bytes.
The round count is big-endian, while every h, m and counter word is little-endian.
This differs from the standard ABI's padded argument layout.
The following table completely defines its published Test vector 5 input, using half-open byte ranges:

| Field | Range | Encoded bytes |
| --- | --- | --- |
| Rounds | `[0,4)` | `0000000c` |
| h | `[4,68)` | The two consecutive 32-byte rows below. |
| m | `[68,196)` | `616263` followed by exactly 125 zero bytes. |
| Low counter | `[196,204)` | `0300000000000000` |
| High counter | `[204,212)` | `0000000000000000` |
| Final flag | `[212,213)` | `01` |

```text
48c9bdf267e6096a3ba7ca8485ae67bb2bf894fe72f36e3cf1361d5f3af54fa5
d182e6ad7f520e511f6c3e2b8c68059b6bbd41fbabd9831f79217e1319cde05b
```

Here `h_0=0x6a09e667f2bdc948`, `m_0=0x0000000000636261`, the remaining m words are zero, and `(t_0,t_1)=(3,0)`.
The 125 padding bytes do not make t equal 128.
The published 64-byte result matches RFC Appendix A's BLAKE2b-512 result:

```text
ba80a53f981c4d0d6a2797b69f12f6e94c212f14685ac4b74b12bb6fdbffa2d1
7d87c5392aab792dc252d5de4533cc9518d38aa8dbf1925ab92386edd4009923
```

The coincidence with a complete hash depends on this initialization, block, counter, flag and twelve-round choice.
Test vector 6 instead sets f to zero and publishes a different state.
Vectors 4 and 7 use zero and one round respectively, which are accepted compression inputs rather than standard whole-message BLAKE2b evaluations.
The EIP's operation charge is one gas per round, making this twelve-round operation cost 12 before surrounding call costs.
Any length other than 213 or flag byte outside 0 and 1 is malformed.

## Selected Lean boundaries and evidence

At the pinned EthLib commit, SHA-256's [`pad`][sha-pad], [`compress`][sha-compress] and [`hash`][sha-hash] expose these distinct stages for whole-byte messages.
BLAKE2b's [`compress`][blake-compress] accepts a round count, state, block, two counter words and Boolean flag.
The precompile's [`compute`, `output` and `blake2f`][precompile] add decoding, errors and gas handling.
Its chosen check order is length, gas, then flag, which the code distinguishes from EIP-152's unspecified ordering of simultaneous failures.
The complete selected proofs of [`blake2f_invalidInputLength` and `readFlag_invalid`][errors] establish those error outcomes under their stated size or flag inequalities, not hash security.

A private Python transcription of RFC F reproduced published vectors 4–7, and `hashlib.blake2b` independently matched the complete `abc` digest.
Byte calculations checked the 213-byte encoding, SHA padding and word order, counter-dependent separation of `abc` from `abc` followed by zero, and the digest-size distinction.
No EVM/precompile call, EthLib execution, Lean build, proof replay, timing or security experiment ran.
The billions-of-rounds Test vector 8 was not evaluated.

[fips-record]: https://csrc.nist.gov/pubs/fips/180-4/upd1/final
[fips]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
[rfc]: https://www.rfc-editor.org/rfc/rfc7693.html
[eip]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-152.md
[sha-pad]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Sha256.lean#L76-L96
[sha-compress]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Sha256.lean#L151-L162
[sha-hash]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Sha256.lean#L176-L200
[blake-compress]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Blake2b.lean#L125-L148
[precompile]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Precompiles/Blake2f.lean#L81-L122
[errors]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Proofs/Evm/Precompiles/Blake2f.lean#L51-L69
