---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 2
catalogue_family: A03
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - signatures
---

# Signed-data preimages and domains

A signature authenticates a digest, so a contribution must establish which bytes produce that digest.
ERC-191 distinguishes signing formats, while EIP-712 defines a typed-data encoding and signing domain.
This note traces their signable bytes through the published Ether Mail example.

## Sources examined

| Source | Sections and material read |
| --- | --- |
| [ERC-191, pinned](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-191.md) | Specification, Registry of Version Bytes, and Example. |
| [EIP-712, pinned](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-712.md) | Specification and its encoding definitions; JSON-RPC Returns and Example; encoding rationale; Test Cases; Security Considerations. |
| [Example.js](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/assets/eip-712/Example.js) and [Example.sol](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/assets/eip-712/Example.sol) | Complete published example and its expected hashes. |

Both standards are Final at these revisions.
Their signing rules do not establish activation of a protocol upgrade or support in a particular wallet API.

## Choose the signing envelope

ERC-191 starts signed data with byte `19`, a version byte, version-specific data, and the message.
Its registered variants have different meanings:

| Version | Version-specific data | Message |
| --- | --- | --- |
| `00` | Intended validator's address. | Application-defined bytes. |
| `01` | EIP-712's 32-byte domain separator. | The message's 32-byte struct hash. |
| `45` | The remaining text of the personal-sign prefix and decimal message length. | Arbitrary message bytes. |

For `personal_sign`, hash this concatenation with Keccak-256:

```text
19 || ASCII("Ethereum Signed Message:\n") || ASCII(decimal(byte_length(message))) || message
```

Here `\n` denotes one newline byte, and `||` means byte concatenation.
The `E` is already version byte `45`; do not insert another version byte before it.
EIP-712's Specification fixes the length to unpadded ASCII decimal, counting message bytes.
For example, UTF-8 text `é` has bytes `c3 a9`, so its decimal length field is ASCII `2`, not `1` or `02`.
This is a hand-derived encoding case, not a published signature vector.

## Hash the type and its values

EIP-712 defines:

```text
typeHash = keccak256(encodeType(type))
hashStruct(value) = keccak256(typeHash || encodeData(value))
digest = keccak256(19 01 || domainSeparator || hashStruct(message))
```

`encodeType` preserves the primary struct's member order.
It appends each referenced struct type once, including transitive dependencies, sorted by type name after the primary type.
For the example, the complete type text is:

```text
Mail(Person from,Person to,string contents)Person(string name,address wallet)
```

`encodeData` also preserves member order and encodes each member into 32 bytes.
Integers and addresses use their padded numeric encoding; fixed bytes are padded at the end.
Dynamic `bytes` and `string` contribute the Keccak-256 hash of their contents.
A nested struct contributes its `hashStruct`; an array contributes the hash of its concatenated element encodings.
Recursive type definitions are supported, but hashing cyclical instance data is undefined.

These rules are not the ABI encoding of an unchanged message struct: strings and nested structs are replaced by hashes, without ABI dynamic offsets.
The source's Solidity example applies `abi.encode` after those replacements and prepends the type hash.
The final envelope instead concatenates two prefix bytes and two 32-byte hashes, without padding the prefix to an ABI word.
`Example.sol` uses `abi.encodePacked` for that final envelope.
Packing the raw message fields would skip the struct-hashing rules.
Hash the resulting bytes, not the ASCII spelling of their hexadecimal representation.

## The complete Ether Mail preimage

The published example supplies these inputs:

| Part | Value |
| --- | --- |
| Domain type | `EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)` |
| Domain fields | `name = "Ether Mail"`, `version = "1"`, `chainId = 1` |
| Verifying contract | `0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC` |
| Sender | `Person(name = "Cow", wallet = 0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826)` |
| Recipient | `Person(name = "Bob", wallet = 0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB)` |
| Contents | `"Hello, Bob!"` |

First hash each `Person`, then encode the two person hashes and the contents hash as the three Mail members.
Hash the Mail type hash followed by those members.
Independently hash the domain type hash followed by its four encoded members.
The expected values below come directly from `Example.js` assertions:

```text
Mail typeHash:
a0cedeb2dc280ba39b857546d74f5549c3a1d7bdc2dd96bf881f76108e23dac2

domainSeparator:
f2cee375fa42b42143804025fc449deafd50cc031ca257e0b194a650a912090f

hashStruct(message):
c52c0ee5d84264471806290a3f2c4cecfc5490626bf912d01f240d7a274b371e
```

The entire signable preimage is the following three fragments concatenated in order.
Whitespace and line breaks are display formatting:

```text
1901
f2cee375fa42b42143804025fc449deafd50cc031ca257e0b194a650a912090f
c52c0ee5d84264471806290a3f2c4cecfc5490626bf912d01f240d7a274b371e
```

Its length is `2 + 32 + 32 = 66` bytes.
The published expected Keccak-256 digest is:

```text
be609aee343fb3c4b28e1df9e632fca64fcfaede20f02e86244efddf30957bd2
```

One helper name needs care: `Example.js` calls the type-hash-prefixed encoding `encodeData`.
For Mail, that helper returns 128 bytes; the specification's `encodeData` names only the 96 bytes of member encodings.
Do not prepend the type hash a second time when following the helper.
The JavaScript example also explicitly leaves array encoding unimplemented.

## What domain separation does not establish

EIP-712 permits selecting the domain fields relevant to the application; unused fields are omitted from its domain type.
Including a chain ID and verifying contract distinguishes those contexts when the verifier checks the intended domain.
It does not prevent the same authorized action from being submitted twice within that domain.
The Replay Attacks section leaves rejection or idempotent handling of repeated messages to the application.

The 66-byte preimage is distinct from a serialized signature, public-key recovery, or a low-s acceptance rule.
The pinned RPC Returns section mentions EIP-155 in its `v` description, while the example asserts `v = 28`.
This note does not reconcile RPC recovery-value conventions or prescribe transaction-style `v` arithmetic for wallet signatures.

## EthLib connections and review limits

At the recorded commit, [Keccak.keccak256](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Hash/Keccak.lean#L465) hashes bytes; [Ecdsa.verify and isLowS](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Secp256k1/Ecdsa.lean#L45) separate verification from the transaction low-s rule.
These are relevant primitives, not evidence of a complete typed-data implementation.

The pinned text and example assertions were read, and an independent source review found no material discrepancy in the selected passages or explanation.
A separate calculation used PyCryptodome 3.23.0 for Keccak-256 and `eth-abi` 5.2.0 for static member words.
It reproduced all four displayed published constants, the complete 128-byte helper result, and the 66-byte preimage without running the published JavaScript.
It also checked the 96-byte normative member encoding and the personal-sign byte-length example.
No JavaScript example, wallet RPC, EthLib build, signature recovery, or Lean proof was run.
