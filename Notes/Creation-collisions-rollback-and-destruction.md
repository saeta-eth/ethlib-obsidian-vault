---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - contract-creation
  - rollback
---

# Creation collisions, rollback, and destruction

Computing a CREATE2 address does not establish that creation can succeed there.
A collision, a reverting initializer, and an enclosing frame's rollback affect different state boundaries.
SELFDESTRUCT also depends on whether the executing contract was created in the same transaction.

## Sources and applicability

All eight EIPs below are Final at the recorded EIPs revision; that status does not establish a network's activation schedule.
Each complete short document was read, including its rationale and compatibility qualifications.

| Sources | Selected rules |
| --- | --- |
| [EIP-1014][1014], [EIP-684][684], [EIP-161][161] | CREATE2 preimage and published examples; collision conditions; destination nonce initialization and empty-account rules. |
| [EIP-140][140], [EIP-150][150] | REVERT outcomes, memory/gas qualifications, and creation's gas-forwarding cap. |
| [EIP-170][170], [EIP-3860][3860] | Runtime-code versus initcode limits, initcode metering, and distinct invalid-transaction/exceptional-instruction cases. |
| [EIP-6780][6780] | SELFDESTRUCT balance transfers, delayed deletion eligibility, and the creation-transaction exception. |
| [Yellow Paper, pinned Shanghai text][yp] | §7 Contract Creation and selected Appendix H.2 CREATE/CREATE2 entries, for address and creator-nonce boundaries. |

The worked state cases assume the selected rules apply, with empty destination storage and no later resource-accounting amendments.
Delegation and later state-gas proposals are outside this composition; the [authorization note](Authorization-order-and-rollback.md) preserves that distinction.
No claim here covers every branch of the pinned EthLib fork model.

## Address bytes are not deployed runtime code

EIP-1014 calculates the last 20 bytes of:

```text
Keccak256(ff || factory_address_20 || salt_32 || Keccak256(initcode))
```

The outer preimage has exactly `1 + 20 + 32 + 32 = 85` bytes.
It commits to initialization code, which executes to produce runtime code, rather than directly to that returned runtime code.
The [Keccak note](Keccak-SHA3-padding-and-domains.md) explains why SHA3-256 cannot substitute for this hash.

Use published **EIP-1014 Example 1**:

```text
factory  = deadbeef00000000000000000000000000000000
salt     = 0000000000000000000000000000000000000000000000000000000000000000
initcode = 00
Keccak256(initcode)
         = bc36789e7a1e281436464229828f817d6612f7b477d66591ff96a9e064bcc98a
outer digest
         = 4c68996c7a7c9bcfca7f1485b928f69bb1d91cd65274e3c79d8986362984fda3
address  = b928f69bb1d91cd65274e3c79d8986362984fda3
```

The address is published; the displayed intermediate hashes are derived calculations.
The initializer `00` is STOP and returns empty runtime code.
That does not make a successfully created account's nonce zero.

EIP-1014's quoted upfront gas is 32006 without memory expansion.
With EIP-3860's additional two gas per initcode word, the corresponding base-plus-hash-plus-initcode amount is 32008.
Neither figure is a complete transaction-gas claim.

## A collision and two rollback boundaries

Assume a hypothetical factory at that address starts a frame with nonce 7.
The calculated destination already holds 5 wei but has nonce zero, empty code, and empty storage.
Both creation attempts use zero endowment, the same salt and `00` initializer, and sufficient gas; depth, balance, and other admission checks pass.
Balance alone is not EIP-684's collision condition.

| Point in this conditional trace | Factory nonce | Destination nonce/code/balance |
| --- | --- | --- |
| Factory-frame checkpoint. | 7 | `0`, empty, 5 wei. |
| First CREATE2 succeeds. | 8 | `1`, empty, 5 wei. |
| Second identical CREATE2 collides. | 9 | `1`, empty, 5 wei. |
| Factory frame subsequently executes REVERT. | Restored to 7. | Restored writes: `0`, empty, 5 wei. |

The destination nonce becomes nonzero before initialization under EIP-161, as EIP-1014 explains for CREATE2.
EIP-684 requires creation to throw as an invalid first initcode instruction when the destination already has a nonzero nonce **or** nonempty code.
Thus the second attempt fails even though the first returned no runtime code.
The failed attempt still consumes the creator nonce increment when its containing frame continues.
The Yellow Paper passes creation a state after that increment; failure restores that inner boundary.
An enclosing frame's later REVERT restores its earlier writes, including both increments.
This is a frame-local account-state trace, not a statement that every transaction-level effect or execution-tracking field disappears.

For a different initializer that executes REVERT, its CREATE2 address generally changes too.
The child creation deploys no code, returns zero to its caller, and exposes its error bytes through return data.
Its own writes are undone, while the creator's increment precedes that child's snapshot.
EIP-140 preserves unused gas when REVERT can pay its memory cost; insufficient gas or stack underflow instead consumes the frame's gas as an exceptional halt.

EIP-150 forwards `N - floor(N/64)` gas to a creation child after the relevant costs.
For isolated `N = 6400`, that is 6300, leaving 100 in the parent.
A collision exhausts the creation allotment under the invalid-instruction rule; it does not consume gas withheld in the parent merely by being a child failure.
These numbers illustrate forwarding, not the complete gas trace above.

## Reject at the right boundary

EIP-170 limits returned runtime code to 24,576 bytes; a larger return makes creation fail with out-of-gas behavior.
EIP-3860 instead bounds initcode at 49,152 bytes and charges `2 * ceil(length/32)` before address calculation and execution.
Equality satisfies each size cap, without guaranteeing success under other checks.

Its numbered Rules distinguish two over-limit cases:

- Oversized initcode in a creation transaction makes the transaction invalid.
- Oversized initcode supplied to CREATE or CREATE2 exceptionally aborts the executing instruction's frame, rather than merely returning zero as a successful parent step.

EIP-3860's Backwards Compatibility paragraph uses broader includable-transaction/exception wording that does not match Rule 1.
Its numbered Rules supply these cases; that disagreement is retained rather than treating every creation failure as transaction invalidity.
The pinned Yellow Paper is used for the nonce boundary, not as a substitute for these amendment rules.

## SELFDESTRUCT does not reopen the address immediately

Under EIP-6780, an older contract halts its current frame and transfers its entire balance, while retaining its account, code, and storage.
A contract created in the same transaction retains the prior deletion behavior.
Creation begins with the creation operation even when a balance already existed at the address.
That is contract creation, not necessarily the first existence of a funded account.

For a distinct beneficiary holding 11 wei and a contract holding 5 wei, the balance outcome is 0 and 16 wei in either case.
Deletion eligibility differs:

| Case, assuming execution commits | Code/storage deletion | Self-beneficiary starting with 5 wei |
| --- | --- | --- |
| Contract created before this transaction. | None under EIP-6780. | Remains 5 wei; no burn. |
| Contract created in this transaction. | Prior deletion behavior at transaction end. | Becomes zero; 5 wei is burned. |

Neither path receives a SELFDESTRUCT refund under EIP-6780's stated post-EIP-3529 rules.
A different initializer may SELFDESTRUCT during its creation, but the STOP-only address example does not execute that behavior.
EIP-1014 explicitly says SELFDESTRUCT does not immediately clear nonce or code, so same-transaction destroy-and-recreate remains blocked by collision checks.
A later enclosing REVERT also rolls back the corresponding persistent effects; EIP-161's addendum explicitly includes empty-account deletion rollback.

## EthLib observations and limits

At the recorded pin, [`launchCreate`][lean-create] increments the creator before returning either a collision result or a child message.
[`prepareCreation` and `depositCode`][lean-message] separate destination setup from restoring failed initialization.
[`processCreateMessageWith`][lean-process] passes its incoming state as the deposit snapshot, before destination preparation.
[`restoreTxState`][lean-restore] restores writes while retaining recorded created-account and read sets.
[`finishSelfdestruct`][lean-destroy] distinguishes deletion eligibility through the transaction's created-account set.

The actual deployment predicate also checks storage, and later fork branches change gas and SELFDESTRUCT behavior.
Those observations are not an audit composing every later amendment, nor a claim that EIP-684 alone describes the whole current definition.
The [proxy note](Proxies-delegation-and-storage-context.md) covers the separate code-versus-storage execution context.

Independent calculations reproduced the displayed example values.
An isolated PyCryptodome 3.23.0 calculation reproduced the published CREATE2 address and intermediate hashes.
Integer calculations checked forwarding, size constants, upfront-cost composition, and the assumed nonce/balance arithmetic.
No initializer, transaction, SELFDESTRUCT, EVM, EthLib binding, or Lean proof was executed.

[1014]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1014.md
[684]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-684.md
[161]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md
[140]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-140.md
[150]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-150.md
[170]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-170.md
[3860]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3860.md
[6780]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6780.md
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[lean-create]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Call/Create.lean#L169-L211
[lean-message]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Call/MessageCall.lean#L165-L243
[lean-restore]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/Tracker.lean#L421-L431
[lean-destroy]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/SelfDestruct.lean#L107-L159
[lean-process]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Call/MessageCall.lean#L349-L368
