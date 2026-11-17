---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: S03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
ssz_specs_commit: "4c2f08805080c6b4b12ede7bf48ad6a80229f4b1"
tags:
  - ethlib
  - external-sources
  - ssz
  - progressive-containers
---

# Progressive SSZ presence and compatible schemas

An absent progressive-container field and a present zero-valued field can contribute the same zero leaf.
Their roots still commit to different field layouts.
Stable field addresses let a verifier retain its addressing logic across compatible schemas; they do not make an old Merkle branch verify against a changed root.

## Sources and roles

| Source | Status or version; selected reading |
| --- | --- |
| [EIP-7495, SSZ ProgressiveContainer][7495] | Review at the EIPs pin: complete type, serialization, Merkleization, rationale, and security text. |
| [EIP-7916, SSZ ProgressiveList][7916] | Review at that pin: complete progressive-tree algorithm, type/serialization rules, and resource-limit discussion. |
| [EIP-8016, SSZ CompatibleUnion][8016] | Draft at that pin: complete compatibility, selector, serialization, Merkleization, and rationale text. |
| [Executable SSZ specification][ssz] | Commit `4c2f0880…`: README and selected `container.py`, `layout.py`, `roots.py`, `trees.py`, `mixins.py`, `collections.py`, `union.py`, `paths.py`, and `gindex.py`. |

All EIP links use `8194528a…`.
Status establishes neither adoption nor fork activation.
The Python project's README identifies it as the official specification; its selected definitions give executable checks beside the EIP prose.
EthLib is a peer specification, not independent normative authority for a protocol rule.
A selector-width ambiguity between the prose and executable source is retained below rather than silently resolved.
The [earlier SSZ note](SSZ-offsets-roots-and-branches.md) covers bounded lists and ordinary container offsets.

## Two schemas, one zero leaf

Use EIP-style schema notation, with active positions numbered from zero:

```text
A: ProgressiveContainer(active_fields=[1,0,1])
    tag:    uint8 = 7       # position 0
    marker: uint8 = 9       # position 2

B: ProgressiveContainer(active_fields=[1,1,1])
    tag:     uint8 = 7      # position 0
    counter: uint8 = 0      # position 1
    marker:  uint8 = 9      # position 2
```

Each schema assigns its declared fields, in order, to the set bits.
The layout belongs to the type; it is not a per-value optional-field switch.
EIP-7495 explicitly defers a general `Optional` type.
Legal declarations have at least one field, at most 256 layout positions, a final set bit, and exactly one set bit per field.

Serialization follows ordinary `Container` rules and emits only declared fields:

```text
A bytes: 07 09       (2 bytes)
B bytes: 07 00 09    (3 bytes)
```

Neither encoding contains the active-fields word or a schema tag.
A decoder therefore needs the schema: the executable container decoder rejects A's two-byte scope when B requires three.
Compatible Merkleization does not promise interchangeable wire encodings or automatic conversion between versions.
These bytes are an original worked example, not a published fixture.

Each container field contributes its own 32-byte root; these three `uint8` positions are not packed together.
Write `Z` for 32 zero bytes, `C7 = 07 || zero_bytes(31)`, and `C9 = 09 || zero_bytes(31)`.
A's inactive position contributes `Z`; B's present `uint8(0)` also roots to `Z`.
Both content trees therefore receive the same ordered chunks `[C7,Z,C9]`.

## Commit to the layout separately

Let `H(x,y) = SHA256(x || y)` for two 32-byte inputs.
EIP-7916's progressive tree starts with a one-chunk subtree, then a four-chunk subtree, then sixteen, continuing by factors of four.
An empty continuation is the zero node itself, not a larger all-zero subtree root.
For our three positions:

```text
M4 = H(H(Z,C9), H(Z,Z))  # four-chunk subtree [Z,C9,Z,Z]
P  = H(C7, H(M4,Z))      # shared progressive contents root

W05 = 05 || zero_bytes(31)  # layout [1,0,1]
W07 = 07 || zero_bytes(31)  # layout [1,1,1]
root_A = H(P,W05)
root_B = H(P,W07)
```

EIP-7495 mixes the packed active bits into the right child, with the contents on the left.
Lowest layout positions occupy the lowest bits of the word.
The distinct hash inputs here are exactly 64 bytes: the same `P`, followed by `W05` or `W07`.
Without that layout commitment, this particular absent-versus-zero pair would have identical content roots.

An isolated SHA-256 calculation gives:

```text
P      = 9f18c1f8668a95140d440f6cecf0435a03d7d901b165cce69c4ac3babdefd764
root_A = aedfb2d01919beeb8a2cf250f421d1aaefcffc16cb74d54056ce9e98099f0616
root_B = e9356938628239eab8fb4d67d42b21007bcb917ab14a84e6e474e374bd8a516b
```

These computed outputs distinguish this pair, without claiming a general collision-resistance proof.

## Stable addresses still need current siblings

The executable [`progressive_chunk_gindex`][gindex] includes the outer mix-in edge.
Positions 0, 1, and 2 map to generalized indices 4, 40, and 41 respectively.
The active-fields word is the right child at index 3.
Thus `marker` stays at index 41 in A and B despite having different declaration ordinals.
A has no declared `counter` field; a numerical address for its vacant position does not assert field presence.

A branch for `C9` at index 41 has these siblings, from leaf to root:

```text
[Z, H(Z,Z), Z, C7, W05]   # A
[Z, H(Z,Z), Z, C7, W07]   # B
```

Both use the same addressing path, but the final sibling changes with the layout.
The A branch reconstructs `root_A`, not `root_B`.
Similarly, a zero leaf at position 1 alone cannot distinguish A's vacancy from B's counter.
A verifier making a presence claim must interpret the authenticated layout, including bit 1, or an authenticated variant that determines it.

## Progressive lists and compatible unions

EIP-7916 defines `ProgressiveList[type]` without a type-level capacity used for Merkleization.
Serialization follows ordinary lists; roots instead use the progressive tree and mix in the element count, not the packed-chunk count.
Basic elements may share chunks, unlike our container fields.
The source says implementations SHOULD enforce context-specific resource bounds.
The executable project permits an optional `LIMIT` checked on values, while leaving it out of the root layout.
Unbounded Merkleization is not an instruction to accept unlimited network data.

EIP-8016 restricts union selectors to 1–127 and requires compatible options.
For progressive containers, shared active positions retain shared names and compatible types, and a shared name cannot move to another position.
The selected executable `_layouts_agree` makes both checks explicit; our A/B pair satisfies them.
Matching field counts alone would not suffice.

For `CompatibleUnion({1:A,2:B})`, serialization prefixes the option's byte selector:

```text
A option: 01 07 09
B option: 02 07 00 09
```

The selected option occupies the left child beneath the union root, opposite its selector commitment.
Wrapping an existing type therefore adds an edge and changes its previous direct addresses.
EIP-8016 explicitly warns that introducing a compatible-union wrapper is not backwards compatible; adding compatible options to an existing wrapper is a different operation.

One Merkleization detail remains unresolved here.
EIP-8016 describes the selector operand as `uint8` serialization in `hash(root, selector)`; a literal one-byte reading gives a 33-byte hash preimage.
The executable [`selector_word`][mixins] explicitly zero-extends it to 32 bytes, giving a 64-byte preimage, and documents that distinction.
This note supplies no union-root vector or claim that those readings agree.
The container-root example does not depend on choosing between them.

## EthLib observations and checks

At the recorded EthLib pin, [`layoutSlots` and `merkleLayout`][lean-layout] retain inactive positions as absent nested slots and mix in the active-fields word.
[`hashTreeRootAtM` and `layoutChunksAtM`][lean-root] turn those slots into zero chunks before progressive Merkleization.
[`progressiveChunkGindex`][lean-index] gives the selected address arithmetic.
Its [`selectorWord`][lean-selector] uses the same 32-byte extension observed in Python.
These are readings of definitions, not a full compatibility theorem review.

A separate source review checked the cited passages and the limits of the explanation.
An independent calculation also reproduced the displayed example values.
The standalone calculation checked both roots, both index-41 branches, and rejection of the old branch against the new root.
It did not execute the official Python package, EthLib, a generated fixture suite, or Lean proofs.
Nested schema changes, every compatibility-rule pair, and the selector-width source boundary require further review before broader conformance claims.

[7495]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7495.md
[7916]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7916.md
[8016]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8016.md
[ssz]: https://github.com/ethereum/ssz-specs/tree/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1
[gindex]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/gindex.py#L89-L119
[mixins]: https://github.com/ethereum/ssz-specs/blob/4c2f08805080c6b4b12ede7bf48ad6a80229f4b1/src/ssz/mixins.py#L51-L70
[lean-layout]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Layout.lean#L81-L174
[lean-root]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Root.lean#L30-L64
[lean-index]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Merkle/Gindex.lean#L68-L85
[lean-selector]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/Ssz/Codec/Layout.lean#L75-L79
