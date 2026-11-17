---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: N02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
consensus_specs_commit: "834e40604ae4411e565bd6540da50b008b2496dc"
shuffle_paper_version: "arXiv:1208.1176v2"
tags:
  - ethlib
  - external-sources
  - consensus
  - validators
---

# Validator weight, shuffling and committee boundaries

A permutation preserves validator positions, a committee divides those positions, and a proposer filter uses effective balances.
These are separate claims with different inputs and evidence.
A correct permutation alone establishes neither unpredictable assignments nor stake-proportional proposer outcomes.
This note traces one constructed shuffle and two balance-filter decisions without treating them as a complete epoch transition.

## Sources and scope

[EIP-7251][7251], [EIP-7549][7549] and [EIP-7514][7514] are Final at the pinned EIPs snapshot.
Status does not specify the activation configuration of a particular chain.
Reading covered EIP-7251's consensus constants, consensus-layer sketch, selected rationale and committee/proposer/churn security discussion, plus the complete EIP-7549 and EIP-7514 texts.
The consolidation contract and its bytecode were outside this reading.

The primary shuffle construction is Hoang, Morris and Rogaway's [author-linked arXiv version 1208.1176v2][paper], specifically the generalized-domain construction, its ideal-random-components setup, Theorem 5's statement and §4's security interpretation.
This version is identified separately from the catalogue's CRYPTO 2012 DOI edition; their numerical bounds and theorem numbering must not be substituted silently.
The proof derivations were not reviewed completely.

EIP-7251 links the exact [consensus-specs commit used here][support].
Selected [phase0 helpers][phase0] and the [Electra proposer modification][electra] provide supporting executable/provenance text under [EthLib's source policy][policy].
They are peer specifications, rather than independent primary authority for every inherited rule.
Later proposer changes are outside the worked algorithm.

## Why a shuffle remains a permutation

In the paper's generalized construction, a round pairs position `x` with `p - x` in the finite group, here addition modulo `n`.
Both members use the same bit because its input represents the unordered pair, using `max(x, flip)`.
The round either swaps both positions or leaves both in place.
Applying that round again restores each position; reversing the round order inverts their composition.
This reasoning needs no claim that the bits are random.

The paper's security analysis instead assumes uniformly chosen round keys and functions; §4 discusses a pseudorandom-function construction.
Its security notion concerns oracle access to a keyed permutation and its inverse.
That result does not by itself establish Ethereum seed unbiasability, fair outcomes against adaptive validators, or a numerical security bound for a public SHA-256-derived instance.

## A complete helper-level assignment

Choose `n = 10`, 90 rounds, seed `00` repeated 32 times, and the distinct validator-index sequence `[10,11,12,13,14,15,16,17,18,19]`.
These are constructed helper inputs, not a published consensus test vector or a complete beacon-state fixture.
The supporting helper hashes `seed || round_byte` for its pivot, reading eight little-endian bytes modulo ten.
For the swap bit it hashes the seed, round byte, and four-byte little-endian encoding of `position // 256`.
Bits are read least-significant first within each byte.

For round zero, the pivot hash is:

```text
7f9c9e31ac8256ca2f258583df262dbc7d6f68f2a03043d5c99a4ae5a7396ce9
```

The pivot is 5.
The bucket-zero hash is:

```text
ab24a95f44ceca5d2aed4b6d056adddd8539f44c6cd6ca506534e830c82ea8a8
```

Positions 0 and 5 both read position 5's bit, which is 1, so they swap.
Running all 90 rounds gives the following computed positions and the validator indices read at those positions:

```text
positions = [9,7,4,1,8,0,5,6,3,2]
indices   = [19,17,14,11,18,10,15,16,13,12]
```

The committee helper with count 3 takes half-open bounds `floor(10*j/3)` through `floor(10*(j+1)/3)`:

| Committee j | Positions | Validator indices |
| --- | --- | --- |
| 0 | [0,3) | [19,17,14]. |
| 1 | [3,6) | [11,18,10]. |
| 2 | [6,10) | [15,16,13,12]. |

Concatenation preserves all ten entries exactly once, with sizes 3, 3 and 4.
This three-slice exercise does not satisfy or claim a complete beacon-epoch committee configuration.
Changing a validator's effective balance does not create repeated positions in this partition.

## Proposer filtering uses a different obligation

EIP-7251 preserves the 32 ETH activation minimum and introduces a 2048 ETH maximum for the compounding profile.
The supporting `get_max_effective_balance` distinguishes compounding credentials from the profile capped at the minimum.
For this example, assume appropriate compounding credentials wherever a balance exceeds 32 ETH.
Give indices 10 through 19 effective balances `[32,64,96,128,256,512,1024,2048,2000,32]` ETH.
Use the same zero seed directly as the proposer-helper seed for this constructed case.

The Electra helper cycles through shuffled positions and accepts candidate balance `b` when `b * 65535 >= (2048 * 10^9) * r`, with `b` in Gwei.
For draw `i`, `r` is the little-endian 16-bit value at byte offset `2*(i mod 16)` of `SHA256(seed || LE64(i // 16))`.
Here `LE64` encodes the counter in eight little-endian bytes.
For both initial draws that digest is:

```text
2c34ce1df23b838c5abf2a7f6437cca3d3067ed509ff25f11df6b11b582b51eb
```

| i | Candidate | Balance, Gwei | Selected bytes → r | b * 65535 | Maximum * r | Decision |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 19 | 32000000000 | 2c34 → 13356 | 2097120000000000 | 27353088000000000 | Reject; continue. |
| 1 | 17 | 2048000000000 | ce1d → 7630 | 134215680000000000 | 15626240000000000 | Accept; return 17. |

The inclusive comparison matters: a 32 ETH candidate accepts values 0 through 1023, while a maximum-balance candidate accepts every 16-bit value.
These integer decisions do not prove a complete proposer-output distribution or a bound on search duration.
The committee partition reads positions without a balance filter; the proposer path also applies the balance comparison.

## Seeds, lifecycle and signed committee information

The supporting active-set predicate is `activation_epoch <= epoch < exit_epoch`.
Its epoch seed hashes the duty domain, eight little-endian epoch bytes and a historical RANDAO mix selected using `MIN_SEED_LOOKAHEAD`.
The proposer path additionally hashes the epoch seed with the slot.
Our direct zero helper seed establishes no relationship to such a state-derived seed and no resistance to RANDAO manipulation.

EIP-7514 caps activation churn at eight: a previously computed churn limit of twelve becomes eight for activation, without imposing that same cap on exits.
EIP-7251 subsequently describes balance-based churn and explicitly queued top-ups; the earlier count cap must not be applied indiscriminately to that accounting.
EIP-7549 fixes the signed `AttestationData.index` to zero and puts committee information in outer `committee_bits`.
It does not erase membership validation: its mutation discussion conditions avoiding first-seen-cache pollution on checking committee membership before registration.
Neither an outer bitfield nor a [BLS aggregate signature](BLS-ciphersuites-and-verification-preconditions.md) alone supplies that membership evidence.

## Selected EthLib connections and limits

At the pinned EthLib commit, [Shuffle.lean][lean-shuffle] defines the pair-bit shuffle and committee slicing; [Accessors.lean][lean-access] defines seeds, active-duty access and fork-dependent balance filtering.
[ShufflePermutation.lean][lean-permutation] states `shuffledPositions_perm` for `count < 2^64`.
Its [`computeCommittee_partition` theorem][lean-proof] assumes positive committee count, `indices.size * count < 2^64`, and successful computation of every committee, concluding that their concatenation is a permutation of the input list.
The selected proofs were read; they were not replayed here and do not assert cryptographic pseudorandomness.
[Proposers.lean][lean-proposers] also distinguishes later proposer algorithms, so the Electra trace is not a description of every fork in that code.

A standalone Python SHA-256/integer calculation reproduced the displayed constructed shuffle, slices and filter products.
No beacon transition, EthLib binding, Lean build, proof check, live duty query or fairness experiment ran.
Transferring a security statement requires a selected fork and preset, established active-set and seed conditions, and the required probabilistic model.

[7251]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7251.md
[7549]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7549.md
[7514]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7514.md
[paper]: https://arxiv.org/pdf/1208.1176v2
[support]: https://github.com/ethereum/consensus-specs/tree/834e40604ae4411e565bd6540da50b008b2496dc
[phase0]: https://github.com/ethereum/consensus-specs/blob/834e40604ae4411e565bd6540da50b008b2496dc/specs/phase0/beacon-chain.md#compute_shuffled_index
[electra]: https://github.com/ethereum/consensus-specs/blob/834e40604ae4411e565bd6540da50b008b2496dc/specs/electra/beacon-chain.md#modified-compute_proposer_index
[policy]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/CONTRIBUTING.md#sources-of-truth
[lean-shuffle]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Shuffle.lean#L59-L196
[lean-access]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Accessors.lean#L275-L482
[lean-permutation]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proofs/ShufflePermutation.lean#L136-L168
[lean-proof]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proofs/ShufflePermutation.lean#L356-L390
[lean-proposers]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proposers.lean#L39-L50
