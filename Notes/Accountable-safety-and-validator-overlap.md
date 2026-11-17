---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
catalogue_family: N01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
paper_version: "arXiv:1710.09437v4, 2019-01-22"
paper_sha256: "000eb8877d8041a6dda1546e45e2f002f6a1a38a49ff8e58ea2a427efa97bb64"
tags:
  - ethlib
  - external-sources
  - accountable-safety
---

# Accountable safety and validator overlap

Conflicting finalization and a bound on faulty stake play different roles in accountable safety.
The theorem derives slashable overlap from conflicting finalizations.
A separate assumption that slashable stake is strictly below one third then excludes that conflict.
Neither statement guarantees progress or authenticates the votes used in the argument.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) explains some separate signature assumptions.

## Source and scope

The primary source is Buterin and Griffith's [*Casper the Friendly Finality Gadget*, arXiv 1710.09437v4][paper], revised 22 January 2019.
Reading covered §2's definitions, §2.1's accountable-safety proof and liveness distinction, and §3's membership amendment.
The PDF digest above identifies the retrieved artifact.
This is a bounded comparison with EthLib's fixed-set model at the recorded commit, not an audit of deployed Ethereum consensus.
Gasper's proofs, weak subjectivity, and full concrete execution refinements were not reviewed for this note.

The paper's §2 starts with fixed membership and counts deposit weight, not validator identities.
Its supermajority threshold includes equality at two thirds.
Justification links can skip checkpoint heights, but ordinary finalization requires a justified checkpoint's link to its direct child.
Section 3 changes membership through forward and rear validator sets, requires quorums in both, and adds evidence-inclusion conditions to finalization.
Those historical dynasty rules do not justify arbitrary replacement of the validator set in a fixed-set argument.

## The model's objects and hypotheses

EthLib's [`Checkpoint` and `Vote`][model] represent a checkpoint as a block/epoch pair and a vote as a validator plus source and target checkpoints.
An epoch is a natural number.
The block preorder expresses ancestry: `a <= b` means that `a` is in `b`'s chain.
[`Conflicting`][conflict] means neither block precedes the other.
Different epochs alone do not make two checkpoints conflicting.

For a finite validator set `V`, [`stake`][stake] sums a natural-valued weight function `w`.
The fixed-set theorem shares these objects across both finalization claims:

| Object or condition | Role in the argument |
| --- | --- |
| One `V` and one `w` | Both quorums use the same membership, weights, and total stake. |
| One genesis and ancestry relation | Both claims refer to compatible checkpoint histories. |
| One vote set | Both finalization witnesses and their conflicting vote evidence belong to the same logical view. |
| Ordered links | A source has a strictly earlier epoch and its block lies in the target block's chain. |
| Positive total stake | Makes a fractional lower bound informative. The theorem also permits zero, where its inequality is vacuous. |

This shared logical view does not require simultaneous network delivery.
It does require justifying the common objects when combining evidence from different branches or clients.
Using each branch's own weights without a comparison bound changes the problem.
Each signer is counted once per link, regardless of how many copies of its vote are received.

[`SupermajorityLink`][link] requires ordering and `2 * total <= 3 * signer_stake`.
Its target's validator set supplies the denominator.
[`JustifiedBy`][justified] starts with genesis and extends justification along a link from an already justified source.
Thus ordering and quorum alone do not make an arbitrary target justified.

[`FinalizedBy`][finalized] includes genesis and the model's more general `k`-finalization for `k >= 1`.
That definition requires ordered intermediate blocks, justified intermediate epochs, and a link from the checkpoint to epoch `e + k`.
Its `k = 1` case corresponds to the paper's direct-child pattern.
A skip link by itself is therefore not an unrestricted finalization rule.
The model also permits one block to represent several epochs, so its checkpoint representation must not be confused with the paper's checkpoint-height construction.

## The two slashable vote patterns

For the same validator, [`DoubleVote`, `SurroundVote`, and `Slashable`][slashable] express these patterns:

```text
double vote: two distinct votes, with t1.epoch = t2.epoch
surround:    s1.epoch < s2.epoch < t2.epoch < t1.epoch
```

The surround inequalities are all strict.
Sharing a source boundary or a target boundary is not enough for that predicate.
The first pattern compares target epochs, rather than requiring identical source epochs.
`Slashable` requires both votes in the view and both validator fields equal to the validator being assessed.

The proof separates two jobs.
[`FinalizedBy.slashableLinksBy`][structural] follows justification/finalization witnesses and finds two links whose common voters break one of those conditions.
The same-target-epoch case gives double votes, while the crossing case can produce a surround.
The quorum arithmetic then bounds the weight of common voters.
Under one set and weights, [`stake_inter`][overlap] turns two two-thirds quorums into overlap of at least one third.
Merely finding two large sets, without incompatible vote contents, would not establish slashability.

## Twelve equal weights

Use twelve validators, each of weight one, and two branches from genesis `g`:

```text
g(epoch 0) -> A(epoch 1) -> A2(epoch 2)
          -> B(epoch 1) -> B2(epoch 2)

Q_A = {1,2,3,4,5,6,7,8}
Q_B = {5,6,7,8,9,10,11,12}
```

Assume A and B conflict and each branch follows the displayed ancestry.
Every member of `Q_A` votes `g -> A` and `A -> A2`.
Every member of `Q_B` votes `g -> B` and `B -> B2`.
Both quorums have weight eight, exactly two thirds of twelve.
The genesis links justify A and B, and their links to the next epoch finalize them in the one-step case.

The overlap is `{5,6,7,8}`, with weight four:

```text
weight(Q_A intersect Q_B)
  = weight(Q_A) + weight(Q_B) - weight(Q_A union Q_B)
  = 8 + 8 - 12 = 4
3 * 4 = 12
```

Those four validators have distinct target-epoch-1 model votes, so each has double-vote evidence in the constructed view.
They also double-vote at epoch 2 in this constructed view.
This meets the lower bound exactly and violates the separate strictly-below-one-third fault assumption.
Requiring nine voters would silently strengthen the source's inclusive threshold.
The example is hand-derived model data, not serialized attestations, BLS signatures, or an actual network trace.
With unequal weights, four identities need not represent one third of stake.

## What the conclusion establishes

[`accountable_safety`][safety] takes two model-finalized checkpoints with conflicting blocks and concludes:

```text
total_stake <= 3 * slashable_stake
```

It assumes no bound on slashable stake.
To derive a safety corollary, add `3 * slashable_stake < total_stake`; a non-strict upper bound does not suffice.
The conclusion gives a weight bound on validators with evidence, not an executable procedure identifying all offenders or a guarantee of evidence inclusion and penalties.
It also does not imply liveness, which needs separate hypotheses about available proposals and votes.

EthLib has dynamic and slack variants in the same module.
Their bounds account for membership changes, weight changes, or weakened quorum denominators.
The fixed-set one-third conclusion cannot be carried over while discarding those loss terms.
This note leaves their detailed bounds for another contribution question.

## From model votes to attestation data

[`ffgVotes`][votes] maps a set of validator/attestation-data pairs into model votes.
[`accountable_safety_isSlashableAttestationData`][bridge] retains one validator set and weight function and assumes both model finalizations over that mapped set.
It concludes a one-third weight bound on validators with two data values satisfying `isSlashableAttestationData`.
The set named `signed` is supplied to the theorem.
Its name does not run BLS verification or prove that a validator produced a signature.

The forward implication from model slashability to the concrete data predicate works for every root-to-block map.
The exact characterization requires an injective map and includes two additional cases.
They are different attestation data describing the same link, and a surround whose inner vote does not go forward.
Do not replace this qualified bridge with an unconditional equivalence or a claim about every running client.

Review covered the selected primary passages, model definitions, fixed-set theorem and structural proof, overlap lemma, and cited attestation bridge statements.
The sources and explanation received a separate review.
An independent calculation checked the four-link example's ordering, justification, finalization, and exact double-vote set.
It also checked all 245025 pairs of eight-voter subsets of twelve validators; their minimum overlap is four.
No Lean build, proof replay, client execution, signature check, full dynamic-model review, or liveness proof audit was performed.
The arithmetic example remains a source-derived explanation, not a formal proof of deployed-system safety.

[paper]: https://arxiv.org/pdf/1710.09437v4
[model]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L72
[conflict]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L129
[stake]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Stake.lean#L27
[link]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L141
[justified]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L166
[finalized]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L186
[slashable]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/Model.lean#L102
[structural]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/AccountableSafety.lean#L194
[overlap]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Stake.lean#L172
[safety]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Models/Ffg/AccountableSafety.lean#L592
[votes]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Transition/FfgVotes.lean#L60
[bridge]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Consensus/Proofs/Transition/FfgVotes.lean#L211
