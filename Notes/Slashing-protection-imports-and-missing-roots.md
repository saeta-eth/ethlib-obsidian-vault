---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: I05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
eip3076_sha256: "59778ec2e103cc99d4a0da159b64a6137634bc6d10d20e016e31c8ee1444ace6"
tags:
  - ethlib
  - external-sources
  - validator-signing
  - slashing-protection
---

# Slashing protection imports and missing roots

An imported signing history constrains what a validator may sign next; it does not authorize signing or authenticate the history itself.
A missing signing root removes evidence that a request repeats the same message.
Merging records must also preserve refusal boundaries from earlier imports, including boundaries that a sparse file represents implicitly.

## Source and interpretation

The primary source is [EIP-3076][eip] at `8194528a…`, whose recorded status is **Last Call** and whose interchange version is **5**.
Reading covered the complete schema, example, five Conditions, Additional Information, rationale, compatibility, and security recommendations.
The old Last Call deadline does not establish a later status, and version 5 is explicitly incompatible with earlier draft versions.
EthLib observations below pin `4f5f4111…`; implementation choices do not strengthen the EIP's requirements.

The file identifies a chain through `metadata.genesis_validators_root` and groups history by validator BLS `pubkey`.
A block record has a slot; an attestation record has source and target epochs.
Both may omit `signing_root`.
That root is the result of `compute_signing_root` with the proposer or attester domain, including the relevant fork and genesis validators root.
It is not a BLS signature, private key, or claim that the message is currently a valid duty.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) and [keystore note](Validator-key-derivation-and-keystores.md) cover separate cryptographic and key-storage boundaries.

The EIP uses strings for integers to avoid language-dependent numeric precision loss.
Its example writes decimal strings, as does this fixture.
The schema itself supplies no decimal patterns, integer bounds, or hex-width checks; it also has no explicit schema dialect and uses array-valued `items`.
Treating that schema alone as a complete value validator would overstate it.
EthLib's stricter parser choices are identified separately below.
An omitted root is represented by an absent field, not JSON `null`, which fails the declared string type if supplied.

## One complete constructed file

This file follows the source's documented shape.
The public-key identifier comes from its example; the chain identifier, epochs, slots, and root bytes here are constructed.
No underlying blocks, attestations, signatures, or root preimages are supplied or claimed.

```json
{
  "metadata": {
    "interchange_format_version": "5",
    "genesis_validators_root": "0x1111111111111111111111111111111111111111111111111111111111111111"
  },
  "data": [
    {
      "pubkey": "0xb845089a1457f811bfc000588fbb4e713669be8ce060ea6be3c6ece09afc3794106c91ca73acda5e5457122d58723bed",
      "signed_blocks": [
        {"slot": "10", "signing_root": "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"},
        {"slot": "12"}
      ],
      "signed_attestations": [
        {"source_epoch": "3", "target_epoch": "5", "signing_root": "0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"},
        {"source_epoch": "4", "target_epoch": "6"}
      ]
    }
  ]
}
```

The identifiers occupy 32 and 48 bytes respectively; supplied signing roots occupy 32 bytes.
All numeric fields are strings.
An empty history uses empty lists, not invented zero-slot or zero-epoch records.

## The five refusal conditions

The source requires these checks for the relevant public key after import:

| Condition | Required refusal |
| --- | --- |
| 1 | A block slashable with any imported block. An imported block without a root makes every new block at its slot count as slashable. |
| 2 | Block slot at or below the minimum imported block slot, except a repeat established through its signing root. |
| 3 | An attestation slashable with any imported attestation. |
| 4 | Source epoch strictly below the minimum imported source epoch. |
| 5 | Target epoch at or below the minimum imported target epoch, except a repeat established through its signing root. |

The minimums here are `10`, `3`, and `5`.
The source-epoch inequality is strict; source equality is allowed by condition 4.
The block-slot and target-epoch inequalities include equality and have repeat exceptions.
Those exceptions do not cancel another refusal condition.
Empty lists supply no minimum; they do not create a floor at zero.

Assume candidate roots truthfully identify their stated messages, and a “repeat” means the exact recorded message.
These conditional outcomes concern only the selected protection checks:

| Candidate against the uncollapsed fixture | Outcome |
| --- | --- |
| Block at slot 10, root `aa…aa`, exact repeat. | Can pass conditions 1–2. |
| Block at slot 9, different root. | Refused by condition 2 even without a recorded slot-9 conflict. |
| Any block at slot 12. | Refused by condition 1 because that record's root is missing. |
| Exact attestation repeat `(3,5,bb…bb)`. | Can pass conditions 3–5. |
| Attestation at target 5 with signing root different from `bb…bb`. | Refused as a double vote under condition 3. |

The EIP explicitly states the missing-root rule for blocks; it does not repeat that exact sentence for attestations.
A missing attestation root cannot establish the repeat exception, but that alone does not resolve every candidate above the minimum target.
EthLib conservatively treats an unknown-root record as different from every candidate at the same target.
That is an observed implementation rule, not an additional verbatim EIP requirement.

## Why union is insufficient

The [complete-database advice][complete] says implementers **MUST** import all messages and enforce all five conditions.
Enforcing the additional floor rules at all times, including ordinary local records, is its recommendation.
It also recommends a distinct internal dummy value for unknown roots; its suggested internal null is not a JSON `null` export rule.

Suppose this file is imported first, then a sparse file containing a block at slot 30.
Simply retaining both files' records and checking their combined minimum, 10, could reopen slot 20.
The later file's condition 2 forbids that slot: its own minimum is 30.
The source warns against such gaps and describes an explicit low watermark or pruning as remedies.
Union must preserve the import boundaries as well as the visible rows.

The [minimal-database advice][minimal] recommends maxima instead: greatest block slot, greatest source epoch, and greatest target epoch, merged with existing values.
For this fixture those limits are `12`, `4`, and `6`.
With roots discarded, a new block must have slot greater than 12; a new attestation needs source at least 4 and target greater than 6.
This deliberately refuses some exact repeats that a complete history could recognize.

Importing the same records again leaves these numerical maxima unchanged.
Merging later limits `(30,10,12)`, then importing the old file again, keeps `(30,10,12)`.
These are max-arithmetic consequences, not a claim that the EIP defines a particular duplicate-import transaction or database API.
Likewise, duplicate public-key entries cannot justify forgetting other records for that key.
The source recommends exporting only while the validator client or signer is stopped, and importing only while the validator client is stopped.
It does not elevate those recommendations to MUST.

## EthLib observations and limits

[`Interchange.ofJson`][parser] requires version `"5"`, 48-byte identifiers, 32-byte roots, and canonical unsigned decimal strings below `2^64`.
It preserves an absent root as `none`, and gathers records from every entry for the requested key.
[`importFile`][import] implements the minimal strategy by combining held and imported data, taking independent maxima, and discarding signing roots.
It retains the held genesis validators root without comparing the file's root.
The EIP identifies the chain but defines no explicit mismatch error; an integration still needs its chain-selection policy before treating merged history as appropriate for that chain.

The selected [import theorems][proofs] state that, if the resulting checker accepts a candidate after a sequence of imports, it meets the corresponding conditions of each input file, including the initial held file.
Their statements permit repeated files and duplicate keys.
This is conditional on checker acceptance and its record-level predicates, not a theorem authenticating arbitrary imported history or authorizing a signature.

A standalone Python check parsed the fixture, evaluated the source schema's used keyword subset, checked every entry's widths and decimal fields, and reproduced the minimum/maximum arithmetic.
No general JSON Schema interoperability test, BLS point/signature validation, validator call, database concurrency test, EthLib execution, or Lean proof replay ran.
The selected definitions and theorem statements were examined.
Independent JSON and integer checks reproduced the fixture, widths, refusal cases, conservative limits, and repeated-import arithmetic.

[eip]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3076.md
[complete]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3076.md#advice-for-complete-databases
[minimal]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3076.md#advice-for-minimal-databases
[parser]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/SlashingProtection.lean#L161-L228
[import]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/SlashingProtection.lean#L416-L471
[proofs]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Consensus/Proofs/SlashingProtection.lean#L814-L857
