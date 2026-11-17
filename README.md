# EthLib research vault

A public research notebook for contributing to [EthLib](https://github.com/ethereum/EthLib), a library of Ethereum definitions and proofs in Lean.
Notes explain external specifications, worked examples and research assumptions relevant to EthLib.
This is a personal notebook; EthLib's contribution rules, definitions, proofs, and documentation remain the references for work on the library.

## Start reading

Use the [external source index](Notes/External-source-index.md) to find a note by subject and question:

- [Source authority and representations](Notes/External-source-index.md#source-authority-and-representations): document status, RLP, SSZ, ABI and JSON-RPC encodings.
- [Execution](Notes/External-source-index.md#execution): transactions, state changes, gas, precompiles, proofs and formal semantics.
- [Cryptography](Notes/External-source-index.md#cryptography): hashes, signatures, KDFs, KZG and security assumptions.
- [Consensus and data availability](Notes/External-source-index.md#consensus-and-data-availability): finality, committees, light clients, PeerDAS and block obligations.
- [Networking](Notes/External-source-index.md#networking): discovery, transport, peer identity, framing and gossip.
- [APIs and history](Notes/External-source-index.md#apis-and-history): Engine, Beacon, Builder, signing and archive interfaces.
- [ERCs and application interfaces](Notes/External-source-index.md#ercs-and-application-interfaces): tokens, account authority, proxies, wallets, names and governance.

Choose the question relevant to the contribution, then read the note's source revisions, assumptions and limits before using the cited primary text.
The [source catalogue](Notes/Ethereum-source-catalogue.md) maps 70 families to specifications and research papers.
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

Follow the [contribution guide](CONTRIBUTING.md) for source selection, evidence and review.

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
