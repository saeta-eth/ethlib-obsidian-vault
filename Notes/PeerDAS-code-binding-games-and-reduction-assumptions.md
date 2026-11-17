---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C11
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
paper_record: "ePrint 2025/1683; received 2025-09-16, approved 2025-09-18"
paper_read_scope: "Archive metadata and abstract only; PDF unavailable"
tags:
  - ethlib
  - external-sources
  - cryptography
  - peerdas
  - security-games
---

# PeerDAS code-binding games and reduction assumptions

Conflicting answers at one position and mutually inconsistent answers at different positions require different winning conditions.
A batch check introduces another distinction: its combined equation can vanish even when individual equations fail.
This note explains those distinctions through selected pinned Lean definitions and a small scalar example.
The cited paper's full argument remains unread because its PDF was unavailable.

## Source roles and access boundary

The [archive record for Proving the Security of PeerDAS without the AGM][paper] names Benedikt Wagner and Arantxa Zapico and records receipt on 2025-09-16 and approval on 2025-09-18.
Its abstract reports replacing reliance on the algebraic group model (AGM) with a standard-model analysis under adaptive rational strong Diffie–Hellman (ARSDH).
The direct PDF returned HTTP 403, and the authors' public research pages pointed to that same PDF.
No paper PDF, exact PDF revision or PDF hash was obtained.
Consequently, the paper's definitions, reduction losses, efficiency arguments and theorem correspondence are not claimed as reviewed.
Its detailed Lean citations are pointers for future source review, rather than substitutes for reading the paper.

[EIP-7594][7594] is Final at the recorded EIPs commit, and its complete text was read.
It describes erasure-coded rows, authenticated cells and columns crossing rows, and permits batch verification of cell proofs in the transaction wrapper.
That EIP supplies protocol requirements.
The game statements below are observations about EthLib at its recorded commit, not independent confirmation that it formalizes the unread paper exactly.
The [custody and recovery note](PeerDAS-custody-sampling-and-recovery-boundaries.md) separates those cryptographic checks from networking and availability assumptions.

## The selected single-cell games

[`PositionBindingWins`][position] asks for one commitment and two different cells at the same index, with both openings accepted individually.
[`CodeBindingWins`][code] instead requires distinct indices, acceptance of every opening, and absence of any polynomial of degree below 4096 explaining all opened cells.
Distinct indices prevent reducing that second condition to a duplicate-position test.
Each actual cell contains 64 evaluations; the codeword uses 128 cell positions.
The degree bound is part of the winning condition, not a conclusion that acceptance alone establishes without a security argument.

The selected games sample the setup secret uniformly from the entire scalar field, give the adversary 4096 G1 powers and 65 G2 powers, and construct the setup from those powers.
The corresponding exponents run from zero through 4095 and zero through 64.
[`WellFormed`][setup] requires the monomial and Lagrange points to represent the appropriate powers and basis evaluations at that secret.
It is not a claim that a deployed ceremony kept the secret unknown.
The [KZG setup note](KZG-openings-and-setup-assumptions.md) treats that distinction separately.

For a well-formed setup, the selected [single-cell acceptance proof][one-cell] obtains the scalar relation:

```text
c - I_j(tau) = w_j * (tau^64 - h_j^64)
```

Here `I_j` interpolates the cell's evaluations, `h_j` identifies its coset, and `c` and `w_j` represent the commitment and proof points.
Its batch-error polynomial has degree zero because this check has one cell.
An accepted root therefore forces that constant error to be zero, without a random-oracle assumption for this step.

## A conditional scalar example

Use the field F17 and polynomials of degree below two, with one scalar per position for this illustration.
These are not Ethereum's 64-point cells or BLS12-381 parameters.
The polynomial `f(X) = 3 + 2X` gives:

| Position | f(position) | Constructed altered answer |
| --- | --- | --- |
| 1 | 5. | 5. |
| 2 | 7. | 7. |
| 3 | 9. | 10. |

The first two answers fix the line: slope two and intercept three.
It predicts nine at position three, so the altered triple has no explanation of degree below two.
The quadratic `4 + 9X + 9X^2` does explain the triple modulo 17, but lies outside the chosen bound.
There is no repeated position.
If all three openings for one commitment were accepted under a corresponding game, the inconsistency would satisfy the code-binding pattern.
By contrast, two accepted answers five and six at position one would satisfy the position-binding pattern.
No commitment, proof bytes or actual acceptance is constructed here.

This illustration stipulates an adversarial output, not a validator-corruption fraction or a data-withholding network model.
Changing the permitted degree changes the consistency question.
It cannot demonstrate that such accepted malicious openings can be generated efficiently.

## What the reduction statements establish

With a target-group model, [`positionBindingAdvantage_le`][position-bound] bounds the selected position-game advantage by the corresponding reduction's bilinear strong Diffie–Hellman (BSDH) advantage.
[`codeBindingAdvantage_le`][code-bound] bounds the code-game advantage by the sum of its BSDH and ARSDH reduction advantages.
The sampled setup and adversary input are part of those definitions.
The reduction searches use that input and the adversary's output, rather than receiving the secret directly.

The [selected assumption predicates][assumptions] require a BSDH solution with `c != tau` representing `1/(tau-c)` in the target group.
ARSDH requires a set S of 4096 scalars and represented scalars satisfying `a != 0` and `a = b * product_{s in S}(tau-s)`.
These conditions alone say nothing about difficulty when the secret is known.
The selected Lean theorems state inequalities for arbitrary adversaries; they supply neither a computational cost model nor a hardness hypothesis making the right-hand advantages small.
The selected statements and proofs were read, without Lean replay or a complete dependency audit.

## Batch acceptance needs its own model

[`BatchCodeBindingWins`][batch-game] checks all opened cells of one commitment in one batch, at distinct indices.
Its [`AcceptsAt` predicate][batch-acceptance] combines successful structural processing with an error-polynomial root at a supplied challenge.
For a well-formed setup, a selected equivalence relates that predicate at the concrete hash's own challenge to the actual verifier returning true.
It does not establish that the executable verifier uses an arbitrary substituted challenge.

For a separate scalar illustration, take nonzero errors `e_0 = 1` and `e_1 = 16` in F17.
Their polynomial is `P(R) = 1 + 16R = 1-R`, which vanishes exactly at R equal to one.
Under an explicitly uniform challenge independent of these fixed errors, that is one of 17 choices, despite both individual errors being nonzero.
This is neither a hash prediction nor a concrete pairing forgery.

The selected [`batchCodeBindingAdvantage_le` theorem][batch-bound] assumes a total bound q on the adversary's oracle queries for every setup input.
It adds `(q+1)*127/r` to the two reduction advantages, where r is the scalar-field order and the extra query belongs to the verifier.
Distinct valid indices bound this particular batch to 128 cells, hence error degree at most 127.
Its oracle is idealized as uniform into the field; the module explicitly distinguishes that model from SHA-256 reduced modulo r.
No transfer bound for that distribution difference is supplied here.
A network column contains cells across commitments, so this one-commitment game must not silently replace a column-game analysis.

Standalone Python checked all 289 lines over F17 against the altered triple, the displayed quadratic, and all 17 residual-polynomial challenges.
No KZG/BLS operation, sampling or corruption experiment, EthLib call, Lean build, proof replay or paper-proof audit ran.
The first research follow-up is to obtain and pin the paper PDF, then compare its exact games, reduction assumptions and efficiency bounds with these selected statements.

[paper]: https://eprint.iacr.org/2025/1683
[7594]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7594.md
[position]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L350-L369
[code]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L423-L447
[setup]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/Commitment.lean#L38-L55
[one-cell]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L245-L309
[position-bound]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L388-L418
[code-bound]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasGames.lean#L554-L658
[assumptions]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/Arsdh.lean#L61-L99
[batch-game]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasBatchGames.lean#L158-L182
[batch-acceptance]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasBatchGames.lean#L67-L106
[batch-bound]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Kzg/PeerDasBatchGames.lean#L270-L311
