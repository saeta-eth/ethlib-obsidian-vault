---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 3
delivery_batch: "follow-up-8"
catalogue_family: E07
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - system-calls
  - execution-requests
---

# System calls and execution request commitments

A privileged system call, an extracted request and a header commitment establish different facts.
Execution can update a history contract or collect requests without establishing that a validator withdrawal or consolidation will pass consensus-layer processing.
The useful boundary is the exact bytes execution produces, their order and the hash that commits to them.

## Sources and applicability

The primary sources are [EIP-4788][4788], [EIP-2935][2935], [EIP-6110][6110], [EIP-7002][7002], [EIP-7251][7251] and [EIP-7685][7685], all Final at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Reading covered the history contracts' interfaces and block-processing rules, deposit extraction, withdrawal/consolidation request structures and system-call rules, and EIP-7685's specification and request-validity rationale.
The embedded contract bytecode, deployment transactions and linked consensus state transitions were not audited here.
Final status alone does not establish activation on a particular network or at a particular block.

## Privilege, timing and failure policy

These system calls use caller `0xfffffffffffffffffffffffffffffffffffffffe`, a dedicated gas limit of `30_000_000` and zero value.
Their gas is excluded from the block gas limit; the request-contract EIPs also explicitly exclude consumed gas from overall block gas usage.
They do not charge an ordinary transaction sender under EIP-1559 fee-burn semantics.
An ordinary user's call to the same address does not acquire this privileged caller identity.
The history-call rules require execution to completion and explicitly allow silent failure when code is absent.

| Source | When active, call timing | Input or result | Missing-code policy |
| --- | --- | --- | --- |
| EIP-4788 | Before transactions | Input is the 32-byte parent beacon block root. | Fail silently. |
| EIP-2935 | Before transactions | Input is the 32-byte parent execution block hash. | Fail silently. |
| EIP-7002 | At block end, after transactions and the stated withdrawal-request validations | Empty calldata; return dequeued withdrawal requests. | The block MUST be invalid. |
| EIP-7251 | At block end, after transactions and the stated consolidation-request validations | Empty calldata; return dequeued consolidation requests. | The block MUST be invalid. |

The two request-contract EIPs also require invalidation if their system call fails or returns an error.
Each request must appear in the exact order returned by its dequeue operation.
The withdrawal queue releases at most 16 requests per block; the consolidation queue releases at most two.
Both system operations update the queue and fee-excess state and reset the current-block request count.
These requests are separate from the execution block's withdrawals that credit balances.

EIP-4788's history is keyed by execution timestamp: one 8191-entry ring stores timestamps and another stores their parent beacon roots.
A read requires exactly 32 big-endian input bytes, a nonzero timestamp and a matching stored timestamp; otherwise it reverts.
EIP-2935 instead stores parent execution hashes in an 8191-entry ring indexed by block number and checks the requested range `[block.number-8191, block.number-1]`.
Its activation does not backfill earlier hashes, and it does not extend the `BLOCKHASH` opcode's 256-block window.
EIP-4788 additionally forbids its system transaction in an active genesis block and specifies the zero-root placeholder there.

## Extracted records and typed bodies

EIP-7685 defines a typed object as `request_type || request_data`, where the type occupies one byte and the body contains zero or more records.
The body's encoding belongs to its request type.
The framework does not prescribe a universal request source or validation time, and its rationale explicitly leaves further validation to the consensus layer.

EIP-6110 extracts deposits from receipt logs filtered by the configured deposit-contract address and first event topic, preserving log order.
Its selected layout check requires 576-byte ABI event data with specified offsets and field lengths.
The resulting record concatenates public key, withdrawal credentials, amount, signature and index into `48+32+8+96+8 = 192` bytes.
Amount and index retain their eight-byte little-endian encodings; the ABI offsets, sizes and padding are not request fields.
The [ABI note](ABI-encoding-and-event-topics.md) explains that outer event layout separately.

One source question remains explicit: EIP-6110's constant table literally writes `b'0'`, which ordinary Python interprets as byte `0x30`, while pinned EthLib assigns numeric type `0`.
This note does not silently repair that mismatch or use deposits for its worked commitment.
Withdrawal type `0x01` and consolidation type `0x02` are explicit in their respective primary texts.

## A complete constructed withdrawal request

Take the following synthetic fields, with no assertion that the public-key bytes encode a valid BLS point or identify a validator:

```text
source_address (20 bytes)
0102030405060708090a0b0c0d0e0f1011121314
validator_pubkey (48 bytes)
2122232425262728292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748494a4b4c4d4e4f50
amount = 0x0102030405060708 = 72623859790382856
amount as big-endian input:  0102030405060708
amount as little-endian output: 0807060504030201
```

Under EIP-7002, adding a withdrawal request takes 56 calldata bytes: the public key followed by the big-endian amount.
The contract requires a sufficient request fee and queues the caller's `msg.sender` as `source_address`.
Conditionally assume that caller is the address above, the call succeeds and this record is dequeued.
The returned fixed-width body then has 76 bytes, with the amount reversed into little-endian order.
Its EIP-7685 object `r` has 77 bytes:

```text
byte 0:       type 01
bytes 1..20:  source_address
bytes 21..68: validator_pubkey
bytes 69..76: amount, little-endian

r, concatenate the following hex lines:
010102030405060708090a0b0c0d0e0f1011121314
2122232425262728292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748494a4b4c4d4e4f50
0807060504030201
```

This is a source-derived constructed example, not a published vector or an executed contract call.
Its byte layout says nothing about whether consensus will authorize or fulfill the request.

## Two levels of SHA-256

EIP-7685 first hashes each nonempty typed object, including its type byte.
It then hashes the concatenation of those 32-byte intermediate digests in ascending request-type order:

```text
h_i = SHA256(type_i || request_data_i), only if len(request_data_i) > 0
requests_hash = SHA256(h_0 || h_1 || ...)

For the single 77-byte r above:
SHA256(r) =
005014989cbea8f85d016adf6d6c35387ea8c584a520f5983be4bcddbafe58a2
requests_hash = SHA256(the 32 decoded bytes above) =
58a66dff729a69ef3e1711b551501a7878acfbd245a12ab9d0b2d9ed31b817ed
```

Hexadecimal text is for display; each hash consumes decoded bytes.
Two withdrawal records belong in one type-`0x01` body and therefore one intermediate hash, rather than one intermediate hash per record.
The framework leaves within-type ordering to the request-type rules.
Ascending commitment order alone does not specify a universal order for executing different system contracts.

A type-only object such as `01` is excluded completely.
Thus no objects, one empty withdrawal body, or several empty typed bodies produce the same empty commitment:

```text
SHA256(empty bytes) =
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

The empty case does not hash `SHA256(01)` into the outer input.
This commitment is also distinct from an SSZ `hash_tree_root` of the request records.

## Pinned EthLib observations and checks

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`depositRequestOfEvent`][lean-deposit] checks the selected ABI layout, and [`requestsHash`][lean-hash] implements the two hash levels and size filter.
The hash helper consumes its input order; it does not sort or validate request types.
[`startSystemCalls`][lean-start] orders beacon-root storage before history storage, while [`REQUEST_CONTRACTS`][lean-contracts] orders withdrawal extraction before consolidation extraction.
[`checkedSystemCall`][lean-checked] checks request-contract code and call errors.
These are observations of definitions, not claims of theorem coverage; later-fork additions visible in these files are outside this source reading.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Python byte concatenation, integer conversion and `hashlib.sha256` checked the field sizes, endian conversion, inner/outer hashes and empty-body controls shown here.
No EVM call, public-key validation, consensus transition, EthLib binding, Lean build or proof replay ran.
Further work can connect a committed request to a particular consensus transition only after reading that transition's authorization and state conditions.

[4788]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4788.md
[2935]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2935.md
[6110]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6110.md
[7002]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7002.md
[7251]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7251.md
[7685]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7685.md
[lean-deposit]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Requests.lean#L113
[lean-hash]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Requests.lean#L150
[lean-start]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/SystemCall.lean#L221
[lean-contracts]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/SystemCall.lean#L234
[lean-checked]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/SystemCall.lean#L179
