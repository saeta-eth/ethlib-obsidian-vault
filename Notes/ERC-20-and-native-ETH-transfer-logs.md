---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 1
catalogue_family: A01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - transfer-logs
---

# ERC-20 events and native ETH transfer logs

ERC-20 and EIP-7708 use the same Transfer event shape for different assets and emission rules.
The shape alone does not identify the asset or establish that the event should have been emitted.
This note compares pinned texts without asserting that EIP-7708 is active on a particular network.

## Sources examined

| Source | Revision and sections read | Role |
| --- | --- | --- |
| [ERC-20](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-20.md) | Complete text, especially transfer, transferFrom, and Events/Transfer. | Token-interface requirements; Final at this snapshot. |
| [EIP-7708](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7708.md) | Complete text, especially ETH Transfer Logs, Rationale, and Test Cases. | Native ETH log proposal; Last Call at this snapshot. |
| [EIP-140](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-140.md) | Motivation and Specification. | REVERT and rollback of execution changes, including logs. |
| [EIP-8246](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8246.md) | Complete text, alongside EIP-7708's SELFDESTRUCT discussion. | Related burn-semantics dependency. |
| [EIP-7799](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7799.md) | Complete text, with attention to fee and system logs. | Separate system-log proposal; Draft at this snapshot. |

## Shared format, distinct rules

ERC-20 declares Transfer with indexed from and to addresses and a uint256 value.
Its transfer and transferFrom rules MUST emit the event for token transfers, including zero-value transfers.
It gives no self-transfer exemption from that general event rule; this does not imply that every transfer call succeeds.
For newly created tokens, emission with the zero address as from is SHOULD, not MUST.
The standard does not define a general burn operation.

EIP-7708 specifies a LOG3-shaped record for native ETH movements:

| Field | EIP-7708 value |
| --- | --- |
| Emitting address | `0xfffffffffffffffffffffffffffffffffffffffe`, the system address. |
| First topic | The Keccak-256 hash of `Transfer(address,address,uint256)`, given explicitly in the EIP. |
| Second and third topics | From and to addresses, each prefixed with zeros to fill a 32-byte word. |
| Data | Amount in wei as a 32-byte big-endian unsigned integer. |

ERC-20 does not require that system emitting address.
Interpret an event together with its emitting address, asset, and operation.

## Cases that separate the standards

| Case | ERC-20 | Pinned EIP-7708 |
| --- | --- | --- |
| Zero-value transfer | MUST emit when the token transfer occurs. | Zero value is a negative emission case. |
| Transaction or CALL moving value to the same account | No self-transfer exemption is stated for token transfers. | Its transaction and CALL rules require a different account. |
| Nonzero native ETH transaction to another account | Does not define native ETH transaction logs. | Transfer log precedes EVM execution logs. |
| Internal native ETH movement | Does not define native ETH execution. | The specified CALL, creation, and SELFDESTRUCT cases log when the movement executes. |
| Token creation | SHOULD emit with from equal to the zero address. | Does not define token minting. |

Do not import a rule from one column merely because the record format matches.

## A worked byte-layout example

Assume a retained, successful native ETH movement of 5 wei from address A to a different address B under the pinned EIP-7708 rule.
Use its system emitting address and fixed first topic.
Encode A and B with 12 leading zero bytes each; the addresses then occupy the remaining 20 bytes of their topics.
The data word is:

```text
0000000000000000000000000000000000000000000000000000000000000005
```

This value follows by writing 5 in base 256 into a 32-byte big-endian word; it was not computed from EthLib.
Changing the amount to zero removes this native transfer log under EIP-7708.
An ERC-20 zero-value token transfer still has its required Transfer event.
The two amounts also use different units: wei and the token's own integer units.

## Rollback and other balance changes

EIP-140 makes REVERT roll back execution changes, including logs.
EIP-7708's Test Cases separately identifies reverts and rolled-back inner calls as cases with no retained transfer log.
A transiently created record is therefore not sufficient evidence of a final receipt log.

EIP-7708 excludes fee payments, base-fee burn, and withdrawals from its transfer-log design.
Do not claim that its logs alone describe every ETH balance change.
EIP-7799 proposes additional fee and block-level system logs with separate commitments and API changes.
Its requirements and applicability must be assessed separately.

EIP-7708's SELFDESTRUCT discussion relies on EIP-8246 and leaves the earlier burn edge case unspecified by EIP-7708 itself.
Preserve that dependency when comparing historical forks.

## EthLib connection and review limits

[Evm/TransferLog.lean](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Evm/TransferLog.lean) is the entry point for the log shape and fork-conditional insertion helpers.
Use the primary source to check emission and ordering, then trace its caller and rollback path for the contribution being made.
This note does not establish complete agreement of that implementation with the EIP.

The source passages above were read and the byte-layout example was derived from the stated encoding rule.
No network adoption audit, Lean proof check, execution test, or complete comparison with every later amendment was performed.
