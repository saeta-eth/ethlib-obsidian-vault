---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C10
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
target_eprint_revision: "2025-05-18; metadata only, corrected argument unread"
eprint_record_sha256: "3bc1c8c87fa39e3c48824d9fe93fb6473ea06e955c5d1a3b7070b369e64722ae"
author_manuscript_commit: "ce20308c4b1015370a2641fb77884ea267d9fda3"
author_pdf_sha256: "3517c8f0cdb46ae046fc63cd0199b10b0d538716b645061a06a2a94998c55e84"
tags:
  - ethlib
  - external-sources
  - kzg
  - fft
---

# Amortized KZG openings and cell-proof computation

Many opening proofs share coefficient sums that can be reused before evaluating them across a root-of-unity domain.
An FFT computes those evaluations efficiently; a convolution can compute the shared sums.
Cell proofs use quotients by a coset's vanishing polynomial, so their shared sums differ from ordinary single-point openings.

## Source versions and an incomplete reading

The official [ePrint 2023/033 record][record], *Fast amortized KZG proofs* by Dankrad Feist and Dmitry Khovratovich, reports a section 2.2 bug fix and latest revision 2025-05-18, the last of three revisions.
The corrected PDF and version-history page returned HTTP 403, including attempted timestamped PDF routes.
That corrected argument remains unread; the accessible record's SHA-256 identifies metadata, not the unavailable paper bytes.

The primary fallback is the authors' [older manuscript][author-pdf], *Fast amortized Kate proofs*, fixed at repository commit `ce20308c4b1015370a2641fb77884ea267d9fda3`.
The four-page PDF is dated 9 March 2020 and has the frontmatter SHA-256; the repository commit is dated 18 October 2022.
Its accompanying [TeX][author-tex] has SHA-256 `c4e28d810a4da1ab143bd2c0893ad2bd92c43c72186d11f0c347c93c13dc29ff`.
Reading covered the PDF's §§1.3, 2–2.1 and selected §3/§3.2 coset formulas, with TeX as notation support.
These older sources are not asserted identical to the corrected ePrint revision.
The example below is independently derived from quotient identities, not a validation or repair of that unread correction.

[EIP-7594][7594] is Final at the recorded EIPs commit and is the source for Ethereum's cell-proof transport obligations.
The manuscript explains an algorithmic relationship, not Ethereum's byte encodings, cell order, verification model or security reduction.
The [KZG note](KZG-openings-and-setup-assumptions.md) covers the opening/setup distinction; the [PeerDAS note](PeerDAS-custody-sampling-and-recovery-boundaries.md) separates cell authentication from availability.

## Why separate point proofs share work

Write `[a] = a*G` for a group element, with setup powers `[tau^i]`.
For `f(X) = sum_{i=0}^d a_i*X^i` of degree at most d, the single-point proof at z commits to `q_z(X) = (f(X)-f(z))/(X-z)`.
Expanding the quotient and regrouping powers of z gives:

```text
H_j = sum_{i=0}^{d-1-j} a_{i+j+1} * [tau^i],  0 <= j < d
pi(z) = sum_{j=0}^{d-1} H_j * z^j
```

The H values depend on the polynomial and setup, not the chosen opening point.
Evaluate this group-valued polynomial at N roots of unity to produce N separate opening proofs.
The older manuscript identifies Toeplitz structure in computing H and an FFT for evaluating it.
This shares generation work; it does not turn all the outputs into one multi-point proof or establish security.

For the arithmetic below, define the zero-indexed forward transform by `DFT_w(v)[j] = sum_k v[k]*w^(j*k)`.
The inverse uses `w^(-1)` and multiplies by `N^(-1)`.
Padding two vectors sufficiently prevents circular-convolution wraparound, giving `conv(A,B) = IDFT(DFT(A)*DFT(B))`, with pointwise multiplication inside.
Here both vectors are scalars; an actual SRS-based convolution has a group-valued factor and uses scalar multiplication.
Root order, padding length, coefficient reversal and output order are part of the construction.

## Complete scalar control over F17

Use arithmetic modulo 17, `f(X)=3+5X+2X^2+7X^3`, and public `tau=6`.
This tiny field and disclosed scalar are deliberately insecure and do not instantiate cryptographic KZG.
Group brackets are replaced by their scalar contents solely to check the linear identities.
The four-point domain has primitive root 4 and ordered points `[1,4,16,13]`.

The shared coefficients are:

```text
H_0 = 5 + 2*6 + 7*6^2 = 14 mod 17
H_1 = 2 + 7*6 = 10 mod 17
H_2 = 7
H_3 = 0  (padding)
```

To obtain them by convolution, use primitive eighth root 2 and these eight-entry vectors:

```text
A = [7,2,5,0,0,0,0,0]
B = [1,6,2,0,0,0,0,0]  # 6^2 = 2 mod 17
DFT_2(A) = [14,14,10,3,10,6,11,5]
DFT_2(B) = [9,4,6,7,14,14,9,13]
pointwise product = [7,5,9,4,4,16,14,14]
c = IDFT_2(product) = [7,10,14,0,10,0,0,0]
H = [c[2],c[1],c[0],0] = [14,10,7,0]
```

Each unpadded input has length three, so its linear convolution needs five positions; padding to eight prevents aliasing.
Now `DFT_4(H)` gives these scalar quotient evaluations:

| z | f(z) | q_z(6) |
| --- | --- | --- |
| 1 | 0 | 14 |
| 4 | 10 | 13 |
| 16 | 10 | 11 |
| 13 | 9 | 1 |

Here `f(6)=2`, and every row satisfies `(6-z)*q_z(6) = 2-f(z)` modulo 17.
For example, z=4 gives quotient `6+13X+7X^2`, whose value at 6 is 13; `2*13 = 2-10 = 9 mod 17`.
The zero-indexed point order above is explicit and should not be replaced by an unexplained rotation of outputs.

## A cell is a coset opening

For a coset of size ell with shift y, its vanishing polynomial is `X^ell-y^ell`.
Interpolate the cell values as a remainder R of degree below ell, then commit to `Q=(f-R)/(X^ell-y^ell)`.
In this toy example, take ell=2 and cosets `{1,16}` and `{4,13}`.
Both quotients are `Q=2+7X`, so `Q(6)=10`, while their remainders differ:

```text
f = (X^2-1)*(2+7X) + (5+12X)
f = (X^2-16)*(2+7X) + (1+15X)     over F17
```

Thus the two cell-proof scalars are `[10,10]`, distinct objects from the four point-proof outputs above.
For larger polynomials, the analogous reusable sums are `T_d=sum_i a_(i+ell*d)*[tau^i]`, and the cell proof evaluates `sum_{d>=1} T_d*c^(d-1)` at `c=y^ell`.
This is the connection to computing many coset proofs together, not an assertion that Ethereum cells contain two elements.

## Selected EthLib algorithm and remaining work

At the recorded commit, [`computeCellsAndKzgProofsPolynomialcoeff`][lean-cells] uses 64-element cosets and 128 cell outputs.
For 4096 coefficients, its first phase computes 63 multi-scalar multiplications of setup points and shifted coefficient slices.
Its second phase pads those results and uses a G1 FFT followed by bit reversal to put proofs in cell order.
These observations do not say that the first phase itself uses FFT-based Toeplitz multiplication.
[`fftG1`][lean-fft] has explicit power-of-two/equal-length shape conditions; it returns the input unchanged outside that shape.
The toy calculation uses valid transform sizes, not that fallback behavior.

EIP-7594 requires wrapped proofs for extension cells and checking them against the corresponding blobs and commitments.
Choosing a faster generation method does not remove those validation obligations.
A further contribution needs the corrected paper revision, reconciled indexing/coset conventions, and a precise operation or complexity claim before assessing an optimization.
This bounded entry leaves the corrected argument and any end-to-end optimization assessment open.

Standalone Python compared direct modular DFTs, schoolbook convolution, all four quotient identities and both coset divisions.
No elliptic-curve group, pairing, BLS field, Ethereum cell proof, setup validation, benchmark, EthLib call, Lean build or proof replay ran.

[record]: https://eprint.iacr.org/2023/033
[author-pdf]: https://github.com/khovratovich/Kate/blob/ce20308c4b1015370a2641fb77884ea267d9fda3/Kate_amortized.pdf
[7594]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7594.md
[lean-cells]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Kzg/Cells.lean#L167
[lean-fft]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Kzg/Fft.lean#L168
[author-tex]: https://github.com/khovratovich/Kate/blob/ce20308c4b1015370a2641fb77884ea267d9fda3/main.tex
