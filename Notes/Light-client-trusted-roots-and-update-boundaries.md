---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: N04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
consensus_specs_commit: "834e40604ae4411e565bd6540da50b008b2496dc"
beacon_api_commit: "6bc8c7bde19d0499f88e67417419eb12331438f4"
tags:
  - ethlib
  - external-sources
  - light-client
  - trust
---

# Light-client trusted roots and update boundaries

A bootstrap proves a committee against a block root the client already trusts.
It does not establish that root's trustworthiness.
Later signatures, Merkle branches and participation thresholds justify different checks; they must not be collapsed into “the update verifies.”

## Source roles and bounded baseline

This note reads the [Altair light-client sync protocol][sync] at consensus-specs commit `834e40604ae4411e565bd6540da50b008b2496dc`.
That repository is a peer specification under EthLib's contribution policy, so it provides supporting algorithm text and provenance, not independent normative authority for inherited EthLib rules.
[EIP-7251][7251] links this repository revision for its Electra changes; that link fixes a convenient snapshot, not EIP-7251 authority over Altair light clients.

Reading covered the Altair containers, branch/period/safety helpers, bootstrap, validation, application, normal processing and forced-update algorithms, plus selected full-node bootstrap and preset/helper passages.
The [Beacon API][api] is separately fixed at `6bc8c7bde19d0499f88e67417419eb12331438f4`, whose OpenAPI label is `Dev - Ethereum Proof-of-Stake Consensus Specification v1.7.0-beta.3`.
Selected light-client endpoint and Altair schemas were read; their embedded consensus links point to v1.3.0, a different source version.
Neither snapshot establishes network activation or compatibility across every later fork.
The example uses Altair-form objects and mainnet preset numbers under a hypothetical fork schedule, not historical mainnet slots.

## Bootstrap binds a committee to an external anchor

`initialize_light_client_store(trusted_block_root, bootstrap)` first checks the light-client header and requires `hash_tree_root(bootstrap.header.beacon) == trusted_block_root`.
It then verifies the current committee's hash-tree root against the header's state root.
In this Altair layout, the current committee has generalized index 54 and a five-node branch.
A valid branch establishes inclusion in that committed state, conditional on the root and hash assumptions; it does not independently choose the correct chain.

Initialization puts the supplied header into both `finalized_header` and `optimistic_header`, installs the current committee, and leaves the next committee unknown.
Those assignments do not authenticate the caller's anchor or establish its finality.
Acquiring an appropriate trusted root, chain configuration and genesis-validators root remains a caller responsibility outside this calculation.

The [bootstrap API][bootstrap] takes a beacon block root, not a state root, and returns a versioned bootstrap as JSON or SSZ.
It says servers SHOULD provide results as defined by the linked full-node helper and need the block's post-state.
Its 404 means a bootstrap cannot be produced for that root, not a cryptographic absence proof.
The [API evidence note](Beacon-API-identifiers-and-response-evidence.md) explains why a response or selector alone supplies no trust anchor.

## Validate the update before changing the store

The baseline validation requires at least one participating sync-committee position, followed by the slot ordering:

```text
current_slot >= signature_slot > attested_slot >= finalized_slot
```

`current_slot` comes from the client's local clock.
The advertised signature slot is untrusted input constrained by these checks.
If the next committee is unknown, the signature period must equal the store's finalized-header period.
If it is known, the signature period may also be the immediately following period; it cannot skip further.
The update must additionally supply a newer attested header or the relevant missing next committee.

A present finality branch binds the finalized header's root to the attested state; a present committee branch binds the next committee there.
Absent branches require the corresponding default-zero objects.
Genesis finality has a special zero-root convention, not the ordinary hash of an arbitrary slot-zero header.
When a committee branch is present, already-known next-committee data must agree if the attested period is the store period.

The signature uses committee keys selected by the participation bits, taking the current or known-next committee according to the signature period.
Its domain uses `DOMAIN_SYNC_COMMITTEE`, the fork version at `max(signature_slot, 1)-1`, and the supplied genesis-validators root.
The signed object is the attested beacon header.
A BLS check under unrelated keys or a different chain domain cannot replace these bindings; see the [BLS preconditions note](BLS-ciphersuites-and-verification-preconditions.md).
These checks also do not execute all consensus or execution transitions.

## Optimistic advance, normal finality and committee rotation

After validation, normal processing tracks the maximum participation observed in the current period.
Its safety threshold is `floor(max(previous_max, current_max)/2)`.
A newer attested header advances the optimistic field only when participation is strictly greater than that threshold.
This is not the two-thirds rule.

Normal processing applies an update only with `3*participants >= 2*committee_size` plus either a newer finalized header or the specified opportunity to learn a finalized next committee.
That learning case requires committee and finality branches with finalized and attested headers in the same period while the next committee is unknown.
With the next committee already known, finalized progress into the next period makes that committee current and installs the update's next committee.
Using next-period keys for a signature does not by itself rotate the store.
Participation counts committee bit positions, not stake-weighted finality votes or an asserted number of distinct validators.

## A complete conditional threshold and period exercise

Use the selected [Altair][altair] and [phase0][phase0] preset numbers: 512 positions, 32 slots per epoch and 256 epochs per committee period.
One period has 8192 slots.
Start every row independently from finalized/optimistic slot 8000, current committee C0, known-next C1, previous maximum 512 and current maximum 400.
The starting store is stipulated; its reachability from bootstrap is not established.
Take attested slot 8191, finalized slot 8100, signature slot 8192 and local slot 8193.
Assume the applicability, relevance, header, branch, committee and BLS checks all succeed; no corresponding block or signature is supplied.
The signature is in period 1 and therefore uses C1, while attested/finalized headers remain in period 0.
Updating the participation maximum leaves the safety threshold at 256 in every row.

| Participants | Strictly above 256? | `3*p >= 1024`? | Optimistic slot after normal processing | Finalized slot after normal processing |
| --- | --- | --- | --- | --- |
| 256 | No | No | 8000 | 8000 |
| 257 | Yes | No | 8191 | 8000 |
| 341 | Yes | No | 8191 | 8000 |
| 342 | Yes | Yes | 8191 | 8100 |

If C1 were unknown, this same update would fail the signature-period check regardless of participation.
In the 342-position case, finalized slot 8100 is still in period 0, so C0/C1 remain installed.
Now conditionally supply a valid follow-up with finalized 8192, attested 8193, signature 8194 and local slot 8195, all 512 positions participating, and a valid next-committee proof for C2.
Normal application rotates current/next from C0/C1 to C1/C2.
These are constructed arithmetic cases, not a complete bootstrap, published vector or observed chain.

## Forced updates and limits

The baseline says force-update processing MAY be called based on use-case-dependent heuristics.
It applies a stored best update when the local slot is strictly beyond the stored finalized slot plus the update timeout.
If that update's finalized header is not newer, its attested header replaces `finalized_header` before application, without the same finality evidence as the normal path.
A consumer must distinguish that progression policy from consensus finality; the field name alone is insufficient.
No force update occurs in the example.

At the recorded EthLib commit, [`initializeLightClientStore`][lean-init], [`validateLightClientUpdate`][lean-validate], and [`processLightClientUpdate`][lean-process] expose the anchor, signature-check input, clock, genesis root and thresholds as selected definitions.
[`processLightClientStoreForceUpdate`][lean-force] separately implements the timeout path.
Their fork-aware layout checks cover a broader scope than the Altair example reviewed here; no whole-module correctness claim follows from these observations.

Python checked slot order, period division, all four threshold rows, the unknown-next rejection condition and the stated rotation arithmetic.
No trusted-root acquisition, Merkle or BLS verification, SSZ root, bootstrap/update fixture, consensus/execution transition, RPC, network, EthLib binding, Lean build or proof replay ran.

[sync]: https://github.com/ethereum/consensus-specs/blob/834e40604ae4411e565bd6540da50b008b2496dc/specs/altair/light-client/sync-protocol.md
[api]: https://github.com/ethereum/beacon-APIs/tree/6bc8c7bde19d0499f88e67417419eb12331438f4
[bootstrap]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/apis/beacon/light_client/bootstrap.yaml
[lean-init]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/LightClient/Sync.lean#L351
[lean-validate]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/LightClient/Sync.lean#L454
[lean-process]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/LightClient/Sync.lean#L554
[lean-force]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/LightClient/Sync.lean#L529
[7251]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7251.md#L580
[altair]: https://github.com/ethereum/consensus-specs/blob/834e40604ae4411e565bd6540da50b008b2496dc/specs/altair/beacon-chain.md#sync-committee
[phase0]: https://github.com/ethereum/consensus-specs/blob/834e40604ae4411e565bd6540da50b008b2496dc/specs/phase0/beacon-chain.md#time-parameters
