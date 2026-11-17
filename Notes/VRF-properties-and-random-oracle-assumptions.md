---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C15
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
mrv_source: "FOCS 1999, MIT-hosted 11-page PDF"
mrv_sha256: "2ebe8e9692a9f9bea943e9aac092982aefbf4ae84775745ec904a24843fcfedf"
br_source: "CCS 1993; author-hosted PDF cover dated 2021-08-02"
br_sha256: "df6dfcf320455354e6cc4c8de09f68e348af8c1a86a174c7e2b89652312a6bf9"
tags:
  - ethlib
  - external-sources
  - vrf
  - random-oracle
---

# VRF properties and random-oracle assumptions

A publicly checkable output is not automatically unpredictable, pseudorandom or fairly selected.
These properties quantify over different keys, information and choices.
Before using a verifiable random function in a protocol, state which property the application needs and which experiment establishes it.

## Sources and exact reading

Micali, Rabin and Vadhan's [*Verifiable Random Functions*][mrv] is the MIT-hosted 11-page FOCS 1999 paper identified by the frontmatter hash.
Reading covered selected introductory motivation and §§3.1–3.2, including the VRF and verifiable unpredictable function (VUF) definitions on PDF pages 3–5.
Its RSA construction, transformations and complete security proofs were not reviewed.

Bellare and Rogaway's [*Random Oracles are Practical*][br] is an author-hosted 21-page PDF whose cover identifies CCS, November 1993, while displaying the date 2 August 2021.
That cover date is not a separate 2021 protocol result.
Reading covered the cover/abstract, the paradigm and background in §§1.1 and 1.3–1.4, and §2's oracle definition on PDF pages 1–6.
The historical construction and hash-function examples are not contemporary recommendations, and their full proofs were not audited.
The hashes identify the exact retrieved documents, not every edition of either paper.

## Three VRF requirements

MRV's formal definition uses probabilistic key generation G, deterministic evaluation F producing a value and proof, and probabilistic verification V.
Verification's randomness matters when stating acceptance, even though a particular implementation may verify deterministically.

| Requirement | What the selected definition says |
| --- | --- |
| Complete provability | For the overwhelming-probability good generated keys specified by the definition, every honest evaluation's proof has high verifier-acceptance probability. Domain/range correctness is also required. |
| Unique provability | For every public key, including a dishonest one, input and pair of distinct candidate values with proofs, at least one has low verifier-acceptance probability. |
| Residual pseudorandomness | An efficient outsider cannot substantially distinguish the fresh value from a uniform range value after seeing the public key and permitted evaluation/proof answers. |

Unique provability concerns the output value, not a unique proof string.
It prevents two distinct values from both being convincingly provable at the same input and public key.
It does not require every public key to have an accepted value.
Completeness addresses honest generation separately, including its exceptional-key probability and verifier coins.

The pseudorandomness experiment lets the adversary choose its fresh challenge input after making evaluation queries.
It receives either the actual value or a uniform value, without the challenge proof, and may make further queries at other inputs.
The challenge input must remain unqueried in both stages.
Giving its proof would change the experiment: public verification could identify the actual value.
The claim concerns an outsider; the secret-key holder can evaluate its own function.

MRV's VUF definition replaces residual pseudorandomness with inability to guess the whole value at an unqueried input.
That is a different goal.
For example, a long value `0 || U` with a uniform unknown suffix can be hard to guess exactly while its leading zero distinguishes it from a uniform string.
In the two-bit analogue, the best exact guess succeeds with probability 1/2, and a classifier distinguishing that source from uniform two-bit strings succeeds with probability 3/4 under equal source probabilities.
This is an illustrative distribution comparison, not a cryptographically secure VUF construction.

## A random oracle is a modelled public function

Bellare–Rogaway's §2 chooses each bit of each oracle answer uniformly and independently across inputs.
All parties, including the adversary, can query that same public function.
Repeated queries to the same input return the same answer; they do not draw fresh randomness.
The paper uses infinitely long answers as convenient notation because any actual protocol reads finitely many bits.

The paradigm first proves a protocol in this model, then substitutes a concrete function.
Sections 1.1 and 1.3 explicitly distinguish that heuristic substitution from the model's theorem and from a secret-key pseudorandom-function argument.
They also require the problem/protocol to be independent of the chosen hash, while leaving that independence informal.
A concrete hash's collision resistance alone does not supply every random-oracle property used by a proof.
An applicable bound therefore needs its oracle interface, answer distribution, permitted queries and challenge timing.

## A complete finite selection and query-count example

Choose a function H uniformly from all maps `{a,b,c} -> {0,1,2,3}`.
There are `4^3 = 64` equally likely tables.
Define selection as obtaining output zero.

| Procedure | Successful tables | Selection/guess probability |
| --- | --- | --- |
| Fix input a before seeing any values; select if H(a)=0. | 16 | 1/4 |
| Query a and b, then choose a zero-valued input if one exists. | 28 | 7/16 |
| Query a twice and use its value. | 16 | 1/4 |
| Query a and b, then guess the unseen H(c) as H(a). | 16 | 1/4 |

The second row has `64 - 3*3*4 = 28` successes: it fails exactly when both queried outputs are nonzero.
Both the second and third rows make two oracle calls, but they expose two fresh table entries and one fresh entry respectively.
Consistency prevents repeated queries from increasing the chance of selection.
Choosing between inputs increases that chance even when the underlying function is perfectly uniform.
The last row concerns guessing a fresh value, rather than selecting a favorable known one.

No key, proof system or actual VRF is instantiated by this table.
Its implication for protocol design is conditional: freedom to choose keys, inputs, eligible indices or publication timing needs a separate fairness argument.
An accepted proof for the chosen input does not remove that freedom.

## Selected EthLib statements and their hypotheses

At the recorded commit, [`verify_opening`][complete] proves an honest hash-tree opening returns its specified output.
[`verify_unique`][unique] says two accepted openings at one root/index give equal outputs or an explicit Keccak-256 collision.
That is a conditional reduction statement, not unconditional collision impossibility.
These are observations about EthLib's hash-tree construction, not an assertion that it implements MRV's RSA construction.

The selected [`prEvent_guess_le`][guess] fixes the challenge index in advance and samples its 32-byte secret uniformly, independently of arbitrary fixed other secrets.
The adversary gets the derived root, hash queries and openings outside that challenge.
Its hash/opening query bounds must hold for every root and on every execution path.
The bound is `(1+2Q)/2^256`, with `Q=q_H+(q_O+1)*2^33`, including the game's internal tree/opening hash budget.
This statement is neither MRV's adaptive-challenge indistinguishability experiment nor a guarantee for holder-chosen challenge secrets.

The separate [`prEvent_selected_le`][selection] assumes a holder commits to a known tree, independent uniform randomness arrives afterwards, and the mix is a uniform random table.
For fixed allowed indices I and target outputs S, it bounds selection by `|I|*|S|/2^256 + q/2^256 + Pr[collision]`, with q bounding pre-randomness mix queries.
Its collision term and randomness assumptions remain explicit.
No selection guarantee for arbitrary registered roots or biased/known registration-time randomness is inferred.

Standalone Python enumerated all 64 tables, checked the four counts and query-entry distinction, and checked the two-bit distribution comparison.
No VRF evaluation/proof, Keccak call, curve operation, oracle instantiation theorem, protocol selection deployment, EthLib call, Lean build or proof replay ran.

[mrv]: https://people.csail.mit.edu/silvio/Selected%20Scientific%20Papers/Pseudo%20Randomness/Verifiable_Random_Functions.pdf
[br]: https://cseweb.ucsd.edu/~mihir/papers/ro.pdf
[complete]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Vrf/Verify.lean#L64
[unique]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Vrf/Verify.lean#L194
[guess]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Vrf/Unpredictability.lean#L496
[selection]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Vrf/Selection.lean#L282
