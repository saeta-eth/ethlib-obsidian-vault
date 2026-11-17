---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E09
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellow_paper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - account-proofs
  - state-commitments
---

# Account trie proofs and absence boundaries

An account proof authenticates a value or its absence relative to a selected state root.
It does not establish that the root belongs to the canonical chain or that an RPC response's unverified zero is correct.
Follow the hashed key, encoded account and authenticated path before interpreting the returned fields.

## Sources and scope

The primary sources are the Shanghai [Yellow Paper][yp] at `efc5f9a1f356cba376c978eedb63cb0363c2aa85`, [EIP-1186][1186] and [EIP-161][161].
At EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`, EIP-1186 is Stagnant and EIP-161 is Final.
Reading covered the Yellow Paper's §4.1 account/state mapping, Appendices C–D's compact paths and trie composition, and the selected Appendix B RLP rules.
Both EIPs were read, concentrating on proof fields, absence and empty-account conditions.
This is one constructed proof, not a published vector, network observation or complete proof-verifier audit.

## Keys, account values and node references

The state trie maps `Keccak256(address)` to the account's RLP bytes, where address means its 20 decoded bytes.
It does not traverse the hexadecimal characters of the address or use those 20 bytes directly as the path.
Each account value is:

```text
RLP([nonce, balance, storageRoot, codeHash])
```

The account's storage trie separately hashes each 32-byte storage key and stores the RLP-encoded integer value.
Encode a numeric slot as 32 big-endian bytes before hashing it; a JSON quantity's display spelling is not that preimage.
The account proof authenticates `storageRoot` first, and a storage proof is checked against that root.
The [RPC note](JSON-RPC-quantities-and-block-state.md) explains the separate quantity/data and block-selector boundaries.

EIP-1186 calls its `storageHash` field “SHA3 of the StorageRoot,” wording that can suggest another hash.
The Yellow Paper defines the account field as the storage trie's root hash already, and EIP-1186's rationale identifies the returned field with that account tuple.
The construction here uses that committed root directly; it does not silently add a second hash.

Appendix D's trie traverses high then low nibbles of each key byte.
A leaf has two items: a hex-prefix-encoded remaining path and the value bytes.
An extension instead carries a nonterminal shared path and a child reference; a branch has sixteen child positions and a value position.
Appendix C's compact-path flags distinguish leaf/extension and odd/even path length.

A nonempty child with an RLP encoding shorter than 32 bytes is embedded as its RLP structure in the parent.
At exactly 32 bytes or more, the parent carries the child's Keccak-256 hash instead.
An absent child uses the empty byte string.
The root commitment is always the hash of the root-node encoding, including when that encoding is short.
An embedded child is already available inside its parent, rather than requiring another independently supplied node to reveal it.

## A complete one-leaf construction

Construct a state containing one account at address `0x1111111111111111111111111111111111111111`, with nonce 0, balance 7 wei, no storage and no code.
This does not assert a reachable Ethereum history for that state.
Its balance makes it nonempty under EIP-161 even though its nonce is zero and its code is empty.

Using Ethereum Keccak-256, the constants and hashed key are:

```text
empty storage trie root = Keccak256(80)
56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
empty code hash = Keccak256(empty bytes)
c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470
key = Keccak256(the 20 address bytes)
e2c07404b8c1df4c46226425cac68c28d27a766bbddce62309f36724839b22c0
```

Here `80` is the RLP encoding of the empty byte string, not the empty list `c0`.
The account encoding has 70 bytes; concatenate these lines:

```text
f8448007
a056e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
a0c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470
```

Its list contents have 68 bytes: nonce zero is `80`, balance seven is `07`, and each 32-byte hash has prefix `a0`.
The [RLP note](RLP-canonical-encoding.md) explains why integer zero does not use byte `00`.

The single leaf consumes all 64 nibbles of the hashed address.
The even-length leaf flag therefore gives compact path `20 || key`, occupying 33 bytes.
In the outer leaf, that path has prefix `a1` and the 70 account bytes have prefix `b846`.
The account is a byte-string value containing RLP, not a nested four-item list in the leaf.
The outer content occupies `34+72 = 106` bytes, so prefix `f86a` gives a 108-byte node.

The complete proof array contains just that node:

```json
[
  "0xf86aa120e2c07404b8c1df4c46226425cac68c28d27a766bbddce62309f36724839b22c0b846f8448007a056e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421a0c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470"
]
```

Hashing the 108 decoded node bytes gives the constructed root R:

```text
ccbbed3b888816e11a386a6f2f7b38958d2de00c12c343051cfb52717fc65c98
```

Conditionally take R as the expected root for this exercise.
Hash-check the node against R, decode its two-item structure, check the leaf flag and full hashed path, then decode the account bytes.
The resulting fields support nonce 0 and balance 7 with exactly the two displayed hashes.
No storage slot was requested, so this example supplies no separate storage-proof array entries.

## Absence needs an authenticated stopping point

For address `0x2222222222222222222222222222222222222222`, the hashed key is:

```text
2ab0a4443bbea3fbe4d0e1503d11ff1367842fb0c8b28a5c8550f27599a40751
```

The same node still hashes to R, but its leaf path differs from this key at the first nibble.
Because the authenticated root is itself a leaf, there is no other branch below which the requested key could occur.
This proves absence relative to R, conditional on the hash commitment's security, rather than absence at any real network height.
EIP-1186 also describes an empty branch entry as an absence stopping point.
A missing node, hash mismatch or malformed encoding is not affirmative evidence of absence.

A returned balance of zero alone does not distinguish absence from a present account with code or a nonzero nonce.
EIP-161 defines empty using code, nonce and balance together and deletes touched empty accounts at transaction end under its rules.
Its addendum requires empty-account deletions to revert when the state reverts.
None of these statements turns a proof at one root into a statement about every earlier or later root.

## Trusted roots and selected EthLib definitions

EIP-1186's account/storage fields and proof nodes must be related to the state root selected for the requested block.
Trusting a root requires a separate authenticated header and chain-selection context; accepting a root supplied alongside a proof proves no such context.
A height or `latest` selector alone does not eliminate reorganization concerns.

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`encodeAccount`][account] constructs the four-field account value, and [`verifyProof`][proof] returns either value bytes or absence after traversing supplied nodes.
[`AccountProof.verifyAccount`][verify] checks decoded account fields against the answer, while its slot checker anchors proofs in that account's storage root.
The [`absentFields`][absent] policy accepts zero balance/nonce and selected zero-or-empty hash representations for an absent account.
The module explicitly identifies missing EIP-1186 rules for those absent-account hash fields; that local policy is not attributed to a universal RPC requirement here.
These are definition observations, not theorem-coverage findings.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Python RLP construction and PyCryptodome 3.23.0 Keccak checked the widths, displayed account/node encodings, root, different-address path and changed-balance/nested-list negative controls.
No generic verifier, live RPC, storage proof, EthLib binding, Lean build or proof replay ran.

[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[1186]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1186.md
[161]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md
[account]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/Account.lean#L62
[proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Trie/Proof.lean#L161
[verify]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/AccountProof.lean#L190
[absent]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/AccountProof.lean#L176
