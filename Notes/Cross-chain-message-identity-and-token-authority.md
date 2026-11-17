---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A14
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
solidity_commit: "73712a01b2de56d9ad91e3b6936f85c90cb7de36"
tags:
  - ethlib
  - external-sources
  - cross-chain
  - token-authority
---

# Cross-chain message identity and token authority

A cross-chain message identifier tracks an execution; it does not authorize token creation.
ERC-5164 describes dispatch and execution, while ERC-7802 describes local token mint/burn entry points.
Connecting them requires a chosen transport, trusted callers, and a token-mapping policy that neither interface supplies.
No particular bridge or integration target is assumed.

## Source versions and reading scope

At ERCs commit `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`, [ERC-5164][5164] has status Last Call and [ERC-7802][7802] has status Draft.
ERC-5164's printed Last Call deadline is 2023-11-15; that date does not change its recorded status or establish adoption.
Reading covered both complete texts: metadata, specifications, rationale, compatibility, security, and ERC-7802's reference implementation.
Linked bridge implementations and the dependencies' full specifications were not evaluated.
The byte calculation also uses selected Solidity v0.8.30 [ABI rules][abi] for selectors, static arguments, and packed encoding.
No implementation or proof-coverage claim is made for the recorded EthLib revision.

## Dispatch, identity, and retry are separate obligations

ERC-5164 requires a dispatcher on the origin chain and an executor on the destination chain.
The dispatcher MUST reject an unsupported destination, forward the message to an executor there, emit `MessageDispatched`, and return a unique `messageId`.
It MAY require payment.
Returning the ID or observing dispatch does not report successful destination execution. [MessageDispatcher][dispatcher]

The ID MUST be unique across chains and dispatchers.
The source suggests hashing `(chainId, dispatcherAddress, messageNonce)` with a monotonically increasing nonce, but fixes neither an encoding nor one required hash construction.
There is therefore no single source-defined message-ID preimage to reproduce here.
Uniqueness also does not authenticate the destination, payload, or source actor.
Those values still need the selected transport's evidence.

An executor SHOULD verify all message data with the bridge transport layer.
It MUST NOT successfully execute a message twice, and MUST revert the transaction when execution fails so the message can be retried.
It may execute different IDs in any order; nonce order at dispatch is not an execution-order guarantee.
After successful execution it MUST emit `MessageIdExecuted`.
The specified duplicate and target-failure custom errors are SHOULD choices, alongside the MUST-revert outcomes. [Execution and errors][executor]

For each call, the executor MUST append ABI-packed `(messageId, fromChainId, from)` after the original application data.
The trailer contains 32 bytes, 32 bytes, and 20 bytes: 84 bytes in total.
It does not change the EVM caller into the remote sender.
In the example below, the token sees the local executor as `msg.sender`; the remote actor exists only in the trailer.
Accepting arbitrary callers' self-written trailers would not authenticate that actor. [Calldata][calldata]

## Token permission does not prove a remote burn

ERC-7802 defines `crosschainMint(address,uint256)` and `crosschainBurn(address,uint256)`.
The token issuer chooses their authorized callers; the interface does not prescribe a bridge registry, ERC-20 allowance check, or remote-token mapping.
It also leaves zero-amount acceptance to the issuer.
Its reference contract uses one immutable `TOKEN_BRIDGE` and a caller check, illustrating one policy rather than requiring that policy. [Design philosophy and implementation][7802]

Successful calls MUST trigger the corresponding `CrosschainMint` or `CrosschainBurn` event, whose `_sender` MUST equal local `msg.sender`.
For minting, an ERC-20 `Transfer` event is SHOULD; the burn text more weakly says implementations might consider a `Transfer` event.
Do not strengthen either into the dedicated events' MUST requirement. [Methods and events][methods]

The required ERC-165 response for interface ID `0x33331994` describes local support.
ERC-7802 explicitly leaves receiving-chain interface consistency to the bridge.
It defines no proof that a particular origin token was burned, locked, finalized, or mapped to this destination token.
Its security section warns that a malicious authorized caller can mint or burn improperly.
It also advises against wrapped-native-token use unless the bridge can symmetrically mint and burn the native asset with proper collateralization. [Security Considerations][security]

## Complete constructed call bytes

Assume destination executor `E=0x3333333333333333333333333333333333333333` is authorized by the destination token.
Assume its transport and source policy accept origin chain 1 and source `S=0x2222222222222222222222222222222222222222`.
Choose the opaque ID `0xaa` repeated 32 times, recipient `0x6666666666666666666666666666666666666666`, and amount 25 atomic units.
These public teaching values are not a deployed bridge or a published vector.

The first four bytes of each canonical-signature Keccak-256 digest give:

```text
crosschainMint(address,uint256) -> 18bf5077
crosschainBurn(address,uint256) -> 2b8c49e3
18bf5077 XOR 2b8c49e3          = 33331994
```

This reproduces ERC-7802's published interface ID, excluding inherited ERC-165 methods as its formula specifies. [Interface Identification][interface]
The complete 152-byte target calldata is the concatenation of these six lines, without whitespace:

```text
18bf5077
0000000000000000000000006666666666666666666666666666666666666666
0000000000000000000000000000000000000000000000000000000000000019
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
0000000000000000000000000000000000000000000000000000000000000001
2222222222222222222222222222222222222222
```

The original function data occupies `[0,68)`; the ID occupies `[68,100)`, chain `[100,132)`, and remote sender `[132,152)`.
The ordinary address argument has 12 leading zero bytes, while the packed trailing address has only 20 bytes.
This is a constructed encoding, not a claim that every token decoder accepts trailers or authenticates them.
The [ABI note](ABI-encoding-and-event-topics.md) explains the two encoding modes.

## Conditional retry ledger

Assume this token accepts its ABI arguments with the appended 84-byte trailer and its issuer's permission policy authorizes E.
ERC-7802 alone does not require acceptance of arbitrary trailing calldata.
Start with recipient balance 10, local total supply 100, and this ID not successfully executed.
The outcomes below are conditional source-rule cases, not a simulated transport or contract:

| Attempt | Executed ID | Recipient balance | Local supply |
| --- | --- | --- | --- |
| Target fails; execution transaction reverts. | No | 10 | 100 |
| Retry succeeds and mints 25. | Yes | 35 | 125 |
| Same-ID delivery repeats; executor rejects it. | Yes | 35 | 125 |

On success the token event identifies E as caller, not S; the executor's event identifies chain 1 and the message ID.
Reentrant or concurrent execution paths must also respect the no-double-success rule, but this ledger establishes no general reentrancy argument.
No global supply conservation follows: this example contains no authenticated origin burn or lock.
The [callback note](Token-callback-order-and-hook-authority.md) supplies separate context about local call order and authority.

Transport security and finality depend on a chosen bridge and application model, neither specified by these interfaces.
The [payment-URI note](Payment-URI-revisions-and-request-boundaries.md) concerns an earlier request boundary: naming a chain or requested action does not authorize its execution.
Independent calculations reproduced the displayed example values.
Python with PyCryptodome 3.23.0 reproduced the two selectors, published XOR result, byte lengths/field slices, and conditional ledger arithmetic.
No transport, token call, EVM, wallet, EthLib binding, Lean build, or proof replay ran.
Interface conformance alone establishes no bridge trust profile, finality bound, or security proof.

[5164]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5164.md
[dispatcher]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5164.md#messagedispatcher
[executor]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5164.md#messageexecutor
[calldata]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5164.md#execution
[7802]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7802.md
[methods]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7802.md#methods
[interface]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7802.md#interface-identification
[security]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7802.md#security-considerations
[abi]: https://github.com/argotorg/solidity/blob/73712a01b2de56d9ad91e3b6936f85c90cb7de36/docs/abi-spec.rst
