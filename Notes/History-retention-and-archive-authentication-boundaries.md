---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
delivery_batch: "follow-up-12"
catalogue_family: I06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
e2store_formats_commit: "31cf6b577123bf1703eb3713f0b242f65fea806a"
nimbus_e2store_commit: "613f4a9a50c9c4bd8568844eaffb3ac15d067e56"
tags:
  - ethlib
  - external-sources
  - history
  - archive-formats
---

# History retention and archive-authentication boundaries

An archive frame locates bytes; an authenticated history also needs commitments tied to a selected chain.
Retention policy, file indexes, internal consistency, and trusted checkpoints answer different questions.
Keep those questions separate when importing historical data or describing what an archive decoder establishes.

## Sources and scope

[EIP-4444][4444] is Draft and [EIP-7643][7643] is Stagnant at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
The [e2store-format repository][formats] is pinned at `31cf6b577123bf1703eb3713f0b242f65fea806a`.
Its README links the [Nimbus base description][base] at `613f4a9a50c9c4bd8568844eaffb3ac15d067e56`, which supplies the generic framing used below.
The repository README marks Era1 deprecated; this note uses it to explain historical execution archives, without selecting a replacement format.

Reading covered complete EIP-4444, EIP-7643's definitions and security text, selected format/type descriptions, and the linked base framing/index sections.
Only selected EIP-7643 CSV rows were inspected; neither the full table nor the accumulator was independently verified.
These source versions do not establish network activation, data availability, or a deployed client's retention configuration.

## Retention is a qualified proposal

The pinned EIP-4444 uses `HISTORY_PRUNE_EPOCHS = 14299`, matching the stated consensus-layer block-retention window.
Its Specification says clients SHOULD NOT serve older headers, bodies, and receipts on p2p, and MAY prune those objects locally.
Its Abstract instead says clients must stop serving them; this note preserves the Specification's explicit requirement strength.
A blanket one-year retention summary would misdescribe this source snapshot.

For bootstrapping, clients MUST use a valid weak-subjectivity checkpoint.
The proposal assumes such a checkpoint is already configured and leaves its acquisition outside scope.
It discusses out-of-band history distribution and import/export, while leaving several missing-history RPC consequences unspecified.
Obtaining an archive from a distribution network therefore does not supply the checkpoint's validity or authenticate the file's contents. [Specification and compatibility][4444]

## What the pre-PoS root commits to

EIP-7643's accumulator commits to pre-Merge header hashes and their total difficulties.
Its `HeaderRecord` contains `bytes32 block_hash` and `uint256 total_difficulty`.
An `EpochRecord` is an SSZ list of at most 8192 records; the outer accumulator contains up to 2048 completed epoch roots plus its current epoch.
These 8192-block groups are different units from EIP-4444's consensus-layer epochs.

The stated root for data before block 15537394 is:

```text
ec8e040fd6c557b41ca8ddd38f7e9d58a9281918dc92bdb72342a38fb085e701
```

Counting blocks from zero gives `15537394 = 1896*8192 + 5362`.
That arithmetic does not recompute any SSZ root or authenticate a downloaded record.
The EIP recommends community verification before accepting its proposed root. [Specification and security][7643]

Era1's per-file accumulator is the root of one bounded list of header records, not this entire outer accumulator.
Matching a file's contents to its own stored accumulator establishes a consistency relation, without independently establishing the expected history.
Header commitments can then anchor transactions, receipts, or state only when those corresponding objects and paths are checked.
The [receipt note](Receipts-status-gas-and-block-commitments.md) distinguishes those commitment domains.

## Generic framing and a complete offset exercise

The linked Nimbus document defines an eight-byte header: two tag bytes followed by six little-endian length bytes, excluding the header.
The tag is a two-byte vector; `65 32` means the Version tag, ASCII `e2`.
Treating the printed type label `0x6532` as a little-endian integer would incorrectly reverse those wire bytes.
Unknown records can be skipped using their advertised length, subject to having enough input bytes. [General structure][base]

The Version descriptions call its record eight bytes but print a nine-element byte array.
The exercise follows the explicit eight-byte framing and zero-length Version rule; the contradictory displayed array remains a source issue.
Start with Version, then the base document's published generic record with tag `22 32` and four payload bytes:

```text
Version, 8 bytes:
65 32 00 00 00 00 00 00

Generic record, 12 bytes:
22 32 04 00 00 00 00 00 01 02 03 04
```

The generic record starts at file offset 8, and the next record starts at 20.
Append a constructed `SlotIndex`, tag `69 32`, with starting slot 100, one offset, and count 1.
The [SlotIndex description][slotindex] uses signed eight-byte little-endian offsets from the beginning of the index record.
Its offset is therefore `8-20 = -12`, encoded as `f4 ff ff ff ff ff ff ff`.
The data occupies `8+8+8 = 24` bytes and the entire index `8*1+24 = 32` bytes.

These three lines concatenate to the complete 52-byte exercise:

```text
6532000000000000
223204000000000001020304
69321800000000006400000000000000f4ffffffffffffff0100000000000000
```

The offset word itself starts at file position 36, but the target calculation uses index-record start 20: `20+(-12)=8`.
Using the word's position as the base would point elsewhere.
An index offset of zero denotes no data in this format; that marker alone is not an authenticated absence proof.
This is a generic framing/SlotIndex illustration, not a conforming Era1 archive, beacon block, or historical proof.

## Source differences and selected EthLib definitions

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`E2store.encodeHeader` and `readHeader`][lean-framing] explicitly define a different layout: two ordered tag bytes, four little-endian length bytes, and two zero reserved bytes.
Its module states that layout as its own definition because the cited format descriptions do not fully lay out the header.
The linked Nimbus six-byte length has a larger range; its upper two length bytes are not reserved in that text.

For example, the header-only sequence `22 32 00 00 00 00 01 00` advertises length `2^32` under Nimbus's layout.
EthLib's selected `readHeader` instead rejects its nonzero reserved byte.
No four-gigabyte payload was allocated, and this header alone is not a complete Nimbus record.
The small worked lengths lie where both layouts agree.
This comparison reports source/definition scope rather than a coverage or repair claim.

[Era1's BlockIndex prose][era1] gives its field order but leaves widths/sign and the phrase “index's location” less explicit than SlotIndex.
The selected [`decodeEra1`][lean-era1] states its choices: SlotIndex-style eight-byte fields, signed offsets from the index-record start, and little-endian total difficulty.
Those choices must not be silently attributed to every format description.
The definition checks index positions, consecutive header numbers, internal parent/difficulty consistency, and the per-file accumulator.
These observations do not establish canonical chain membership or replace execution validation.

For beacon-era files, the format's extended verification starts from a well-known finalized checkpoint and checks state roots while traversing history.
That trust anchor remains external to ordinary framing and decompression. [Verifying era files][era]
Python checked the displayed bytes, signed offset/base arithmetic, header-only length countercase, and cutoff division.
No archive decoder, Snappy/RLP/SSZ processing, accumulator reconstruction, chain replay, checkpoint validation, EthLib binding, Lean build, or proof replay ran.

[4444]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4444.md
[7643]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7643.md
[formats]: https://github.com/eth-clients/e2store-format-specs/tree/31cf6b577123bf1703eb3713f0b242f65fea806a
[base]: https://github.com/status-im/nimbus-eth2/blob/613f4a9a50c9c4bd8568844eaffb3ac15d067e56/docs/e2store.md
[slotindex]: https://github.com/eth-clients/e2store-format-specs/blob/31cf6b577123bf1703eb3713f0b242f65fea806a/types/0x6932.md
[era1]: https://github.com/eth-clients/e2store-format-specs/blob/31cf6b577123bf1703eb3713f0b242f65fea806a/formats/era1.md
[era]: https://github.com/eth-clients/e2store-format-specs/blob/31cf6b577123bf1703eb3713f0b242f65fea806a/formats/era.md#verifying-era-files
[lean-framing]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/E2store.lean#L93
[lean-era1]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/History/Era1.lean#L257
