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
| [RLP: canonical bytes and typed values](RLP-canonical-encoding.md) | S01 | Which bytes encode generic items, and which extra conditions belong to integer fields or typed envelopes? | Pinned Yellow Paper Appendix B, Coglio's selected decoder sections, EIP-2718; nested and boundary calculations. |
| [ABI encoding and event topics](ABI-encoding-and-event-topics.md) | S04 | Where do offsets start, and which event fields encode directly or contribute a hash? | Pinned Solidity v0.8.30 ABI sections; complete argument encoding, packed ambiguity, and indexed-value rules. |
| [Signed-data preimages and domains](Signed-data-preimages-and-domains.md) | A03 | Which bytes form an ERC-191/EIP-712 message, and which replay obligations remain with the application? | Pinned ERC-191, EIP-712 and published examples; independent recomputation of the complete Ether Mail digest. |
| [Typed transaction envelopes and signing](Typed-transaction-envelopes-and-signing.md) | E01 | How do transaction bytes, signing preimages, receipt payloads, and blob network wrappers differ? | Pinned EIP-155/2718/2930/1559/4844/7702 sections; unsigned type-2 byte example and fee arithmetic. |
| [Authorization order and rollback](Authorization-order-and-rollback.md) | E02 | When is a transaction invalid, when is a tuple skipped, and which state survives execution failure? | Pinned EIP-3607/7702 rules; assumed-recovery nonce/code trace; separately scoped EIP-2780/8037 amendments. |
| [Proxies, delegation, and storage context](Proxies-delegation-and-storage-context.md) | A10 | Which account supplies code, and which account's address, caller, value, and storage does that code use? | Pinned EIP-7/140/7702 and ERC-1167/1967/2535 sections; runtime bytes, slot calculation, and source-derived context trace. |

## Planning and discovery

- The [phased research plan](Contribution-research-plan.md#delivery-phases) defines the next batches and their completion conditions.
- The [source catalogue](Ethereum-source-catalogue.md) contains 70 candidate families and 16 research papers or design notes.
- The [standards inventory](Ethereum-standards-inventory.md) contains 1,232 numbered documents and keeps unassessed sources visible.

Catalogue entries are candidate notes, not completed source explanations.
The inventory establishes metadata provenance, not a semantic review of every standard.

## Use in a task

Name the note and the exact primary source section relevant to the contribution.
For example: "Read the transfer-log comparison and the pinned EIP-7708 ETH Transfer Logs section before changing emission conditions."
Record the EthLib commit and any source revision that differs from the note.
Keep unresolved source readings explicit.
