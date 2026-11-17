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
| [Ethereum source catalogue](Ethereum-source-catalogue.md) | 70 candidate note families, each with sources, a concrete question or output, a provisional priority, and a reason. Includes a register of 17 research papers or design notes. |
| [Ethereum standards inventory](Ethereum-standards-inventory.md) | Every distinct numbered EIP/ERC document in the recorded source snapshots: 607 EIPs and 625 ERCs. Contains titles, document statuses, declared dependencies, and candidate-family links where identified. |

The inventory was generated from the official repositories at these commits:

- [ethereum/EIPs, `8194528a`](https://github.com/ethereum/EIPs/tree/8194528a2ee9a4893a398a5871c470ab0ac3b09a).
- [ethereum/ERCs, `dbba0f44`](https://github.com/ethereum/ERCs/tree/dbba0f4479dd817d19d7d7771dea0dcd76c68d17).

The census resolves 365 Moved entries to their ERC destinations, leaving 1,232 distinct documents.
It includes all statuses and categories in those snapshots.
It excludes unmerged proposals and files outside the numbered document paths; RFCs, API repositories, and papers have their separate catalogue entries.

Of those documents, 138 are mapped to at least one candidate family and 1,094 remain Unassessed for note relevance.
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
The batch sections record delivery history; [Current coverage](#current-coverage) gives the remaining first entries after the latest batch.

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

After follow-up batch 1, nine Phase 2 families still needed their first source note:

| Family | Next bounded question |
| --- | --- |
| S03 | How do missing and zero-valued fields differ across one pair of progressive schemas? |
| S05 | How do quantity/data encodings and a block-hash selector determine one RPC state view? |
| C01 | Which padding and domain choices distinguish one Keccak-256 calculation from SHA3-256? |
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

At the Phase 3 delivery, these source mappings brought the total to 131 mapped documents and 1,101 Unassessed documents without changing the metadata census or family priorities.
After follow-up batch 1, ten Phase 3 families still needed their first source note:

| Family | Next bounded question |
| --- | --- |
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

At the initial Phase 4 delivery, ten families still needed their first source note:

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

Phase 5 added RFC 8032 and RFC 7748 as primary support for W05's identity and DH example, bringing additional source entries to 57.
I01's Engine API entry now pins the revision read by the note and cited by EthLib.
The numbered inventory, source mappings, and family priorities stay unchanged.
The notes preserve source limitations: shared fixed-set assumptions for the one-third bound, a libp2p actor-wording defect, and unspecified total precedence among simultaneous Engine failures.

At the initial Phase 5 delivery, seventeen families still needed their first source note:

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

### Phase 6 batch and follow-up questions

This batch develops two application questions and one bounded assessment record.
A07 separates vault arithmetic from executable capacity, A12 separates name bytes from resolver authority, and A11 assesses four wallet RPC sources.
The assessment also compares fresh metadata with the frozen census and adds a literature candidate with its reading depth stated.

| Selected family and note | Connection and contribution value | Additional knowledge and bounded result |
| --- | --- | --- |
| A07: [vault rounding and limits](ERC-4626-rounding-and-vault-limits.md) | Arithmetic and interface contributions need to distinguish fee-free estimates, previews, limits, and actual transfers. | A small proportional model traces four rounding directions, with qualified asynchronous and share-token contrasts. |
| A12: [ENS resolver boundaries](ENS-namehash-and-resolver-boundaries.md) | Name and hash callers need the correct input bytes and distinct ownership, resolver, recipient, and metadata roles. | Published namehash vectors connect the original name protocol to address, reverse-name, and text profiles within an explicit normalization scope. |
| A11: [wallet assessment and catalogue refresh](Wallet-RPC-assessment-and-catalogue-refresh.md) | Provider integrations need to separate permissions, chain configuration, active-chain changes, and batched call submission. | Four source assessments, a conditional switch request, a fresh census comparison, and a bounded literature reading extend the discovery record. |

Each note received a separate source review.
Independent calculations checked the model's roundings, published namehash values, interface selectors, and DNS byte contrast.
A separate metadata comparison checked all 1,232 records on the tracked fields and the refreshed archive digest.
These checks did not execute a vault, resolver, Provider, wallet, or EthLib.

The four assessed EIPs, 2255, 3085, 3326, and 5792, now map to A11.
At the Phase 6 delivery, these mappings brought the total to 135 and left 1,097 Unassessed entries in the baseline census.
A11's When needed priority is retained; the assessment does not settle every request schema or source discrepancy.
RFC 1035 joins A12's encoding references, and R17, the fixed-version DeFi systematization paper, supplies contextual reading for A07.
At that delivery, the catalogue had 17 literature entries and 59 additional source entries, with all 70 family priorities unchanged.

The refresh observed an unchanged EIPs head and a newer ERCs head, `50b3a3cdd2c12e2b81288b56f073c0ee812f95a8`.
It found no added or removed IDs, tracked metadata changes, or changed redirects: the census remains 607 EIPs and 625 ERCs with 365 resolved redirects.
Two ERC document bodies changed, 8048 and 8049; their text was not semantically reviewed.
The six ERCs used by the vault and ENS notes have identical bytes across the compared revisions.
The inventory keeps its original pins, with the refresh comparison recorded separately so that metadata equality does not imply body equality.

At the initial Phase 6 delivery, four families still needed their first entry:

| Family | Next bounded question |
| --- | --- |
| A08 | Which metadata schema, update signal, or royalty query supplies one concrete consumer guarantee? |
| A13 | Which URI revision and caller determine the parsing and chain-selection rules of one payment request? |
| A14 | Which trust and finality assumptions qualify one cross-chain interface result? |
| A15 | How do one governance clock and historical voting query define the time of a vote? |

A07 retains detailed asynchronous and fee models; A12 retains normalization and extended resolver reviews; A11 retains full method-specific explanations and integration cases.
The six initial delivery batches do not exhaust their queues or the inventory.
Continue with a bounded question from the remaining families, revisit a source ambiguity, or assess another Unassessed cluster when it supports a contribution.

### Follow-up batch 1: serialization, storage, and key derivation

This batch takes three remaining families from the representation, execution, and cryptography queues.
S02 and E03 fill foundational gaps before more complex schema and call-state questions.
C04 fills a shared prerequisite for the published validator-key and Noise notes, while retaining its When needed priority.
The selection uses the first worked result and its reuse, with all 70 family priorities unchanged.

| Family and narrow question | Connection and contribution value | Additional knowledge, coverage, and effort |
| --- | --- | --- |
| S02: How does one list-containing container pass from bytes to a root and branch? | Serialization and proof callers need distinct offsets, limits, lengths, and authenticated chunk addresses. | A small `List[uint16, 4]` example connects historical prose to selected executable-project passages. One schema keeps this first note bounded. |
| E03: How do repeated writes affect storage gas and the refund counter? | Execution work needs to compose amendments rather than copy an isolated gas table. | Two London-scoped traces separate cold costs, dirty-slot changes, counter deltas, and the transaction cap. Later resource models remain separate work. |
| C04: How do salt, input material, and context enter HKDF? | Key derivation and secure-channel work need exact HMAC argument roles and expansion bounds. | RFC 5869 Test Case 1 exposes both blocks and truncation, without treating a known answer as a security theorem. Full scheme reductions remain unread. |

Source readiness is recorded in [the SSZ note](SSZ-offsets-roots-and-branches.md), [the storage note](Storage-gas-and-refund-accounting.md), and [the HKDF note](HKDF-extract-expand-and-context.md).
The SSZ reading separates historical consensus-specs prose from the executable project's stated specification role.
The storage reading composes pinned EIPs for the assumed London rules and retains EIP-3529's conflicting rationale figure.
The HKDF reading names its RFC sections and published vector, separating caller assumptions from algorithmic checks.

Each note received a separate review against its named sources and selected EthLib definitions.
Independent Python 3.14.8 calculations reproduced the SSZ bytes, roots, and branch, the storage traces and assumed refund cap, and RFC 5869's published PRK/OKM with expansion intermediates.
Additional controls checked small SSZ decoding boundaries, branch changes, the storage sentry, and the HKDF counter arithmetic and context inputs.
These are source-model and example checks, without executing EthLib, the official SSZ package, or an EVM.

EIP-1283 moves from Unassessed to E03 because EIP-2200 explicitly inherits its original/current/new vocabulary.
The assessment covers that definition passage as a historical prerequisite, not its superseded standalone gas schedule or complete argument.
At the follow-up batch 1 delivery, this brought the inventory to 136 mapped documents and 1,096 Unassessed documents, with its 1,232 source records and baseline pins retained.
The historical SSZ Merkle-proof document joined S02's references, bringing additional source entries to 60.
The literature register stays at 17 entries.

At the follow-up batch 1 delivery, 20 of the 70 families had a first note or assessment entry, and 50 still needed one.
The remaining first-entry counts were nine for Phase 2, ten for Phase 3, ten for Phase 4, seventeen for Phase 5, and four for Phase 6.
S02 retains multi-chunk and nested-variable layouts, E03 retains rollback and later gas amendments, and C04 retains additional vectors and protocol-specific context encodings.
The initial notes do not exhaust these families.


### Follow-up batch 2: RPC state, node records, and permit authority

This batch develops one interface, networking, and application question from the remaining families.
S05 establishes state-query encoding and selection before further proof or API notes.
W01 extends the RLP foundation into a signed discovery record, and A04 extends signed-data preimages into conditional approval changes.
All three retain their When needed priority because the batch fills concrete coverage gaps rather than assigning implementation work.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| S05: [quantities and block state](JSON-RPC-quantities-and-block-state.md) | RPC callers need exact wire types and a stable state identifier before interpreting returned claims. | Two constructed requests connect EIP-1474 encodings, EIP-1898 selectors, and EIP-1186 proof roles. Reading records source discrepancies and separates selected Lean profiles. |
| W01: [ENR bytes and identity](ENR-record-bytes-and-identity.md) | Discovery work needs the signed content and key-derived address, independently of an advertised endpoint. | The published EIP-778 vector connects RLP, Keccak, compressed keys, and selected devp2p rules. Packet exchange and live connectivity stay outside the note. |
| A04: [permit authority](Permit-nonces-and-allowance-authority.md) | Signature and application callers need nonce ownership, deadline semantics, and the exact action authorized. | ERC-2612 and ERC-4494 supply a conditional state case and explicit signing bytes. The note retains different failure wording and ERC-4494's example discrepancy. |

These sources were already mapped in the catalogue, so no inventory assessment or priority changes are needed.
Each note received a separate source review, followed by a navigation and catalogue review.
Independent calculations checked the constructed RPC JSON and widths, the published ENR's RLP/content hash and node address, and the synthetic permit's signing bytes and conditional state trace.
They used Python 3.12.15 with `rlp` 4.1.0 and PyCryptodome 3.23.0 where needed, without executing EthLib, a signature verifier, a token, or a network endpoint.
At the follow-up batch 2 delivery, the census remained 1,232 documents, with 136 mapped and 1,096 Unassessed.
The catalogue then contained 60 additional source entries and 17 literature entries.
S05 retains complete trie verification and wider method profiles, W01 retains discovery packet/state-machine analysis, and A04 retains signature validation and full token integration cases.


### Follow-up batch 3: hash domains, discovery keys, and receiver callbacks

This batch connects a foundational hash choice to two callers that depend on exact encodings and actor roles.
C01 explains the hash-domain distinction used by selectors, addresses, and namehash.
W02 builds on ENR and HKDF before broader discovery-state questions, while A02 separates interface advertising from acceptance of a particular transfer.
Their original priorities remain unchanged.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| C01: [Keccak and SHA-3 padding](Keccak-SHA3-padding-and-domains.md) | Hash callers need the correct domain and preimage, rather than selecting by output width. | Original Keccak, FIPS 202, and the pinned Yellow Paper distinguish bit suffixes, byte padding, and a rate boundary. Selected algorithm sections were read without a full cryptanalysis review. |
| W02: [challenge and key binding](Discovery-v5-challenge-and-key-binding.md) | Discovery callers need to bind the expected peer, stored challenge, endpoint, and key direction. | A conditional exchange connects exact signature/HKDF inputs to separate published primitive fixtures. Complete packet/state-machine execution stays outside the note. |
| A02: [receiver interfaces and callbacks](Token-receiver-interfaces-and-callbacks.md) | Token integrations need separate interface claims, operator arguments, and acceptance results. | ERC-165/721/1155 supply selector calculations and conditional transfer traces. Alternate-hook wording and operator tensions remain explicit. |

Each note received a separate source review, including selected Lean definitions where the note cites them.
Independent calculations reproduced hash outputs and padding boundaries, published discovery-v5 ECDH/HKDF keys and the identity-preimage hash, and receiver selectors, XOR, and balance arithmetic.
No ECDSA verification, contract execution, live handshake, EthLib execution, or Lean proof replay ran.
At the follow-up batch 3 delivery, the original Keccak reference brought additional source entries to 61.
The numbered census then had 136 mapped and 1,096 Unassessed entries, with 17 literature entries unchanged.
C01 retains bit-oriented APIs and cryptanalysis, W02 retains complete packet/state transitions, and A02 retains implementation-specific callback and hybrid-policy cases.


### Follow-up batch 4: precompile contracts, signing history, and contract policy

This batch separates three caller boundaries: a precompile's byte contract, a validator's imported history, and a signing account's validation policy.
E06 extends execution knowledge beyond integer arithmetic, I05 connects key movement to future refusal conditions, and A05 distinguishes arbitrary contract policy from ECDSA recovery.
Their original When needed priorities remain unchanged.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E06: [MODEXP bytes and gas](Modexp-precompile-bytes-and-gas.md) | Precompile callers need declared lengths, output width, and a chosen fork's gas relation in addition to a modular power. | Short-exponent examples compose EIP-198/2565 and the pinned Yellow Paper. The long-exponent wording difference and later amendments remain separate questions. |
| I05: [slashing-protection imports](Slashing-protection-imports-and-missing-roots.md) | Signing-history contributions need refusal floors, repeat evidence, and preserved boundaries across imports. | A constructed EIP-3076 file exposes unknown roots, minimums, conservative maxima, and repeated imports, with parser/proof limits identified. |
| A05: [contract signatures](Contract-signatures-and-counterfactual-validation.md) | Signature callers need the expected account's current policy and exact wrapper-first validation order. | ERC-1271/6492 supply a complete synthetic ABI wrapper and conditional deployment/preparation trace. Reference-helper branches remain distinct from broad prose assurances. |

EIP-7823 and EIP-7883 move from Unassessed to E06 because they directly amend the same precompile's length acceptance and price.
Reading covered their selected Specifications as later-amendment context, not complete arithmetic proofs, deployment evidence, or a calculation of the later fork's full behavior.
Both are Final at the pinned EIPs revision; status is separate from applicability to the Berlin examples.
These assessments bring the census to 138 mapped and 1,094 Unassessed entries, with all 1,232 source records and metadata pins retained.
The catalogue keeps 61 additional source entries and 17 literature entries; its added Yellow Paper link for E06 already existed in other families.

Each note received a separate source review, including selected Lean definitions and theorem statements where cited, followed by an integration review.
Independent calculations checked nine bounded ModExp byte/gas cases, the complete interchange fixture and conditional refusal/import arithmetic, and the contract-signature selector and ABI wrapper.
No EVM, signer, BLS/signature verification, deployment, EthLib execution, or Lean proof replay ran.

E06 retains long-exponent and later-fork cases, I05 retains full database concurrency and signer integration, and A05 retains deployed-helper and account-policy audits.
The initial entries do not exhaust these families.

### Follow-up batch 5: schema presence, compression boundaries, and metadata consumers

This batch follows three representation questions through to their consumers.
S03 extends the existing SSZ note into schema changes, W04 connects compressed bytes to response boundaries, and A08 extends receiver/ABI knowledge into metadata and royalty queries.
Their original When needed priorities remain unchanged; the selection adds bounded source knowledge without assigning new EthLib implementations.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| S03: [progressive SSZ presence](Progressive-SSZ-presence-and-compatible-schemas.md) | Schema and proof callers need to distinguish field presence, zero leaves, and stable addresses from reusable proof bytes. | A three-position schema pair connects serialized fields, active-field words, SHA-256 roots, and branches. EIP and executable-source readings preserve the union selector-width ambiguity. |
| W04: [Snappy frames and response boundaries](Snappy-frames-and-response-chunk-boundaries.md) | Networking callers need separate raw-block, framing, decoded-length, and method-context boundaries. | Complete abc streams connect the primary Snappy descriptions and RFC 3720 CRC rules to a pinned supporting consensus profile. Resource bounds and stricter decoder choices remain distinct. |
| A08: [metadata updates and royalty queries](Metadata-updates-and-royalty-query-limits.md) | Application consumers need to distinguish resource/schema claims, refresh signals, query units, and payment actions. | Pinned ERC-1046/4906/2981 with selected ERC-721/1155 and ABI passages support constructed metadata, a complete event layout, and an explicit percentage policy. No token implementation is presumed. |

The notes record the selected source passages and their limits.
S03 separates Review/Draft proposal status from adoption and identifies the executable project's specification role.
W04 uses RFC 3720 only for CRC-32C, while the consensus text supplies the selected networking profile rather than overriding the primary formats.
A08 keeps optional extensions, MUST requirements, recommendations, and assumed royalty policy separate.

Independent calculations reproduced the progressive-container bytes, layout masks, roots, and branches, including stale-branch and wrong-path controls.
They also checked four published CRC controls, the derived abc checksum and complete frames, and the metadata JSON, royalty selector, event topic/data, and quotient/remainder arithmetic.
These example checks did not execute the official SSZ package, a Snappy decoder, an EVM, a metadata server, a royalty payment, EthLib, or Lean proofs.

RFC 3720 and the newly pinned supporting consensus URL bring the catalogue to 63 additional source entries; the generic consensus URL remains available in other families.
All selected numbered sources were already mapped, so the census remains 1,232 documents with 138 mapped and 1,094 Unassessed, and the literature register remains at 17.
The 70 questions and priority assignments are unchanged.
S03 retains nested schema and full compatibility review, W04 retains complete streaming/parser and client integration cases, and A08 retains content-authenticity and marketplace-policy assessments.
The first entries do not exhaust these families.

### Follow-up batch 6: creation checkpoints, encryption modes, and account authority

This batch connects three interfaces to the conditions their callers must preserve.
E04 follows a creation through collision and rollback, C03 separates encryption from authenticated decryption, and A06 separates account validation from execution authority.
The selection retains E04's First priority and C03/A06's When needed priorities.
It supplies source context for bounded contributions without assigning new EthLib implementations.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E04: [creation collisions and rollback](Creation-collisions-rollback-and-destruction.md) | Execution callers must place creator nonces and new-account writes on the correct sides of a failure checkpoint. | A published CREATE2 address and conditional creation trace connect collision rules, initcode limits, enclosing rollback, and the same-transaction SELFDESTRUCT exception. Later amendments retain their own scope. |
| C03: [AES modes and authentication failure](AES-counter-modes-and-authentication-failure.md) | Cryptographic callers must distinguish block transforms, counter uniqueness, and authentication failure. | The complete published AES-128 CTR example supports derived partial-block and bit-flip cases. A separate GCM example and selected Lean definitions distinguish algorithm outputs, supported tags, and caller-managed IV history. |
| A06: [account validation and module authority](Account-abstraction-validation-and-module-authority.md) | Account consumers must distinguish validation, fee sponsorship, executor permission, and protocol delegation. | Exact nonce and inner-call bytes accompany an explicitly assumed sponsored-operation trace. Version differences and conflicting validation-failure wording remain visible. |

The notes identify the selected source passages, versions, and reading limits.
The AES note keeps the 2001/2007 algorithms separate from NIST's revision notices, including the June 2026 pre-draft announcement that supplies no revised specification.
The account note compares the modular-account proposals without treating their interfaces or permissions as interchangeable.
The creation note separates baseline rules and account-write rollback from later code branches and other transaction bookkeeping.

Independent calculations checked published CREATE2 addresses and CTR blocks, derived length/gas and partial-block cases, GCM output and modified-input rejection in a separate library, and the account nonce/mode/call bytes.
Conditional state traces explain selected source rules rather than report EVM execution.
No EthLib binding, Lean build, proof replay, deployed account, or EntryPoint ran.

The two NIST revision notices bring the catalogue to 65 additional source entries.
All selected numbered standards were already mapped, so the inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; the literature register remains at 17.
The 70 questions and priority assignments are unchanged.
Each first entry leaves deeper fork composition, lifecycle history, or implementation-specific account behaviour for later work.

### Follow-up batch 7: word boundaries, password cost, and token hooks

This batch follows three caller obligations through small, explicit examples.
E05 distinguishes word interpretation from memory ranges, C05 separates password-derivation validity from affordable resources, and A09 separates hook registration from spending authority.
Their When needed priorities remain unchanged.
The selection supplies bounded source knowledge without assigning new EthLib implementations.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E05: [word arithmetic and shift boundaries](EVM-word-arithmetic-and-shift-boundaries.md) | Execution callers need exact signed rounding, large-shift, overlap, and memory-growth semantics before replacing instructions or proving rewrites. | Selected Yellow Paper rules and seven opcode EIPs support published shift/copy controls and derived arithmetic. Environment values and opcode availability retain their fork and caller context. |
| C05: [password KDF parameters and resource boundaries](Password-KDF-parameters-and-resource-boundaries.md) | Keystore callers need algorithm bounds and resource-admission policy to remain separate from password preparation and decryption. | RFC 8018/7914 and selected ERC-2335 passages support complete small controls, block counts, and qualified memory/work models. Expensive fixture derivation and current protection-cost recommendations remain outside scope. |
| A09: [token callback order and hook authority](Token-callback-order-and-hook-authority.md) | Token consumers need to identify who selects a callback, which balances it observes, and what its rejection undoes. | Pinned ERC-223/777/1363/1820 and ERC-820's supersession notice support lookup/selector calculations and a conditional nested-send ledger. Source tensions and operator authority remain distinct. |

The notes identify their selected source passages and applicability limits.
The password note distinguishes Informational RFCs from deployment policy and explicitly qualifies EthLib's buffer-payload and mixed-work models.
The callback note retains ERC-223 return/event-order tensions and the difference between ERC-1820's prose and embedded registration exceptions.
These readings do not establish general reentrancy safety, an application's resource budget, or arbitrary environment validity.

Independent calculations matched all 38 printed EIP-145 controls, published MCOPY overlap cases, and derived zero-length and memory-expansion arithmetic.
They reproduced RFC 7914's complete small PBKDF2/scrypt outputs, the two single-iteration HMAC blocks, and the selected table/core counts and memory/work model totals.
They also reproduced the registry-name hashes, callback selectors, and the nested-send ledger's conserved total of 110.
No EVM, token callback, registry, heavy scrypt case, EthLib binding, Lean build, or proof replay ran.

All selected numbered standards and external references were already catalogued.
The inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed, the additional-source count remains 65, and the literature register remains at 17.
The 70 questions and priority assignments are unchanged.
Each first entry leaves broader opcode composition, password-security policy, or application-specific hook behaviour for later work.

### Follow-up batch 8: execution requests, curve checks, and payment URI revisions

This batch follows three representations to the decisions they do and do not establish.
E07 separates system calls from request commitments, C06 separates curve and signature checks from key selection, and A13 compares payment-request revisions before choosing a consumer profile.
E07 and C06 retain their First priorities; A13 remains Unresolved.
The URI entry is a bounded assessment, not a selected implementation target.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E07: [system calls and execution request commitments](System-calls-and-execution-request-commitments.md) | Execution and consensus callers must distinguish privileged invocation, request extraction, and the commitment to ordered request bodies. | Selected passages from six pinned EIPs support a complete constructed withdrawal request and two-level SHA-256 calculation. The EIP-6110 deposit-type literal discrepancy remains explicit. |
| C06: [curve validation and recovery boundaries](Curve-validation-ECDSA-and-recovery-boundaries.md) | Signature callers must distinguish coordinate/scalar bounds, valid keys, verification, recovery candidates, and transaction admission. | Selected SEC 1/2 and EIP-2/155 passages connect published generator bytes to a constructed signature pair and parity-adjusted recoveries. Authentic key selection and general security arguments remain separate. |
| A13: [payment URI revision assessment](Payment-URI-revisions-and-request-boundaries.md) | A transaction-request consumer needs a chosen revision/profile before syntax or amounts can inform an approval workflow. | Complete ERC-67/681/831 texts and selected RFC 3986 sections support published URI cases, exact amount conversion, and delimiter-decoding distinctions. The revision and consumer workflow remain undecided. |

Separate source reviews checked the cited passages and the conditions stated in the notes.
The system-call note preserves source-specific timing and failure rules without auditing embedded bytecode or linked consensus transitions.
The curve note keeps SEC key-validity requirements separate from its authentic-key recommendation and Ethereum's transaction/precompile differences.
The URI assessment preserves the withdrawn, final, and stagnant revisions' different roles and their grammar tensions.

Independent calculations checked the constructed request's field widths, endian conversion, inner/outer hashes, and empty-body controls.
They reproduced the curve generator encodings, the constructed signature equations, low/high classification, and the parity-dependent recovery cases using standalone integer and affine-group arithmetic.
They also checked the two published ASCII URI cases and the native amount using exact arithmetic.
No EVM call, consensus transition, wallet, resolver, payment, EthLib binding, Lean build, or proof replay ran.

RFC 3986 brings the catalogue to 66 additional source entries.
All selected numbered standards were already mapped, so the inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; the literature register remains at 17.
The 70 questions and priority assignments are unchanged.
Each first entry leaves broader fork composition, cryptographic security, or consumer conformance for later work.

### Follow-up batch 9: receipts, deterministic nonces, and cross-chain authority

This batch connects three byte representations to the conditions their consumers must check.
E08 separates receipt outcomes from other block commitments, C07 separates seed conversion from nonce rejection, and A14 separates message identity from token authority.
E08 and C07 retain their When needed priorities; A14 remains Unresolved.
The cross-chain entry assesses the interfaces without selecting a bridge or assigning an EthLib implementation.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E08: [receipt status and block commitments](Receipts-status-gas-and-block-commitments.md) | Execution consumers need to distinguish included-transaction outcomes, cumulative gas, surviving logs, and separately committed block objects. | Selected pinned EIPs and the Shanghai Yellow Paper support a complete constructed type-2 receipt and its status-zero variant. Later gas amendments, proposed logs, and a full header schema remain separate. |
| C07: [deterministic nonce conversions](Deterministic-nonces-and-RFC6979-conversions.md) | Signing callers need exact fixed-width conversions, candidate bounds, and the selected hash/order profile before changing deterministic nonce generation. | RFC 6979's published P-256/SHA-256 control supports a nonce and HMAC-state calculation. The secp256k1 profile, local retry bound, distinct-hash variant, and low-s normalization retain their own conditions. |
| A14: [cross-chain message and token-authority assessment](Cross-chain-message-identity-and-token-authority.md) | A bridge consumer needs separate source evidence, replay state, and local mint/burn permissions. | Complete ERC-5164/7802 texts and selected ABI rules support a constructed call with an 84-byte identity trailer and a conditional retry ledger. Bridge trust, finality, token mapping, and acceptance of trailing calldata need explicit assumptions. |

The notes identify their selected source passages and applicability limits.
The receipt note separates Final rules from Last Call/Draft log proposals and qualifies its London gas example.
The nonce note keeps the Informational RFC's procedure and published P-256 control separate from code-profile observations and general security arguments.
The cross-chain assessment preserves Last Call/Draft status, transport-verification SHOULD, and the replay/failure MUST requirements.
Its Unresolved priority does not change when this first assessment is indexed.

Independent calculations reproduced the complete 268-byte constructed receipt, status-zero encoding, and conditional gas subtraction.
They reproduced RFC 6979's published P-256 nonce from the stated SHA-256, private-key, and order inputs, without checking a complete signature or curve operation.
The nonce note distinguishes calculated HMAC intermediates from the published output.
The calculations also checked the bridge-call selectors, published interface ID, 68-byte function data, 84-byte trailer, 152-byte combined calldata, and conditional ledger arithmetic.
No EVM call, transaction execution, receipt-trie root, bridge transport, token call, EthLib binding, Lean build, or proof replay ran.

All selected numbered standards and external references were already catalogued.
The inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; the additional-source count remains 66 and the literature register remains at 17.
The 70 questions and priority assignments are unchanged.
Each first entry leaves broader fork composition, signature security, or a chosen bridge's conformance and trust model for later work.

### Follow-up batch 10: account proofs, peer compatibility, and governance history

This batch connects three consumer decisions to the evidence they require.
E09 distinguishes a proof relative to a root from trust in that root, W03 separates peer compatibility checks, and A15 separates historical weight from governance descriptions.
All three retain their When needed priorities.
The entries supply bounded source knowledge without selecting an EthLib implementation or a DAO execution model.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E09: [account trie proofs and absence boundaries](Account-trie-proofs-and-absence-boundaries.md) | State-proof consumers need the authenticated path and encoded account before interpreting returned fields or absence. | Selected Yellow Paper rules and EIP-1186/161 support a complete constructed one-leaf proof and conditional absence case. Header trust and EthLib's local absent-field policy remain separate. |
| W03: [RLPx capabilities and fork-ID filtering](RLPx-capabilities-and-fork-ID-filtering.md) | Networking consumers must distinguish permitted extensions, shared capability versions, and claims about a fork schedule. | Pinned devp2p and four EIPs support an invented message-ID allocation, published historical CRC controls, and conditional fork-ID cases. Cryptographic handshake execution and complete peer admission remain outside scope. |
| A15: [governance clocks and vote history](Governance-clocks-vote-history-and-DAO-metadata.md) | A voting consumer needs a known clock domain and snapshot convention before using delegated weight or DAO metadata. | Complete ERC-5805/6372/4824 texts support clock boundaries, same-clock delegation arithmetic, and constructed metadata JSON. A specific consumer and its execution policy still need selection. |

The notes identify their source versions, reading depth, and applicability limits.
The account note retains EIP-1186's storageHash wording and separates its absent-account field gaps from the selected code policy.
The networking note preserves source-specific requirement strengths, historical block-fork controls, and the separately scoped timestamp amendment.
Its EIP-1898 citation supplies only the genesis-hash literal and does not change the catalogue's family assignments.
The governance note distinguishes lookup SHOULD clauses from the separate past-value constancy MUST, retaining signed-delegation example discrepancies.
DAO metadata descriptions do not select a voting rule or grant execution authority.

Python calculations checked the account, compact path, and complete leaf-node encodings, root hash, different-key absence case, and changed-balance/nested-list controls.
They reproduced the published EIP-2124 CRC32 values and EIP-6122's fictional timestamp control, and checked the invented capability ranges and conditional fork-filter cases.
They also checked the uint48 boundary, fixed-history delegation sums, and constructed JSON syntax.
No generic trie verifier, live RPC, chain validation, RLPx connection, frame authentication, voting contract, JSON-LD processor, EthLib binding, Lean build, or proof replay ran.

All referenced numbered standards and external source entries were already catalogued.
The inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; the additional-source count remains 66 and the literature register remains at 17.
The 70 questions, source-family assignments, and priority recommendations are unchanged.
Each first entry leaves a complete verifier audit, network admission policy, or chosen governance consumer for later work.

### Follow-up batch 11: activation, transport identity, and signing operations

This batch separates three kinds of conditions that a consumer must preserve.
E11 qualifies execution rules by their activation, W06 separates transport key derivation from identity, and I04 assigns key-movement obligations to the operation and actor that supply them.
All three retain their When needed priorities.
The notes support future contribution decisions without selecting a new EthLib implementation.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E11: [historical fork activation and Merge boundaries](Historical-fork-activation-and-Merge-boundaries.md) | Historical validation needs the activation condition and surviving transaction forms, as well as the amended header rules. | Five pinned EIPs and selected Shanghai Yellow Paper passages support legacy v arithmetic and a conditional terminal-difficulty trace. Complete PoW validity and PoS canonical selection remain separate. |
| W06: [TLS/QUIC transcripts and Initial keys](TLS-QUIC-transcripts-and-initial-key-boundaries.md) | Transport consumers need to know which transcript, key, certificate, and intended identity each check covers. | Selected primary RFCs and pinned libp2p profiles support the complete published QUIC v1 Initial-key control. Certificate, Finished, packet, and connection execution remain outside scope. |
| I04: [key-management and remote-signing obligations](Key-management-and-remote-signing-obligations.md) | Key movement must preserve signing history and distinguish local deletion from remote signer registration or signing. | Exact API schema revisions and selected keystore/interchange rules support conditional per-key deletion outcomes. A successful HTTP response does not establish all keys were removed or all copies stopped signing. |

The notes identify their source versions, reading depth, and applicability limits.
The historical note preserves EIP-155's SHOULD/MUST distinction and EIP-4399's override of the original zero mixHash constant.
The transport note keeps four encryption levels separate from three packet-number spaces and treats public Initial derivation separately from authenticated identity.
The key-management note keeps local deletion atomicity, retained history, remote registration, and optional signingRoot verification within their stated schema scopes.
ERC-2335 remains Review and EIP-3076 remains Last Call at the selected snapshots; their metadata is distinct from the interchange format version.

Independent calculations reproduced the scaled threshold and signature-v arithmetic and the empty-ommer hash.
They reproduced RFC 9001 Appendix A.1's five complete labels and nine secret/key/IV values, and checked the constructed deletion JSON, byte lengths, status/history alignment, and repeat-history conditions.
No PoW/PoS block validation, signature, TLS/QUIC packet, certificate, network endpoint, actual key deletion, remote signing, EthLib binding, Lean build, or proof replay ran.

The catalogue now pins both official API repositories at the revisions read by I04.
The inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; the additional-source count remains 66 and the literature register remains at 17.
The 70 questions, source-family assignments, and priority recommendations are unchanged.
Each entry leaves complete transition validation, connection processing, or a chosen signing deployment for later work.

### Follow-up batch 12: state-tree choices, gossip routing, and archive evidence

This batch makes three evidence boundaries explicit.
E10 compares competing state layouts, W07 separates routing decisions, and I06 separates storage policy and file structure from authenticated history.
E10 retains its Unresolved priority after this first assessment; W07 and I06 retain When needed.
The entries identify useful contribution questions without resolving a proposed fork or complete network deployment.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| E10: [alternative state-tree designs and migration boundaries](Alternative-state-tree-designs-and-migration-boundaries.md) | A state-layout contribution needs a chosen design and migration rule before assigning meaning to keys or absence. | Selected EIP-6800/7864/7523 passages support shared basic-data packing and Verkle location/preimage arithmetic. Draft inconsistencies, symbolic stems, witness algorithms, and full migration rules remain explicit limits. |
| W07: [Gossipsub validation, mesh, and caches](Gossipsub-validation-mesh-and-cache-boundaries.md) | Networking contributions need to preserve the distinct effects of validation, duplicates, cache age, score, and backoff. | Selected pinned pubsub and Gossipsub v1.0/v1.1 passages support constructed local cases. Ethereum topic rules, complete router behaviour, and network delivery guarantees remain separate. |
| I06: [history retention and archive authentication](History-retention-and-archive-authentication-boundaries.md) | Historical-data consumers need a trusted source of canonical commitments as well as correctly framed and indexed bytes. | Pinned retention/accumulator proposals and archive documents support a complete constructed SlotIndex and a length-header boundary. No complete archive, historical root, or chain was validated. |

The state-tree assessment records conflicting binary-draft parameters rather than choosing repairs or a final hash function.
Its worked packing follows the explicit big-endian field table while retaining the Verkle text's general little-endian convention as a separate source statement.
Its code-location examples do not assert code exists in the hypothetical no-code account.
The gossip note distinguishes Accept, Ignore, and Reject from message-ID equality and cache retention, preserving selected version-specific conditions.
The archive note preserves EIP-4444's 14,299-epoch SHOULD NOT serving rule and MAY pruning rule rather than a blanket retention requirement.
It records Nimbus's six-byte length field alongside the selected EthLib definition's four-byte length and two reserved bytes.
These descriptions identify their respective sources without claiming the layouts agree on every header.

Independent calculations checked the 32-byte account packing, Verkle index/suffix boundaries, and complete 64-byte preimages without computing Pedersen commitments.
They checked the published generic record, constructed 52-byte framing/index exercise, signed offset, and the 2^32 length-header boundary without allocating that payload.
A separate list-window calculation reproduced the gossip table, including advertisement before shifting, payload eviction, and separate seen-cache assumptions.
No commitment/witness proof, migration, live router, archive decoder, historical-root reconstruction, chain replay, EthLib binding, Lean build, or proof replay ran.

The catalogue adds the exact Nimbus framing document linked by the pinned archive repository.
The inventory remains 1,232 documents with 138 mapped and 1,094 Unassessed; there are now 67 additional source entries and 17 literature entries.
The 70 questions, numbered-source family assignments, and priorities are unchanged.
Choosing a target tree, a consensus topic profile, or a trusted archive checkpoint remains later work.

### Follow-up batch 13: validator selection and external API evidence

This batch separates mathematical, server-reported, and signed evidence.
N02 distinguishes a permutation from a selection-security claim, I02 qualifies a node's response, and I03 separates a bid from its execution and release obligations.
All three retain their When needed priorities.
The notes supply bounded context without selecting a live chain configuration or external service.

| Family and first result | Connection and contribution value | Additional knowledge, scope, and readiness |
| --- | --- | --- |
| N02: [validator weight, shuffling, and committee boundaries](Validator-weight-shuffling-and-committee-boundaries.md) | Consensus contributions need to distinguish index partitions, balance filters, state-derived seeds, and changing lifecycle accounting. | Three pinned EIPs, a separately versioned shuffle paper, and EIP-linked supporting helpers supply a constructed shuffle and two Electra decisions. A complete epoch fixture, probabilistic model, or security reduction remains outside scope. |
| I02: [Beacon API identifiers and response evidence](Beacon-API-identifiers-and-response-evidence.md) | API consumers need the requested object domain, selected representation, and chain view before interpreting roots or flags. | Exact schemas support complete constructed root/error JSON and selected identifier cases. A node report or HTTP 404 supplies neither independent authentication nor global absence evidence. |
| I03: [builder/relay bids and payload release](Builder-relay-bids-and-payload-release-boundaries.md) | Builder consumers need distinct signature objects, payment evidence, and conditions around proposer commitment and payload availability. | Exact parent and nested schema revisions support unsigned registration bytes and conditional replacement arithmetic. Bellatrix clauses, later endpoint changes, and incomplete pseudocode remain explicitly separated. |

The shuffle paper's selected arXiv revision is distinct from its DOI edition, and no numerical bound is transferred to a public Ethereum seed.
The note preserves the hypotheses of selected partition statements and distinguishes their reading from Lean replay.
The Beacon note keeps StateId and BlockId alternatives separate and preserves required endpoint fields despite generic compatibility descriptions.
The builder note retains required local building, advisory parallel scheduling and cancellation, and the prohibition on local fallback after publishing a signed blinded block.
It also separates a declared bid amount, relay simulation requirements, and an actual transfer.

Independent Python calculations reproduced the ten-position shuffle and inverse on those inputs, three helper slices, displayed SHA-256 values, and inclusive 16-bit balance-filter cases.
They checked complete constructed root/error JSON and its field widths, the 84-byte fixed SSZ registration, and exact wei conversions and replacement arithmetic.
No complete beacon transition, duty query, fairness experiment, BLS operation, signing-root computation, RPC, relay simulation, payment, auction, EthLib binding, Lean build, or proof replay ran.

The catalogue pins the selected supporting/API repositories and adds the separately versioned shuffle paper; its literature reading record now reflects the examined passages.
There remain 1,232 inventory documents, 138 mapped and 1,094 Unassessed; additional source entries increase from 67 to 70 and literature entries remain 17.
The 70 questions, numbered-source family assignments, and priorities are unchanged.
A selected fork/preset, trusted checkpoint procedure, or complete external-builder deployment still requires its own assessment.

## Current coverage

56 of the 70 families have a first source note or bounded assessment entry in the [source index](External-source-index.md).
14 families still need a first entry.
An entry covers its stated question and reading depth, rather than the entire family.

| Original phase queue | Families awaiting a first entry |
| --- | --- |
| 1 | None. |
| 2 | None. |
| 3 | E12 |
| 4 | C02, C10, C11, C12, C13, C14, C15, N06 |
| 5 | N03, N04, N05, N07, N08 |
| 6 | None. |

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
The 17 literature entries include metadata-only candidates and scheme-design notes as well as papers with selected passages examined.
The two Phase 1 notes and the three entries in each of Phases 2 through 6 record their completed reading and review limits.
The Phase 2 byte calculations used Python 3.12.15, `rlp` 4.1.0, `eth-abi` 5.2.0, and PyCryptodome 3.23.0 in an isolated temporary environment.
The Phase 3 calculations used separate RLP/Keccak tools and source-derived state traces, with no EthLib execution.
The Phase 4 calculations used Python's hash, HMAC, PBKDF2, and Unicode functions, PyCryptodome 3.23.0 for AES, and scalar or symbolic arithmetic.
The Phase 5 calculations used set and list operations and PyCryptodome 3.23.0 for the public Ed25519 example.
The Phase 6 calculations used exact rational arithmetic, PyCryptodome 3.23.0 for Keccak-256, and comparisons of refreshed metadata and document bytes.
Follow-up batch 1 adds three reviewed notes, with Python 3.14.8 standard-library byte, hash, HMAC, and gas-model calculations.
Follow-up batch 2 adds three reviewed notes, with independent JSON, RLP, Keccak, curve-coordinate, and conditional permit-state calculations.
Follow-up batch 3 adds three reviewed notes, with hash-padding, curve/HKDF, signature-preimage, selector, and conditional transfer calculations.
Follow-up batch 4 adds three reviewed notes, with ModExp byte/gas cases, interchange JSON/refusal arithmetic, and a contract-signature ABI wrapper calculation.
Follow-up batch 5 adds three bounded source notes, with progressive-layout roots and branches, published CRC controls and complete Snappy frames, and metadata/event/royalty calculations.
Follow-up batch 6 adds three bounded source notes, with CREATE2 address and gas calculations, published CTR blocks and derived GCM cases, and account nonce/mode/call bytes.
Follow-up batch 7 adds three bounded source notes, with published shift/copy controls, password-derivation outputs and qualified cost arithmetic, and hook lookup/selector and nested-send calculations.
Follow-up batch 8 adds two bounded source notes and a URI revision assessment, with request bytes and hashes, standalone curve/signature/recovery arithmetic, and fixed URI-field and exact amount checks.
Follow-up batch 9 adds two bounded source notes and a cross-chain assessment, with constructed receipt bytes, a published deterministic nonce and HMAC-state calculation, and bridge-call/conditional-ledger checks.
Follow-up batch 10 adds three bounded source entries, with a constructed account proof and conditional absence, published fork-hash controls and capability arithmetic, and clock/delegation/metadata calculations.
Follow-up batch 11 adds three bounded source entries, with conditional fork/signature arithmetic, published QUIC Initial-key controls, and constructed deletion/history JSON checks.
Follow-up batch 12 adds a state-tree design assessment and two bounded source notes, with layout/preimage arithmetic, conditional gossip transitions, and archive-framing/index checks.
Follow-up batch 13 adds three bounded source notes, with a constructed shuffle/filter trace, root-response JSON checks, and unsigned builder-registration bytes and value arithmetic.
Detailed reading and note writing for the remaining catalogue questions stay planned work.
No Lean build, proof check, binding test, or performance measurement was run for this documentation revision.
