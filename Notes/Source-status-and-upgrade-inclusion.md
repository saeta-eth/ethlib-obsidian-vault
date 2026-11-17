---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: P01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - protocol-upgrades
---

# Document status and upgrade inclusion

An EIP's document status, its inclusion in a network upgrade, and EthLib's support for it answer different questions.
None of those records substitutes for the others.
This note explains the distinction at the pinned sources below; it does not audit the activation of a particular upgrade.

## Sources examined

| Source | Revision and sections read | Role |
| --- | --- | --- |
| [EIP-1](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1.md) | EIP Types and EIP Work Flow, including EIP Process. | Document categories and lifecycle. |
| [EIP-7723](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7723.md) | Specification, including Upgrade Meta EIPs and the five inclusion stages. | Inclusion process for a named upgrade. |
| [EthLib architecture](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/ARCHITECTURE.md#when-a-change-enters-ethlib) | When a Change Enters EthLib. | Repository admission policy. |

At this snapshot, EIP-1 is Living and EIP-7723 is Last Call.
Those are the statuses of the process documents themselves.

## Three separate records

| Record | What it establishes | What still needs checking |
| --- | --- | --- |
| EIP document status | Where the text stands in its editorial lifecycle. | Its inclusion in the target upgrade and activation on the target network. |
| Inclusion stage in an upgrade meta EIP | The proposal's treatment in that named upgrade. | The exact revision and activation conditions for the target upgrade and network. |
| EthLib implementation at a commit | Which definitions and rules that checkout supplies. | Agreement with the source, operation-specific support, and proof hypotheses. |

EIP-1 defines Final as a final standard whose later changes are limited to errata and non-normative clarification.
It does not identify a target network or activation point through that status alone.
Stagnant and Withdrawn describe the document process too; historical text may still be useful when its role is explicit.

## Inclusion belongs to an upgrade

EIP-7723 assigns stages separately for each upgrade:

1. **Proposed for Inclusion:** the proposal is put forward for that upgrade.
2. **Considered for Inclusion:** client teams intend to try including it in devnets; this is insufficient for mainnet deployment.
3. **Scheduled for Inclusion:** teams have a strong intent to include it and consider it sufficiently mature, subject to later changes.
4. **Included:** the upgrade has activated the EIP.

**Declined for Inclusion** means exclusion from that upgrade's current plan.
It does not prevent proposing the EIP for a later upgrade.
A scheduled proposal may return to Considered or become Declined under the process.
Do not treat these stages as an irreversible sequence.

The meta EIP records inclusion.
After activation, EIP-7723 requires the activated EIPs to move to Included and other stage lists to be removed.
The stages generally concern Core EIPs; the process also permits assigning stages to non-Core EIPs.

## A conditional example

Suppose an EIP marked Final is listed for upgrade U.
The meta EIP for U lists it as Considered for Inclusion.
These facts establish a final document and a candidate for devnet work, not activation in U.

A fork-specific claim also depends on the relevant source revision, activation evidence and operation support at the chosen commit.
EthLib's admission policy is a separate question.
This example is a deduction from the two process definitions, not a claim about a particular EIP or network.

## Applying the distinction to EthLib

At the inspected commit, EthLib puts Considered or Scheduled work on the named upgrade branch.
Its architecture uses a mainnet activation recorded by the meta EIP as the signal for merging the upgrade into master, with only its Scheduled EIPs.
It also states separate entry rules for blob-parameter changes, standards outside the EIP process, and maintainer-selected exceptions.
Final alone is not that policy's admission rule.

Recognising a fork name also does not establish that every operation implements its rules.
That claim depends on the relevant definition and export support boundary.

## Review limits

The cited process sections and repository policy were read for this note.
No network activation table, complete amendment history, Lean proof, or test suite was checked.
