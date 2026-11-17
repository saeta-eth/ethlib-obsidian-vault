---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: I02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
beacon_api_commit: "6bc8c7bde19d0499f88e67417419eb12331438f4"
tags:
  - ethlib
  - external-sources
  - beacon-api
  - response-evidence
---

# Beacon API identifiers and response evidence

A Beacon API response identifies an object in the serving node's view.
Its root, availability result, and optimistic/finalized flags answer different questions.
Keep the endpoint's object type and schema separate from independent evidence that the object belongs to a trusted chain.

## Source snapshot and scope

This note uses the official [Beacon API repository][api] at commit `6bc8c7bde19d0499f88e67417419eb12331438f4`.
Its OpenAPI label is `Dev - Ethereum Proof-of-Stake Consensus Specification v1.7.0-beta.3`, and its schemas assume the mainnet preset.
Neither that label nor a listed fork name establishes deployed support or network activation.

Reading covered StateId/BlockId parameters, state/block-root operations, V2 block retrieval, auxiliary fork/finality operations, and selected shared types and headers.
All fork-specific block schemas, consensus transitions, light-client verification, and server implementations remain outside this reading.
The EthLib commit records contribution context only; this source-only note makes no coverage or implementation-gap claim.

## Endpoint families determine the object being named

| Endpoint family | Selected operation and returned object |
| --- | --- |
| State root | `GET /eth/v1/beacon/states/{state_id}/root`: HashTreeRoot of the selected BeaconState. |
| Block root | `GET /eth/v1/beacon/blocks/{block_id}/root`: HashTreeRoot of BeaconBlock/BeaconBlockHeader. |
| Full block | `GET /eth/v2/beacon/blocks/{block_id}`: a fork-dependent signed block, as JSON or SSZ. |
| State context | `/states/{state_id}/fork` and `/finality_checkpoints`: fork information or checkpoint objects associated with that state. |

A state root is not an execution-layer state-trie root, and a beacon block root is not an execution block hash.
The equal 32-byte widths do not identify the hashing domain or make those values interchangeable.
The [account-proof note](Account-trie-proofs-and-absence-boundaries.md) covers the separate execution-state commitment.

[StateId and BlockId][identifiers] describe different accepted names:

| Selector | State identifier description | Block identifier description |
| --- | --- | --- |
| `head` | Canonical head in this node's view. | Canonical head in this node's view. |
| `genesis` | Listed alias. | Listed alias. |
| `finalized` | Listed alias. | Listed alias. |
| `justified` | Listed alias. | Not listed. |
| `64` | Constructed slot spelling. | Constructed slot spelling. |
| A `0x`-prefixed root | State-root lookup. | Block-root lookup. |

Do not carry `justified` into block requests merely because state requests list it.
Both parameter schemas are strings with descriptive alternatives, rather than complete union parsers.
The nearby Uint64 primitive is also only a string with an example; this note derives no canonical decimal, leading-zero, or overflow-rejection rule from that schema alone.
The operations publish `current` as an invalid-ID example, returning HTTP 400.

The Root response type requires `0x` followed by 64 hexadecimal digits and permits uppercase or lowercase hex digits.
That lexical check establishes 32 bytes, not that a BeaconState or block hashes to them. [Primitive types][primitives]

## Complete constructed state-root exchange

Consider this illustrative request, with no request body:

```http
GET /eth/v1/beacon/states/head/root HTTP/1.1
Host: example.invalid
Accept: application/json
```

A constructed HTTP-200 response body is:

```json
{
  "execution_optimistic": true,
  "finalized": false,
  "data": {
    "root": "0x1111111111111111111111111111111111111111111111111111111111111111"
  }
}
```

The repeated bytes are a public shape example, not a computed state root or network observation.
The [getStateRoot schema][state-root] requires `execution_optimistic`, `finalized`, and `data`, with `data.root` required in turn.
Both flags are JSON booleans, not strings or numeric status codes.
The response does not include the state, its slot, a proof path, or a signature authenticating this server assertion.

`ExecutionOptimistic` means the response references an unverified execution payload and may later be invalidated.
`Finalized` means it references finalized history as determined by fork choice.
Thus this example reports optimistic information outside finalized history, without proving that either report is correct independently.
Even false optimistic status is not a substitute for checking a trusted root or validating the returned object.

The shared primitive description discusses assuming false when `execution_optimistic` is absent, and additional finality calls when `finalized` is absent.
Those generic fallback descriptions do not remove this selected operation's explicit required fields.
A compatibility consumer can choose a fallback policy, but must distinguish it from conformance to the selected response schema.

On a successful root-identified state lookup, the operation says the returned root equals that identifier.
Equality alone therefore does not prove to the caller that the underlying state was recomputed or independently authenticated.
Likewise, repeated `head` calls do not promise one immutable snapshot: the selector is relative to the node's view when resolved.

## Availability and fork-aware encoding

The selected state operation documents HTTP 404 as “State not found”; block operations use “Block not found.”
Their error schema requires numeric `code` and string `message`, for example:

```json
{
  "code": 404,
  "message": "State not found"
}
```

This is an endpoint outcome, not an authenticated assertion of global nonexistence.
The response does not distinguish all possible causes, such as unavailable local history versus an unknown identifier.
Diagnosing retention, querying another source, and checking canonical history require additional evidence.
The [history note](History-retention-and-archive-authentication-boundaries.md) explains why available bytes and authenticated history are separate concerns.

For [V2 block retrieval][block], `Accept` selects JSON or `application/octet-stream` SSZ bytes; the operation also documents HTTP 406 when a representation cannot be supplied.
Its success response declares the required `Eth-Consensus-Version` header, identifying the consensus version to aid deserialization.
JSON additionally requires `version`, `execution_optimistic`, `finalized`, and `data`, with `data` referring to fork-dependent signed-block schemas.
The selected phase0 envelope, for example, requires `message` and `signature`; this note supplies no full-block fixture. [Version header][root-spec]

The API's root description says endpoints are independently versioned.
The `/v2/` route is therefore not consensus fork number two, and its JSON `version` is not a client software version.
The state-root operation above advertises only JSON; SSZ support must be checked per operation rather than inferred from another endpoint.
For binary blocks, choose the corresponding object schema using version and network/preset context; a byte stream does not carry the JSON metadata object.

## Contribution question and checks

A bounded consumer contribution can retain the requested identifier, resolved root, response metadata, representation, and expected chain context as distinct inputs.
It should state how mutable aliases, missing local data, unknown versions, and compatibility fallbacks affect its result.
If finality verification is required, it also needs an independently justified checkpoint/update-verification procedure; a server boolean alone supplies none.
No such consensus procedure or deployment policy is selected here.

Python checked the constructed JSON, required boolean/root fields, 32-byte root pattern, uppercase acceptance, and short/missing-prefix controls.
The identifier table checks only its displayed cases, not a complete parameter parser.
No RPC, BeaconState, SSZ serialization/hash, signature, fork-choice transition, EthLib binding, Lean build, or proof replay ran.

[api]: https://github.com/ethereum/beacon-APIs/tree/6bc8c7bde19d0499f88e67417419eb12331438f4
[root-spec]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/beacon-node-oapi.yaml
[identifiers]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/params/index.yaml
[primitives]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/types/primitive.yaml
[state-root]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/apis/beacon/states/root.yaml
[block]: https://github.com/ethereum/beacon-APIs/blob/6bc8c7bde19d0499f88e67417419eb12331438f4/apis/beacon/blocks/block.v2.yaml
