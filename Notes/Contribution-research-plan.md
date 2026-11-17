---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: working
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
upstream_commit: "04d4532f5b40cd4193c1804ce60993a95c118ac9"
tags:
  - ethlib
  - external-sources
  - contribution-planning
---

# Research plan for external Ethereum knowledge

Build a source-based knowledge collection that helps Codex understand Ethereum rules, interfaces, encodings, and security arguments before changing EthLib.
Cover the protocol, relevant ERC application standards, and surrounding research.
Select individual notes through an explicit assessment process, keeping unexplored sources visible.

The vault can supply background beyond EthLib's immediate implementation scope.
Including a source note does not itself propose implementing that standard in EthLib.
The contribution's scope and the library's inclusion policy are separate decisions.

## Catalogue and inventory

| File | Purpose and present coverage |
| --- | --- |
| [Ethereum source catalogue](Ethereum-source-catalogue.md) | 70 candidate note families, each with sources, a concrete question or output, a provisional priority, and a reason. Includes a register of 15 research papers or design notes. |
| [Ethereum standards inventory](Ethereum-standards-inventory.md) | Every distinct numbered EIP/ERC document in the recorded source snapshots: 607 EIPs and 625 ERCs. Contains titles, document statuses, declared dependencies, and candidate-family links where identified. |

The inventory was generated from the official repositories at these commits:

- [ethereum/EIPs, `8194528a`](https://github.com/ethereum/EIPs/tree/8194528a2ee9a4893a398a5871c470ab0ac3b09a).
- [ethereum/ERCs, `dbba0f44`](https://github.com/ethereum/ERCs/tree/dbba0f4479dd817d19d7d7771dea0dcd76c68d17).

The census resolves 365 Moved entries to their ERC destinations, leaving 1,232 distinct documents.
It includes all statuses and categories in those snapshots.
It excludes unmerged proposals and files outside the numbered document paths; RFCs, API repositories, and papers have their separate catalogue entries.

Of those documents, 129 are mapped to at least one candidate family and 1,103 remain Unassessed for note relevance.
Mapping is a proposed knowledge connection, not evidence that the entire source has been reviewed or that EthLib implements it.
The paper register is a literature seed rather than a complete literature survey.

## Delivery phases

Deliver each phase through a small batch of source notes, review it, then commit and push that batch before starting the next phase.
The family queues below assign all 70 families a home; they do not require exhausting an entire subject before advancing.
Keep unfinished families in the catalogue and bring a prerequisite forward when a note needs it.

| Phase | Family queue | Initial delivery | Completion condition |
| --- | --- | --- | --- |
| 1. Publish the foundation and pilot notes | P01, A01 | Publish this plan, the catalogue, inventory, source index, document-status note, and transfer-log comparison, with README navigation. | The seven documentation files are reviewed, their references and inventory totals checked, and their commit pushed. |
| 2. Shared representations and signing | S01-S05; C01, C04, C06, C07; A02-A06 | Three narrow notes: a canonical RLP case, an ABI/log encoding case, and a complete signed-data preimage. | Pinned sections, source-derived examples, exceptional cases, and source guarantees are reviewed and published; record next questions for the remaining families. |
| 3. Execution state and application behaviour | E01-E12; A09-A10 | Typed transaction envelopes, ordered authorization processing, and a proxy/delegation comparison after its prerequisites. | The signing and state-change traces preserve failure branches and fork applicability, pass documentation checks, and are committed and pushed. |
| 4. Cryptographic constructions and assumptions | C02, C03, C05, C08-C16; N06 | Validator keys, BLS preconditions, and a small KZG opening/setup explanation. | Each published note distinguishes representation checks, algebraic results, and computational claims, with its models and assumptions. |
| 5. Consensus, networking, and interface contracts | N01-N05, N07-N08; W01-W07; I01-I06 | One accountable-safety hypothesis map, one Noise peer-identity trace, and one Engine method's ordered outcomes. | The reviewed and published batch names actors, source roles, amendment applicability, and remaining obligations. |
| 6. Application breadth and catalogue renewal | A07-A08; A11-A15 | A vault-rounding case, one name/resolver question, and a bounded assessment of an unassessed source cluster. | Publish the notes and assessment record, compare refreshed metadata, and add a literature candidate with an explicit review depth. |

These queues contain 2, 14, 14, 13, 20, and 7 families, respectively, with no family assigned twice.
A completed phase means its selected documentation batch was published with its evidence and limits reviewed.
It does not mean the family queue is exhausted or a mathematical claim has been formally verified.
Published commits record delivery; the [source index](External-source-index.md) records the source notes that exist.
After Phase 6, continue bounded batches from the remaining catalogue and unassessed inventory.

Before each commit, check the changed prose, local links, source pins, metadata totals where affected, and the exact staged file list.
Record commands actually run and their results.
Use executable tests or proof checks when a batch changes or evaluates executable material.

## Assess sources systematically

For each candidate, record four judgments before assigning a writing priority:

1. **Connection:** identify a direct EthLib rule, a dependency, an interface, an adjacent concept, or an exploratory future use.
   Existing code is one signal, not a requirement for including background knowledge.
2. **Contribution value:** state the implementation decision, proof obligation, interoperability question, or ambiguity the note would help resolve.
3. **Additional knowledge:** identify the cross-source relationship, amendment history, worked example, or security assumption the note would explain.
   A list of links alone does not supply that explanation.
4. **Coverage and effort:** identify the neglected subject it covers, its prerequisites, and a small result that would make the first version useful.

Record source readiness separately: exact version, available primary text, examined sections, unresolved conflicts, and applicable forks.
Metadata inspection, reading an abstract, checking selected sections, and reviewing a full argument are different levels of evidence.
A draft, historical, stagnant, or withdrawn document can still provide useful context when its role is stated.

Use these writing priorities:

- **First:** a concrete question with broad reuse, important cross-source interactions, or a foundational assumption.
- **When needed:** a concrete connection whose detailed explanation should follow a relevant contribution or coverage need.
- **Unresolved:** a visible candidate needing a clearer target, applicability decision, or design comparison.
- **Unassessed:** the source is in the inventory but its note relevance has not yet been evaluated.

The catalogue has 22 First families, 44 When needed families, and 4 Unresolved families.
First is a pool for choosing a small batch, not a claim that those families are universally optimal or that every note should be written immediately.
Unassessed and Unresolved are not rejection categories.
Record a reason when changing a priority, and retain the source even when writing is deferred.
The present tiers are preliminary judgments; complete the four-part assessment and source-readiness record before selecting a family for drafting.

## Coverage map

| Subject | Catalogue families | Kinds of additional context to develop |
| --- | --- | --- |
| Process and representations | P01, S01-S05 | Authority and upgrade inclusion; RLP; SSZ; ABI; JSON-RPC encodings and state views. |
| Execution | E01-E12 | Transactions, authorization, gas, storage, calls, rollback, opcodes, precompiles, system requests, state proofs, historical forks, and formal-semantics comparisons. |
| Cryptography | C01-C16 | Hashes, domain separation, AES modes, MACs and KDFs, curve encodings, ECDSA, BLS, KZG, hash-based signatures, VRF properties, oracle models, and validator keys. |
| Consensus | N01-N08 | Finality and fork choice; dynamic stake; committees; weak subjectivity; light clients; data availability; PBS; inclusion lists; historical consensus models. |
| Networking | W01-W07 | ENR, discovery, RLPx, capabilities, Snappy, request/response, Noise, libp2p, TLS, QUIC, and gossipsub. |
| Interfaces and history | I01-I06 | Engine, Beacon, Builder, Relay, key-management and remote-signing APIs; slashing interchange; history retention and archive formats. |
| ERCs and application interfaces | A01-A15 | Transfers, discovery and receiver callbacks, signed data, permits, contract signatures, account abstraction, vaults, metadata, hooks, proxies, providers, names, payments, cross-chain interfaces, and governance. |

No ERC family is excluded merely because it concerns applications.
For example, compare ERC-20's Transfer event with native ETH logs under EIP-7708: a shared event format leaves different emission conditions to examine.
Contract signatures, typed signing, proxy storage, and tokenized-vault arithmetic also offer potential knowledge beyond modules already present in EthLib.
The catalogue states the proposed use without assuming those application standards are implemented.

## Choose the next writing batch

Start with the current contribution's question, follow its source dependencies, and select the smallest useful note.
Without an active contribution, choose across the coverage map so that one familiar area does not consume the whole queue.
Record why each selected note comes before another; use prerequisites and the value of the first worked result, not its EIP/ERC label alone.

Useful starting choices include an encoding or transaction note, an ERC interface comparison, a paper-assumptions note, and a neglected networking or API topic.
These are examples of a balanced batch, not a fixed list of permitted notes.
Keep the other candidates and unassessed sources available for later tasks.

Before drafting, state a narrow question and a first deliverable.
For instance, A01 can compare event fields, zero-value cases, emitter identity, and rollback without attempting to describe every token standard.
N01 can map one safety theorem's hypotheses before discussing the whole consensus protocol.
Split a family into separate notes when different actors, algorithms, or revisions need different explanations.

## Requirements for each source note

Use the existing [research-note template](../Templates/Research-note.md), adding these details where relevant:

- **Source identity:** title, author or standards body, exact section, immutable commit or paper version, and review date.
- **Source role:** normative requirement, primary interface specification, research model, historical context, reference behaviour, or the note author's deduction.
- **Applicability:** document status, amendments, dependencies, applicable fork, and any unresolved adoption question.
- **Rules and examples:** inputs, outputs, validation order, bounds, failure cases, and source-derived calculations or published vectors.
- **Requirement strength:** preserve MUST, SHOULD, MAY, quantifiers, bounds, and exceptional conditions.
- **Security assumptions:** the adversary, network and oracle models, setup assumptions, and hypotheses beside each result they qualify.
- **Contribution connection:** the EthLib entry points or candidate obligations, with implementation and coverage claims checked at a recorded commit.
- **Review limits:** say which sections were examined and what remains unread, ambiguous, or unproved.

Read the primary text for the rule being implemented when using a note in a contribution.
The note should supply an original explanation and a reading path, with enough references to check each claim.

## Preserve source and version distinctions

Follow EthLib's [source policy](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/CONTRIBUTING.md#sources-of-truth).
Research papers support their stated models and theorems; another implementation or formalization supplies comparison evidence rather than authority for an Ethereum rule.
Consensus-specs and execution-specs retain their supporting/provenance role under that policy.
The SSZ project describes its executable implementation as the official specification; record that role explicitly while following EthLib's code-citation requirements.

Keep these concrete distinctions from the earlier research:

- EthLib selects BLS signature draft-04 and separately follows RFC 9380 for hash-to-curve; later drafts need an explicit comparison.
- Its ERC-2333/2334/2335 citations pin `ethereum/ERCs` commit `be307c948f28897adb8f25afcc253c2c909de35d`; compare that text before importing later amendments.
- The ERC key-generation retry rule and EthLib's 16-attempt bound are separate facts.
- The KZG module documents conflicting readings of its normative text; preserve the disagreement rather than choose silently.
- RFC XMSS and FIPS SLH-DSA provide comparison material for the leanXMSS and leanSPHINCS variants, whose own sources and parameters must be recorded.
- A paper's assumptions and historical fork coverage accompany any comparison with present protocol rules.

Use [EIP-7723's inclusion stages](https://eips.ethereum.org/EIPS/eip-7723) and record an EIP's document status separately.
Apply EthLib's [inclusion policy](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/ARCHITECTURE.md#when-a-change-enters-ethlib) before proposing implementation work.
Do not infer a missing theorem or implementation solely from an external source or catalogue entry.

## Continue the assessment

Review bounded clusters from the Unassessed inventory, including dependencies declared by a selected document and adjacent standards discovered while reading it.
Record the source ID, family, proposed question, connection, priority, rationale, and review depth for each assessment.
Keep declared dependencies distinct from additional relationships found through research.

For papers, follow references and later work from primary archives, authors, and venues.
Expand beyond the seed register into consensus attacks and defenses, execution semantics, availability arguments, and cryptographic reductions as tasks require.
Keep historical predecessors, competing proposals, and rejected designs when they clarify assumptions or amendment history.

Refresh the standards inventory by resolving new official repository commits and comparing added, removed, and changed metadata.
The inventory records the source paths, exact revisions, archive digests, and transformation method.
Recheck affected source notes when a requirement or amendment changes; no recurring refresh has been scheduled.

## Use in a Codex task

Use the catalogue as a topic index and make the vault available to the task.
Name the relevant note, family, and primary source section explicitly; load the material needed for that question.
Internal workflow notes on proof reading, Python-to-Lean calls, fixture provenance, and fork caps remain supporting material.
Keep EthLib's contribution rules and complete proof-coverage descriptions in their existing homes, with links from the vault.

## Evidence and limits

The 1,232-document inventory is a metadata census of the two pinned repositories, not a semantic review of all their requirements.
The 70-family catalogue combines metadata checks, selected external-source reading, and inspected EthLib source references.
The 15 literature entries include metadata-only candidates and scheme-design notes as well as papers with selected passages examined.
The two Phase 1 source notes record their completed reading and review limits.
Detailed reading and note writing for the other families remain planned work.
No Lean build, proof check, binding test, or performance measurement was run for this documentation revision.
