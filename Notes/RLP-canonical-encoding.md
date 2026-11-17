---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: S01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - serialization
---

# RLP: canonical bytes and typed values

Recursive Length Prefix (RLP) encodes byte strings and lists of items.
Generic items, integer fields, and typed transaction envelopes have different acceptance conditions.

## Sources examined

| Source | Version and passages read | Role |
| --- | --- | --- |
| [Ethereum Yellow Paper, Paper.tex][yp] | Commit `efc5f9a1f356cba376c978eedb63cb0363c2aa85`, Shanghai text, Appendix B: item types, `R_b`, `R_l`, `s`, `BE`, and scalar interpretation. | Encoding rules and scalar requirements. |
| [Alessandro Coglio, Ethereum's Recursive Length Prefix in ACL2][coglio] | EPTCS 327 (2020), arXiv `2009.13769v1`, sections 4.2, 4.2.1 and 4.3. | Research explanation of parsing, canonical rejection, and the two inverse properties; not authority for an Ethereum rule. |
| [EIP-2718: Typed Transaction Envelope][envelope] | EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`, Transactions, Receipts, and the opaque-payload rationale. | Limits on applying RLP to a typed envelope. |

## Bytes, lists, and integers

The Yellow Paper distinguishes byte-string leaves from lists, including their empty forms.
An unsigned integer first becomes its shortest big-endian byte string.
Zero therefore uses no digits.

| Intended value | Item supplied to RLP | Encoded bytes |
| --- | --- | --- |
| Empty byte string | Empty leaf | `80` |
| Empty list | Empty list | `c0` |
| Integer 0 | Empty leaf | `80` |
| One zero byte | Leaf `00` | `00` |
| Integer 128 | Leaf `80` | `81 80` |

The generic item `00` is valid RLP but fails the integer-zero rule.
Likewise, `82 00 01` encodes bytes, but not canonical integer 1, whose encoding is `01`.
Leading-zero rejection depends on scalar interpretation.

## Prefixes and length boundaries

A one-byte string below `0x80` encodes as itself.
Otherwise, let `n` count payload bytes and `k` count its minimal big-endian length bytes.
A list's payload concatenates its children's complete encodings.

| Item and payload length | Prefix before the payload |
| --- | --- |
| Byte string, `n < 56`, excluding the exception above | `0x80 + n` |
| Byte string, `56 <= n < 2^64` | `0xb7 + k`, then the `k` length bytes |
| List, `n < 56` | `0xc0 + n` |
| List, `56 <= n < 2^64` | `0xf7 + k`, then the `k` length bytes |

At 55 payload bytes the string and list prefixes are `b7` and `f7`.
At 56 bytes, `56 = 0x38`, so the prefixes become `b8 38` and `f8 38`.
The total sizes are 56 and 58 bytes, derived from these rules.

Each byte-string length and each list's encoded payload must be below `2^64`.
The length needs at most eight bytes, keeping the prefix in range.
Application resource limits are separate.

## A complete nested encoding

Encode `["cat", ["dog", []], empty-bytes]`.
Here the quoted words denote the specified ASCII byte strings; RLP itself chooses no text encoding.

1. `cat` has bytes `63 61 74`; its three-byte payload gets prefix `83`.
2. `dog` similarly becomes `83 64 6f 67`; the empty list becomes `c0`.
3. The inner list's payload is those four bytes followed by `c0`: five bytes, so its prefix is `c5`.
4. The last outer item, the empty byte string, becomes `80`.
5. The outer payload has `4 + 6 + 1 = 11` bytes, so its prefix is `0xc0 + 11 = 0xcb`.

The full twelve-byte result is:

```text
cb 83 63 61 74 c5 83 64 6f 67 c0 80
```

Reading back, `cb` bounds eleven bytes; `c5` bounds the five bytes of `dog`'s encoding followed by `c0`.
Each `83` consumes three data bytes, and `80` consumes none.
Both lists end at their declared boundaries.
All bytes and counts above were derived by hand.

## What a strict decoder rejects

Appendix B defines the encoder, not a complete parser algorithm.
Strict whole-input decoding accepts exactly the encoder's outputs.
Coglio explains why a round trip alone is insufficient: a decoder could also accept extra forms.
Successful decoding must re-encode to the complete input.

| Input | Reason for rejection as one complete item |
| --- | --- |
| `81 05` | Byte `05` encodes as itself. |
| `b8 03 64 6f 67` | Length 3 requires short prefix `83`. |
| `b9 00 38` followed by 56 copies of `61` | Leading-zero length; use `b8 38`. |
| `f8 01 80` | One payload byte requires `c1`. |
| `f9 00 38` followed by 56 copies of `80` | Leading-zero list length; use `f8 38`. |
| `83 64 6f` | Missing third payload byte. |
| `80 80` | Unread bytes after one item. |

A stream parser can instead read the first item of `80 80` and return the remainder.
Scalar interpretation adds the field check described above.

## Typed envelopes and EthLib use

EIP-2718 defines typed transactions and receipts as a type byte followed by an opaque payload.
It does not require every future payload to use RLP.
Select the envelope type first, then apply that type's payload specification; do not send every complete typed transaction to a whole-item RLP decoder.

For an EthLib contribution, inspect the pinned [encoder][encode] and [decoder][decode], then the caller's field rules.
The statement [`decode_eq_ok_iff`][canonical] relates successful generic decoding to successful encoding at the recorded commit.
Its scope does not establish transaction validity or remove size and scalar conditions.

## Review limits

The listed external passages and relevant EthLib declarations were read.
An independent source review found no material discrepancy in the selected passages or explanation.
Python package `rlp` 4.1.0 reproduced the nested encoding, both 55/56-byte boundary cases, and the empty/scalar byte encodings, and rejected all seven listed malformed whole inputs.
No Lean build, proof replay, binding test, or client comparison was run.
The worked bytes are source-derived calculations checked with a separate codec, not an imported test vector or evidence that EthLib passed them.
Signed numbers, floating-point conventions, and each transaction type's complete schema are outside this note.

[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[coglio]: https://arxiv.org/pdf/2009.13769v1
[envelope]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2718.md
[encode]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Serialization/Rlp/Encode.lean
[decode]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Serialization/Rlp/Decode.lean
[canonical]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Serialization/Proofs/Rlp/Canonical.lean#L382
