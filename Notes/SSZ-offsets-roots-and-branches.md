---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 2
delivery_batch: "follow-up-1"
catalogue_family: S02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
consensus_specs_commit: "802485ec74a6986542bd53bf153189623aab440c"
ssz_specs_commit: "4c2f08805080c6b4b12ede7bf48ad6a80229f4b1"
tags:
  - ethlib
  - external-sources
  - ssz
  - merkle-proofs
---

# SSZ offsets, roots, and branches

SSZ serialization tells a decoder where a variable field's bytes begin; Merkleization instead commits to its typed value.
An offset, a list's element count, and its declared limit have different roles.
This example follows one container from bytes to a root, then authenticates the chunk containing a list element.

## Sources and roles

| Source | Selected reading and role |
| --- | --- |
| [Historical `ssz/simple-serialize.md`][prose] at `802485ec…` | Constants, fixed/variable sizes, unsigned-integer and composite serialization, deserialization boundaries, and Merkleization. Archived prose explanation. |
| [`ethereum/ssz-specs`][official] at `4c2f0880…` | README and selected `container.py`, `collections.py`, `offsets.py`, `layout.py`, `roots.py`, `mixins.py`, `trees.py`, `paths.py`, `gindex.py`, and `verification.py`. Executable specification. |
| [Historical `ssz/merkle-proofs.md`][historical-proof] at `802485ec…` | Generalized Merkle tree index, SSZ object to index, and single-item proof calculation. Archived addressing and branch explanation. |

The Python project's README explicitly identifies it as the official specification.
EthLib's SSZ README identifies that project as its reference and the consensus-specs text as the prose it replaced.
Here the archived prose explains the rules, while the executable specification supplies exact checks at its pinned revision.
EthLib's definitions below are connections to a peer formal specification, not independent authority for a protocol rule.
This note does not audit every difference between those sources or claim conformance from matching an example.

## A container with one variable field

Use this schema notation, with field order fixed as written:

```text
Example:
    tag: uint16
    values: List[uint16, 4]

value:
    tag = 513
    values = [1, 2, 3]
```

The list permits zero through four elements.
Its element type has fixed width, but the list itself has variable width because its count can change.
SSZ does not include field names or a schema identifier in these serialized bytes; the decoder must know the declaration.
This is an original worked case, not a published upstream fixture.

Unsigned integers serialize little-endian, so `tag = 513 = 0x0201` becomes `01 02`.
The container's fixed section has two tag bytes and a four-byte offset for `values`: six bytes total.
The offset is measured from the start of this container's serialization, not from its own position or the start of a larger enclosing message.
It is therefore `06 00 00 00`.
The three list elements follow as six bytes, with no separate length prefix:

```text
byte positions   0..1   2..5          6..11
contents         tag    list offset   list body
hex              01 02  06 00 00 00   01 00 02 00 03 00

complete: 010206000000010002000300
```

The serialized container has 12 bytes.
Given that complete input scope, the decoder assigns the list the bytes from offset 6 to the end.
Dividing that six-byte scope by the two-byte element width recovers three elements.
This differs from a list of variable-size elements, whose own offset table also identifies its count.

## Malformed bytes and valid extensions

The executable specification's [container decoder][container] requires the first variable offset to equal the end of the fixed section.
Its [list decoder][collections] checks element-size alignment and the recovered element count.
The [offset helpers][offsets] also enforce the composite's exclusive `2^32` byte ceiling.
These are source-derived boundaries for this schema:

| Input change | Consequence |
| --- | --- |
| Give fewer than six total bytes. | The input cannot contain the container's fixed section. |
| Replace offset 6 with 5 or 7. | The first variable field overlaps the fixed section or leaves a gap; neither is an alternative encoding. |
| Append one byte to the valid encoding. | The list scope becomes seven bytes, which is not divisible by two. |
| Supply five complete `uint16` elements after the fixed section. | The count exceeds the declared limit of four. |
| Append two zero bytes to the valid encoding. | This is a valid different value: `values = [1,2,3,0]`. |

“Extra bytes” therefore needs a schema-specific explanation.
Two appended bytes can be another element rather than unused input.
The exclusive composite-size ceiling is an algorithmic boundary here, not a large-input test performed for this note.

## Packing, list length, and the container root

Let `H(left || right)` mean SHA-256 of two 32-byte nodes.
SSZ packs basic list elements together before Merkleizing them; it does not hash each `uint16` into a separate leaf.
The list's maximum packed width is `4 * 2 = 8` bytes, so its declared chunk capacity is `ceil(8 / 32) = 1`.
Its data tree consequently contains one chunk, whose root is that chunk itself.

Define these exact 32-byte words:

```text
T = 01 02             || 30 zero bytes       # tag root
C = 01 00 02 00 03 00 || 26 zero bytes       # packed list data
L = 03                || 31 zero bytes       # uint256 little-endian count
```

The list mixes in its actual **element count**, 3, rather than its six serialized bytes or its limit, 4.
The container then Merkleizes its two field roots in declaration order:

```text
list_root = H(C || L)
          = 8c0cd61afd522875dc93f9cfd0d3514d3bc6f60271d00ec8fb0c10881159b3b9

container_root = H(T || list_root)
               = df649acaf93641c54dec8860200af06809a66c46fcfa9d585bd58d800892afc0
```

Offsets are not leaves in this tree, and this root calculation is not SHA-256 of the 12 serialized bytes.
The type determines packing, field positions, and tree capacity.
The limit is not serialized or directly mixed in: a limit of eight `uint16` elements would also occupy one chunk and give this value the same root.
The root alone cannot recover that declared limit.
Appending a zero fourth element leaves `C` unchanged but changes `L` to the encoding of 4.
That illustrates why zero padding cannot stand in for the list's length commitment.

## One branch reaches a packed chunk

A generalized index numbers the root as 1 and a node's children as `2k` and `2k+1`.
For this object, the tree positions are:

```text
                  1: container_root
                 /                 \
             2: T                3: list_root
                                 /          \
                              6: C         7: L
```

The target chunk has generalized index 6, binary `110`: discard the leading root bit, then follow right and left.
Its branch, ordered from leaf upward, contains `[L, T]`, at sibling indices `[7, 2]`.
Rebuild it as `H(C || L)`, then `H(T || previous)`.
The [executable verifier][verification] uses the index bits from the bottom upward to choose each sibling's side and requires the branch depth and node widths to match.

`values[1]` is the two-byte range `[2,4)` inside `C`, decoding to 2.
It does not have its own hashed leaf: the proof reaches the whole packed chunk.
The [path helpers][paths] explicitly retain that distinction between a chunk address and an element's byte range.
Interpreting the element also needs the schema, the authenticated length 3, and the index-bound check `1 < 3`.
Branch equality against a supplied root alone does not validate the entire object's schema or establish that the root came from a trusted protocol context.

## EthLib connection and checked scope

At the recorded EthLib commit, [`assemble` and `serialize`][lean-serialize] build fixed sections and variable bodies.
[`merkleLayout`][lean-layout] specifies basic-element packing, capacities, field-root leaves, and list count words; [`hashTreeRoot`][lean-root] evaluates those layouts with SHA-256.
[`calculateMerkleRoot` and `verifyMerkleProof`][lean-verify] rebuild a branch while checking its index, depth, and node widths.
These are selected definitions read, not a claim about all SSZ proofs or implementation refinements.

An isolated Python calculation reproduced the serialization, both displayed roots, and the branch at 6.
Reversing the sibling order produced a different root.
The source passages, explanation, and cited definitions received a separate review.
An independent standard-library calculation reproduced the bytes, roots, and branch, with controls for changed sibling order, path, data, and length.
It also checked the displayed small malformed-input boundaries in a schema-specific decoder.
Neither calculation executed the official SSZ package or EthLib.
No Lean build, proof replay, or full malformed-input suite ran.
Remaining scope includes nested variable-size elements, bitlist delimiters, progressive types, unions, multiproofs, and broader source-to-code correspondence.

[prose]: https://github.com/ethereum/consensus-specs/blob/802485ec74a6986542bd53bf153189623aab440c/ssz/simple-serialize.md
[official]: https://github.com/ethereum/ssz-specs/tree/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1
[historical-proof]: https://github.com/ethereum/consensus-specs/blob/802485ec74a6986542bd53bf153189623aab440c/ssz/merkle-proofs.md
[container]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/container.py#L192-L254
[collections]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/collections.py#L552-L608
[offsets]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/offsets.py
[verification]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/verification.py#L18-L53
[paths]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/paths.py#L176-L207
[lean-serialize]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Serialize.lean#L83-L172
[lean-layout]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Layout.lean#L121-L167
[lean-root]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Root.lean#L28-L86
[lean-verify]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Merkle/Verify.lean#L34-L65
