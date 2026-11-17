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
| [SSZ offsets, roots, and branches](SSZ-offsets-roots-and-branches.md) | S02 | How do one container's bytes relate to its list root, length mix-in, and proof address? | Pinned historical prose and executable-project passages; a packed-list example and a two-sibling branch. |
| [Storage gas and refund accounting](Storage-gas-and-refund-accounting.md) | E03 | How do original/current/new values, warmth, and a transaction refund cap interact? | Pinned storage EIPs composed for London; two conditional SSTORE traces and a retained rationale discrepancy. |
| [HKDF extract, expand, and context](HKDF-extract-expand-and-context.md) | C04 | Which bytes distinguish salt, keying material, expansion context, and the output bound? | RFC 2104 and RFC 5869 sections; published SHA-256 Test Case 1, expansion intermediates, and caller assumptions. |
| [JSON-RPC quantities and block state](JSON-RPC-quantities-and-block-state.md) | S05 | Which encodings and block selectors identify a state query, and what does a returned proof establish? | Pinned EIP-1474/1898/1186 passages; conditional requests and a separately trusted state root. |
| [ENR record bytes and identity](ENR-record-bytes-and-identity.md) | W01 | Which bytes authenticate an ENR and derive its node address, and what remains an endpoint claim? | Pinned EIP-778 and devp2p passages; published record, content preimage, and v4 key/address encodings. |
| [Permit nonces and allowance authority](Permit-nonces-and-allowance-authority.md) | A04 | How do owner and token nonces distinguish two signed approvals from token transfer authority? | Pinned ERC-2612/4494 with ERC-20 and EIP-712 context; conditional state trace and synthetic signing bytes. |
| [Keccak, SHA-3 padding, and domains](Keccak-SHA3-padding-and-domains.md) | C01 | Which suffix and padding bits distinguish Ethereum Keccak-256 from SHA3-256? | FIPS 202, original Keccak reference, and pinned Yellow Paper; complete byte blocks and boundary calculations. |
| [Discovery v5 challenge and key binding](Discovery-v5-challenge-and-key-binding.md) | W02 | How do the challenge, identity signature, shared point, and directional keys bind different inputs? | Pinned devp2p v5.1 passages and separate published primitive fixtures; conditional actor trace and caller obligations. |
| [Token receiver interfaces and callbacks](Token-receiver-interfaces-and-callbacks.md) | A02 | How do interface discovery and magic return values differ for one accepted transfer? | Pinned ERC-165/721/1155 sections; selector/XOR calculations and conditional callback traces with source tensions retained. |
| [MODEXP precompile bytes and gas](Modexp-precompile-bytes-and-gas.md) | E06 | How do declared lengths, zero padding, fixed-width output, and fork-specific gas qualify modular arithmetic? | Pinned EIP-198/2565 with Yellow Paper conventions; short-exponent examples and separately scoped later amendments. |
| [Slashing protection imports and missing roots](Slashing-protection-imports-and-missing-roots.md) | I05 | Which refusal floors and repeat evidence survive import, missing roots, and repeated files? | Complete pinned EIP-3076; constructed version-5 file, conditional checks, minimal limits, and selected Lean statements. |
| [Contract signatures and counterfactual validation](Contract-signatures-and-counterfactual-validation.md) | A05 | How do contract policy, wrapper detection, and deployment/preparation differ from ECDSA recovery? | Pinned ERC-1271/6492, synthetic ABI wrapper, ordered validation, and the published helper's qualified side-effect behavior. |
| [Progressive SSZ presence and compatible schemas](Progressive-SSZ-presence-and-compatible-schemas.md) | S03 | How do absent fields, zero values, and layout commitments affect compatible schemas and proof addresses? | Pinned EIP-7495/7916/8016 and executable SSZ passages; complete schema-pair bytes, roots, and branches, with selector-width ambiguity retained. |
| [Snappy frames and response chunk boundaries](Snappy-frames-and-response-chunk-boundaries.md) | W04 | Which lengths, checksums, and method context delimit one decoded response payload? | Pinned raw/framing formats, RFC 3720 CRC passages, and supporting consensus profile; complete abc streams and selected decoder limits. |
| [Metadata updates and royalty query limits](Metadata-updates-and-royalty-query-limits.md) | A08 | What do a metadata resource, update event, and royalty query each establish for a consumer? | Pinned ERC-1046/4906/2981 with selected metadata and ABI context; constructed JSON, event layout, and explicit royalty-policy arithmetic. |
| [Creation collisions, rollback, and destruction](Creation-collisions-rollback-and-destruction.md) | E04 | Which nonce and account changes survive a failed creation, collision, or enclosing rollback? | Pinned creation EIPs and Yellow Paper passages; published CREATE2 address and conditional checkpoint trace, with later amendments separate. |
| [AES counter modes and authentication failure](AES-counter-modes-and-authentication-failure.md) | C03 | How do block encryption, CTR bytes, GCM authentication, and IV history impose different obligations? | Versioned FIPS 197 and SP 800-38A/38D passages; published CTR blocks, derived partial-block and GCM calculations, and separate revision notices. |
| [Account abstraction validation and module authority](Account-abstraction-validation-and-module-authority.md) | A06 | How do validation results, executor permissions, sponsorship, and protocol delegation differ? | Pinned ERC-4337/7579/6900 and EIP-7702 passages; nonce/mode bytes and a conditional sponsored execution trace, with version and source tensions retained. |
| [EVM word arithmetic and shift boundaries](EVM-word-arithmetic-and-shift-boundaries.md) | E05 | How do word width, signed interpretation, shifts, and copy boundaries qualify opcode results? | Selected Yellow Paper and EIP-145/5656 rules; arithmetic boundaries, published shift/copy controls, and isolated memory/gas calculations. |
| [Password KDF parameters and resource boundaries](Password-KDF-parameters-and-resource-boundaries.md) | C05 | Which password-derivation bounds define the algorithm, and which limits reflect caller resource policy? | RFC 8018/7914 and selected keystore context; published derivation outputs, HMAC blocks, and separately qualified memory/work calculations. |
| [Token callback order and hook authority](Token-callback-order-and-hook-authority.md) | A09 | Who controls hook discovery, which balances does a callback observe, and what does rejection undo? | Pinned ERC-223/777/1363/1820 with ERC-820 supersession context; lookup hashes, callback selectors, and a conditional nested-send ledger with source tensions retained. |

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
