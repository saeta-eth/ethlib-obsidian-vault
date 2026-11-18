# Contributing research notes

Write a note that answers an external Ethereum question relevant to an EthLib contribution.
Start with the question, its sources and the explanation a reader needs.
Use the [source index](Notes/External-source-index.md) to find related notes and the [catalogue](Notes/Ethereum-source-catalogue.md) to identify source relationships.

## Choose the question and sources

Choose a narrow question about a rule, encoding, interface, construction or research assumption.
Explain the decision or uncertainty the note helps a contributor resolve.
Existing EthLib code is one connection, but background knowledge can also be useful before a corresponding module exists.
A note's inclusion does not propose implementing its subject in EthLib.

For each source, record its title, author or standards body, exact section, revision and URL.
Prefer primary standards, papers, interface specifications and author or venue publications.
Read the relevant primary passage before stating a rule or using a note in a contribution.
Follow declared dependencies and relevant amendments, keeping those relationships distinct from additional comparisons.
The [standards inventory](Notes/Ethereum-standards-inventory.md) keeps documents whose relevance is Unassessed visible.
An unmapped source or an unresolved applicability choice is not a rejection.

Use a separate note when different actors, algorithms or revisions require different explanations.
Keep sources and unresolved questions available when a note covers only part of a family.

## Write the explanation

Use the [research-note template](Templates/Research-note.md), adding fields or sections only when they help answer the question.
Record the source revision and the EthLib commit before making technical claims.
Name a source's role: protocol requirement, interface contract, research model, historical context or comparison formalization.
Follow [EthLib's source policy](https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/CONTRIBUTING.md#sources-of-truth) when relating those sources to library rules.

State the important inputs, outputs, validation order, bounds and failure cases.
Preserve MUST, SHOULD and MAY, together with hypotheses, quantifiers and exceptional conditions.
State the adversary, network and oracle models and setup assumptions beside each security result.
Give a source-derived example or published vector when it clarifies the rule.
Distinguish source statements, Lean theorems, test observations and deductions.
Link code and theorem names at the recorded EthLib commit, with material hypotheses beside the result.

Keep conflicting interpretations explicit.
Record examined sections and what remains unread, inaccessible, ambiguous or unproved.
Metadata inspection, an abstract, selected passages and a complete argument review establish different reading depths.
Document status, applicability to a fork, network adoption and EthLib support also require separate evidence.

Do not add phase or batch fields for organizing the writing or publication of knowledge notes.
Keep writing schedules, effort estimates and publication chronology out of their prose.
State unresolved technical conditions directly.
Omit generic review-completion narration when it adds no source scope, result or evidence.
Source revisions, review dates and applicability conditions belong with the explanation they qualify.
Git records the publication history.

## Review evidence and references

Record only checks actually run, with their inputs, expected-value sources, commands, results and relevant tool versions.
A standalone calculation, an EthLib test and a Lean proof replay provide different evidence.
State the limits of each check and the work not performed.
Do not infer complete implementation, proof coverage or computational security from a source link or one successful example.

Read revised passages alongside the original and their sources.
Check that technical conditions and requirement strength survive the edit.
Check local links, source references and any affected catalogue or inventory counts.
Add the note to the appropriate subject in the source index.
Keep the catalogue's stable family identifiers so that its source and inventory links remain useful.

Set `last_reviewed` and use `status` to distinguish `working`, `reviewed` and `superseded` notes.
A reviewed note records the scope of its review rather than certifying every statement in a source or every behaviour of the library.

## Maintain the collection

Recheck a note when an applicable requirement, amendment or source revision changes.
For a source-census refresh, record the official repository commits, archive digests and transformation method.
Compare added, removed and changed metadata before updating the inventory.
The inventory is a metadata census, and its family mappings do not establish a semantic review of every document.

For a literature extension, follow primary archives, authors and venues beyond the seed register.
Retain historical predecessors or competing designs when they clarify assumptions or amendment history.
The original material's [CC0 dedication](LICENSE) does not replace the terms or notices of material from other sources.

### Weekly maintenance

Complete the following process each week and when a contribution reveals an applicable source change.
The maintainer is responsible for source checks, technical review, validation and publication.
Keep the schedule and run reports in the maintenance automation, outside the knowledge notes.

#### Prepare the checkout

Confirm that the checkout and its origin belong to [this vault](https://github.com/saeta-eth/ethlib-obsidian-vault).
Record the starting vault commit and inspect tracked and untracked local changes.
Preserve those changes, including Obsidian settings, scratch notes and canvases.
Use a private clean checkout based on the latest published `main` for maintenance work when local changes are present.
Do not discard, stash or overwrite another contributor's work to prepare an update.

#### Check sources

Use the notes, source index, catalogue and inventory to enumerate the recorded sources and exact revisions.
Compare upstream source text and metadata with those revisions, including applicable amendments, dependencies, errata and later paper versions.
Check referenced EthLib definitions and theorem hypotheses against their recorded commit when assessing whether their explanation remains applicable.
Keep changes to EthLib itself outside vault maintenance.

For the standards inventory, compare document bodies separately from metadata.
An unchanged title, status or dependency list does not establish unchanged requirements.
For a versioned standard or paper, distinguish a new edition, an erratum, a withdrawal and a changed publication record.
Report inaccessible sources and incomplete comparisons explicitly.
A failed download, abstract or metadata check does not establish review of the underlying argument.

#### Update affected notes

Map changed sources to notes through their citations, dependencies and catalogue families.
Read the affected primary passages before revising a claim.
Preserve requirement strength, hypotheses, bounds, fork conditions, source roles and explicit uncertainty.
Retain applicable historical revisions; add a comparison or separate note when a newer revision needs a different explanation.
Update EthLib references only after examining the cited definitions, proofs and material hypotheses at the new commit.

Revise only notes affected by verified changes and the navigation or catalogue entries they require.
For a census refresh, reproduce the baseline before comparing the new snapshot.
Record the new official commits, archive digests, comparison method and changed document IDs.
Keep metadata coverage distinct from semantic review, and preserve stable family identifiers.
Defer a change whose meaning remains uncertain while continuing independent updates that can be verified.

#### Review and validate

Compare complete old and new passages with their primary sources.
Use an independent specialist review for changed technical claims or source interpretations.
Check affected calculations, examples, vectors and references, and record what each check covers.
Verify local links, source pins and any changed inventory or catalogue counts.
State checks not performed and unresolved source readings.

Advance `last_reviewed` and change `status` only to reflect review actually completed at the stated scope.
Do not mark an inaccessible or unread argument as reviewed.
Do not replace a valid historical pin merely because a newer revision exists.

#### Publish and synchronize

Stage only reviewed maintenance files, leaving existing local changes and private files out of the commit.
Describe the source change, its effect on the notes and the checks actually run in the commit message.
Push reviewed commits to `main` without forcing or rewriting published history.
If the remote advances, reconcile the maintenance changes in the private checkout and repeat the affected review and validation.
Leave unresolved conflicts unpublished and report the blocker.

After publication, fast-forward the local vault checkout used by Codex only when doing so preserves all local work.
Otherwise leave that checkout untouched and report that it still needs synchronization.
Verify the published commit and the preservation of unrelated local files.
When there is no verified content change, report the result without creating a commit solely for dates or source pins.

Report source coverage, changed notes, deferred questions, checks, commit and push results, and local synchronization in the maintenance chat.
Do not put run logs, task lists or publication chronology in knowledge notes.
