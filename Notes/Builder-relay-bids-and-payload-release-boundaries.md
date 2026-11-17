---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: I03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
builder_specs_commit: "fce64468a81a87892d7cb9b4858272bb3d010e50"
relay_specs_commit: "a4e0a50d9d46bfe881fd74cc7e0db366371f4eed"
tags:
  - ethlib
  - external-sources
  - builder-api
  - relay-api
---

# Builder/relay bids and payload-release boundaries

A signed bid states an offer under a key; it does not itself transfer the offered amount.
A proposer then commits to a blinded block before receiving its full execution payload.
This ordering creates a trust and availability boundary that signature verification alone cannot remove.

## Versions and reading scope

This note fixes the official [Builder API][builder] and [Relay API][relay] repositories at the frontmatter commits.
Both OpenAPI files identify their version as `dev`.
The Builder API's Bellatrix Honest Validator document additionally labels itself work-in-progress.
Reading covered Bellatrix containers, registration/signing/bidding/release clauses, validator bid/fallback rules, selected endpoint YAML, and referenced registration/primitive/payload schemas.
It did not review complete later-fork actor rules or relay implementations.

The builder repository's Beacon API gitlink is `01e99a6530b22941f619bde525cbf8bb61991d42`; referenced schemas were read at that revision.
The relay repository independently pins Beacon API `339eea96b41c787dad47765fc781303fb40aa886` and builder-specs `01487014ea047d2ed570920fb2284470f0b5dd30`.
Selected relay schema dependencies were read at its own Beacon revision; its older builder gitlink was recorded, not substituted for the separately selected builder source.

The worked flow uses Bellatrix's external-builder interface.
At the selected builder commit, the registration and header endpoints are deprecated from Gloas, and the blinded-block endpoint is marked deprecated with replacement text.
These source labels do not establish fork activation or a deployed relay's capabilities.
This note does not equate external relays with enshrined proposer-builder separation proposals such as EIP-7732 or EIP-7805.

## Three signatures have different objects

A [ValidatorRegistrationV1][builder-bellatrix] contains `fee_recipient`, preferred `gas_limit`, `timestamp`, and the validator's BLS `pubkey`.
Its wrapping signature is by that validator.
It supplies building preferences, not a bid or permission to sign a beacon block.
The registration endpoint says a valid registration must be integrated into builder state; invalid registration receives 400.

A Bellatrix `BuilderBid` instead contains an execution payload header, `value: uint256`, and a builder public key.
Its [schema][bid-schema] describes `value` as the payment in wei to the fee recipient.
The signature binds this bid object, including its stated value; checking that signature does not execute the payment transaction or establish the signer's operational trustworthiness.
The [builder signing section][builder-signing] says registrations and bids should use `compute_signing_root(message, domain)` with:

```text
domain = compute_domain(DOMAIN_APPLICATION_BUILDER,
                        fork_version=None, genesis_validators_root=None)
DOMAIN_APPLICATION_BUILDER = 0x00000001
```

That four-byte constant is a domain type, not the entire signing domain.
Do not sign the JSON text or substitute the raw concatenated serialization below for the specified signing root.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) separates a valid signature equation from key and application preconditions.

A `SignedBlindedBeaconBlock` carries the proposer's signature under the beacon-proposal domain for its state and epoch, not the builder application domain.
Its body holds the selected payload header in place of the transactions-bearing payload.
The builder must check consensus validity and the expected proposer's signature before revealing the payload. [Blinded block processing][release]

## From builder submission to payload release

The Relay API separately lets a builder submit a full payload and a signed `BidTrace` through `POST /relay/v1/builder/blocks`.
The trace names the slot, parent/block hashes, builder/proposer keys, proposer fee recipient, gas limit/used, and value.
This is not the same object as a `BuilderBid`.
The [submission endpoint][submit] says the relay will simulate the block and verify proposer payment in a builder-to-proposer payment transaction at the block's end.
It describes the builder signature as covering the SSZ-encoded `message` and publishes that message without the transactions for accountability.
These selected relay clauses do not supply a complete independent signing-domain recipe; this note does not silently import the BuilderBid recipe for BidTrace.

For the proposer-facing path, the relay follows the Builder API, according to its README.
The Bellatrix flow is:

| Step | Object and conditional obligation |
| --- | --- |
| Register | P sends its signed preferences through `POST /eth/v1/builder/validators`. |
| Request | P requests `/eth/v1/builder/header/{slot}/{parent_hash}/{pubkey}` for its slot, execution parent and validator key. |
| Offer | A 200 response carries a versioned signed bid; inability to produce a valid header requires 204, while an invalid request requires 400. |
| Commit | P selects at most one bid for that slot and submits the resulting signed blinded block through `POST /eth/v1/builder/blinded_blocks`. |
| Reveal | On a valid signed blinded block, the builder reveals the payload; v1 success returns that payload. |

The [validator document][validator] requires local building if an external bid is not timely.
It says the local task should begin in parallel, and the other process should be cancelled once either proposal signature is produced.
Once P publishes the signed blinded block, it MUST NOT use the local build process as a fallback even if the external path fails.
This boundary prevents a timeout after commitment from being treated like a timeout before signing.
The returned payload still needs its corresponding header and other applicable validity checks; an HTTP success is not a consensus proof.
The v2 endpoint has a different contract: success is 202 and the builder MUST publish the unblinded block, rather than returning the v1 payload response. [v2 endpoint][v2]

## A complete constructed registration and value example

Choose a 20-byte fee recipient `11` repeated 20 times, gas limit 30,000,000, timestamp 1,700,000,000, and a 48-byte public-key field `22` repeated 48 times.
These synthetic bytes are not asserted to encode a valid BLS key or registration.
The four fixed-size SSZ fields occupy `20+8+8+48 = 84` bytes, with integers little-endian and no offsets:

```text
1111111111111111111111111111111111111111
80c3c90100000000
00f1536500000000
2222222222222222222222222222222222222222222222222222222222222222
22222222222222222222222222222222
```

Concatenate the five lines for the complete unsigned registration serialization.
It has no signature, slot, bid value, or payload; it is not a complete registration HTTP request.
The [SSZ note](SSZ-offsets-roots-and-branches.md) distinguishes serialization from the hash-tree root used for signing.

Independently suppose a builder offers `25000000000000000` wei, or 0.025 ETH, then submits a replacement worth `20000000000000000` wei, or 0.020 ETH.
The relay endpoint explicitly says a new submission by the same builder key overwrites its previous submission even when less profitable.
The stored amount in this conditional example is therefore 0.020 ETH; no source clause here makes the store retain the maximum, selects an auction winner, or establishes an actual transfer.

## Source defects, code connections and limits

The selected Bellatrix pseudocode has unresolved names and object-shape inconsistencies in registration and bid construction.
Its validator `verify_bid_signature` references undefined `signed_registration`, and `process_bid` calls the verifier without asserting its return.
These snippets must not be treated as a complete enforcing verifier; this note preserves their defects rather than silently repairing them.

At the recorded EthLib commit, [`processRegistration`][lean-registration] and [`processBid`][lean-bid] implement ordered checks, including refusal when signature verification returns false.
[`builderDomain`][lean-domain] uses the configured genesis fork version and a zero genesis-validators root; that is not a claim that every network's genesis version is zero.
[`unblind`][lean-unblind] checks equality between the revealed payload's header and the blinded header.
[`SubmissionStore.submit`][lean-store] models replacement for the same builder in its selected slot.
These are selected definition observations, not end-to-end relay or proof-coverage findings.

Python checked the 84-byte layout, integer fields, wei arithmetic and conditional replacement.
No BLS signing/verification, signing-root computation, valid payload, relay simulation, payment, auction, HTTP request, network, EthLib binding, Lean build or proof replay ran.

[builder]: https://github.com/ethereum/builder-specs/tree/fce64468a81a87892d7cb9b4858272bb3d010e50
[relay]: https://github.com/flashbots/relay-specs/tree/a4e0a50d9d46bfe881fd74cc7e0db366371f4eed
[builder-bellatrix]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/specs/bellatrix/builder.md#containers
[builder-signing]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/specs/bellatrix/builder.md#signing
[bid-schema]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/types/bellatrix/bid.yaml
[release]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/specs/bellatrix/builder.md#blinded-block-processing
[validator]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/specs/bellatrix/validator.md#proposer-slashing
[submit]: https://github.com/flashbots/relay-specs/blob/a4e0a50d9d46bfe881fd74cc7e0db366371f4eed/apis/relay/builder/blocks.yaml
[v2]: https://github.com/ethereum/builder-specs/blob/fce64468a81a87892d7cb9b4858272bb3d010e50/apis/builder/blinded_blocks_v2.yaml
[lean-bid]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/BuilderApi/Bid.lean#L119
[lean-domain]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/BuilderApi.lean#L147
[lean-unblind]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/BuilderApi.lean#L92
[lean-store]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/RelayApi.lean#L87
[lean-registration]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/BuilderApi/Registration.lean#L105
