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
| [Ethereum source catalogue](Ethereum-source-catalogue.md) | 70 candidate note families, each with sources, a concrete question or output, a provisional priority, and a reason. Includes a register of 16 research papers or design notes. |
| [Ethereum standards inventory](Ethereum-standards-inventory.md) | Every distinct numbered EIP/ERC document in the recorded source snapshots: 607 EIPs and 625 ERCs. Contains titles, document statuses, declared dependencies, and candidate-family links where identified. |

The inventory was generated from the official repositories at these commits:

- [ethereum/EIPs, `8194528a`](https://github.com/ethereum/EIPs/tree/8194528a2ee9a4893a398a5871c470ab0ac3b09a).
- [ethereum/ERCs, `dbba0f44`](https://github.com/ethereum/ERCs/tree/dbba0f4479dd817d19d7d7771dea0dcd76c68d17).

The census resolves 365 Moved entries to their ERC destinations, leaving 1,232 distinct documents.
It includes all statuses and categories in those snapshots.
It excludes unmerged proposals and files outside the numbered document paths; RFCs, API repositories, and papers have their separate catalogue entries.

Of those documents, 131 are mapped to at least one candidate family and 1,101 remain Unassessed for note relevance.
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

### Phase 2 batch and follow-up questions

This batch develops three representation questions before the execution-state work in Phase 3.
They supply reusable byte-level examples for callers, log readers, and signing code without requiring a new EthLib feature.
The [source index](External-source-index.md) links each note's pinned reading and review limits.

| Selected family and note | Connection and contribution value | Additional knowledge and bounded result |
| --- | --- | --- |
| S01: [canonical RLP](RLP-canonical-encoding.md) | Encoding and decoding callers need to separate generic items from scalar field rules. | One nested encoding, 55/56-byte boundaries, and rejection cases connect Appendix B to parser inverse properties and typed-envelope limits. |
| S04: [ABI encoding and event topics](ABI-encoding-and-event-topics.md) | Log readers and call boundaries need a schema, relative offsets, and the correct indexed-value encoding. | One complete `(uint256,string)` argument block connects ordinary ABI encoding to indexed hashes, packed ambiguity, and decoder acceptance limits. |
| A03: [signed-data preimages](Signed-data-preimages-and-domains.md) | Hashing and signature callers need an exact message format before reasoning about verification. | The complete published Ether Mail preimage connects ERC-191 envelopes, EIP-712 type/domain hashes, and application replay obligations. |

Each note received a separate source review.
Independent calculations reproduced the RLP and ABI examples and all four displayed EIP-712 published hash constants.
These checks evaluate the examples with separate Python packages; they do not test EthLib or replay its proofs.
The signed-data note keeps the pinned RPC text's `v` description and the example's `v = 28` explicit; signature serialization remains outside its result.

Eleven families in the Phase 2 queue still need their first source note:

| Family | Next bounded question |
| --- | --- |
| S02 | How does one variable-length SSZ list pass from container offsets to a length-mixed root and branch? |
| S03 | How do missing and zero-valued fields differ across one pair of progressive schemas? |
| S05 | How do quantity/data encodings and a block-hash selector determine one RPC state view? |
| C01 | Which padding and domain choices distinguish one Keccak-256 calculation from SHA3-256? |
| C04 | How do salt, PRK, info, and the expansion bound appear in one published HKDF vector? |
| C06 | Which input checks belong to point decoding, ECDSA verification, recovery, and transaction low-s acceptance? |
| C07 | Which bit/integer conversions and rejected candidates occur in an applicable RFC 6979 vector? |
| A02 | How do interface discovery, a receiver callback, and a magic return determine one token-transfer outcome? |
| A04 | Which nonce, deadline, and domain authorize one permit, and what action does it permit? |
| A05 | How does contract-defined signature validity differ from ECDSA recovery for one counterfactual account? |
| A06 | Which actors validate one UserOperation, and where do protocol delegation and account modules enter? |

The three covered families also retain work: trie-field schemas for S01; nested ABI layouts and custom errors for S04; arrays, compact signatures, domain discovery, and signature-value conventions for A03.
Their initial notes do not establish complete family coverage.
Phase 3 can use these representation notes while taking the next selected batch from its own queue.

### Phase 3 batch and follow-up questions

This batch connects the Phase 2 byte encodings to transaction admission, authorization state, and execution context.
It selects E01 and E02 before A10 so that the proxy comparison can use the transaction and authorization boundaries already explained.
Each note describes a bounded reading at exact source revisions, with later amendments identified separately.

| Selected family and note | Connection and contribution value | Additional knowledge and bounded result |
| --- | --- | --- |
| E01: [typed transactions](Typed-transaction-envelopes-and-signing.md) | Transaction codecs and signature callers must distinguish wire bytes, signing bytes, and inclusion conditions. | One complete unsigned type-2 preimage and conditional fee calculation connect RLP to per-type signing domains and blob network wrappers. |
| E02: [authorization order and rollback](Authorization-order-and-rollback.md) | State-transition work must distinguish an invalid transaction, a skipped tuple, and a failed execution. | An assumed-recovery trace follows warming, nonce changes, delegation replacements, and refund credits before a reverting frame. |
| A10: [proxy/delegation context](Proxies-delegation-and-storage-context.md) | Call and storage work must track code source separately from the account whose state it uses. | A complete minimal-proxy runtime and published storage-slot calculation support a context comparison with EIP-7702 delegation. |

The notes received separate source reviews and independent checks of their displayed bytes, slot hashes, arithmetic, and conditional state trace.
The state and context traces are deductions under their stated assumptions, not executed EVM vectors or Lean proofs.
The transaction note preserves the unresolved EIP-7702 outer-signing wording; its complete numeric signing example uses EIP-1559's explicit formula.
The authorization note keeps the original EIP-7702 rules separate from the pinned Last Call EIP-2780/8037 amendments.

The reading also assesses two previously Unassessed inventory entries:

| Source and family | Connection and reason for assessment | Review depth and limit |
| --- | --- | --- |
| EIP-7 → A10 | Its caller/value rules are a direct prerequisite for explaining a proxy's `DELEGATECALL` context. | Overview, Specification, and Rationale; gas amendments are outside the proxy trace. |
| EIP-2780 → E02 | Its runtime-gas rules explain why failure before the first frame differs from failure inside it. | Selected runtime-gas and authorization-processing sections; Last Call at the inventory pin, without a network activation claim. |

These source mappings bring the total to 131 mapped documents and 1,101 Unassessed documents without changing the metadata census or family priorities.
Eleven Phase 3 families still need their first source note:

| Family | Next bounded question |
| --- | --- |
| E03 | How do original/current/new values determine the gas and refund of one storage-write sequence? |
| E04 | Which state and value changes roll back across one failed call or creation? |
| E05 | What boundary inputs distinguish one opcode amendment from its historical behaviour? |
| E06 | How does one precompile's input/output/error contract differ from the underlying mathematical operation? |
| E07 | Which fields and ordered steps form one system-contract request? |
| E08 | Which fields belong to one fork's header, receipt, and log commitments? |
| E09 | How does one account or storage path establish inclusion or absence under its pinned trie rules? |
| E10 | Which assumptions and migration rules distinguish two proposed state-tree designs? |
| E11 | Which activation conditions select protected or unprotected signing rules for one historical transaction? |
| E12 | Which historical fork and exception rules qualify one comparison with a formal EVM semantics paper? |
| A09 | How does one token hook's callback order affect failure propagation and reentrant calls? |

E01 retains signature recovery and complete amendment coverage; E02 retains later gas-meter traces; A10 retains upgrades, initialization, and storage-layout compatibility.
The initial notes do not exhaust these families.
Phase 4 can take its selected cryptographic batch from the next queue.

### Phase 4 batch and follow-up questions

This batch connects validator key material to BLS verification, then separates KZG's use of the same curve from signature rules.
It selects one bounded question in each of C16, C08, and C09.
The notes distinguish encoded bytes, algebraic identities, and computational claims under a stated model.

| Selected family and note | Connection and contribution value | Additional knowledge and bounded result |
| --- | --- | --- |
| C16: [validator keys and keystores](Validator-key-derivation-and-keystores.md) | Key-generation and storage contributions need to separate scalar derivation, role paths, password bytes, and decryption acceptance. | Published master/child and PBKDF2 fixtures connect three ERCs; the note compares both source revisions and preserves the retry-limit difference. |
| C08: [BLS verification preconditions](BLS-ciphersuites-and-verification-preconditions.md) | Signature callers must preserve subgroup checks, exact domains, and verified proofs of possession. | An algebraic rogue-key trace, exact tags, and a qualified security-game bound connect the draft's scheme rules to caller obligations. |
| C09: [KZG openings and setup](KZG-openings-and-setup-assumptions.md) | Commitment work needs a degree bound, consistent setup points, canonical input bytes, and a separate binding assumption. | A small quotient opening and known-trapdoor forgery connect the 2010 paper to Ethereum's representation rules and qualified reduction statements. |

Each note received a separate source review.
Independent calculations reproduced ERC-2333 Test Case 0 and the published ERC-2335 PBKDF2 checksum/decryption.
Separate checks covered password and domain-tag bytes, symbolic BLS cancellation, and toy KZG arithmetic.
The BLS and KZG examples are algebraic explanations, not executed pairing or signature vectors.
The Python password examples use Unicode 15.0.0, while EthLib records Unicode 16.0.0; checking those examples does not compare the full normalization tables.

Source distinctions remain explicit: BLS draft-04 versus RFC 9380, the ERC procedures versus EthLib's 16-attempt key-generation bound, and the unresolved KZG normative-text and endianness readings.
The key-generation note records that its three ERC source files are byte-identical at EthLib's pin and the inventory pin.
The KZG report has a dated version and file digest; selected sections and reduction statements were read without a full security or ceremony audit.
No inventory source mapping or family priority changes in this batch.

Ten Phase 4 families still need their first source note:

| Family | Next bounded question |
| --- | --- |
| C02 | How does one compression-function precompile's length and state contract differ from a whole-message hash? |
| C03 | Which counter, IV, and authentication obligations qualify one AES-mode example? |
| C05 | How do one password KDF's algorithm bounds differ from caller-selected work and memory limits? |
| C10 | Which FFT and convolution steps produce several KZG cell openings under a pinned corrected paper? |
| C11 | What is the adversary's winning condition in one PeerDAS code-binding reduction without the algebraic group model? |
| C12 | Which state and one-time-key reuse conditions distinguish RFC XMSS from the selected leanXMSS variant? |
| C13 | Which parameters and address/hash rules distinguish the selected leanSPHINCS variant from FIPS SLH-DSA? |
| C14 | How do one target-sum code and retry budget enter a WOTS+C security argument? |
| C15 | Which oracle and adversary assumptions separate VRF uniqueness, unpredictability, and pseudorandomness? |
| N06 | How does one custody or sampling response connect cell authenticity, recovery, and availability? |

C16 retains deeper path and scrypt cases; C08 retains aggregate-PoP security games; C09 retains multipoint, batch, and setup-generation arguments.
The initial notes do not exhaust these families.
Phase 5 can take its selected consensus, networking, and interface batch from the next queue.

### Phase 5 batch and follow-up questions

This batch selects one question from consensus research, peer authentication, and the execution/consensus interface.
N01 supplies a bounded safety argument, W05 separates channel and peer identity, and I01 separates method errors from payload validation outcomes.
These notes give each boundary its actors, source roles, and remaining obligations.

| Selected family and note | Connection and contribution value | Additional knowledge and bounded result |
| --- | --- | --- |
| N01: [accountable safety](Accountable-safety-and-validator-overlap.md) | Consensus proofs need explicit membership, weights, ancestry, vote views, and threshold conditions. | A twelve-unit example connects quorum overlap and incompatible votes to the paper's fixed-set argument and a qualified attestation-data bridge. |
| W05: [Noise peer identity](Noise-handshakes-and-libp2p-peer-identity.md) | Networking callers must distinguish a channel's static DH key, its authenticated signing identity, and the expected remote peer. | The XX actor trace and exact identity bytes connect Noise revision 34 to pinned libp2p rules, including early-data limits and EthLib's narrower identity profile. |
| I01: [Engine newPayloadV3](Engine-newPayloadV3-outcomes-and-order.md) | Interface work must distinguish JSON-RPC errors, payload statuses, ancestry-dependent hashes, and caller forkchoice updates. | A Cancun-scoped decision table and overlapping-failure case separate inherited API requirements from EthLib's observed order. |

Each note received a separate source review.
Independent calculations checked the quorum example, identity preimage/protobuf/Peer ID bytes, and the Engine examples' array order and timestamp/ancestry conditions.
PyCryptodome 3.23.0 reproduced the public RFC 8032 signature control and checked a signature on the constructed identity message, including rejection of changed messages.
This did not run a Noise handshake, EthLib, a consensus client, or an Engine endpoint.

The catalogue adds RFC 8032 and RFC 7748 as primary support for W05's identity and DH example, bringing its additional source entries to 57.
I01's Engine API entry now pins the revision read by the note and cited by EthLib.
The numbered inventory, source mappings, and family priorities stay unchanged.
The notes preserve source limitations: shared fixed-set assumptions for the one-third bound, a libp2p actor-wording defect, and unspecified total precedence among simultaneous Engine failures.

Seventeen Phase 5 families still need their first source note:

| Family | Next bounded question |
| --- | --- |
| N02 | How do one shuffle and weighted committee rule depend on the seed and validator lifecycle? |
| N03 | Which membership and weight-change bounds qualify one checkpoint-age argument? |
| N04 | How do a trusted root, committee branch, and signature threshold qualify one light-client update? |
| N05 | Which amendment revision and applicability decision define one light-client candidate? |
| N07 | Which actors and deadlines distinguish one enshrined PBS or inclusion-list obligation? |
| N08 | Which network and adversary model qualifies one historical GHOST or backbone result? |
| W01 | Which signed record bytes bind one ENR identity and advertised discovery endpoint? |
| W02 | How do one WHOAREYOU exchange and identity proof establish discovery-v5 session keys? |
| W03 | Which compatibility and capability rules govern one RLPx handshake? |
| W04 | How do decoded length, framing checksum, and context constrain one response chunk? |
| W06 | How do one TLS or QUIC transcript and certificate bind the remote libp2p identity? |
| W07 | Which validation result and cache state determine one gossipsub transition? |
| I02 | How do one Beacon API identifier and state-dependent flags determine its response? |
| I03 | Which signature, release condition, and payment claim belong to one Builder/Relay operation? |
| I04 | Which atomicity and signing-history obligations qualify one key-management operation? |
| I05 | How do missing roots and repeated imports affect one slashing-protection interchange case? |
| I06 | Which framing and authentication checks qualify one history/archive read? |

N01 retains dynamic stake and liveness arguments; W05 retains handshake-security and interoperability analysis; I01 retains other methods and inherited-validation cases.
The initial notes do not exhaust these families.
Phase 6 can take its selected application and catalogue-renewal batch from the next queue.

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
The 16 literature entries include metadata-only candidates and scheme-design notes as well as papers with selected passages examined.
The two Phase 1 notes and the three notes in each of Phases 2 through 5 record their completed reading and review limits.
The Phase 2 byte calculations used Python 3.12.15, `rlp` 4.1.0, `eth-abi` 5.2.0, and PyCryptodome 3.23.0 in an isolated temporary environment.
The Phase 3 calculations used separate RLP/Keccak tools and source-derived state traces, with no EthLib execution.
The Phase 4 calculations used Python's hash, HMAC, PBKDF2, and Unicode functions, PyCryptodome 3.23.0 for AES, and scalar or symbolic arithmetic.
The Phase 5 calculations used set and list operations and PyCryptodome 3.23.0 for the public Ed25519 example.
Detailed reading and note writing for the remaining catalogue questions stay planned work.
No Lean build, proof check, binding test, or performance measurement was run for this documentation revision.
