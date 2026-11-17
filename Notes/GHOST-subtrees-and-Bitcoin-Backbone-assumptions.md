---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: N08
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ghost_source_version: "FC 2015 preproceedings, paper_30.pdf"
backbone_source_version: "EUROCRYPT 2015, LNCS 9057, pp. 281–310"
tags:
  - ethlib
  - external-sources
  - consensus
  - proof-of-work
---

# GHOST subtrees and Bitcoin Backbone assumptions

A rule that selects one branch of a block tree does not by itself establish a safe transaction ledger.
Original GHOST changes which proof-of-work branch a node extends, while the Bitcoin Backbone paper separates chain agreement, contributions and transaction progress under a specified model.
This note compares those roles through one constructed tree.
Neither paper supplies the rules of Ethereum's later latest-message fork choice or quorum finality.

## Exact sources and access limits

The GHOST source is Sompolinsky and Zohar's [Secure High-Rate Transaction Processing in Bitcoin][ghost], the FC 2015 preproceedings PDF, not a later paper with a different title.
Its 244214-byte PDF has SHA-256 `ba004f0936fe1063867869ff3aa13a7b64e4f804a69aba480a2ff7820c03d8c1`.
Reading covered §3, §5 including Algorithm 1 and the §5.1 convergence/security arguments, and the beginning of §6 for scope.
The paper's simulations, benchmark claims and remaining proofs were not audited.

The Backbone argument read here is Garay, Kiayias and Leonardos's [EUROCRYPT 2015 publisher version][backbone], LNCS 9057, pages 281–310.
Its 533948-byte PDF has SHA-256 `d39f8563edb366bb3a4d5f190fd5c6db3ef5479e885f9705a139b2a285ba9572`.
Reading covered §2's communication/query model, selected §3 algorithms, Definitions 2–3, §4's growth lemma and property statements, and §5.1's complete ledger definitions and lemma statements.
This is a selected reading, not a complete proof review.
The [ePrint 2014/765 record][eprint] lists a latest revision dated 2020-08-14, the last of 17 revisions.
The archive PDF and version-history page returned HTTP 403, so that revision's arguments remain unread and are not substituted for the conference text.

## What original GHOST counts

Algorithm 1 starts at genesis and repeatedly chooses the child with the heaviest observed subtree until it reaches a leaf.
Its displayed cardinality includes the child and all descendants, not just descendants on the eventual selected path.
Footnote 8 specifies combined proof-of-work as the intended weight, using subtree size for concise notation.
Counting blocks therefore needs an equal-difficulty assumption in this example.
The rule selects a path, rather than adding every supporting block's transactions to that path's ledger.

Section 3 models miners on a directed graph with link delays and Poisson block-production rates determined by computational shares.
Its attack model gives the attacker efficient sequential production without internal propagation delay.
Those are research assumptions, not consequences of the word “heaviest.”
The selected §5.1 arguments concern eventual convergence and attack behaviour within that model.
No confirmation depth, success probability or operational security threshold is derived here.

## A complete tree with different heads

Assume all the following blocks are valid, visible in one node's view, and have equal difficulty.
An edge goes from parent to child, and genesis G has depth zero:

```text
G
|-- A -- A1 -- A2 -- A3
`-- B
    |-- B1 -- B11
    |-- B2
    `-- B3
```

The complete parent relation is:

```text
G:none, A:G, A1:A, A2:A1, A3:A2,
B:G, B1:B, B11:B1, B2:B, B3:B
```

There are ten nodes, including genesis.
The A subtree has four blocks, while the B subtree has five.
The unique longest branch ends at A3, four edges from G.
Original GHOST instead makes these local decisions:

| Current block | Child subtree sizes | Chosen child |
| --- | --- | --- |
| G | A: 4; B: 5. | B. |
| B | B1: 2; B2: 1; B3: 1. | B1. |
| B1 | B11: 1. | B11. |
| B11 | No children. | Stop. |

The resulting path is `G → B → B1 → B11`, only three edges long.
B2 and B3 contribute to the first decision although neither belongs to that path.
The trace has no tie and needs no invented tie-breaking convention.
Removing B3 produces a 4-to-4 tie at G, which this example deliberately leaves unresolved.

This is a finite calculation about a fixed view.
It gives no probability that another view has the same head, that these blocks were honestly mined, or that a transaction remains accepted.
Changing the visibility set can change a decision without changing any block already present.
An implementation case must therefore specify the available tree, validation assumptions, weight function and tie policy separately.

## What the Backbone properties add

The selected conference version studies a fixed population in a synchronous, round-based, random-oracle model with bounded hashing queries.
It permits adaptive corruption and rushing, but honest broadcasts arrive by the next round.
The environment cannot contribute extra oracle queries, and the analysis isolates one protocol instance.
These restrictions matter when relating adversarial hashing power to communication speed.

Its properties answer different questions:

| Source | Question answered |
| --- | --- |
| Definition 2, common prefix. | Is each honest chain minus its last k blocks a prefix of the other honest chain in that view? |
| Definition 3, chain quality. | Is the adversarial fraction bounded in each specified-length consecutive segment? |
| Lemma 1, growth. | Does chain length advance by at least the number of successful honest rounds in the interval? |
| Definition 5, persistence. | Does a transaction reported more than k blocks deep retain its position in any honest report from that round on? |
| Definition 5, liveness. | For a generated or neutral transaction supplied continuously to all honest parties for u rounds, does some honest party report it more than k blocks deep? |

The conference's growth statement is a lemma, rather than an imported definition from another revision.
Here “neutral” means that the transaction conflicts with no other transaction under the paper's ledger relation.
Its liveness lemma also assumes an unambiguous transaction-generation oracle.
The property lemmas impose quantitative production and adversary bounds beyond merely naming these properties.
Honest successful-round production must exceed adversarial production by a stipulated margin, with a further synchronization-dependent inequality for common prefix and persistence.
The paper's honest-majority interpretation depends on sufficiently fast synchronization relative to proof-of-work production.
The constructed tree checks none of those probabilistic hypotheses.

## Ethereum and contribution boundaries

At the recorded EthLib commit, [`groupVotes` and `tallyVotes`][lean-tally] use latest validator messages and eligible balances, and [`getWeight`][lean-weight] includes conditional proposer boost.
The selected [`getHead` definitions][lean-head] walk viable children from a justified checkpoint.
Those code observations distinguish the recorded latest-message machinery from counting every proof-of-work block in the GHOST tree.
They do not establish a refinement between either paper and EthLib, nor a complete description of every fork branch.
The [accountable-safety note](Accountable-safety-and-validator-overlap.md) addresses quorum overlap separately.

The useful next contribution is a precise model comparison: choose the source revision, state the adversary and delivery assumptions, and identify the exact chain or ledger property to relate.
One cannot transfer a proof-of-work theorem merely because both algorithms descend through heavy children.
Standalone Python checked the parent relation, subtree sizes, depths, both selected paths and the tie control.
No mining simulation, benchmark, probability bound, transaction execution, EthLib call, Lean build or proof replay ran.

[ghost]: https://fc15.ifca.ai/preproceedings/paper_30.pdf
[backbone]: https://link.springer.com/content/pdf/10.1007/978-3-662-46803-6_10.pdf
[eprint]: https://eprint.iacr.org/2014/765
[lean-tally]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/ForkChoice/Head.lean#L133-L179
[lean-weight]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/ForkChoice/Head.lean#L271-L295
[lean-head]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/ForkChoice/Head.lean#L393-L425
