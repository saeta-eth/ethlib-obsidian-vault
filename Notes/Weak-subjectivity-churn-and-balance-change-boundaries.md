---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: N03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
gasper_paper_version: "arXiv:2003.03052v3"
tags:
  - ethlib
  - external-sources
  - consensus
  - weak-subjectivity
---

# Weak subjectivity, churn and balance-change boundaries

A checkpoint-age calculation is conditional on how voting weight can change after that checkpoint.
Counting validator entries and exits does not capture every balance change, and a calculated age limit does not authenticate the checkpoint supplied to it.
This note separates those obligations and traces one proposed churn calculation, including an unresolved source-unit discrepancy.

## Sources and reading scope

At the pinned EIPs snapshot, [EIP-7251][7251] is Final and [EIP-8061][8061] is Last Call, with a recorded deadline of 2026-11-01.
Passing that deadline does not alter the recorded status.
EIP-8061 says its backwards-incompatible changes MUST be activated through a scheduled network upgrade; neither its status nor this note establishes activation.
Reading covered EIP-7251's consensus constants, consensus-layer sketch and selected churn/security rationale, and EIP-8061's complete text.

The research source is [Combining GHOST and Casper, arXiv 2003.03052v3][gasper], specifically its fixed-stake setup, rewards discussion, safety argument and §§8.6–8.7.
Its research model supplies context for changing sets and weights, rather than the later EIP's period algorithm.
This reading does not audit every safety/liveness proof or the paper's correspondence to deployed forks.

## Three different changes

The paper assumes fixed validator sets and stake sizes in its main analysis.
Section 5's accountable-safety result makes conflicting finality expose at least one-third slashable stake under that model.
Section 8.6 instead bounds the slashable overlap using activations and exits measured from a reference set; elapsed time matters through the chosen churn policy.
This does not make an arbitrary overlap calculation a proof of actual finalized history.

Section 8.7 separately describes nonparticipating validators losing stake without necessarily committing a slashable offense, while participating weight can become a supermajority.
This changes weights even without replacing identities.
That section places the relevant transition and fork-handling questions outside its mathematical scope; its historical timescale is not used here as a current estimate.

EIP-7251 adds another distinction: a 32 ETH activation minimum coexists with a 2048 ETH maximum for the compounding profile.
Consolidating indices, adding a pending deposit, and changing an existing validator's effective balance consequently need different accounting.
The [shuffle note](Validator-weight-shuffling-and-committee-boundaries.md) separates index partitioning from balance-weighted proposer filtering.
A validator count alone supplies neither total active balance nor the distribution of weights among those identities.

EIP-8061's deposit pseudocode makes pending-deposit processing conditional on finalization, queue limits and other state conditions.
It postpones deposits for exiting validators, while deposits to already withdrawn validators can increase balances without consuming active-entry churn.
Its normal processing branch consumes churn for deposits that fit the available balance allowance.
Therefore, “a deposit exists” cannot be substituted for “this much effective voting weight activated now.”

## A proposed period has separate churn terms

EIP-8061 proposes separating activation, exit and consolidation allowances.
The exit share uses quotient `2^15` and a configured minimum, rounded down to the effective-balance increment.
Activations use that share with a cap; consolidations use quotient `2^16`, rounded down, without a minimum or maximum in that helper.
The allowance functions do not themselves consume a queue or schedule all state changes.

Its period pseudocode computes, in Gwei:

```text
delta = floor(2*exit/3) + floor(activation/3) + consolidation
period = MIN_VALIDATOR_WITHDRAWABILITY_DELAY
       + floor(SAFETY_DECAY * total / (200 * delta))
```

The factor two in the denominator corresponds to the rationale's exit/activation/consolidation effects of `4/3`, `2/3` and `2`.
The divisions in the code are separate integer divisions; they must not be replaced silently by a single real-valued formula.
A different churn policy, decay target, balance unit, or state changes the input to this calculation.
Inactivity penalties and changes to effective balances need their own bounds rather than an assumption that membership churn covers them.

## Complete conditional arithmetic, with the unit choice exposed

The source prose specifies a 256 ETH activation cap, but its configuration table lists `uint64(256)` and its Gwei-returning helper compares directly against that constant.
Those passages do not explicitly reconcile the unit conversion.
This trace uses the prose's 256 ETH interpretation, written as `256000000000` Gwei, and separately computes the literal 256-Gwei alternative below.
It does not repair the source or choose a deployment configuration.

Choose total active balance `S = 36000000 ETH = 36000000000000000 Gwei`, increment 1 ETH, a minimum churn of 128 ETH, decay parameter 10 and withdrawal delay 256 epochs.
The selected minimum lies below the computed exit share.
These are fixed illustrative calculation inputs, not a live stake observation or a simulated sequence of epochs.

| Term | Calculation | Gwei result |
| --- | --- | --- |
| Exit E | Round `floor(S/32768)` down to a 1 ETH increment. | 1098000000000. |
| Activation A | `min(256000000000, E)`. | 256000000000. |
| Consolidation C | Round `floor(S/65536)` down to a 1 ETH increment. | 549000000000. |
| Exit contribution | `floor(2*E/3)`. | 732000000000. |
| Activation contribution | `floor(A/3)`. | 85333333333. |
| Weighted churn delta | Sum the preceding two contributions and C. | 1366333333333. |

The complete period arithmetic is:

```text
10*S              = 360000000000000000
200*delta         =    273266666666600
floor(10*S/(200*delta)) = 1317
period            = 256 + 1317 = 1573 epochs
```

The displayed intermediate products are below `2^64`, and the divisor is positive.
At the source's conversion of 225 epochs per day, that is `1573/225`, approximately 6.991 days, conditional on these inputs and unit interpretation.
Using the literal `A = 256` Gwei instead gives `delta = 1281000000085` and 1661 epochs.
That difference prevents treating either result as the prescribed configuration without resolving the unit discrepancy.
Neither result establishes that a particular checkpoint remains safe.

## What the selected Lean statements establish

At the recorded EthLib commit, [Churn.lean][lean-churn] defines the distinct fork-dependent allowances and queue reservation; [WeakSubjectivity.lean][lean-period] defines period arithmetic and checkpoint-age checks.
These are source-code observations, not activation evidence.
The selected [`balanceChurnPeriod_eq_ok_iff` theorem][lean-closed] gives the closed form exactly when `10*total < 2^64`, `200*churn < 2^64`, churn is positive, and the resulting delay-plus-quotient is below `2^64`.
This is a statement about checked arithmetic.

The [`safetyMargin_of_balanceChurnPeriod_eq_ok` theorem][lean-margin] additionally assumes a bound of the form:

```text
total <= 3*x + 2*((period-minDelay)*(3*churn))
```

Together with successful period computation, it concludes `70*total <= 300*x`, equivalent to the `7/30` margin for decay 10.
The supplied bound is a separate hypothesis; computing 1573 does not establish it for execution traces, balance leaks or an adversarial history.
The selected statements and their proofs were read, without Lean replay.

The [checkpoint check][lean-check] first needs a successful block-root lookup matching the supplied checkpoint and a state in its epoch.
It then needs successful period/current-slot computations and checked addition before comparing the current epoch with the inclusive endpoint.
For checkpoint epoch 100, the illustrative period 1573 ends at 1673.
The time comparison accepts 1673 and rejects 1674, conditional on all preceding checks succeeding.
This is neither a finalized-chain proof nor a trusted-bootstrap procedure.
As the [Beacon API note](Beacon-API-identifiers-and-response-evidence.md) explains, a node's returned root or finalized flag supplies no independent checkpoint authentication.

## Checks and remaining questions

Standalone Python integer calculations checked the Gwei allowances, separate floors, both unit interpretations, overflow bounds and conditional endpoint comparisons.
No beacon transition, deposit/exit queue, inactivity-leak run, checkpoint authentication, RPC, EthLib binding, Lean build or proof check ran.
Using the period as a checkpoint-safety limit requires a selected fork and configuration, resolved units, justified bounds on membership and effective-weight changes, and an explicit trust basis for the bootstrap checkpoint.

[7251]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7251.md
[8061]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8061.md
[gasper]: https://arxiv.org/abs/2003.03052v3
[lean-churn]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Churn.lean#L46-L132
[lean-period]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/WeakSubjectivity.lean#L116-L164
[lean-closed]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proofs/WeakSubjectivity.lean#L60-L90
[lean-margin]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proofs/WeakSubjectivity.lean#L109-L129
[lean-check]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/WeakSubjectivity.lean#L166-L183
