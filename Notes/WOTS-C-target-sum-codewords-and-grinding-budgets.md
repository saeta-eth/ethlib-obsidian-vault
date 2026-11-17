---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C14
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
paper_version: "NIST-hosted author paper, September 14, 2022"
comparison_design_commit: "4af2cdddd7cd41b9df2cc7e123e314a91b0b4462"
tags:
  - ethlib
  - external-sources
  - hash-signatures
  - wots
  - target-sum-codes
---

# WOTS+C target-sum codewords and grinding budgets

A fixed digit sum prevents deriving another distinct codeword by walking every chain forwards.
Requiring a digest to have that sum also makes the signer search for an admissible digest.
Counting such digests explains a retry cost under an explicit random-function assumption, but does not establish a forgery bound.
This note separates those three claims.

## Exact paper and reading scope

The primary argument read here is Hülsing, Kudinov, Ronen and Yogev's [SPHINCS+C: Compressing SPHINCS+ With (Almost) No Cost][paper], hosted by NIST for its fourth PQC standardization conference.
Its title page dates it September 14, 2022.
The 1258853-byte PDF has SHA-256 `c520c9a6f1c792f952a278ba259bcdf1271fa69c16dd765f7b9db35f1852f5a9`.
The [ePrint 2022/778 record][record] records receipt on June 16 and revision on September 14, 2022.
Its PDF returned HTTP 403, so byte identity with that archive version was not established.

Reading covered §3's WOTS+C construction, §3.1's security experiment, §3.2's argument outline, §3.3's counting/retry analysis, and selected §5 discussion of tweaks and security notions.
The full reduction proofs, quantum analysis, complete SPHINCS+C construction, benchmarks and parameter-search results were not audited.
NIST hosts an author research paper here, rather than a standard prescribing the leanXMSS variant.

## Incomparability is a combinatorial fact

For digits `x_i` in `0..w-1`, a target-sum code requires `sum_i x_i = T`.
Suppose every digit of x is at most the corresponding digit of y and both sums equal T.
Then each digit must agree, because any strict increase would increase the sum.
Consequently, two distinct vectors of the same sum each have a coordinate below the other's.

A chain disclosure allows forward computation under that chain's public parameters.
Incomparability therefore obstructs deriving a different admissible vector using only forward steps from one disclosure.
It does not prove hash preimages or collisions difficult, and distinct messages can raise a separate question about their digest encodings.
The [stateful signature note](Stateful-hash-signatures-and-leanXMSS-boundaries.md) treats one-time-key allocation and reuse separately.

Section 3 removes the checksum chains and adds prescribed zero positions to omit further chains.
The paper's description uses initial zero chains, while §3.3 generalizes this condition to fixed zero bits.
The leanXMSS comparison fixes the last six base-4 digits to zero.
These choices must not be exchanged as byte formats without specifying the digest mapping.
The paper's introductory Algorithms 2–3 also list a signature component r without defining it there, so this note takes no complete wire format from that fragment.

## Counting a complete small admissible set

Choose a constructed eight-bit digest interpreted as four base-4 digits.
Retain its first three digits, require their sum to be three, and require the fourth digit to be zero.
The ten retained vectors are:

```text
(0,0,3) (0,1,2) (0,2,1) (0,3,0) (1,0,2)
(1,1,1) (1,2,0) (2,0,1) (2,1,0) (3,0,0)
```

All 256 possible digests are equally likely under the stipulated uniform-digest model.
Each retained vector has exactly one extension with the final digit zero.
Thus the success probability per fresh trial is `p = 10/256 = 5/128`.
Ignoring the zero digit would incorrectly use `10/64`, four times too large.
The example specifies digits and their constraints, without constructing messages, randomizers or signatures.

For length L and digit range `0..w-1`, define `N(L,w,T)` as the coefficient of `z^T` in `(1+z+...+z^(w-1))^L`.
The recurrence counts ordered strings:

```text
N(0,w,0) = 1
N(0,w,T) = 0 for T > 0
N(L+1,w,T) = sum_{0 <= d < w, d <= T} N(L,w,T-d)
```

An unattainable target gives zero admissible strings, regardless of the retry budget.
This recurrence counts a set, without asserting anything about the distribution of a concrete hash.

## A retry cap changes both outcomes and averages

Assume successive tests use distinct fresh random-function inputs and hence independent uniform digests.
Section 3.3 explicitly uses this random-function and independent-trial model.
For positive p, the uncapped expected trial count is `1/p`, and failure after A trials has probability `(1-p)^A`.
Counting tests until success or exhaustion instead gives `sum_{i=0}^{A-1}(1-p)^i = (1-(1-p)^A)/p`.

For the constructed set:

| Quantity | Result |
| --- | --- |
| Uncapped mean tests | 25.6. |
| Hard cap A | 32 tests. |
| Failure within that cap | `(123/128)^32`, approximately 0.2794116676. |
| Mean tests until success or exhaustion | Approximately 18.4470613092. |

The smaller capped mean includes exhausted attempts and is not an improved successful-signature time.
Repeating an identical random-function input returns its existing output, rather than another independent trial.
Applying the formula to an adaptive execution with an existing oracle history requires justification beyond this count.

## The actual 58-digit comparison

The [pinned leanXMSS design][xmss] uses 58 retained base-4 digits of target 106 and six zero digits in a 128-bit digest.
The count is:

```text
N(58,4,106) = 323504418562481994618807509487594
p = N(58,4,106) / 2^128
  ≈ 0.000000950694041215616
1/p ≈ 1051863.11962294 trials
```

The denominator includes all 64 digits, including the six fixed to zero.
For the design's cap `A = 2^32`, `A*p ≈ 4083.199815523146`.
Under the same independent-uniform assumption, `(1-p)^A <= exp(-A*p)`.
This is arithmetic about one encoding search, not a measured Keccak runtime or a theorem about every signing history.
Approximately `2^20` trials to find an admissible digest is also not a `2^20` forgery attack.

## Security budgets and selected Lean statements

In paper §3.1, q counts signing-oracle queries in the EU-CMA experiment, with the one-time case limited to one.
It is not the number of hashes tried by the signer.
The pinned leanXMSS design uses a different budget in its stated classical random-oracle theorem: every hash call of the full experiment counts, including repeated inputs.
Its experiment also prohibits repeated leaf-index requests.
That scheme-specific statement cannot be transferred from the generic codeword count or from SPHINCS+C's security experiment.

At the recorded EthLib commit, [`incomparable_of_sum_eq`][incomparable] assumes equal digit sums and distinct vectors and concludes the two opposing strict-coordinate inequalities.
[`countRow_eq_card`][count] equates the dynamic-program entry with the cardinality of digit strings for every target within the supplied bound.
Both selected statements and their proofs were read.
Neither theorem assumes a hash distribution or concludes cryptographic unforgeability.
The generic [`findFirst`][search] tests successive trials up to its fuel and returns the first admissible result or none.
This definition supplies an explicit exhaustion outcome, without making the tests probabilistically independent.

Python enumerated the small set, checked its capped calculations, and compared the actual count from dynamic programming with the paper's inclusion-exclusion count.
No hash evaluation, signing run, empirical timing, allocator, EthLib binding, Lean build, proof replay or full reduction audit ran.
A probability or security interpretation of these counts requires an explicit encoding, query model and retry cap.

[paper]: https://csrc.nist.gov/csrc/media/Events/2022/fourth-pqc-standardization-conference/documents/papers/sphincs-plus-c-pqc2022.pdf
[record]: https://eprint.iacr.org/2022/778
[xmss]: https://github.com/leanEthereum/leanVM/blob/4af2cdddd7cd41b9df2cc7e123e314a91b0b4462/doc/lean_xmss/main.tex
[incomparable]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Wots/TargetSum.lean#L88-L101
[count]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Proofs/Wots/TargetSum.lean#L198-L219
[search]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Wots.lean#L37-L49
