---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C03
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
fips197_version: "Updated 2023-05-09"
fips197_sha256: "62c86eb567f13edb8f71826e985da870b04ef6381634f303cdb16e84d47becd1"
sp800_38a_version: "December 2001"
sp800_38a_sha256: "66821162de1e7130c5fb5eedb22140d8d6d013ec51af4550bb095c2a9481a00e"
sp800_38d_version: "November 2007"
sp800_38d_sha256: "d99f3921ccebca049e7522426553aba071dae14ec3d5b6041e8c111a6cb57bba"
tags:
  - ethlib
  - external-sources
  - aes
  - authenticated-encryption
---

# AES counter modes and authentication failure

AES transforms one fixed-size block under a key.
CTR turns that primitive into confidentiality for variable-length data, while GCM adds authenticated decryption with a failure result.
These operations establish different facts, and their counter or IV obligations remain with the caller.

## Fixed editions and selected reading

| Primary source | Edition and passages read |
| --- | --- |
| [FIPS 197-upd1, Advanced Encryption Standard][fips] | Updated May 9, 2023: block/key parameters, §5 and §5.1's forward-cipher algorithm. |
| [SP 800-38A, Methods and Techniques][ctr-source] | December 2001: §6.5, Appendix B's counter requirements and incrementing function, and complete F.5.1 AES-128 CTR example. |
| [SP 800-38D, GCM and GMAC][gcm-source] | November 2007: §§5.2.1–5.2.2, algorithms in §7, §8's uniqueness requirement, and opening authentication-assurance discussion in Appendix B. |

NIST's [FIPS publication page][fips-page] says the 2023 update makes no technical changes to AES.
Revision planning is separate from those algorithm editions.
As checked on 2026-11-17, NIST's [38A revision decision][ctr-plan] identifies IV/counter clarification and authentication guidance as goals.
The [June 2026 GCM pre-draft notice][gcm-plan] explicitly says no draft document is available and proposes removing support for tags below 96 bits.
Those plans do not silently amend the 2007 rules used below.
This reading does not assess the complete cryptanalysis, validation requirements or every subsequent proposal.

## The AES block is always 16 bytes

FIPS 197 specifies AES-128, AES-192 and AES-256, with 128-, 192- and 256-bit keys respectively.
All three operate on 128-bit blocks.
AES-256 therefore does not mean a 256-bit data block.
The forward cipher applies the key schedule and rounds to one block; it supplies no message framing, counter management or authentication tag.

CTR applies that forward cipher even when decrypting:

```text
O_j = AES_K(T_j)
C_j = P_j XOR O_j
P_j = C_j XOR O_j
```

For the last partial block, §6.5 uses only the leading bits of `O_j` that the data needs.
No padding bytes are appended to the message.
The counter blocks must be distinct across all messages under one key, not merely within each call.
Appendix B's incrementing construction and selection of initial counters are separate parts of satisfying that requirement.

## A complete published CTR control

SP 800-38A F.5.1 publishes this 64-byte AES-128 CTR case.
Each plaintext/ciphertext line is one 16-byte block, and the key is public test material:

```text
K  = 2b7e151628aed2a6abf7158809cf4f3c
T1 = f0f1f2f3f4f5f6f7f8f9fafbfcfdfeff

P1 = 6bc1bee22e409f96e93d7e117393172a
P2 = ae2d8a571e03ac9c9eb76fac45af8e51
P3 = 30c81c46a35ce411e5fbc1191a0a52ef
P4 = f69f2445df4f9b17ad2b417be66c3710

C1 = 874d6191b620e3261bef6864990db6ce
C2 = 9806f66b7970fdff8617187bb9fffdff
C3 = 5ae4df3edbd5d35e5b4f09020db03eab
C4 = 1e031dda2fbe03d1792170a0f3009cee
```

The listed counters continue with suffixes `ff00`, `ff01` and `ff02` after T1's `feff`.
The published first forward-cipher output is `ec8cdf7398607cb0f2d21675ea9ea1e4`.
XORing it with P1 gives C1.
That block calculation checks AES on a counter, not decryption through the inverse AES primitive.

F.5.1 contains four full blocks, so the following partial case is derived rather than a separately published vector.
Take the first 17 plaintext bytes, namely P1 followed by `ae`.
With the same initial counter and standard full-block incrementing, the complete ciphertext is:

```text
874d6191b620e3261bef6864990db6ce98
```

The second counter contributes only its first keystream byte, `36`, giving `ae XOR 36 = 98`.
The ciphertext remains 17 bytes.
Changing its first byte from `87` to `86` changes the first decrypted byte from `6b` to `6a`.
CTR has no authentication result that rejects this alteration.
An attacker can make that bit change without knowing the key.

## GCM adds a different acceptance boundary

GCM protects plaintext confidentiality and authenticates the plaintext and additional authenticated data, or AAD.
AAD remains unencrypted.
For a 96-bit IV, Algorithm 4 forms `J0 = IV || 0^31 || 1` and starts data encryption at `inc32(J0)`.
Other supported IV lengths use the specified GHASH-based construction instead.
The tag combines GHASH over padded AAD, ciphertext and their bit lengths with the block-cipher output at J0.
It is not a checksum over plaintext alone.

Algorithm 5 returns plaintext only if the supplied tag equals its computed tag.
Unsupported input lengths or a tag mismatch produce `FAIL`.
Although it computes candidate plaintext internally before the comparison, that plaintext is not the returned result on failure.

For a separate author-derived case, choose a 16-byte zero AES key, 12-byte zero IV, 16-byte zero plaintext, empty AAD and a 16-byte tag.
An independent AES-GCM calculation gives:

```text
C = 0388dace60b6a392f328c2b971b2fe78
T = ab6e47d42cec13bdf53a67b21257bddf
```

These values are calculated here, without attribution to a published GCM vector.
Keeping the original tag while changing C's first byte from `03` to `02` failed authenticated decryption in the calculation.
Separate changes to the tag's first bit and to AAD from empty bytes to `01` also failed.
Those three observations illustrate the acceptance boundary, not a proof that every possible forgery must fail.

## Length, IV and key-wide obligations

The 2007 specification permits tag lengths 128, 120, 112, 104 or 96 bits generally.
It conditionally permits 64 or 32 bits under Appendix C's additional restrictions.
It associates one fixed supported tag length with each key.
It recommends restricting IV support to 96 bits for interoperability, efficiency, and simplicity, while defining behaviour for other supported lengths.
Its byte-aligned plaintext limit is `2^39 - 256` bits, with separate IV and AAD length limits.

Section 8 bounds the probability of reusing a key/IV pair on distinct input data by at most `2^-32` across all instances using that key.
IV-generation constructions and invocation limits require additional application-wide management.
A successful tag check alone does not establish that the encryption side met those historical obligations.
Repeatedly checking a previously valid tuple is also not a replay-detection mechanism.

## Read the EthLib interface precisely

At commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`ciph`][lean-block] exposes AES-128 and AES-256 through the typed `KeySize` interface.
That observed selection does not change FIPS 197's three algorithms.
[`ctr`][lean-ctr] consumes a forward-cipher function, counter width, initial counter and byte array.
Its last block may contain fewer than 16 whole bytes.
It does not implement arbitrary bit-length messages or retain counter history.

[`Gcm.checkLengths`][lean-lengths] accepts tags of 12–16 bytes and explicitly rejects the 4- and 8-byte short-tag cases.
This is an implementation's supported profile within the fixed edition, separate from NIST's revision plans.
[`encrypt` and `decrypt`][lean-gcm] return ciphertext/tag or plaintext/error, but do not remember IV use or a fixed tag length associated with a key.
The caller must preserve those wider invariants.
The module explicitly makes no constant-time claim, including its tag comparison.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
PyCryptodome 3.23.0 reproduced all four published CTR blocks, the first counter's AES output, the derived 17-byte case and its bit alteration.
It also calculated the separate GCM example and its three rejection cases.
No EthLib binding, Lean build, proof replay, complete protocol integration or security reduction ran.

[fips]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf
[fips-page]: https://csrc.nist.gov/pubs/fips/197/final
[ctr-source]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38a.pdf
[gcm-source]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf
[ctr-plan]: https://csrc.nist.gov/news/2023/decision-to-revise-nist-sp-800-38a
[gcm-plan]: https://csrc.nist.gov/pubs/sp/800/38/d/r1/2prd
[lean-block]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Aes/Block.lean#L307
[lean-ctr]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Aes/Ctr.lean#L70
[lean-lengths]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Aes/Gcm.lean#L105
[lean-gcm]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Aes/Gcm.lean#L155
