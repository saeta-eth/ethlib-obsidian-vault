---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
delivery_batch: "follow-up-15"
catalogue_family: N07
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - consensus
  - proposer-builder-separation
  - inclusion-lists
---

# Enshrined PBS and inclusion-list obligations

A payload commitment, a timely reveal, execution validity and a proposer payment establish different facts.
An inclusion list adds another obligation, evaluated against a particular validator's collected lists and the payload's final execution state.
This note traces selected proposal rules without treating them as an activated, fully composed protocol.

## Sources and scope

At the pinned EIPs snapshot, [EIP-7732][7732] is Last Call with deadline 2026-11-01 and declares dependency EIP-7685; [EIP-7805][7805] is Draft with no `requires` header.
An elapsed deadline does not change the recorded status, and absence of that header does not remove textual dependencies.
Reading covered EIP-7732's abstract, selected containers/constants, transition/fork-choice/P2P/actor summaries, payment/validation rationale and security qualifications, and the complete EIP-7805 text.
Linked consensus-specs changes were not read for this note; the proposal summaries do not establish every inherited check or exact wire schema.

The [external Builder/Relay API note](Builder-relay-bids-and-payload-release-boundaries.md) concerns a separate interface and trust model.
Here the proposed builder has a beacon-state registry and balance, with consensus-layer pending payments and withdrawals.
That distinction prevents importing the external relay's signature object, wei-valued bid or payment-simulation contract into an enshrined bid.

## Commitment, reveal and attestation have different actors

EIP-7732 replaces the full payload in the beacon block with a `SignedExecutionPayloadBid`.
Its bid names execution and beacon parents, the committed execution block hash, request root, slot, builder, fee recipient, gas limit and amounts in Gwei.
The beacon-block step checks an active builder and sufficient balance before recording a pending payment.
An `ExecutionPayloadEnvelope` later supplies the payload and execution requests, tied to the selected builder and beacon block.

The Payload Timeliness Committee (PTC) reports whether the payload was timely and blob data was available in its members' views.
PTC members perform basic commitment/signature checks; they are not required to execute the payload before voting.
The fork-choice summary separately requires local payload verification before preferring a full-payload node.
An honest builder may withhold under the stated allowance when the including beacon block was both late and not the head in that builder's view.
That condition does not make arbitrary withholding harmless; the security section separately discusses economically motivated withholding and collusion risks.

Envelope verification does not itself modify the beacon state in this proposal.
If a child builds on the committed parent payload, child-block processing checks the parent's request root and applies its consensus effects, including payment settlement and requests.
If it does not build on that payload, the child's `parent_execution_requests` field must be empty.
The expected-withdrawal mechanism is separate: withdrawals are deducted when their beacon block is processed, and a corresponding execution payload MUST contain the committed expected list.
These stages cannot be collapsed into “the payload arrived, therefore payment and execution completed.”

## Conditional timing and payment arithmetic

Choose a hypothetical 12-second slot.
EIP-7732's 2500/5000/7500 basis-point deadlines become 3 seconds for attestations, 6 seconds for payload arrival and 9 seconds for PTC attestations.
For its PTC size 512, the fork-choice fragment uses strictly more than 256 votes for each of timely payload and available data.
Counts `(256,257)` or `(257,256)` fail that conjunction; `(257,257)` passes it.
Assume local verification and every other applicable condition; these counts alone select no global canonical head.
The alternative involving a proposer-boosted child is not exercised.

For a separate balance-reservation case, choose:

| Input | Gwei |
| --- | --- |
| Builder balance | 2000000000. |
| Assumed minimum deposit | 1000000000. |
| Prior pending payments and withdrawals | 100000000. |
| New bid value, with execution payment zero | 25000000. |

The required reservation is `1000000000 + 100000000 + 25000000 = 1125000000` Gwei, leaving headroom `875000000` Gwei.
The bid value is 0.025 ETH.
Assume builder activity, signatures, parents, preferences and all other bid conditions; this arithmetic only checks balance coverage.
A reservation or pending-payment record is not an execution-layer receipt.

The proposal permits settlement through parent-payload inclusion, or epoch processing of a sufficiently attested beacon block even if its payload was not included.
For an independently stipulated per-slot active balance of 100 ETH, six tenths is 60 ETH.
Same-slot weight 59 ETH misses that threshold; 60 ETH reaches it when the payment is in the designated processing window.
This measures attesting balance, not a count of PTC votes.
Pending payments of a slashed proposer are removed under the stated rule.
Withdrawal queuing, deduction and execution-layer credit must still be distinguished; no transfer was executed here.

## FOCIL freezes a local view before checking payload-end eligibility

EIP-7805 describes its own timeline: validators freeze new-list storage at slot N's 9-second deadline, builders update payloads around 11 seconds, and attesters decide at slot N+1's 4-second deadline.
The builder timing is explicitly provisional pending tests/benchmarks.
Late lists can still be forwarded, and equivocation evidence continues to be recorded until the attestation deadline.
This is not automatically the 3-second attestation schedule calculated above.

Assume all gossip signature, membership, root, size and slot checks succeed for these constructed arrivals at one validator:

| Slot N time | Arrival | Effect in the described policy |
| --- | --- | --- |
| 8.0 seconds | First list from A. | Store for the frozen view. |
| 8.5 seconds | First list from B. | Store for the frozen view. |
| 9.5 seconds | First list from C. | Forward, without adding it to that frozen view. |
| 10 seconds | A different list from A. | Record equivocation; exclude A's lists. |

Only B's retained list constrains this validator in the resulting case.
This does not assert every validator received the same lists.
Gossip acceptance performs no execution-layer transaction validation, so a retained list need not contain executable transactions.

For one transaction T from B's list, use payload-end state S and `gas_left = 30000`.
Each row independently assumes all unrelated conditions succeed:

| T's condition | Result of the selected check |
| --- | --- |
| Already present in the payload. | Continue to the next transaction. |
| Missing, with gas limit 40000. | Continue: insufficient remaining gas. |
| Missing, gas limit 21000, nonce/balance check fails against S. | Continue: invalid at the payload's end. |
| Missing, gas limit 21000, nonce/balance check passes against S. | Return `INCLUSION_LIST_UNSATISFIED`. |

The last outcome makes the consensus layer withhold its attestation under the proposal; the text distinguishes it from execution-block invalidity.
It does not require forcing every listed transaction into every payload, executing T, or rechecking against its earlier mempool state.
Builder construction may need repeated updates to its post-state, which the payload-construction discussion treats separately from this terminal-state check.

## Composition questions and selected EthLib context

EIP-7732 states high-level compatibility with fork-choice inclusion lists, but these summaries do not supply a complete joint transition.
Their timing baselines differ, and both assign domain-type bytes `0x0C000000`: PTC attestation in EIP-7732 and IL committee signing in EIP-7805.
This is a domain-allocation/composition question, not a demonstrated cross-message signature attack.
EIP-7732 also distinguishes in-protocol value from trusted payment promises elsewhere, while gossip bids must carry no execution payment.
All worked bid amounts here use `execution_payment = 0`.

At the pinned EthLib commit, [`canBuilderCoverBid`][lean-builder] checks minimum balance and pending obligations, and [`releasedPayments`][lean-payments] filters by an inclusive weight threshold.
[`InclusionListStore.process`][lean-il] takes timeliness as an input and records differing lists as equivocation; counted entries can exclude untimely lists and equivocators.
Its keys use `dependentRoot`, whereas EIP-7805's displayed container uses `inclusion_list_committee_root`; these schemas must not be substituted silently.
These are selected definition observations, without whole-module, proof-coverage or source-agreement claims.

Python checked the timing fractions, strict-majority boundaries, reservation/threshold arithmetic, conditional arrival policy and terminal-state cases.
No block, state transition, payload execution, valid transaction/signature, P2P network, payment, Merkle/BLS verification, EthLib binding, Lean build or proof replay ran.
A contribution still needs a selected joint fork revision, reconciled timing/domains/schemas and explicit execution and network assumptions.

[7732]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7732.md
[7805]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7805.md
[lean-builder]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Builders.lean#L110-L118
[lean-payments]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Transition/BuilderPayments.lean#L34-L37
[lean-il]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/InclusionList.lean#L58-L79
