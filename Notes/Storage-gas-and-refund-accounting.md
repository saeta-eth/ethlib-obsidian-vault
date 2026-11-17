---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 3
delivery_batch: "follow-up-1"
catalogue_family: E03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - evm
  - storage-gas
  - refunds
---

# Storage gas and refund accounting

An `SSTORE` charge, its refund-counter change, and the transaction's applied refund are different quantities.
Derive each from the slot's original, current, and new values before explaining the final gas used.
A refund does not supply gas to an executing call frame.

## Sources and historical scope

This note assumes the historical London composition of the following rules at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`:

| Source | Selected passages and role |
| --- | --- |
| [EIP-2200][2200], Specification. | Original/current/new metering, refund adjustments, and the low-gas sentry. |
| [EIP-2929][2929], Specification, Storage read changes, and SSTORE changes. | Cold/warm storage access and amended EIP-2200 parameters. |
| [EIP-3529][3529], Specification, Rationale, Backwards Compatibility, and Test Cases. | Reduced clearing refund, removed `SELFDESTRUCT` refund, and the transaction refund cap. |
| [EIP-1283][1283], Specification definitions only. | Historical vocabulary explicitly referenced by EIP-2200, without adopting its earlier numeric rules. |
| [EIP-2930][2930], Parameters and access-list initialization. | The storage-key cost used by EIP-3529's formula, and paid prewarming as a separate input. |

The three main EIPs are Final at this pin, but their metadata is not an activation audit.
In particular, EIP-3529's `FORK_BLOCK` remains `TBD` in the text.
The calculations assume these rules apply, rather than infer a network activation date or block.
Later amendments, including EIPs 7623, 7778, 8037, and 8038, are outside this calculation.
Applying it to a later fork requires reading and composing those rules separately.

## Keep the three values and access state apart

For one `(storage address, key)` pair, write:

```text
o = original value, before the current transaction
c = current value, immediately before this SSTORE
n = new value requested by this SSTORE
```

EIP-1283 describes the original value as the value obtained if the current transaction reverts.
It is not a new snapshot at every internal call.
A slot is clean for the comparison when `o = c`, and dirty when `o != c`.
Restoring the original value makes it clean again for the next comparison, even though it can remain warm.

EIP-2929 tracks accessed storage pairs separately from accessed addresses.
A cold `SSTORE` adds 2,100 gas to the metering below and inserts the pair into the accessed set.
A warm `SSTORE` adds no cold surcharge.
A prior surviving access or transaction access-list entry can make the pair warm.
Prewarming through EIP-2930 has its own upfront gas charge, so it is not a free discount.
The examples start cold and assume no access-list prewarming.

The composed parameters are:

| Quantity | Gas |
| --- | ---: |
| Warm read/no-op/dirty-write base, `SLOAD_GAS`. | 100 |
| Cold storage surcharge. | 2,100 |
| Clean zero-to-nonzero base, `SSTORE_SET_GAS`. | 20,000 |
| Clean nonzero-change base, `SSTORE_RESET_GAS`. | `5,000 - 2,100 = 2,900` |
| Clearing refund, `SSTORE_CLEARS_SCHEDULE`. | `2,900 + 1,900 = 4,800` |
| Restore-original-zero refund. | `20,000 - 100 = 19,900` |
| Restore-original-nonzero refund. | `2,900 - 100 = 2,800` |

The 2,100 cold surcharge does not replace the 100-gas base of a no-op or dirty write.
Thus a cold no-op costs 2,200, while a warm no-op costs 100.

## Calculate the charge and counter separately

First apply EIP-2200's sentry: if `gasleft <= 2300`, `SSTORE` fails the current frame with out-of-gas.
Even a warm 100-gas no-op must pass that check.
Passing the sentry does not guarantee enough gas for the subsequent charge.

For a permitted write, these are the warm base cases:

| Condition | Charge | Refund-counter change |
| --- | ---: | --- |
| `c = n`. | 100 | Zero. |
| `o = c = 0`, `n != 0`. | 20,000 | Zero. |
| `o = c != 0`, `n != c`. | 2,900 | Add 4,800 if `n = 0`. |
| `o != c`, `n != c`. | 100 | Apply the adjustments below together. |

In the dirty case, if `o != 0`, remove 4,800 when `c = 0`, and add 4,800 when `n = 0`.
Additionally, if `n = o`, add 19,900 for original zero or 2,800 for original nonzero.
The adjustments are cumulative, not an exclusive choice of one refund branch.
For example, `o = 1, c = 0, n = 1` changes the counter by `-4,800 + 2,800 = -2,000`.
EIP-2200 warns that a call-frame counter can be negative, so a representation that silently clips each local adjustment to zero changes the accounting.

There is a numerical inconsistency in EIP-3529's Rationale: its nonzero-restore discussion says 4,900 and refers to a preceding 5,000 charge.
The mandatory EIP-2929 parameter amendment instead makes that restore refund 2,800, which EIP-3529's own test tables also show.
These calculations follow the composed Specification formulas and record the differing prose explicitly.

## Trace one slot from zero and back

Assume a non-static frame, sufficient gas for every step and sentry, successful writes, no reverted scope, and no other refunds.
The original value stays zero throughout:

| Write | `o,c,n` | Access | Gross charge | Counter change | Counter after |
| --- | --- | --- | ---: | ---: | ---: |
| Set 1. | `0,0,1` | Cold. | `2,100 + 20,000 = 22,100` | 0 | 0 |
| Restore 0. | `0,1,0` | Warm. | 100 | +19,900 | 19,900 |

The two `SSTORE`s consume 22,200 gas during execution.
Subtracting the full counter gives 2,300, but that is an uncapped storage subtotal, not the transaction's charged gas.
It excludes intrinsic gas, stack preparation, and every other instruction.

For a separate settlement illustration, assume the whole transaction consumes `G = 50,000` gas before refunds.
That includes the 22,200 storage charge and 27,800 gas of other costs.
Let its final eligible refund counter be `R = 19,900`.
Under EIP-3529:

```text
refund_cap     = floor(G / 5) = 10,000
refund_applied = min(R, refund_cap) = 10,000
gas_used       = G - refund_applied = 40,000
```

The unused part of the counter does not become spendable gas during execution.
This is assumed transaction accounting, not an executed bytecode vector.

## Clear, replace, and restore a nonzero original

Start a separate transaction with original value 1 and the same execution assumptions:

| Write | `o,c,n` | Access | Charge | Counter change | Counter after |
| --- | --- | --- | ---: | ---: | ---: |
| Clear. | `1,1,0` | Cold. | `2,100 + 2,900 = 5,000` | +4,800 | 4,800 |
| Replace with 2. | `1,0,2` | Warm. | 100 | -4,800 | 0 |
| Restore 1. | `1,2,1` | Warm. | 100 | +2,800 | 2,800 |

Gross storage cost is 5,200 and the final counter is 2,800, before applying the whole transaction's cap.
The clearing refund was removed when the slot stopped being zero.
It cannot be retained alongside the later restore refund merely by counting every clear as an independent credit.

## Scope rollback and EthLib connections

EIP-2929 makes the accessed sets transaction-wide but requires rollback to their earlier state when an inner scope reverts.
An access already present before entering that scope remains present.
The examples avoid reverts, so their first access stays warm for later writes.

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`sstoreCost`, `sstoreRefund`, and `sstoreStep`][storage] separate the gross price, signed counter delta, and instruction execution.
[`getStorageOriginal`][original] reads the transaction's original storage view, rather than the latest write.
[`incorporateChild`][child] merges a child's refund and accessed sets on success and retains the caller's corresponding substate on failure.
Those inspected declarations connect the counter and access scopes without establishing complete EVM conformance.

[`gasRefund` and `gasUsed`][settlement] separately cap the counter and calculate settlement.
At `.london`, their formula is the one used above.
Their later-fork branches are another reason to name the fork instead of presenting a timeless gas calculator.

The selected source passages and cited definitions were read at their exact pins.
The examples were derived by hand from the composed formulas.
The source composition, explanation, and cited definitions received a separate review.
An independent source-model calculation reproduced both traces, the signed refund deltas, the assumed transaction cap, and sentry/no-op controls.
No Lean build, proof replay, EVM execution, transaction simulation, or full later-amendment review was performed.
This note supplies contribution background and does not identify an implementation gap.

[2200]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2200.md#specification
[2929]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2929.md#sstore-changes
[3529]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3529.md#specification
[1283]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1283.md#specification
[2930]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2930.md#parameters
[storage]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/Storage.lean#L99
[original]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/Tracker.lean#L219
[child]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Call/Child.lean#L76
[settlement]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Transactions/Settlement.lean#L169
