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
| [Validator key derivation and keystores](Validator-key-derivation-and-keystores.md) | C16 | How do seed derivation, validator paths, and keystore checks establish different facts? | Compared ERC-2333/2334/2335 revisions; published master/child and PBKDF2 fixtures; password bytes and bounded retry scope. |
| [BLS ciphersuites and verification preconditions](BLS-ciphersuites-and-verification-preconditions.md) | C08 | Which point checks, domains, and caller possession proofs qualify aggregate verification? | BLS draft-04, RFC 9380, exact tags, symbolic rogue-key example, and selected EthLib game statements. |
| [KZG openings and setup assumptions](KZG-openings-and-setup-assumptions.md) | C09 | What does an opening equation establish, and which setup and binding assumptions remain? | Versioned 2010 report, pinned EIPs, insecure scalar example, and qualified algebraic reductions; source conflict preserved. |
| [Accountable safety and validator overlap](Accountable-safety-and-validator-overlap.md) | N01 | Which shared stake and vote assumptions turn conflicting finalizations into a slashable-weight bound? | Casper FFG v4, fixed-set definitions and qualified attestation bridge; twelve-unit quorum example. |
| [Noise handshakes and libp2p peer identity](Noise-handshakes-and-libp2p-peer-identity.md) | W05 | How do the channel's static key, the signing identity, and a caller's expected Peer ID differ? | Noise revision 34 and pinned libp2p rules; exact Ed25519 identity bytes, separate from EthLib's secp256k1 profile. |
| [Engine newPayloadV3 outcomes and order](Engine-newPayloadV3-outcomes-and-order.md) | I01 | Which response is an error or payload status, and which checks and ancestry determine it? | Pinned Paris/Shanghai/Cancun inheritance and Prague boundary; observed EthLib order distinguished from unspecified source precedence. |
| [ERC-4626 rounding and vault limits](ERC-4626-rounding-and-vault-limits.md) | A07 | How do conversion estimates, previews, executable limits, and integer rounding differ? | Pinned ERC-4626 and selected ERC-7540/7575 contrasts; explicit proportional-model arithmetic. |
| [ENS namehash and resolver boundaries](ENS-namehash-and-resolver-boundaries.md) | A12 | Which bytes identify a name, and what do registry, resolver, reverse-name, and text answers establish? | Pinned ERC-137/181/634, published namehash vectors, and a DNS encoding contrast; normalized ASCII scope. |
| [Wallet RPC assessment and catalogue refresh](Wallet-RPC-assessment-and-catalogue-refresh.md) | A11 | Which wallet standards deserve follow-up, and what changed in the refreshed source census? | Bounded assessment of four EIPs, conditional switch case, metadata/body comparison, and R17 literature candidate; not a complete wallet specification. |

## Planning and discovery

- The [phased research plan](Contribution-research-plan.md#delivery-phases) defines the next batches and their completion conditions.
- The [source catalogue](Ethereum-source-catalogue.md) contains 70 candidate families and 17 research papers or design notes.
- The [standards inventory](Ethereum-standards-inventory.md) contains 1,232 numbered documents and keeps unassessed sources visible.

Catalogue entries are candidate notes, not completed source explanations.
The inventory establishes metadata provenance, not a semantic review of every standard.

## Use in a task

Name the note and the exact primary source section relevant to the contribution.
For example: "Read the transfer-log comparison and the pinned EIP-7708 ETH Transfer Logs section before changing emission conditions."
Record the EthLib commit and any source revision that differs from the note.
Keep unresolved source readings explicit.
