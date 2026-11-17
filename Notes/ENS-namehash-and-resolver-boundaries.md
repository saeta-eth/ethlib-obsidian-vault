---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A12
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - ens
  - name-resolution
---

# ENS namehash and resolver boundaries

An ENS node identifies a name for contract lookups; it does not establish who controls the name or the address returned for it.
Resolution crosses distinct boundaries: normalize the input, compute its node, find its resolver in the registry, then request a particular record.
Reverse names and text records add information without collapsing those boundaries.

## Sources and version limits

All ERC links pin `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.

| Source | Passages examined and role |
| --- | --- |
| [ERC-137][e137], Final at this pin | Complete text, especially Name Syntax, namehash algorithm and published vectors, Registry specification, Resolver specification, and Contract Address Interface. Original name-to-record protocol; appendices are illustrative implementations. |
| [ERC-181][e181], Final | Complete text, especially reverse name construction, registrar methods, and Resolver interface. Address-to-name profile. |
| [ERC-634][e634], Stagnant | Complete text, especially Resolver Profile, Global Keys, and Service Keys. Text-record profile. |
| [RFC 1035 §3.1][dns] | Uncompressed DNS name representation, used only for the byte-format comparison below. |

ERC-137's Name Syntax specifies UTS46 normalization with `transitional=false` and `useSTD3AsciiRules=true`, including case folding before hashing.
That is the pinned document's historical rule, not a validated guide to current ENS normalization.
ENSIP-15 normalization is outside this note's reviewed source set.
The examples assume already-normalized lowercase ASCII labels; they establish no general Unicode acceptance or normalization result.
No deployed registry address, live resolver, or current ownership was queried.

## From normalized labels to a node

Use `K` for Ethereum's Keccak-256 and `||` for byte concatenation.
For the ASCII inputs examined here, a label's bytes are its UTF-8 bytes.
ERC-137's pseudocode calls the hash `sha3`; its published vectors below identify the intended Ethereum hash.
Do not substitute the distinct SHA3-256 function.

```text
node = 32 zero bytes
for label in labels, from right to left:
    labelhash = K(UTF8(label))
    node = K(node || labelhash)
```

Each outer hash receives exactly 64 bytes: a 32-byte parent node followed by a 32-byte label hash.
Concatenate their raw bytes, not hexadecimal text or two strings with `0x` prefixes.
The dot separates labels; it is not included in either label's hash input.
The empty name returns the initial zero node directly, without hashing an empty message.

ERC-137 publishes these known answers:

```text
namehash("") =
0000000000000000000000000000000000000000000000000000000000000000

namehash("eth") =
93cdeb708b7545dc668eb9280176169d1c33cfd8ed6f04690a0bcc88a93fc4ae

namehash("foo.eth") =
de9b09fd7c5f901e23a3f19fecc54828e9c848539801e86591bd9801b019f84f
```

For `foo.eth`, first hash the three bytes `65 74 68` for `eth`.
Hash `zero_node || K(657468)` to obtain the published `eth` node.
Then hash the three bytes `66 6f 6f` for `foo`, and hash `eth_node || K(666f6f)` to obtain the final node.
Hexadecimal arguments in those expressions denote decoded bytes.
This processes the parent before the child, despite the name's left-to-right written order.

Namehash is neither `K(UTF8("foo.eth"))` nor hashing a DNS wire name.
RFC 1035's uncompressed representation prefixes each label with its length and terminates with the root's zero length:

```text
03 66 6f 6f 03 65 74 68 00
```

Those nine bytes describe a different representation from both the seven text bytes and the recursive 64-byte namehash inputs.
The shared dot-separated notation does not make these encodings interchangeable.

## Registry control, resolver code, and an address record

For node `N`, the original protocol separates three values:

| Query | Meaning |
| --- | --- |
| `registry.owner(N)` | The node's owner, called its registrar in ERC-137. |
| `registry.resolver(N)` | The contract responsible for resolving its records. |
| `resolver.addr(N)` | The Ethereum address record supplied by that resolver. |

These addresses can differ.
The node owner controls `setResolver` and can create or assign subnodes through the registry interface.
The resolver supplies record data.
The resolved address need not control either contract or own the name; ERC-181 explicitly observes that anyone can configure a name to point to an address.
Knowing `N` supplies no ownership authority.

Check that a resolver has been specified before attempting record lookup; a zero resolver address does not identify a resolved recipient.
For a resolver supporting `addr`, ERC-137 requires the zero address when the node has no address record.
Its client rule is stronger than merely displaying “missing”: clients MUST treat that zero result like an absent resolver and refuse to send funds to or interact with the address.
A nonzero result alone does not prove the recipient's ownership of the name.

Resolvers may implement different record profiles.
ERC-137 requires `supportsInterface(bytes4)` and support for its own interface ID, `0x01ffc9a7`.
The `addr(bytes32)` profile has ID `0x3b3b57de`; a positive interface response commits the resolver to implementing its functions.
The [ABI note](ABI-encoding-and-event-topics.md) explains function selectors and encoded calls.
ERC-137 does not standardize every resolver's record-update permissions: do not promote an appendix's sample owner check into a universal resolver rule.

## Reverse names require a separate consistency decision

ERC-181 builds the reverse name from the address's 40 lowercase hexadecimal characters, without `0x`, followed by `.addr.reverse`.
Its published address example yields:

```text
112234455c3a32fd11230c42e7bccd4a84e02010.addr.reverse
```

This is the source's example, not a current registry-address assertion.
Compute that name's node, obtain its resolver, and call `name(bytes32)`, whose interface ID is `0x691f3431`.
The resolver must return a valid ENS name or the empty string when none is defined.
The registrar lets an address claim its own reverse node and delegate that node's ownership to another address.
This does not grant ownership of the forward name stored in its `name` record.

Suppose address `B` publishes reverse name `foo.eth`, while forward resolution of `foo.eth` returns distinct address `A`.
The reverse answer and forward answer disagree about the proposed association.
An application relying on that association can require forward resolution of the returned name to equal the original address before accepting it.
This is an application consistency check derived from the two independent mappings; the pinned ERC-181 contains no forward-check MUST or SHOULD to quote.
Even equality does not imply that the address owns the registry node.

## Text records describe rather than authenticate

ERC-634 adds `text(bytes32,string)`, interface ID `0x59d1d43c`.
The value can be arbitrary UTF-8 text; an absent key must return the empty string.
Global keys use lowercase letters, digits, and hyphens, with examples including `email`, `url`, and `avatar`.
Service keys use an owned namespace in reverse-dot notation and contain at least one dot, such as `com.github`.
That naming convention avoids key collisions; a returned username is not proof that the name holder controls the external service account.

For `display`, ERC-634 requires the value to match the ENS name after case folding; clients **should** ignore a value that does not.
Preserve that distinction between the value's MUST condition and the client's stated recommendation.
The text-query interface supplies no general proof of metadata truth and specifies no universal setter authorization rule.

## EthLib connection and review limits

At the recorded EthLib commit, [`Keccak.keccak256`][keccak] takes a `ByteArray` and returns 32 digest bytes.
Its hash definition uses the original Keccak padding; the surrounding source explicitly distinguishes SHA3-256's suffix.
These byte operations are the relevant primitive entrypoints for investigating the node computation.
Reading them establishes neither name normalization nor registry, resolver, or metadata authorization behavior.

This note read the cited sources and selected hash definitions.
The sources and explanation received a separate review.
PyCryptodome 3.23.0 independently reproduced all three published namehash vectors and the four displayed interface identifiers.
The 64-byte outer preimages and nine-byte DNS encoding were also checked.
The displayed node values are the ERC's published vectors; no EthLib namehash execution, Lean build, proof replay, or live ENS lookup was performed.
Wildcard resolution, off-chain lookup, multi-chain address records, and current normalization require separate source review before extending this baseline.

[e137]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-137.md
[e181]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-181.md
[e634]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-634.md
[dns]: https://www.rfc-editor.org/rfc/rfc1035.txt
[keccak]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Hash/Keccak.lean#L288-L476
