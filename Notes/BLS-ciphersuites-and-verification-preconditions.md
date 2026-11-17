---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C08
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
bls_signature_draft: "draft-irtf-cfrg-bls-signature-04, 2020-09-10"
hash_to_curve_rfc: "RFC 9380, 2023-08"
tags:
  - ethlib
  - external-sources
  - bls
  - signatures
---

# BLS ciphersuites and verification preconditions

A BLS pairing equation authenticates signers only within a scheme's validation and key-registration rules.
For Ethereum's proof-of-possession scheme, validating a compressed public key does not establish possession of its secret key.
The caller must preserve that distinction when using aggregate verification.

## Sources and version boundaries

| Source | Selected passages examined |
| --- | --- |
| [BLS signature draft-04][draft] | Sections 1.3, 2.1–2.2, 2.5–2.9, 3.1–3.3, 4.1–4.2.3, 5.1–5.5, and Appendices A and C. Scheme rules and ciphersuites. |
| [RFC 9380][rfc] | Sections 2.2.5, 3–3.1, 5.3.1, 5.3.3, 8.8.2, and 10.1. Hash-to-curve construction, domain separation, and security qualifications. |
| [Pinned EthLib BLS code][signatures] | Active definitions, decoding checks, selected algebraic theorem, and selected security-game statements described below. Implementation evidence. |

Draft-04 is the dated September 2020 Internet-Draft, not an RFC or a claim about the latest BLS draft.
Its hash-to-curve reference is draft-09, dated June 2020.
EthLib's [README][readme] explicitly selects RFC 9380 for hash to curve instead.
This note keeps that version boundary visible; it does not establish equivalence between every historical hash draft and the RFC.
The [validator-key note](Validator-key-derivation-and-keystores.md) covers key derivation and storage; the [KZG note](KZG-openings-and-setup-assumptions.md) covers another use of BLS12-381 with different verification rules.

## Bytes, points, and a pairing equation

The minimal-pubkey-size variant puts public keys in prime-order subgroup `G1` and signatures in prime-order subgroup `G2`.
EthLib represents these as compressed 48-byte and 96-byte vectors, respectively.
These lengths describe representations; they do not prove curve membership, subgroup membership, or secret-key possession.

Draft-04's `KeyValidate` decodes a canonical public key, rejects the identity point, and checks subgroup membership.
`CoreVerify` separately decodes the signature and checks its subgroup before validating the key and evaluating the pairing equation.
The public key's non-identity requirement must not become an invented blanket rejection of identity signatures: the draft's signature check is subgroup membership.
RFC 9380 §10.1 also permits a hash-to-curve output to be the identity, with negligible probability for random inputs under its stated conditions.

EthLib's [decoders][compress] check the compression flag, canonical infinity encoding, field bounds, and whether the coordinates recover a curve point.
`G1.keyValidatePoint` then rejects infinity and checks the public-key subgroup.
`coreVerify` applies `G2.inSubgroup` to the decoded signature.
Decoding alone therefore differs from verification.
Likewise, draft `Aggregate` and EthLib `aggregate` add decoded signatures without validating their subgroups; successful aggregation is not successful signature verification.

Write `P` for the `G1` generator, `x` for a nonzero secret scalar, and `Q = H(m)` for the message's `G2` hash.
Signing gives `R = xQ`, and bilinearity gives:

```text
PK = xP
e(P, R) = e(PK, Q)
e(P, R) * e(-PK, Q) = 1
```

The first two arguments live in additive elliptic-curve groups; pairing outputs live in multiplicative `GT`.
EthLib uses the product form through `pairingCheck`.
This is conditional algebra over valid subgroup elements, not a proof that computing a forgery is infeasible.

## Three defenses against rogue keys

Draft-04 defines three schemes, whose aggregation rules must travel with their ciphersuites.
The **basic** scheme rejects an aggregate when any two input messages are equal.
The **augmentation** scheme signs the canonical public-key bytes concatenated with the message, and verification reconstructs that concatenation.
The **proof-of-possession** scheme, abbreviated PoP, instead adds a separate proof for each public key.
These are different defenses; removing the basic scheme's distinct-message check does not turn it into the PoP scheme.

Draft-04 §3.3 requires every public key used by PoP `Verify`, `AggregateVerify`, or `FastAggregateVerify` to have a proof for which `PopVerify` returns `VALID`.
The obligation applies to all three operations.
`KeyValidate` cannot discharge it.
`PopProve` signs the canonical public-key bytes using a separately domain-separated hash; `PopVerify` checks that proof, including key and signature validation.
An integration must retain the association between the exact public key and its successful proof verification.

For a common message, `FastAggregateVerify` sums the public keys and invokes `CoreVerify` on that sum.
The draft requires `n >= 1`.
The resulting aggregate key must also validate, so individually valid keys whose sum is the identity do not pass this operation.
EthLib adds explicit validation of each input key before summing and rejects an empty list in `fastAggregateVerify`.
Its Ethereum wrapper, `ethFastAggregateVerify`, has an explicit exception: an empty list passes exactly with the canonical infinity signature.
That branch is not evidence of signatures from a nonempty signer set.
Neither wrapper checks a 32-byte message length; application-specific message construction remains a caller obligation.

## Why possession matters: an algebraic trace

This original example derives from the draft's pairing rules and rogue-key defense.
Assume prime-order groups, a victim key `PKv = xP` with `x != 0`, and an attacker-chosen scalar `a` with `a != 0` and `a != x` modulo `r`.
For the common message `m`, let `Q = H(m)`.

| Step | Construction or consequence |
| --- | --- |
| Publish a rogue key | `PKmal = aP - PKv = (a - x)P`, a non-identity subgroup point. |
| Sum both keys | `PKv + PKmal = aP`, also non-identity. |
| Construct one signature | `R = aQ`, using only the attacker's chosen scalar and public hash. |
| Evaluate the equation | `e(P, aQ) = e(aP, Q)`, so the fast aggregate equation passes for both keys. |

The attacker can form the rogue point from public data without knowing `x` or the rogue key's scalar `a - x`.
Key validation alone therefore accepts the kind of point needed for this attack.
A verified PoP is the additional scheme requirement; the example does not construct such a proof for the rogue key.
These are symbolic subgroup points and an algebraic derivation, not real compressed points, signatures, or an implementation test.
EthLib's [`fastAggregateVerify_rogue`][rogue] states this construction with explicit nonzero and unequal scalar hypotheses.

## The two domains are exact bytes

Draft-04 §4.2.3 and EthLib's active definitions use these ASCII byte strings, without a terminating zero byte:

```text
signature DST: BLS_SIG_BLS12381G2_XMD:SHA-256_SSWU_RO_POP_
PoP DST:       BLS_POP_BLS12381G2_XMD:SHA-256_SSWU_RO_POP_
```

Each has 43 bytes; their prefixes distinguish message signing from proof generation even when the message bytes equal public-key bytes.
RFC 9380 requires distinct, nonempty domain separation tags for independent encodings.
Its XMD expansion incorporates `DST_prime = DST || I2OSP(len(DST), 1)`, so these tags each acquire a final length byte `0x2b` inside that construction.
The length byte is not part of either configured DST.
EthLib's [`hashToG2`][hash] uses the RFC's `BLS12381G2_XMD:SHA-256_SSWU_RO_` pipeline: two field elements, curve mappings, addition, and cofactor clearing.
The ciphersuite tag and the underlying hash-to-curve suite ID serve distinct roles.

## Security claims and review limits

Draft-04 §5.5 models the message and public-key hashes as random oracles.
RFC 9380 §10.1 explains the conditions for its hash-to-curve construction's random-oracle replacement and cautions that indifferentiability does not preserve every possible protocol proof.
Neither text proves a concrete elliptic-curve problem hard.

EthLib's selected [single-signature game][game] allows an adversary to choose hash and signing queries during its run; a forgery must concern a message never submitted for signing.
The selected [byte-game bound][bound] states `forgeAdvantage <= q * solverAdvantage` for its named co-CDH solver, assuming `q > 0` and at most `q` hash calls in every run.
The count includes direct hash queries, signing-oracle hashes, repeated messages, and final verification's hash call.
This cited game samples the secret scalar from the whole field, including zero; the nonzero-key refinements are separate statements.
Its [co-CDH problem][cocdh] supplies `aP`, `aP2`, and `cP2` and asks for `acP2`, where `P2` generates `G2`.
The extra `aP2` input matters when comparing assumptions.
This note does not audit the separate aggregate-PoP games or turn the pairing trace into a computational-security theorem.

Review covered the cited source passages and selected definitions and theorem statements.
The explanation and selected statements received a separate source review.
The two ASCII byte counts, XMD length suffix, and symbolic cancellation in the rogue-key example were checked independently.
No Lean build, proof check, pairing execution, signature vector, or constant-time assessment was run for this note.
Further contributions can use these boundaries to investigate caller PoP registration, cancellation of aggregate keys, and wrong-domain rejection without treating an algebraic check as a security audit.

[draft]: https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-04.txt
[rfc]: https://www.rfc-editor.org/rfc/rfc9380
[signatures]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/Signatures.lean#L26-L229
[readme]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/README.md#L16-L35
[compress]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/Compress.lean#L124-L264
[hash]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/HashToCurve.lean#L256-L264
[rogue]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Bls/SignaturesUnique.lean#L124-L156
[game]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Bls/Security/Game.lean#L36-L129
[bound]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Bls/Security/Refinement.lean#L684-L740
[cocdh]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Bls/Security/CoCdh.lean#L7-L39
