---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: working
tags:
  - ethlib
  - external-sources
  - index
---

# External source index

Use these notes to find the primary text and the distinctions relevant to an EthLib contribution.
A note's reviewed status means its stated source passages and explanation were reviewed; it does not establish formal correctness or network adoption.
Read the source section for the rule being changed, and check its version and applicability for the task.

## Source notes

| Note | Family | Question it answers | Review scope |
| --- | --- | --- | --- |
| [Document status and upgrade inclusion](Source-status-and-upgrade-inclusion.md) | P01 | How do document status, inclusion in an upgrade, and EthLib support differ? | Pinned EIP-1 and EIP-7723 process sections and EthLib's admission policy. |
| [ERC-20 events and native ETH transfer logs](ERC-20-and-native-ETH-transfer-logs.md) | A01 | Which event fields are shared, and where do emission, units, and rollback differ? | Pinned ERC-20, EIP-7708 and related proposal text; source-derived byte-layout example. |

## Planning and discovery

- The [phased research plan](Contribution-research-plan.md#delivery-phases) defines the next batches and their completion conditions.
- The [source catalogue](Ethereum-source-catalogue.md) contains 70 candidate families and 15 research papers or design notes.
- The [standards inventory](Ethereum-standards-inventory.md) contains 1,232 numbered documents and keeps unassessed sources visible.

Catalogue entries are candidate notes, not completed source explanations.
The inventory establishes metadata provenance, not a semantic review of every standard.

## Use in a task

Name the note and the exact primary source section relevant to the contribution.
For example: "Read the transfer-log comparison and the pinned EIP-7708 ETH Transfer Logs section before changing emission conditions."
Record the EthLib commit and any source revision that differs from the note.
Keep unresolved source readings explicit.
