# EthLib research vault

A public research notebook for contributing to [EthLib](https://github.com/ethereum/EthLib), a library of Ethereum definitions and proofs in Lean.
Notes preserve source reading, proof attempts, and open questions between contributions.
This is a personal notebook; EthLib's contribution rules, definitions, proofs, and documentation remain the references for work on the library.

## Start reading

The [external source index](Notes/External-source-index.md) links the source notes and explains their review status.
Begin with [document status and upgrade inclusion](Notes/Source-status-and-upgrade-inclusion.md) or [ERC-20 events and native ETH transfer logs](Notes/ERC-20-and-native-ETH-transfer-logs.md).
For representation and signing work, read [canonical RLP](Notes/RLP-canonical-encoding.md), [ABI encoding and event topics](Notes/ABI-encoding-and-event-topics.md), and [signed-data preimages and domains](Notes/Signed-data-preimages-and-domains.md).
For execution-state work, continue with [typed transaction envelopes](Notes/Typed-transaction-envelopes-and-signing.md), [authorization order and rollback](Notes/Authorization-order-and-rollback.md), and [proxy/delegation context](Notes/Proxies-delegation-and-storage-context.md).
For cryptographic work, read [validator key derivation and keystores](Notes/Validator-key-derivation-and-keystores.md), [BLS verification preconditions](Notes/BLS-ciphersuites-and-verification-preconditions.md), and [KZG openings and setup assumptions](Notes/KZG-openings-and-setup-assumptions.md).
For consensus and client boundaries, read [accountable safety](Notes/Accountable-safety-and-validator-overlap.md), [Noise peer identity](Notes/Noise-handshakes-and-libp2p-peer-identity.md), and [Engine newPayloadV3 outcomes](Notes/Engine-newPayloadV3-outcomes-and-order.md).
For application interfaces, read [vault rounding and limits](Notes/ERC-4626-rounding-and-vault-limits.md), [ENS resolver boundaries](Notes/ENS-namehash-and-resolver-boundaries.md), and the [wallet RPC assessment and source refresh](Notes/Wallet-RPC-assessment-and-catalogue-refresh.md).
The first follow-up batch adds [SSZ offsets, roots, and branches](Notes/SSZ-offsets-roots-and-branches.md), [storage gas and refunds](Notes/Storage-gas-and-refund-accounting.md), and [HKDF extract/expand](Notes/HKDF-extract-expand-and-context.md).

The [research plan](Notes/Contribution-research-plan.md#delivery-phases) divides the work into phases, each delivered in small reviewed batches.
The [source catalogue](Notes/Ethereum-source-catalogue.md) lists 70 candidate note families and their reasons for priority.
The [standards inventory](Notes/Ethereum-standards-inventory.md) retains all 1,232 EIP/ERC documents in its pinned snapshots, including sources not yet assessed for note relevance.

## Open the vault

Clone this repository:

```sh
git clone https://github.com/saeta-eth/ethlib-obsidian-vault.git
```

In Obsidian's vault switcher, choose **Open folder as vault**, then select the cloned `ethlib-obsidian-vault` folder.
The shared settings enable the built-in Templates plugin, use standard Markdown links, and put new notes in `Notes/`.
You can also read and edit the Markdown files on GitHub or in any text editor.

## Write a note

1. Create a note in `Notes/` with a name that describes its question.
2. Open the command palette and run **Templates: Insert template**, then select **Research-note**.
3. Record the specification revision and the EthLib commit before writing technical claims.
4. Keep attempted arguments, established results, and unresolved questions distinguishable.
5. Record commands actually run, their results, and the commit tested.

The template fills the title and creation date when Obsidian inserts it.
Outside Obsidian, copy [the note template](Templates/Research-note.md) into `Notes/` and replace `{{title}}` and `{{date}}` yourself.
Set `last_reviewed` when you review a note, and use `status` to record whether it is `working`, `reviewed`, or `superseded`.
A reviewed note has been read at its recorded commit; that status alone establishes no mathematical claim.

## Keep claims traceable

Cite the primary document and exact section for each protocol rule, with a revision or pinned URL where available.
Link to code at the EthLib commit the note describes, and state material hypotheses beside theorem claims.
A test observation records its inputs and expected-value source.
Keep conflicting source readings explicit until they are resolved.

Link to existing EthLib documentation and add your reasoning here.
When a finding is needed to understand or maintain the library, include it in the relevant EthLib contribution.
Start with these references:

- [Contribution rules](https://github.com/ethereum/EthLib/blob/master/CONTRIBUTING.md) describe sources, proofs, and checks.
- [Architecture](https://github.com/ethereum/EthLib/blob/master/ARCHITECTURE.md) describes layers, models, and module boundaries.
- [Soundness](https://github.com/ethereum/EthLib/blob/master/SOUNDNESS.md) describes what the proofs establish and what their use trusts.

## Files and settings

```text
Notes/                        research notes
Templates/Research-note.md    reusable note template
Attachments/                  images and other supporting files
.obsidian/                    shared vault and template settings
```

Only the three shared Obsidian configuration files are tracked.
Workspace layouts, themes, community plugins, and deleted-note files are ignored.
For Obsidian's template commands and variables, see its [Templates documentation](https://help.obsidian.md/Plugins/Templates).

## Licence

Original material in this repository is dedicated under [CC0 1.0 Universal](LICENSE), matching EthLib's terms for original contributions.
Quoted or copied material from other sources retains its own terms and notices; identify its source and licence separately.
