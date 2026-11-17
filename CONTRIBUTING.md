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
