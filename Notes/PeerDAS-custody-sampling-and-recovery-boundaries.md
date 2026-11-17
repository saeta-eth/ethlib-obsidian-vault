---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 4
delivery_batch: "follow-up-15"
catalogue_family: N06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
consensus_support_commit: "9d377fd53d029536e57cfda1a4d2c700c59f86bf"
tags:
  - ethlib
  - external-sources
  - peerdas
  - data-availability
---

# PeerDAS custody, sampling and recovery boundaries

An authenticated cell is a statement about data at one index under a commitment.
It does not establish that enough other cells can be obtained, or that a particular peer will serve them.
Custody, sampling, recovery and cryptographic binding therefore need separate conditions.

## Versions and source roles

[EIP-4844][4844] and [EIP-7594][7594] are Final; [EIP-8070][8070] is Last Call at the recorded EIPs commit.
The latter's passed deadline does not change its recorded status or establish deployment.
EIP-7594 requires EIP-4844 and links detailed Fulu text at consensus-specs `9d377fd53d029536e57cfda1a4d2c700c59f86bf`.
EIP-4844's cryptographic-helper reference instead uses `86fb82b221474cc89387fa6436806507b3849d88`; these are distinct source snapshots.

Reading covered EIP-4844's relevant parameters/types, EIP-7594's specification/networking and selected security discussion, and EIP-8070's sparse-blobpool rules and limitations.
Selected linked Fulu custody, column, sampling, verification and recovery passages were read, including inherited hash/byte helpers.
The EIP is the citation for its rules; linked consensus text records detail/provenance under EthLib's peer-specification policy, not independent code authority.
This note selects a bounded Fulu baseline, without resolving every frozen-reference versus later-implementation question discussed in the [KZG note](KZG-openings-and-setup-assumptions.md).

## From a blob to a column

The selected parameters give a blob 4096 field elements of 32 bytes: 131,072 bytes.
Reed–Solomon extension doubles its evaluations to 8192 elements, arranged as 128 cells of 64 elements, or 2048 bytes per cell.
Each blob contributes one matrix row.
A column at index j contains cell j from every blob in the block, with one corresponding commitment/proof per row. [Cells][cells]

A cell proof checks an opening at that index against its blob commitment.
The column sidecar also carries a proof that its commitment list belongs to the signed block's body.
Those are different bindings, and header/chain validation adds further conditions.
The [Fulu gossip rules][p2p] require the listed validations before forwarding, with distinct Reject/Ignore outcomes.
For by-root responses, the reader SHOULD check sidecar shape, commitment inclusion and KZG proofs before consuming the next chunk.
An advertised custody set or response header does not replace these checks.
The by-root response type and successful-chunk rule name `DataColumnSidecar`, while one prose sentence instead names `DataColumnIdentifier`; this source inconsistency remains unresolved.

EIP-7594 also changes the blob transaction's network wrapper: version 1 carries proofs for all 128 cells per blob, including extension cells, while transmitting the unextended blobs.
It requires count, versioned-hash and blob/commitment/cell-proof consistency checks.
This transport object is distinct from the execution transaction envelope and the consensus column sidecar.

## A complete deterministic custody example

The [custody core][das] uses 128 groups and 128 columns, so each group contains exactly its same-numbered column.
The minimum ordinary custody count is four; the validator minimum is eight and can increase with attached balance.
Custody includes serving data, backfilling when syncing and retaining it for the pruning period.
The core sampling count is `max(8, custody_group_count * columns_per_group)`.
Its deterministic selection extends the custody set rather than replacing it.

Construct node ID zero and choose four custody groups.
Serialize each candidate uint256 ID in 32 little-endian bytes, hash with SHA-256, interpret the first eight digest bytes little-endian and reduce modulo 128.
Increment the candidate ID, discard repeated groups, and sort after collecting the requested count.
For candidate zero:

```text
input = 0000000000000000000000000000000000000000000000000000000000000000
SHA256 = 66687aadf862bd776c8fc18b8e9f8e20089714856ee233b3902a591d0d5f2925
first8 little-endian integer = 8628161281313630310
group = 8628161281313630310 mod 128 = 102
```

The complete first eight candidate results are:

| Candidate ID | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Group | 102 | 1 | 87 | 17 | 75 | 42 | 19 | 117 |

They are distinct, so custody is `[1,17,87,102]` and the core's minimum eight-column sample set is `[1,17,19,42,75,87,102,117]`.
This is an author-derived hash/count control, not a published node identity or network observation.
It demonstrates deterministic assignment, not an independent-random-sampling security argument.
The complementary [peer-sampling text][sampling] separately recommends locally random selection without replacement, diverse responsive peers and randomized peer choice.
Its illustrative allowed-failures table assumes 16 samples; it must not be imported as the eight-sample baseline's parameters.

## Recovery needs distinct cells of the same row

For one block's three blob rows, one complete column contains `3*2048 = 6144` cell-data bytes.
The eight selected columns contain 49,152 cell-data bytes, excluding proofs, commitments, headers and transport overhead.
That is one eighth of the unextended blob bytes but one sixteenth of the extended matrix bytes.
Retrieving these eight columns satisfies the chosen core sampling set; it is far below the 64-column reconstruction threshold.

The [recovery interface][recovery] requires between 64 and 128 distinct, in-range cell indices for each blob, equal index/cell counts and correctly encoded cells.
Sixty-three distinct indices plus a duplicate are insufficient even if 64 records arrived.
Cells from different blob rows cannot be pooled toward one row's threshold.
With 64 complete authenticated columns of the same block, every row has 64 distinct cells; reconstruction is conditional on their belonging to the same valid low-degree codeword per row.
No recovery runs in this example.

The recovery function takes cells and indices, not their original proofs: authentication belongs to the caller's preceding checks.
Position binding prevents different accepted contents at one position, while code binding concerns whether accepted cells fit one polynomial of degree below 4096.
That stronger global condition, setup assumptions and the verification model cannot be established by counting samples.
Likewise, successful local retrieval does not establish global availability against correlated withholding or an eclipsed peer set.
The EIP's network-probability table was not independently reproduced here.

## Sparse blobpool is a separate proposal

EIP-8070 aligns proposed execution-layer sampling with consensus custody, with provider probability 0.15 and sampler behavior otherwise.
Its `8/64` payload fraction refers to unextended blob data, not half of the 128-column extension.
The proposed eth/72 transaction response MUST encode the omitted `blobs` as the RLP empty list `c0`, not the empty byte string `80`.
Its sampler SHOULD observe at least two distinct provider announcements and receive a validated signed transaction before proceeding.
When fetching from a provider, it MUST request extra random columns.
Responders MAY truncate cell responses.
The snapshot leaves announcement-mask expressiveness limits and local disconnection heuristics explicit, with no normative peer-scoring algorithm.
These are versioned proposal obligations, not measurements or a guarantee that the sampled data will arrive.

## Selected EthLib connections and limits

At the recorded commit, [`getCustodyGroups`][lean-custody] exposes a fuel bound for the otherwise unbounded search, and [`verifyDataColumnSidecarKzgProofs`][lean-column] checks cells at the column's shared index.
[`recoverCellsAndKzgProofs`][lean-recovery] additionally requires strictly ascending indices, whereas the selected linked public routine explicitly requires uniqueness without that ordering restriction.
[`CodeBindingWins`][lean-game] records distinct accepted openings with no common degree-bounded polynomial as a security-game winning condition.
These selected definitions do not establish network security or a complete sampling proof.

Python checked the eight SHA-256 assignments, sorted subset relation, byte counts, distinct-index threshold and denominator arithmetic.
No cell proof, setup, Merkle inclusion, polynomial recovery, probability bound, peer response, blobpool, EthLib binding, Lean build or proof replay ran.

[4844]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4844.md
[7594]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7594.md
[8070]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8070.md
[cells]: https://github.com/ethereum/consensus-specs/blob/9d377fd53d029536e57cfda1a4d2c700c59f86bf/specs/fulu/polynomial-commitments-sampling.md#cells
[das]: https://github.com/ethereum/consensus-specs/blob/9d377fd53d029536e57cfda1a4d2c700c59f86bf/specs/fulu/das-core.md
[sampling]: https://github.com/ethereum/consensus-specs/blob/9d377fd53d029536e57cfda1a4d2c700c59f86bf/specs/fulu/peer-sampling.md#sample-selection
[p2p]: https://github.com/ethereum/consensus-specs/blob/9d377fd53d029536e57cfda1a4d2c700c59f86bf/specs/fulu/p2p-interface.md
[recovery]: https://github.com/ethereum/consensus-specs/blob/9d377fd53d029536e57cfda1a4d2c700c59f86bf/specs/fulu/polynomial-commitments-sampling.md#recover_cells_and_kzg_proofs
[lean-custody]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/DataAvailability/Custody.lean#L87
[lean-column]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/DataAvailability/Column.lean#L176
[lean-recovery]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Kzg/Recovery.lean#L138
[lean-game]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L434
