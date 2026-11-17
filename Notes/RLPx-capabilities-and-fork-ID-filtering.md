---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: W03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
devp2p_commit: "88f1c80800d622cc3a542a2125bbe33cc18c9f9c"
tags:
  - ethlib
  - external-sources
  - rlpx
  - fork-id
---

# RLPx capabilities and fork-ID filtering

Permitting an extended handshake, negotiating a shared capability, and accepting a fork ID are different decisions.
None alone establishes that a peer is honest or that its blocks are valid.
Keep the transport, capability, and chain-configuration checks separate when following connection setup.

## Sources and reading scope

[EIP-8][8], [EIP-706][706], [EIP-2124][2124], and [EIP-6122][6122] all have status Final at the recorded EIPs commit.
Reading covered their relevant specifications, EIP-706's decompression caveats, EIP-2124's validation/rationale and selected controls, and EIP-6122's timestamp example and applicability rules.
[EIP-1898's Specification][1898] supplies only the mainnet genesis-hash literal used below; its header is also Final.
Final status does not identify a network's activation point or supported capability versions.

The pinned devp2p [RLPx specification][rlpx] supplies Node Identity, Initial Handshake, Framing/MAC, Capability Messaging, and Hello rules.
Its transport version is 5; handshake `auth-vsn` and `ack-vsn` remain 4.
The pinned [Ethereum Wire Protocol][eth] identifies its own version as `eth/72`; reading here covers Status and the historical `eth/64` introduction of fork IDs.
Those capability versions are distinct from devp2p's version and from chain upgrades.
This note does not reproduce ECIES, session-secret derivation, encrypted frames, or a complete peer-admission policy.

## Accept extensions without waiving authentication

EIP-8 says implementations should ignore Hello-version differences and additional trailing Hello list elements.
Its auth/ack decoding likewise should ignore version mismatches, extra list elements, and bytes following the first RLP list.
Those bytes are permitted padding within the encrypted body, not an instruction to ignore arbitrary transport data.
Its recommendation for at least 100 bytes of auth padding applies during the transition from the older fixed-size format.

The pinned RLPx text uses must for ignoring auth/ack version mismatches, additional body fields, and Hello's extra fields/version differences.
It still describes encrypted handshake packets and first-frame MAC authentication, and recommends frame MAC verification before decryption.
The two-byte handshake size participates as authenticated data in the specified ECIES construction.
Forward compatibility does not authorize accepting a failed MAC or replacing malformed required fields with defaults.
The exact signature construction and its authentication argument are outside this reading; the pinned RLPx prose does not fully define `sig`.

EIP-706 makes compression conditional on the advertised devp2p version: use the older behaviour for a peer advertising only version 4, and Snappy for version 5 or higher.
The initial handshake message is never compressed, and Snappy framing is not used.
The v5 RLPx layout keeps the RLP message ID outside the compressed message data.
Its compressed-size boundary does not replace the recommended decoded-size check before decompression.
The pinned v5 text says messages with uncompressed data larger than 16 MiB should be rejected by closing the connection.
The [Snappy note](Snappy-frames-and-response-chunk-boundaries.md) explains raw-block versus framed-stream formats separately.

## Allocate message IDs from the shared capability set

Hello advertises pairs of case-sensitive ASCII capability names and version numbers.
The peers select exact shared name/version pairs; for a shared name with several common versions, the numerically highest wins.
Unsupported capabilities are ignored.
The selected capabilities receive contiguous ranges in alphabetical name order, starting at `0x10`; `0x00–0x0f` is reserved for p2p.
Each capability must statically specify its message-ID count.
The frame's `capability-id` field is not the multiplexing mechanism in this version.

Consider these invented advertisements, not real `eth` or `snap` message counts:

```text
local:  (aaa,1), (aaa,2), (bbb,1), (ccc,1)
remote: (aaa,2), (aaa,3), (bbb,1)
chosen: (aaa,2), (bbb,1)
```

Assume `aaa/2` requires three IDs and `bbb/1` requires two.
Their ranges are `aaa/2: 0x10–0x12` and `bbb/1: 0x13–0x14`.
Message 1 within bbb therefore uses wire ID `0x13+1=0x14`.
Neither aaa/1 nor aaa/3 is common; ccc gets no range.
A Hello version mismatch does not make an unshared capability version compatible.

## A complete historical fork-hash control

EIP-2124 computes IEEE CRC32 over genesis bytes followed by each already-passed fork block as an eight-byte big-endian integer.
Order matters; duplicate activations at the same block enter once, and rules already active at genesis do not add a fork.
This is ordinary IEEE CRC32, not the CRC-32C used by Snappy framing.

Use this complete 32-byte mainnet genesis hash:

```text
d4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3
```

EIP-2124 publishes the following results:

| Passed forks | Exact checksum input after the genesis bytes | Total bytes | CRC32 |
| --- | --- | --- | --- |
| None | Empty suffix. | 32 | `fc64ec04` |
| Homestead | `0000000000118c30`, encoding 1150000. | 40 | `97c2c34c` |
| Homestead, DAO | `0000000000118c3000000000001d4c00`, encoding 1150000 then 1920000. | 48 | `91d1f948` |

These are published historical controls, not claims about today's last known fork.
`FORK_NEXT` separately advertises the next known activation, or zero if none is known.
The wire identifier is `RLP([FORK_HASH,FORK_NEXT])`: a fixed four-byte checksum plus an RLP integer.
The eight-byte activation encoding belongs to the checksum input; it does not require an eight-byte RLP encoding of every next-fork value.

## Filter a claim against the local schedule

EIP-2124 cross-validates fork IDs rather than comparing them for equality.
Matching hashes can be accepted while announced future forks differ, unless the remote's announced next fork has already passed locally.
A matching earlier local prefix can be accepted only when the remote next value matches the locally following fork.
A matching later prefix completed by locally known future forks can also be accepted; other cases are rejected.

For a derived conditional case, assume the historical mainnet schedule and local head 1500000, with local hash `97c2c34c`.
A remote hash `fc64ec04` with next 1150000 passes rule 2: it describes an earlier prefix and its correct next fork.
The same remote hash with next zero fails that rule as stale.
Acceptance permits this synchronization relationship; it does not prove that the remote will ultimately follow every local rule.
EIP-2124 explicitly leaves some three-way-fork separation unresolved and explains why cryptographic hashing cannot make a peer's self-report honest.

EIP-6122 adds timestamp-scheduled forks, requiring timestamp forks at or after block-scheduled forks and verification by block first, then timestamp.
Its fictional timestamp 1668000000 after Homestead appends `00000000636ba900` and yields published CRC `cb37b2ee`.
That fixture is not the real Shanghai activation time.
The block-only EIP-2124 controls and this amended schedule must retain their separate applicability.

## Selected EthLib connections and checks

At the pinned EthLib commit, [`leadingFields` and `decodeBody`][handshake] retain the required auth/ack prefix and ignore the permitted extra fields/body padding.
That module explicitly implements the EIP-8 packet form rather than the older fixed-size form.
[`ForkId.Schedule.checksum`][checksum] and [`validate`][validate] separate checksum construction from prefix/next-fork validation.
Its `TIMESTAMP_THRESHOLD` is an explicitly documented local choice for interpreting a peer's next value, not a constant mandated by EIP-6122.
These are selected definition observations, not claims about full negotiation execution or proof coverage.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Python `zlib.crc32` reproduced all three displayed EIP-2124 checksums and EIP-6122's fictional timestamp control.
Integer/set calculations checked the invented capability allocation and the conditional prefix/next-value cases.
No RLPx connection, handshake encryption, frame/MAC check, Snappy decoder, EthLib binding, Lean build, or proof replay ran.

[8]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8.md#specification
[706]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-706.md#specification
[2124]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2124.md
[6122]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6122.md
[1898]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1898.md#specification
[rlpx]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/rlpx.md
[eth]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/caps/eth.md#status-0x00
[handshake]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Rlpx/Handshake.lean#L112
[checksum]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Devp2p/ForkId.lean#L139
[validate]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Devp2p/ForkId.lean#L228
