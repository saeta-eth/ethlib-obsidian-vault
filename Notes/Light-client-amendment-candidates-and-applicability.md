---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: N05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
baseline_support_commit: "834e40604ae4411e565bd6540da50b008b2496dc"
tags:
  - ethlib
  - external-sources
  - light-clients
  - proposal-assessment
---

# Light-client amendment candidates and applicability

Adding sync-committee accountability, recording historical update choices, and removing the sync committee are different proposals.
They cannot be applied as one automatic upgrade to an existing light-client implementation.
Applicability remains unresolved without a target proposal revision, fork context, and composition rule.

## Source identities and baseline

[EIP-7657][7657] and [EIP-7658][7658] are Stagnant; [EIP-8390][8390] is Draft at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
None of their headers declares a `requires` list.
That absence does not remove the protocol dependencies described in their text.
Reading covered all three proposal texts, including their code, compatibility, and security sections.
Linked older specification snapshots, proof-system implementations, deployment claims, and economic forecasts were not verified.

The accompanying [baseline note](Light-client-trusted-roots-and-update-boundaries.md) uses selected Altair text at consensus-specs commit `834e40604ae4411e565bd6540da50b008b2496dc` as peer-specification support/provenance.
Its bootstrap begins with a trusted block root and authenticates the current sync committee before processing updates.
EIP-7657 instead links supporting snapshot `b3e83f6691c61e5b35136000146015653b22ed38`; EIP-7658 links `4afe39822c9ad9747e0f5635cca117c18441ec1b`.
Their inherited containers, branches, and generalized indices must not be assumed identical to every later fork.
EIP-7657 selects the evidence-signature fork version at `signature_slot`, while the accompanying baseline uses `max(signature_slot, 1)-1`.
Fork-boundary composition must resolve that difference rather than silently inherit either rule.
This note neither modifies that baseline nor claims EthLib implementation or proof coverage.

## Three distinct changes

| Candidate | Proposed change | Applicability conditions |
| --- | --- | --- |
| EIP-7657 | Add sync-committee slashing evidence and consensus validation. | Applicable evidence shapes, fork-specific branches, and supporting protection/inclusion infrastructure must be selected. |
| EIP-7658 | Track a period's best sync data in BeaconState for historical backfill. | The backfill exchange and its proofs' connection to an accepted historical state must be specified. |
| EIP-8390 | Remove sync committees and the Altair light-client protocol, relying on an external proof-based replacement. | The replacement, transition conditions, and consumer migration require definition and assessment. |

Status alone selects none of these paths and establishes no activation date.
The first two retain the mechanism that the third proposes removing.
Their relationships therefore need explicit fork-composition decisions.

## Accountability requires the proposed evidence conditions

EIP-7657 introduces two pieces of `SyncCommitteeSlashingEvidence`, participant information, and a recent finalized reference.
Its checks include ordered signature/attested/finalized slots and no future evidence.
The first evidence's attested period must be at most one period after the second evidence's attested period.
The candidate violations include conflicting attested headers at one slot, incompatible finalized next committees under specified conditions, and non-linear finalized histories.
Listed validators must be known, ordered, and participants in both signatures; at least one must still be slashable.
Merkle evidence and aggregate signatures are validated before applying slashings. [State transition checks][7657]

Two different root strings alone do not satisfy that procedure.
The text deliberately avoids punishing a validator solely for consistently following an incorrect checkpoint history.
It also acknowledges that penalties are retrospective and do not undo an already completed attack.
Its motivation's illustrative committee-balance budget is not a universal bound for every later balance regime.

The proposal lists slashing-protection databases, remote-signer APIs, evidence gossip, slashers, and Beacon APIs as separately introducible infrastructure.
Tests, reference implementation, and security sections remain TBD in this snapshot.
The [key-management note](Key-management-and-remote-signing-obligations.md) explains why an existing import or signing response cannot establish support for new slashing obligations.

## A conditional backfill-ranking example

EIP-7658 adds `previous_best_sync_data`, `current_best_sync_data`, and `parent_block_has_sync_committee_finality` to BeaconState.
Its `SyncData` holds a sync aggregate and signature slot.
At period rollover, current data moves to previous data and the current tracking state resets.
Block processing tracks the candidate before overwriting the latest block header.
This changes consensus state processing, rather than merely adding a historical-query endpoint. [Containers and transition][7658]

For the ranking fragment, assume 512 participation bits and that attested/signature periods match.
Assume the earlier finality-tracking stage produces the stated old/new booleans.
Use old signature slot 100 and new slot 101 as constructed comparator inputs, without asserting a complete reachable mainnet state.
Supermajority uses `participants*3 >= 512*2`: 341 fails, while 342 passes.

| Old/new participants | Old/new committee-finality flag | Select new? | Deciding rule |
| --- | --- | --- | --- |
| 400 / 342 | False / True | Yes. | Both have supermajority; new committee finality wins before participation count. |
| 340 / 341 | False / False | Yes. | Without supermajority, higher participation wins. |
| 342 / 341 | False / False | No. | Retain the supermajority candidate. |
| 400 / 400 | False / False | No. | Equal ranking retains the earlier signature slot 100. |

This is not a “largest aggregate always wins” rule.
The rationale requires lower `signature_slot` for ties with MUST, and retaining the existing selection for equal slots with SHOULD.
The displayed code also retains the existing selection on equality.
These booleans are inputs/results of proposal processing, not independently verified finality evidence.

The proposal describes this as a subset of update-ranking logic.
It does not replace the baseline store's complete `is_better_update` procedure with this smaller comparator.
Its rationale sketches a backfill protocol using later finalized state, historical block roots, bootstraps, and proof of the selected aggregate/slot.
No complete exchange implementation or proof vector is established by the four arithmetic cases.

## Removal has unresolved replacement and transition questions

EIP-8390 proposes deleting sync-committee fields, `SyncAggregate`, Altair light-client containers/helpers, duties, networking methods, and Beacon API endpoints.
It also retires the corresponding domain types, which implementations MUST NOT reuse under its proposed rules.
Pre-fork blocks and states retain their own decoding schemas; the proposal does not preserve Altair update processing after the transition. [Specification and compatibility][8390]

Under its explicit assumption that reward weight 2 is removed without redistribution and denominator 64 remains, the nominal removed share is `2/64 = 1/32`.
That fraction is not a measured network issuance change, staking-yield forecast, or proof-production benchmark.
The motivation's proving-speed and security assertions were not independently established here.
The draft explicitly leaves replacement proving infrastructure unspecified and requires it to exist before activation.
The [accountable-safety note](Accountable-safety-and-validator-overlap.md) records model hypotheses that cannot be replaced by an unqualified claim about a proof system.

The draft defines `EIP8390_FORK_EPOCH` as an activation epoch, but Networking and Fork transition say “slots at or after” that parameter.
This unit mismatch remains unresolved here; the note chooses neither direct comparison nor an inferred conversion.
The removal text also supplies no composition rule for EIP-7658's added SyncData fields or EIP-7657's evidence using types it removes.
Combining them requires additional transition/schema decisions, not simply concatenating their changes.

## Assessment and limits

The applicability map distinguishes the versioned baseline, candidate change, inherited assumptions, and unresolved consumer obligations.
Applying a proposal requires a chosen revision, reconciled activation units, exact inherited schemas, and a stated trust/security model.
This comparison specifies no replacement deployment or migration procedure.

Python checked the four conditional ranking results, 341/342 boundary, earlier/equal-slot controls, and exact `2/64` fraction.
No BLS signature, Merkle branch, BeaconState transition, backfill client, ZK proof system, deployment benchmark, EthLib binding, Lean build, or proof replay ran.

[7657]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7657.md
[7658]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7658.md
[8390]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8390.md
