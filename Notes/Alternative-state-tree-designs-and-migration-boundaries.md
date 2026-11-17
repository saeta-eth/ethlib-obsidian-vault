---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 3
delivery_batch: "follow-up-12"
catalogue_family: E10
assessment_priority: Unresolved
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - state-trees
  - migration
---

# Alternative state-tree designs and migration boundaries

Changing a state tree changes key placement, value encoding, proof objects and migration obligations together.
A shared account layout does not make two proposed commitments interchangeable.
This first assessment leaves E10 Unresolved because a target design, reconciled specification and migration context still need selection.

## Sources and reading scope

At EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`, [EIP-6800][6800] is Stagnant, [EIP-7864][7864] is Draft and [EIP-7523][7523] is Last Call.
An expired Last Call deadline does not change the recorded status.
Reading covered the tree definitions, node commitments, account/code/storage embeddings, selected rationale and compatibility boundaries of the first two documents, and EIP-7523's complete text.
Linked migration, access-gas and cryptographic specifications were not assessed.
No status here establishes adoption, activation or a preferred replacement for the existing trie.

## Compare the commitment and embedding separately

The [account-proof note](Account-trie-proofs-and-absence-boundaries.md) follows the existing account trie and its separately committed storage trie.
Its account value is an RLP tuple containing nonce, balance, storage root and code hash.
That proof format cannot be carried over merely by substituting a different hash function.

| Boundary | EIP-6800 | EIP-7864 |
| --- | --- | --- |
| Commitment design | Verkle construction using Bandersnatch Pedersen commitments. | Binary node hashes, with stems grouping 256 values. |
| State embedding | One space of 32-byte keys and 32-byte values for account data, code chunks and storage. | One space with a storage-type prefix and variable-length keys; values are 32 bytes. |
| Key derivation | A Pedersen-derived 31-byte stem followed by one suffix byte. | Header/code/storage key helpers, whose inconsistencies remain below. |
| Hash decision | Depends on the stated curve, basis and point-to-field construction. | BLAKE3 is experimental in the draft; the final hash is explicitly undecided. |
| Proof question | Which openings authenticate the requested stem values? | Which authenticated sibling/path data establish the requested value? |

Both proposals put code chunks into the tree and pack several account fields into one basic-data leaf.
Co-locating related values can reduce separate branch openings, but does not supply a proof-verification algorithm or measured witness size by itself.
The Verkle opening and binary sibling-hash checks are different obligations.
Neither was executed here.

## Complete basic-data packing and key-input trace

Choose address `0x1111111111111111111111111111111111111111`, version 0, code size 0, nonce 1 and balance 7.
Both documents give the following big-endian packing for the basic-data value:

| Field | Byte offset | Width | Chosen value |
| --- | --- | --- | --- |
| Version | 0 | 1 | 0. |
| Reserved | 1 | 4 | Zero bytes for this example. |
| Code size | 5 | 3 | 0. |
| Nonce | 8 | 8 | 1. |
| Balance | 16 | 16 | 7. |

The complete 32-byte value is:

```text
0000000000000000000000000000000100000000000000000000000000000007
```

This trace follows the explicit big-endian packed-field description.
EIP-6800 also states a general 32-byte little-endian number convention; this note does not infer an override rule between those passages.
The account fields retain their roles, while their packed widths and encoding differ from the [RLP account representation](RLP-canonical-encoding.md).
This trace does not establish that every possible old state fits those widths or migrates correctly.

For EIP-6800's key placement, first prepend twelve zero bytes to the address:

```text
Address32 =
0000000000000000000000001111111111111111111111111111111111111111
preimage(index) = Address32 || LE32(index)
LE32(0) =
0000000000000000000000000000000000000000000000000000000000000000
LE32(1) =
0100000000000000000000000000000000000000000000000000000000000000
LE32(2^240) =
0000000000000000000000000000000000000000000000000000000000000100
```

Each displayed preimage is completely specified by concatenating Address32 with the corresponding index line, giving 64 bytes.
The Pedersen input transformation starts with `2+256*64 = 16386`, then four little-endian 128-bit input limbs and 251 zero limbs.
No curve operation was performed.
Write `S_i = pedersen_hash(preimage(i))[:31]` symbolically; the final key is `S_i || suffix`.

EIP-6800 uses width 256, header-storage offset 64, code offset 128 and main-storage offset `256^31`:

| Location | Position arithmetic | Tree index | Suffix, hex |
| --- | --- | --- | --- |
| Basic data | Header leaf. | 0 | `00` |
| Code hash | Header leaf. | 0 | `01` |
| Storage slot 0 | `64+0` | 0 | `40` |
| Storage slot 63 | `64+63` | 0 | `7f` |
| Storage slot 64 | `256^31+64` | `2^240` | `40` |
| Code chunk 0 | `128+0` | 0 | `80` |
| Code chunk 127 | `128+127` | 0 | `ff` |
| Code chunk 128 | `128+128` | 1 | `00` |

Slots 0 and 63 share the basic-data stem; slot 64 crosses the special-case boundary.
The code rows illustrate the placement function, not present chunks for the chosen no-code account.
The Pedersen basis is left as a placeholder in the document, so no actual stem, key digest or commitment is claimed.

## Preserve the source inconsistencies

The selected binary draft cannot be treated as one reconciled parameter set:

- Its prose describes the first 64 storage slots and 128 code chunks in the account stem, while its table sets those counts to 4 and 16.
- Its required `CODE_OFFSET > HEADER_STORAGE_OFFSET` invariant conflicts with table values 4 and 20.
- Its storage helper refers to undefined `STORAGE_OFFSET` and `chunk_id`; other helpers call `hash` although a `tree_hash` function is defined.
- Its address-padding snippet contains an escaped byte literal; this trace follows the prose's twelve zero bytes rather than executing that snippet as a correction.

EIP-6800 also leaves its Pedersen basis unspecified, and its displayed root routine builds `data_as_stems` but later iterates `data`.
This reading records those facts without repairing or running either root routine.
The shared packed value above therefore supplies no canonical binary-tree key or interoperable root vector.

## Zero, absence and migration require their own rules

EIP-6800 distinguishes a present zero value from an absent leaf by adding `2^128` to the lower value limb before commitment.
EIP-7864 distinguishes `empty` leaves from present 32-byte values, and hashes a present zero value through its selected H.
Those are leaf-presence rules, not definitions of an empty Ethereum account.

EIP-7523 separately proposes a post-Merge prohibition on accounts with no code, zero nonce and zero balance.
It constrains the Merge pre-state or merged-at-genesis state, and permits assumptions for the specific chains satisfying its listed conditions.
Its security section calls for verification of the historical clearing assertion.
This note does not establish that assertion or apply the proposal universally to arbitrary test states.

The EIP-6800 rationale describes an initially empty new tree receiving changes and copies of accessed state while the MPT is frozen.
EIP-7864's rationale likewise starts a new tree, freezes the MPT and refers to later migration; their Fork sections delegate transition details elsewhere.
A layout comparison alone does not determine lookup precedence, migration completion or availability of historical proof data.
An existing MPT proof remains a statement about its original root and scheme; these index calculations do not convert it into a new-tree proof.
Those delegated rules need a separate reading before proposing a transition implementation.

## Selected context and remaining decision

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`encodeAccount`][account] takes a storage root for the RLP account tuple, and [`stateEntries`][state] constructs account values with separately computed storage roots.
These definitions identify the existing representation relevant to the comparison; they establish neither a missing module nor a commitment to a replacement design.

The next contribution needs a chosen design revision, reconciled parameters, cryptographic/hash specification and migration rule set.
Keep E10 Unresolved until that target and its proof obligations are explicit.
Python checked the packed value, preimage widths, input-limb counts and index/suffix arithmetic.
No Pedersen commitment, binary hash selection, root, witness, proof, migration, EthLib binding, Lean build or proof replay ran.

[6800]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6800.md
[7864]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7864.md
[7523]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7523.md
[account]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/Account.lean#L62
[state]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/Root.lean#L68
