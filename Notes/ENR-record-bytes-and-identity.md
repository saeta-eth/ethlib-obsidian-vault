---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: W01
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
devp2p_commit: "88f1c80800d622cc3a542a2125bbe33cc18c9f9c"
tags:
  - ethlib
  - external-sources
  - enr
---

# ENR record bytes and identity

An Ethereum Node Record binds connectivity information to an identity key through a signature.
It does not establish that an advertised endpoint is reachable or that the reader has the newest record.
This note follows one published record from text to signing bytes, then separates record validation from discovery-protocol checks.

## Sources and scope

The primary sources are [EIP-778, Ethereum Node Records][eip778], [EIP-868, Node Discovery v4 ENR Extension][eip868], and the discovery-packet requirements in [EIP-8][eip8], at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Reading covered EIP-778's specification, rationale and test vector, EIP-868's specification, and EIP-8's Specification section.
These EIPs have Final status at that revision; status alone does not show which software or network implements them.

The [devp2p ENR specification][dev-enr], read completely at commit `88f1c80800d622cc3a542a2125bbe33cc18c9f9c`, explicitly defines the uncompressed-key hash input omitted from EIP-778's wording.
Selected [discovery-v4 sections][discv4] supply node-identity terminology, endpoint proof, packet framing and ENR request/response rules.
This is a bounded comparison of those texts and EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, rather than a survey of discovery v5 or later revisions.

## The canonical record and its content

EIP-778 defines the canonical binary representation as an RLP list:

```text
content = [seq, k1, v1, k2, v2, ...]
record  = [signature, seq, k1, v1, k2, v2, ...]
```

The keys must be sorted and unique.
They are byte sequences, with ASCII preferred; the text does not specify a comparator for arbitrary byte sequences.
EthLib explicitly chooses increasing lexicographic byte order.
The `id` pair names the identity scheme.
Endpoint fields are optional, while verification under the `v4` scheme needs its `secp256k1` public-key pair.
Unknown metadata keys do not by themselves invalidate a record.

The sequence number is an unsigned 64-bit integer.
Nodes **should** increase it whenever their record changes and republish the record.
It orders updates from an identity; it is not a timestamp or an expiration field.
The maximum encoded record size is 300 bytes, and implementations **should** reject larger records.
The text representation is `enr:` followed by base64-encoded RLP bytes; URL-safe base64 without padding is also a **should** requirement.
These distinctions preserve the source's wording while allowing an implementation to choose stricter acceptance rules.

RLP length prefixes describe encoded byte strings and lists, not character counts or key counts.
See [RLP canonical encoding](RLP-canonical-encoding.md) for its integer and length rules.
Removing the signature therefore requires encoding the smaller content list with a new outer length prefix.

## Decode the published example

EIP-778's [test vector][vector] publishes this record:

```text
enr:-IS4QHCYrYZbAKWCBRlAy5zzaDZXJBGkcnh4MHcBFZntXNFrdvJjX04jRzjzCBOonrkTfj499SZuOh8R33Ls8RRcy5wBgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPKY0yuDUmstAHYpMa2_oxVtw0RW_QAdpzBQA8yWM0xOIN1ZHCCdl8
```

Decoding its base64 gives 134 bytes, starting with `f8 84`: an RLP list with 132 payload bytes.
The first item starts with `b8 40`, which introduces the 64-byte signature.
The remaining items decode as follows; hex values below denote bytes, not ASCII hex text.

| Item | Decoded value |
| --- | --- |
| `seq` | `01`, integer 1 |
| `id` | `7634`, ASCII `v4` |
| `ip` | `7f000001`, IPv4 `127.0.0.1` |
| `secp256k1` | `03ca634cae0d49acb401d8a4c6b6fe8c55b70d115bf400769cc1400f3258cd3138` |
| `udp` | `765f`, big-endian integer 30303 |

The four keys occur in increasing ASCII byte order: `id`, `ip`, `secp256k1`, `udp`.
There is no `tcp` entry from which to infer a TCP port.
The loopback address belongs to a published fixture, not an observed remote peer.

The content items occupy `1 + 3 + 3 + 3 + 5 + 10 + 34 + 4 + 3 = 66` bytes after their individual RLP encodings.
Their new list prefix is `f8 42`, producing this complete 68-byte signing preimage:

```text
f84201826964827634826970847f00000189736563703235366b31a103ca634cae0d49acb401d8a4c6b6fe8c55b70d115bf400769cc1400f3258cd31388375647082765f
```

For `v4`, apply Ethereum's Keccak-256 to these RLP bytes and verify the ECDSA signature against the record's public key.
The digest below is an author-derived intermediate, not a hash printed in the EIP:

```text
bc218268b018aecb5d4c5afd5feeb3b920f56eddf09bdf83df9d3de868e1cb95
```

Hashing the base64 text, the full signed record, or the content payload without its list prefix would select different messages.
The ENR signature has fixed-width `r || s` components and omits the recovery identifier `v`.
It is neither a DER-encoded ECDSA signature nor discovery v4's 65-byte packet signature `r || s || v`.
EIP-778 adds no transaction-style low-`s` acceptance requirement.

## Compressed record key, uncompressed hash input

The record stores a 33-byte compressed secp256k1 key: a parity prefix followed by a 32-byte `x` coordinate.
The [pinned devp2p text][dev-v4] defines the node address as `keccak256(x || y)`, with each coordinate zero-padded to 32 bytes.
Thus the hash input has 64 bytes, without the SEC 1 uncompressed-point prefix `04`.
For the fixture, the decompressed coordinates concatenate to:

```text
ca634cae0d49acb401d8a4c6b6fe8c55b70d115bf400769cc1400f3258cd31387574077f301b421bc84df7266c44e9e6d569fc56be00812904767bf5ccd1fc7f
```

Their Keccak-256 hash matches the published node ID:

```text
a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7
```

The discovery-v4 text also calls the identity public key itself a “node ID”; its distance function hashes public keys.
Keep that terminology separate from the 32-byte ENR node address returned by EthLib's `nodeId`.

## Freshness and endpoint checks remain separate

[EIP-868][eip868] adds ENR sequence numbers to ping/pong and introduces requests `[expiration]` with responses `[request-hash, ENR]`.
The recipient **should** check that the record signer is the node that sent the response.
The request hash links the response to the entire request packet.
The sender **should** recently have answered a ping, and an expired request **should** receive no reply.
Discovery v4's separate [endpoint-proof procedure][endpoint] uses a valid pong matching a ping hash to limit traffic amplification.
A valid record signature alone performs none of these request, freshness or endpoint checks.

[EIP-8][eip8] says discovery implementations should ignore extra packet-list elements and bytes after the first RLP value.
That packet-layer extension rule does not erase ENR's sorted-key or canonical-record requirements.
Its 1280-byte packet limit also differs from the 300-byte encoded-record limit.

## What the Lean entry points establish

[Record decoding][lean-decode] rejects oversized records, malformed RLP, invalid sequence scalars, unpaired fields, repeated or unsorted keys, and missing `id`; it does not verify a signature.
[Public-key decoding and `verifyV4`][lean-v4] additionally require `id = v4`, a valid compressed curve point and a 64-byte signature before ECDSA verification.
The [point decoder][lean-point] checks coordinate range, valid compression prefix and existence of the curve point, rejecting infinity.
These are implementation observations; EIP-778 does not state a complete point-validation algorithm.
[ECDSA verification][lean-ecdsa] checks `1 ≤ r,s < n` and public-key validity, without imposing low-`s`.

[`nodeId`][lean-id] validates the public-key representation but explicitly does not verify the record signature.
Its [export wrapper][lean-export] first decodes the record, so successful node-ID extraction still does not mean `verifyV4` succeeded.
Likewise, arbitrary `Enr` values can reach [`Enr.encode`][lean-encode] without size or key-order checks.
Callers must keep these operations' distinct guarantees when composing them.

## Checks and review limits

An isolated Python base64/RLP calculation reproduced the decoded fields, list lengths and content bytes.
PyCryptodome 3.23.0 supplied Keccak-256, and an integer curve calculation decompressed the key and reproduced the published node ID.
Independent RLP, Keccak, and curve calculations reproduced the record/content lengths, signed-content hash, and published node ID, including the distinct result with prefix `04`.
Those calculations did not execute EthLib, verify the ECDSA signature, contact a peer or establish endpoint reachability.
No Lean build or proof replay ran.
Source inspection and deterministic byte agreement are not a cryptographic security proof, a Lean proof or a full conformance result.

[eip778]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-778.md
[eip868]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-868.md#specification
[eip8]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8.md#specification
[vector]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-778.md#test-vectors
[dev-enr]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/enr.md
[dev-v4]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/enr.md#v4-identity-scheme
[discv4]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv4.md
[endpoint]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv4.md#endpoint-proof
[lean-decode]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Enr/Record.lean#L156
[lean-v4]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Enr/V4.lean#L48
[lean-point]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Encoding.lean#L66
[lean-ecdsa]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Secp256k1/Ecdsa.lean#L47
[lean-id]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Enr/V4.lean#L83
[lean-export]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Exports/Enr.lean#L80
[lean-encode]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Enr/Record.lean#L135
