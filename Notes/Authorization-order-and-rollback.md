---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E02
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - authorizations
---

# Authorization order and rollback

A valid EIP-7702 transaction can contain authorizations that it skips.
A skipped tuple can still warm its recovered authority, while accepted delegations survive a later execution revert.
These outcomes depend on validation order and on where failure occurs.
Transaction invalidity and the later proposals' failure before the first frame need separate treatment.

## Sources and applicability

All EIP links below pin commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
The example uses the EIP-7702 baseline, without the later EIP-2780/EIP-8037 pricing changes.
Document status does not establish network activation.

| Source | Passages examined and role |
| --- | --- |
| [EIP-7702][e7702] | Complete text, especially Set code transaction, Behavior, Gas Costs, Transaction origination, and Charge maximum cost upfront. Final at this revision. |
| [EIP-3607][e3607] | Complete text, especially Specification. Earlier sender-code restriction, amended by EIP-7702. Final. |
| [EIP-161][e161] | Specification's empty-account definition. An account with no code, zero nonce, and zero balance is empty. |
| [EIP-2780][e2780] | Runtime gas costs and EIP-7702 authorization processing. Separately versioned pre-execution and pricing amendment, in Last Call. |
| [EIP-8037][e8037] | Gas accounting for halts and reverts, and authorization cross-reference. State-gas amendment, in Last Call. |

The [typed-transaction note](Typed-transaction-envelopes-and-signing.md) covers the outer envelope.
The [signed-data note](Signed-data-preimages-and-domains.md) explains other signing domains.
An authorization uses its own prefix, `0x05`, followed by RLP of its chain ID, delegation address, and nonce.
Its digest is Keccak-256 of those bytes.
The tuple's `address` names the delegation target, not its signer, whom recovery identifies as the **authority**.

## Three validation outcomes

EIP-7702 makes a type-4 transaction invalid if its authorization list is empty or its destination is null.
It also makes it invalid when an authorization field exceeds its structural bound:

| Field | Structural requirement |
| --- | --- |
| Chain ID, `r`, `s` | Each fits an unsigned 256-bit value. |
| Nonce | Fits an unsigned 64-bit value. |
| Address | Has exactly 20 bytes. |
| Signature parity | Fits an unsigned byte. |

These cases do not become valid by skipping the malformed tuple.
An invalid transaction cannot be applied as an included transaction.
Within a structurally valid list, a failed processing check instead skips that tuple and continues.
A tuple passing those checks changes its authority's code and nonce.

The bounds differ deliberately: nonce `2^64 - 1` fits the tuple but fails processing, which requires strictly less than `2^64 - 1`.
Nonce `2^64` fails the structural bound.
Likewise, fitting the parity byte does not establish successful signature recovery.

EIP-3607's Specification requires rejection when the sender's code hash differs from the empty-code hash.
EIP-7702's Transaction origination section permits a valid delegation indicator as sender code, while retaining rejection for other code.
Read the amendment with the earlier restriction.

## The ordered authorization steps

Process the list after incrementing the transaction sender's nonce and before executing its message.
For each tuple, EIP-7702's Behavior section gives this order:

1. Require chain ID zero or the current chain's ID.
2. Require authorization nonce below `2^64 - 1`.
3. Recover the authority from its authorization digest and check the low-`s` condition, `s <= secp256k1n/2`.
4. Add the authority to `accessed_addresses`.
5. Require its code to be empty or a delegation indicator.
6. Require its current nonce to equal the tuple's nonce.
7. Add `25000 - 12500 = 12500` to the refund counter if the authority is not empty.
8. Set its code to `0xef0100 || address`, except that address zero clears the code instead.
9. Increment its nonce.

Stop a failing tuple immediately and continue with the next.
Failure before step 4 does not warm an authority.
Failure at steps 5 or 6 leaves the recovered authority warm, but adds no refund and changes neither nonce nor code.
This is a consequence of the prescribed order, also explicit in EIP-2780's authorization subsection.
Chain ID zero broadens applicability across chains, while the current nonce and other checks still apply.

An accepted authorization changes the nonce seen by subsequent tuples.
Thus the last accepted occurrence for an authority determines its final delegation.
The baseline intrinsic charge includes `25000` for every tuple, including skipped tuples and duplicates.
A refund-counter credit is not an immediate payment and does not establish the final gas refund.

## Worked trace with stale duplicates

Use chain ID 1 and these distinct addresses:

```text
S  = 0x1111111111111111111111111111111111111111
A  = 0x2222222222222222222222222222222222222222
D1 = 0x3333333333333333333333333333333333333333
D2 = 0x4444444444444444444444444444444444444444
C  = 0x5555555555555555555555555555555555555555
```

Assume a valid outer type-4 transaction from S to C, with nonce 9, zero value, empty calldata and access list, and enough gas and funding.
Before it, S has nonce 9 and empty code.
A exists with nonce 4, zero balance, and empty code, so A is not empty under EIP-161.
A is initially outside the accessed-address set `W0`.
The authorization refund contribution starts at zero.

For every tuple below, assume an in-range, low-`s` signature that recovers A for that tuple's exact preimage.
Rows 2 and 3 are identical tuples, including their signature.
These are explicit recovery assumptions, not published signature vectors.

| Row: `(chain ID, target, nonce)` | Outcome | A nonce | A code | Accessed set | Authorization refund credit |
| --- | --- | --- | --- | --- | --- |
| Start after sender nonce increment | S now has nonce 10. | 4 | Empty | `W0` | 0 |
| 1: `(1,D1,3)` | Skipped at nonce check, after warming A. | 4 | Empty | `W0 ∪ {A}` | 0 |
| 2: `(1,D1,4)` | Accepted. | 5 | `0xef0100 || D1` | `W0 ∪ {A}` | 12500 |
| 3: `(1,D1,4)` | Stale duplicate, skipped at nonce check. | 5 | `0xef0100 || D1` | `W0 ∪ {A}` | 12500 |
| 4: `(0,D2,5)` | Accepted using chain ID zero. | 6 | `0xef0100 || D2` | `W0 ∪ {A}` | 25000 |

Two accepted tuples add two nonces and two refund credits.
All four tuples contribute `4 * 25000 = 100000` to the baseline intrinsic charge.
The last accepted target is D2.
If row 4 instead targets address zero with a signature for that changed preimage, it clears the code and still raises A's nonce to 6.

Now assume C's first frame starts and later executes REVERT.
EIP-7702 explicitly preserves the processed delegation indicators after execution failure.
The source-derived authorization state is A nonce 6 with delegation to D2, while S's transaction increment remains 10.
The table calculates authorization-phase credits, not the settled transaction bill.
For sender-as-authority, the corresponding nonce check would start after its transaction increment: a sender starting at 9 needs authorization nonce 10 for its first acceptance.

## A later failure boundary

EIP-2780 separates intrinsic validity from runtime charges before the first frame.
Under that amendment, running out of gas there keeps the transaction included but undoes pre-execution changes, including authorizations, while retaining the sender's nonce increment and gas payment.
Failure after frame entry instead keeps the authorizations.
EIP-8037 correspondingly distinguishes restoring the frame's state-gas baseline from undoing the entire pre-execution phase.
Its authorization charges refer to EIP-2780.
The baseline trace above does not apply its prices or refill formulas.

## EthLib definitions, theorems, and open interpretation

At the recorded commit, [`applyAuthorization`][apply] and [`applyAuthorizations`][fold] implement recovery, warming, acceptance, and ordered processing.
[`applyAuthorization_of_nonce_ne`][nonce] proves the warming-only result for a nonce mismatch, assuming scope, recovery, and readable code.
An internal missing-code error is therefore different from a refused tuple.

[`processTransaction_authority_of_error`][failed] states the surviving authority nonce and code hash when the transaction runs and its message fails, assuming initially readable delegable code.
From Glamsterdam it additionally assumes that the first frame starts.
[`processTransaction_authority_of_firstFrameHalts`][halt] covers the other branch.
These theorem statements were read, not replayed.
The definition [`CallOutput.ofFrame`][output] retains the earlier authorization refund when its frame fails.
That observation is about the definition, not a new refund theorem.

One source distinction remains explicit: EIP-7702 Behavior refunds an authority that is **not empty**, whereas its upfront-charge rationale says **already exists**.
EthLib's [`delegate`][delegate] uses existence and documents the difference for an existing empty account.
The trace avoids that case through A's nonzero nonce.
This note does not resolve the difference or establish complete conformance.
The selected sources, theorem statements, and explanation received a separate review.
An independent calculation followed the source's ordered rules under the stated recovery assumptions and reproduced the table's nonces, code, warming, and refund credits.
It also checked the 100000-gas authorization contribution and the two nonce bounds.
No Lean build, proof check, execution test, signature-vector test, or network-adoption audit was run.

[e7702]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md
[e3607]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3607.md
[e161]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md
[e2780]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2780.md#runtime-gas-costs
[e8037]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8037.md#gas-accounting-for-halts-and-reverts
[apply]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Authorization.lean#L211
[fold]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Authorization.lean#L237
[nonce]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Proofs/Transactions/Authorization.lean#L471
[failed]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Proofs/Transactions/Failed.lean#L1209
[halt]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Proofs/Transactions/Failed.lean#L1242
[output]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Process.lean#L214
[delegate]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Authorization.lean#L142
