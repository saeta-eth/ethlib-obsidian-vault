---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - token-interfaces
  - callbacks
---

# Token receiver interfaces and callbacks

ERC-165 discovers advertised interfaces, while a receiver callback accepts or rejects a particular token transfer.
Keep the token contract, transfer operator, previous owner, and receiving contract distinct when tracing that callback.
A positive interface answer cannot replace the callback's required return value.

## Sources and scope

All three sources below have status Final at the ERCs commit in the frontmatter.
The reading covers the sections in the table, including ERC-165's complete Specification and detection example and ERC-1155's complete Safe Transfer Rules.
The [ABI note](ABI-encoding-and-event-topics.md) supplies the encoding prerequisite.
This note compares interface obligations and conditional transfers, without asserting that a particular deployed contract satisfies them.

| Source | Sections used |
| --- | --- |
| [ERC-165][165] | Specification: interface identifiers, publishing interfaces, and the two detection procedures. |
| [ERC-721][721] | Specification: `ERC721`, `ERC721TokenReceiver`, Caveats, and Rationale: Transfer Mechanism. |
| [ERC-1155][1155] | Specification: token/receiver interfaces and Safe Transfer Rules, Approval, and Backwards Compatibility. |

## Discovery is a separate operation

ERC-165 identifies an interface by XORing its function selectors.
Events, return types, and optional functions do not enter that identifier.
The discovery procedure first uses `STATICCALL` with 30,000 gas to ask `supportsInterface(0x01ffc9a7)`.
A failed call or false answer fails detection.
It then asks about `0xffffffff`, again with 30,000 gas.
Failure or true also fails detection, so a contract that answers true indiscriminately does not pass.
When ERC-165 compliance is not already known, run this detection procedure before querying the desired interface.
These answers do not execute a transfer or establish that every advertised behaviour conforms to its specification. [ERC-165, Specification][165]

ERC-721 tokens implement ERC-165 and advertise token interface `0x80ac58cd`.
Their specified safe-transfer path checks the recipient's code size and invokes its receiver function, without prescribing a separate receiver-interface preflight.
ERC-1155 tokens MUST advertise `0xd9b67a26`.
ERC-1155 receivers MUST implement both receiver functions and ERC-165 support for receiver interface `0x4e2312e0`.
Their `supportsInterface` MUST NOT consume more than 10,000 gas, a stricter bound than ERC-165's general 30,000 limit. [ERC-721, Specification][721] [ERC-1155, receiver ERC-165 rules][1155]

## Calculate the receiver selectors and interface IDs

For these canonical signatures, take the first four bytes of Keccak-256 over the exact ASCII text.
Parameter names and data locations are absent.
The complete receiver calculation is:

| Canonical signature | First four hash bytes |
| --- | --- |
| `supportsInterface(bytes4)` | `0x01ffc9a7` |
| `onERC721Received(address,address,uint256,bytes)` | `0x150b7a02` |
| `onERC1155Received(address,address,uint256,uint256,bytes)` | `0xf23a6e61` |
| `onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)` | `0xbc197c81` |

ERC-721's receiver has one function, so its ID and acceptance value both equal `0x150b7a02`.
ERC-1155's receiver ID is the XOR of its two hook selectors:

```text
  f23a6e61
^ bc197c81
= 4e2312e0
```

That combined ID is not an acceptance value for either hook.
The single hook returns `0xf23a6e61`, and the batch hook returns `0xbc197c81` when accepting.
Supporting ERC-165 separately does not add `0x01ffc9a7` into either named receiver ID. [ERC-165, interface identification][165] [ERC-721, receiver interface][721] [ERC-1155, magic constants][1155]

## Trace three conditional transfers

Let `T` be the token contract, `F` the owner, `O` an authorized caller, and `R` a receiving contract distinct from `T`.
Use data bytes `0x1234`.
Assume authorization and balance checks pass, `R` accepts without further state-changing calls, and no other rejection condition applies.
The examples are independent initial states, not three successive operations.

For ERC-721, `F` owns token 42 and `O` calls `T.safeTransferFrom(F,R,42,0x1234)`.
The token transfers ownership to `R`, resets the token's individual approval, and calls `R.onERC721Received(O,F,42,0x1234)` after the transfer.
Inside that callback, `msg.sender` is `T`, the operator argument is `O`, and the previous-owner argument is `F`.
Returning `0x150b7a02` passes the receiver check.
Throwing or returning another value rejects the transfer and reverts the transaction.
The no-data overload passes empty data instead.
ERC-721 permits additional rejection conditions, so satisfying the listed checks does not guarantee every implementation accepts. [ERC-721, interfaces and Transfer Mechanism][721]

For an ERC-1155 single transfer, start with balances `F[7]=10` and `R[7]=0`.
`O` calls `T.safeTransferFrom(F,R,7,3,0x1234)`.
Before the callback, balances are `F[7]=7` and `R[7]=3`, and `T` emits `TransferSingle(O,F,R,7,3)`.
The chosen hook call is `R.onERC1155Received(O,F,7,3,0x1234)`, with token contract `T` as its caller.
The correct magic passes that check, while a revert or different value MUST revert the transaction, including these balance changes and its transfer event. [ERC-1155, safe-transfer and single-hook rules][1155]

For an ERC-1155 batch, start with `F[7]=10`, `F[8]=20`, `R[7]=0`, and `R[8]=1`.
`O` calls `T.safeBatchTransferFrom(F,R,[7,8],[3,4],0x1234)`.
Matching array lengths preserve the pairing `7->3`, `8->4`.
The balance changes follow that array order and give `F=[7,16]`, `R=[3,5]` for IDs `[7,8]`.
Choose one `TransferBatch(O,F,R,[7,8],[3,4])` before `R.onERC1155BatchReceived(O,F,[7,8],[3,4],0x1234)`.
All referenced balances and corresponding events MUST precede the hook.
Returning `0xbc197c81` passes its receiver check, while `0xf23a6e61` does not.
A length mismatch or insufficient balance MUST revert independently of receiver acceptance. [ERC-1155, batch-transfer, event, and batch-hook rules][1155]

## Conditions that the traces do not erase

Both standards reject the zero recipient in these transfer functions.
ERC-1155 mint and burn event conventions use zero addresses separately and do not waive that transfer check.
ERC-721 specifies code size greater than zero for recipient detection, and ERC-1155 gives it as an example.
ERC-1155 forbids receiver hooks on an externally owned account.
Neither interface advertising nor an acceptance value establishes that tokens can later leave `R`.

ERC-1155's interface comments prescribe named single/batch hooks, while Scenarios 6–7 allow alternative hook choices.
The `safeTransferFrom` rules require the single hook, whereas the batch-transfer rules allow either.
These traces choose the named hooks without resolving that textual tension into a universal dispatch rule.
The receiver comments identify the initiating caller as operator, while detailed hook rules require an approved operator and only SHOULD identify it with `msg.sender`.
Choosing direct authorized caller `O` satisfies both descriptions.
Self-transfer hook exceptions, non-standard transfer APIs, and the explicit hybrid-standard compatibility procedure have separate rules. [ERC-1155, Safe Transfer Rules and Backwards Compatibility][1155]

Callbacks execute recipient code after the specified state changes.
ERC-1155 expressly accounts for forwarding and valid re-entry, requiring balances and events to agree before the external call.
That ordering is not a proof that arbitrary application code prevents reentrancy attacks.
It also supplies no callback requirement for an arbitrary ERC-20 transfer.

## Review limits and contribution use

This is a source-only preparation note at the recorded EthLib revision, with no claim about token-module implementation or proof coverage.
The selectors and XORs were recomputed with PyCryptodome Keccak-256, and the balance arithmetic follows the stated inputs.
The primary passages and explanation received a separate review.
Independent calculations reproduced the displayed selectors, receiver-interface XOR, and conditional balance changes.
No token contract, EVM callback, deployed receiver, or Lean proof was executed.
A bounded contribution can separate an ERC-165 query result from a transfer's callback outcome, then state the relevant authorization, argument, update-order, and rollback conditions explicitly.
Keep alternate-hook and hybrid-policy questions visible when selecting that contribution's scope.

[165]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-165.md#specification
[721]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-721.md#specification
[1155]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1155.md#safe-transfer-rules
