---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C09
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
paper_version: "CACR 2010-10, extended report, 2010-12-01"
paper_sha256: "15cda500272b20c04fc71dda0db8fb31bbc56b1caee4b02172f5ecf41079ff0d"
tags:
  - ethlib
  - external-sources
  - kzg
  - trusted-setup
---

# KZG openings and setup assumptions

A KZG opening checks an evaluation of a committed polynomial.
Before changing it, separate the polynomial identity, valid input encodings, setup consistency, and the assumption that an adversary cannot forge openings.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) supplies the related group-validation context.

## Sources and their roles

| Source | Selected material and role |
| --- | --- |
| Kate, Zaverucha and Goldberg, [*Polynomial Commitments*][paper], CACR 2010-10, dated 1 December 2010. | Extended ASIACRYPT 2010 report. Sections 2, 3.1–3.2, the trapdoor and degree discussions in 3.4, and Appendix C.1 explain the construction and its security model. |
| [EIP-4844][e4844], pinned EIPs commit above. | Parameters, Type aliases, Cryptographic Helpers, and Point evaluation precompile state Ethereum requirements and reference detailed KZG functions. |
| [EIP-7594][e7594], same pin. | Specification and Networking introduce cell openings and their transport obligations. They do not turn a single-point equation into a complete sampling-security argument. |

The paper defines a research construction, not Ethereum's wire format.
For contributions, cite the EIP for its rules.
EthLib's [source policy][policy] permits provenance notes naming an EIP's linked consensus-specs text, without making that peer specification an independent authority.

The [KZG README][readme] records an unresolved choice between the Final EIPs' frozen references, EthLib as the later referenced text, and the consensus text amended through activation and removal.
There is a concrete difference: EIP-4844's [frozen KZG helper][frozen] uses `ENDIANNESS`, whose [global value is little-endian][endian], and leaves setup contents `TBD`.
EIP-4844's precompile explicitly requires big-endian `z` and `y`.
EthLib decodes big-endian.
This note preserves that conflict and does not select a normative reading.

## The opening identity

Write `G1` and `G2` for generators of prime-order groups, and `e` for their bilinear pairing.
Scalars belong to their common scalar field.
A structured reference string (SRS) supplies powers `tau^i * G1` through a maximum degree `d`, with the required `G2` points, including `tau * G2`.
For `deg(f) <= d`, linear combination of the powers computes the commitment:

```text
C = f(tau) * G1
y = f(z)
q(X) = (f(X) - y) / (X - z)
pi = q(tau) * G1
```

The division defines a polynomial because `y = f(z)`.
For nonconstant `f`, its quotient has degree at most `d - 1`.
A constant polynomial has zero quotient.
The prover uses public setup points, without needing `tau` or the scalar `f(tau)`.

The verifier checks the equation implemented by [`verifyKzgProofWith`][core]:

```text
e(C - y * G1, G2) = e(pi, tau * G2 - z * G2)
```

Bilinearity reduces honest verification to `f(tau) - y = q(tau) * (tau - z)`.
This establishes correctness under consistent setup points.
It does not establish computational binding, secrecy of `tau`, or a degree bound on arbitrary adversarial group elements.
The supported polynomial degree is part of the setup and security model, not a number that this pairing equation reads from a commitment.

## A small scalar example

Work in `F17`, with every operation modulo 17:

```text
f(X) = 3 + 2X + X^2
z = 4
y = f(4) = 27 mod 17 = 10
q(X) = X + 6
(X - 4)(X + 6) = X^2 + 2X + 10 = f(X) - 10
```

For illustration, publish `tau = 5`.
Then `f(5) = 4`, `q(5) = 11`, and both sides of the opening identity equal 11:

```text
f(5) - 10 = 4 - 10 = 11
(5 - 4) * 11 = 11
```

Publishing `tau` also demonstrates the failure of binding.
For the false value `y' = 9` at `z = 4`, choose proof scalar `(4 - 9)/(5 - 4) = 12`.
It satisfies the scalar verification equation even though `f(4) != 9`.
Generally, knowledge of `tau` lets an attacker form `pi' = (tau - z)^(-1) * (C - y' * G1)` when `z != tau`.
This is an intentionally insecure scalar illustration, not a BLS12-381 vector, actual pairing test, or usable trusted setup.

## What binding assumes

The setup secret can be discarded after generation.
The intended binding model keeps it unknown to adversaries, although honest verification still works when it is known.

The paper's Definition 3.1 distinguishes **polynomial binding**, which excludes two different polynomial openings of one commitment, from **evaluation binding**, which excludes two accepted values at one point.
These are computational claims about efficient adversaries and setup-generated parameters, not absolute uniqueness of every polynomial under evaluation at `tau`.

Definition 2.3 states the `t`-strong Diffie–Hellman assumption: given powers through `tau^t`, an efficient adversary cannot find `(c, (tau + c)^(-1) * G)` with `tau + c != 0`, except with negligible probability.
Appendix C.1 reduces conflicting evaluations to this problem.
Theorem 3.2 separately assumes discrete-log hardness for its hiding claim.
Ethereum's deterministic blob commitment is not an encryption mechanism or the paper's randomized PolyCommitPed variant.
Do not infer privacy, batch security, or a degree-enforcement theorem from the single-opening reduction.

The paper presents symmetric groups and discusses adaptation to asymmetric pairings.
EthLib's BLS12-381 check uses distinct `G1` and `G2` groups.
Its corresponding reduction must account for the setup information in both groups.

## Ethereum's representation boundary

A blob holds 4096 evaluations of a polynomial of degree below 4096, not its coefficient list.
For domain points `omega_i`, let `L_i` be the Lagrange polynomials satisfying `L_i(omega_j) = 1` when `i = j`, and 0 otherwise.
Then `f = sum_i f(omega_i) * L_i`, so Lagrange setup points `L_i(tau) * G1` yield the same commitment.
The matching order matters: [`TrustedSetup`][setup] stores the Lagrange points in bit-reversed order, matching EthLib's blob domain.
Its 4096 monomial `G1` powers support degree at most 4095.

EIP-4844 specifies 32 bytes per scalar and 48 bytes per compressed commitment or proof.
Its precompile MUST reject `z` or `y` whose integer is at least `BLS_MODULUS`.
Do not reduce untrusted input modulo the field: that would accept multiple encodings of one value.
EthLib's [`bytesToBlsField`][polynomials] rejects such inputs, and blob decoding applies it to every element.
Field reduction used to derive a hash challenge is a separate operation.

For commitments and proofs, EIP-4844 requires BLS `KeyValidate` checks with an explicit exception allowing the identity point.
The zero polynomial commits to the identity.
EthLib's `validateKzgG1Point` preserves that exception, so importing a BLS signature rule that rejects every identity would change KZG behaviour.
EIP-7594 further requires verification of cell proofs and their wrapper counts.
Those obligations need their own multipoint and batching analysis.

## EthLib connections and review limits

Setup file integrity, algebraic consistency, and secrecy are different claims.
[`TrustedSetupData`][setup-data] records a source commit and SHA-256.
`TrustedSetup.ofBytes` checks list lengths and curve decompression, not subgroup membership or powers of one secret.
Neither successful decoding nor a file hash establishes honest generation or destruction of the trapdoor.
[`SOUNDNESS.md` D3][soundness] separates these assumptions and records more specific checks.

Two inspected theorems show the distinction.
[`verifyKzgProof_computeKzgProof`][honest] assumes `TrustedSetup.WellFormed s tau`, a successful commitment computation, and a successful proof computation, then concludes that verification returns `.ok true`.
[`findSdh_valid_bytes`][binding] assumes a well-formed setup and two accepted openings of the same commitment at the same point.
It concludes that the decoded values agree or its search returns a strong Diffie–Hellman solution.
That algebraic reduction does not prove that solving the problem is computationally hard.

The cited source sections, definitions, and these complete theorem statements and proofs were read.
The explanation and selected statements received a separate source review.
An independent calculation checked the source-derived quotient identity at all 17 scalar inputs, the opening equation at `tau = 5`, and the known-trapdoor false opening.
No Lean build, proof replay, Ethereum KZG vector, ceremony audit, or full binding-coverage assessment was performed.

[paper]: https://cacr.uwaterloo.ca/techreports/2010/cacr2010-10.pdf
[e4844]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4844.md
[e7594]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7594.md
[policy]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/CONTRIBUTING.md#sources-of-truth
[readme]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Kzg/README.md#the-normative-text
[frozen]: https://github.com/ethereum/consensus-specs/blob/86fb82b221474cc89387fa6436806507b3849d88/specs/deneb/polynomial-commitments.md#bytes_to_bls_field
[endian]: https://github.com/ethereum/consensus-specs/blob/86fb82b221474cc89387fa6436806507b3849d88/specs/phase0/beacon-chain.md#constants
[core]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Kzg/Core.lean#L167
[setup]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Kzg/TrustedSetup.lean#L29
[polynomials]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Kzg/Polynomials.lean#L57
[setup-data]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Kzg/TrustedSetupData.lean#L5
[soundness]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/SOUNDNESS.md#d3-the-kzg-trusted-setup
[honest]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Kzg/Verify.lean#L112
[binding]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Kzg/Binding.lean#L189
