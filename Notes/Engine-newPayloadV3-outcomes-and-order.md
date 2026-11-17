---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: I01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
execution_apis_commit: "3284ee80c3da3e516c53782dde75157cccf1ebdc"
tags:
  - ethlib
  - external-sources
  - engine-api
  - payload-validation
---

# Engine newPayloadV3 outcomes and validation order

`engine_newPayloadV3` submits an execution payload for processing.
Its response can be a JSON-RPC error, an invalid payload, a payload awaiting validation, or a valid payload.
Keep those outcomes separate before changing a decoder, caller, or validation check.

## Source and fork boundary

This note reads the official [execution-apis][repository] specification at commit `3284ee80c3da3e516c53782dde75157cccf1ebdc`, the exact pin cited by EthLib's `NewPayload.lean`.
The selected sources form an inheritance chain:

| Source | Material used |
| --- | --- |
| [Paris][paris] | `PayloadStatusV1`, Payload validation, `engine_newPayloadV1`, and the forkchoice update rules. |
| [Shanghai][shanghai] | `engine_newPayloadV2`, including its replacement of `INVALID_BLOCK_HASH`. |
| [Cancun][cancun] | `ExecutionPayloadV3` and `engine_newPayloadV3`. |
| [Prague][prague] | Update the methods of previous forks, establishing V3's upper timestamp boundary. |
| [Common rules][common] | Errors, Message ordering, and Load-balancing and advanced configurations. |

These are primary interface rules, not evidence that a particular client implements them.
The bounded method here handles Cancun payloads: timestamps at or after Cancun activation and before Prague activation under the configured schedule.
Method version and payload-structure version differ: Prague's `newPayloadV4` still takes `ExecutionPayloadV3`.
This note does not cover V4, payload building, or every later amendment.

## Decode the request before classifying the payload

V3 has exactly three parameters, in order:

```text
[executionPayload, expectedBlobVersionedHashes, parentBeaconBlockRoot]
```

The payload includes withdrawals, `blobGasUsed`, and `excessBlobGas`.
The root is a separate 32-byte parameter, not another payload field.
Cancun requires the supplied parameters and fields to match the expected structure strictly.
A `null` field counts as absent, so `parentBeaconBlockRoot: null` cannot satisfy its required parameter.
A correctly encoded all-zero root is a value, distinct from absence. That observation concerns shape, not full payload validity.

`expectedBlobVersionedHashes` contains 32-byte values.
Build the actual list by concatenating each blob transaction's versioned-hash list in transaction inclusion order, preserving each transaction's internal order.
Compare arrays, not sets.
With no blob transactions, the expected array must be `[]`.
The [transaction note](Typed-transaction-envelopes-and-signing.md) explains the envelope bytes in `payload.transactions`.
These are transaction bytes, not the pooled-transaction wrapper carrying blobs and proofs.
Matching versioned hashes alone establishes neither blob availability nor KZG proof validity.

## Separate errors from payload statuses

A malformed V3 parameter set produces JSON-RPC `-32602: Invalid params`.
A payload timestamp outside the method's fork range produces `-38005: Unsupported fork`.
Neither is a `PayloadStatusV1` result with `status: INVALID`.
Unrelated processing failures also use an error object under the inherited Paris rule.

After those qualifications, the following cases illustrate the required response categories.
Each row assumes the other relevant checks do not introduce another failure.

| Case | Result | `latestValidHash` |
| --- | --- | --- |
| Expected and actual blob hash arrays differ. | `INVALID` | `null`. |
| Payload block hash differs from the reconstructed header hash. | `INVALID` | `null`. |
| Transactions contain zero-length or invalid entries. | `INVALID` | `null`. |
| Requisite data for acceptance or validation is missing. | `SYNCING` | `null`. |
| A permitted side-branch payload remains unvalidated under the conditions below. | `ACCEPTED` | `null`. |
| Full payload validation succeeds. | `VALID` | The submitted payload's block hash. |
| Full validation finds an invalid payload or ancestor. | `INVALID` | The ancestry-dependent value described below. |

V3 inherits Shanghai's replacement of `INVALID_BLOCK_HASH` with `INVALID`.
Do not return every constructor of the shared Paris status enumeration merely because that enumeration still lists five values.

`ACCEPTED` requires nonempty transaction entries, a valid block hash, a payload that does not extend the canonical chain, no completed full validation, and known ancestors forming a well-formed chain.
It does not mean execution succeeded.
Both `ACCEPTED` and `SYNCING` have `validationError: null`.
An invalid response may carry an explanatory string or `null`; callers must use the status rather than parse that string.

## Required checks do not define every tie-break

The inherited Paris rules require checking nonempty transactions and the computed block hash even during syncing.
Cancun likewise requires immediate comparison of blob hash arrays in all cases, including active sync.
Missing parent state must not become a reason to omit those checks.
If a canonical extension has the requisite data locally, another branch's active sync must not prevent full validation.

The selected text does not expressly define a total precedence among all simultaneous failures.
Consider an illustrative schedule with Cancun activation at 1000 and Prague at 2000.
Submit a well-shaped V3 request at timestamp 2000 whose expected hashes are `[B, A]` while its blob transactions contain `[A, B]`, where `A` and `B` are distinct 32-byte hashes.
The request violates both the fork-range rule and array equality.
The numbered additions state both obligations, without explicitly saying which response wins this overlap.
Do not turn their numbering into an unqualified portable precedence claim.

EthLib's [`checkNewPayload`][checks] checks decoded parameters, then the active fork, then block construction and hash, then the blob hash array.
For that overlap, its fork check returns `.unsupportedFork` before the array comparison.
That is observed implementation order, not a resolution of the source's unspecified tie-break.
A contribution testing overlapping failures must state which ordering it is pinning and why.

## Read latestValidHash along the submitted branch

`latestValidHash` is not a synonym for the execution client's current head.
For full-validation failure, Paris requires the hash of a fully validated ancestor such that every higher ancestor on the submitted branch is invalid.
If that qualifying ancestor is a proof-of-work block, return the all-zero 32-byte hash.
If the client cannot determine the qualifying ancestor, return `null`.
The zero hash and `null` therefore express different cases.
The explicit early failures above require `null`, without a search for a substitute head hash.

For example, let `A` be a fully validated proof-of-stake block, followed by invalid `B`, followed by submitted `C`.
If the client determines this ancestry and validation fails along it, the qualifying hash is `hash(A)`, not `hash(B)` or `hash(C)`.
Shanghai changes the terminal-PoW validation step with a `MAY NOT validate` clause.
This note does not copy the Paris check into V3 as unconditional work.

`VALID` reports validation; the consensus client communicates canonical head selection through `engine_forkchoiceUpdated`.
The common rules distinguish inserting payloads through `newPayload` from writing the chain head through forkchoice updates.
They require consensus callers to preserve forkchoice-event order and execution clients to process those updates in receipt order.
The update rules also require a valid head before applying the forkchoice state.

## EthLib connections and review limits

[`Payload.lean`][payload] separates `EngineError` from `PayloadStatus` and its optional hash.
`newPayloadWithoutState` performs the initial checks, then reports syncing if they pass.
`newPayloadOnChain` additionally needs the last recent block to match the parent before executing the state transition.
These inspected helper paths do not implement every client's caching and side-branch policy.
Their lack of an `ACCEPTED` branch does not remove that source-defined outcome.

The inspected [`newPayloadOnChain_of_not_parent`][parent-proof] assumes successful initial checks and that no last recent block has the required parent hash.
It concludes `.ok (SYNCING, no hash, no reason, no new block/state)` in the helper's tuple representation.
[`ExecutionEngine.lean`][bridge] separately adapts beacon requests to block and execution checks through a Boolean notification interface.
That adapter is not the complete JSON-RPC response model.

The cited passages, definitions, and the parent-mismatch theorem's full statement and proof were read.
The sources and explanation received a separate review.
Independent calculations checked the illustrative timestamp range, ordered blob arrays, empty array, zero-root distinction, and qualifying ancestor in the examples.
The examples and decision table are source-derived review cases, not executed client or Lean tests.
No build, proof replay, network request to an Engine endpoint, or complete inherited-validation audit was performed.

[repository]: https://github.com/ethereum/execution-apis/tree/3284ee80c3da3e516c53782dde75157cccf1ebdc
[paris]: https://github.com/ethereum/execution-apis/blob/3284ee80c3da3e516c53782dde75157cccf1ebdc/src/engine/paris.md
[shanghai]: https://github.com/ethereum/execution-apis/blob/3284ee80c3da3e516c53782dde75157cccf1ebdc/src/engine/shanghai.md#engine_newpayloadv2
[cancun]: https://github.com/ethereum/execution-apis/blob/3284ee80c3da3e516c53782dde75157cccf1ebdc/src/engine/cancun.md#engine_newpayloadv3
[prague]: https://github.com/ethereum/execution-apis/blob/3284ee80c3da3e516c53782dde75157cccf1ebdc/src/engine/prague.md#update-the-methods-of-previous-forks
[common]: https://github.com/ethereum/execution-apis/blob/3284ee80c3da3e516c53782dde75157cccf1ebdc/src/engine/common.md
[checks]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Engine/NewPayload.lean#L201
[payload]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Engine/Payload.lean#L133
[parent-proof]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Engine/Proofs/NewPayload.lean#L446
[bridge]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Engine/ExecutionEngine.lean#L171
