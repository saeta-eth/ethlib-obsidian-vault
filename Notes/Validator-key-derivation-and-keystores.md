---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C16
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
ethlib_ercs_commit: "be307c948f28897adb8f25afcc253c2c909de35d"
tags:
  - ethlib
  - external-sources
  - validator-keys
---

# Validator key derivation and keystores

Deriving a key, assigning its role, and decrypting its storage file answer different questions.
ERC-2333 derives secret scalars from a seed, ERC-2334 assigns tree paths, and ERC-2335 stores encrypted secret bytes.
A successful decryption does not establish the scalar's range, its public key, its path, or authority to operate a validator.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) covers signature verification separately.

## Sources and revisions

The links pin the inventory's ERC revision `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.
EthLib's [derivation][old2333], [path][old2334], and [keystore][old2335] sources use revision `be307c948f28897adb8f25afcc253c2c909de35d`.
A comparison of all three complete official files at that earlier revision with the inventory copies found each pair identical.
This establishes document equality for these files, not implementation conformance.
All three documents have status Review at both pins.

| Source | Passages examined |
| --- | --- |
| [ERC-2333: BLS12-381 Key Generation][2333] | Version, tree structure, all derivation procedures, Lamport rationale, hardened-key discussion, and Test Case 0. The long intermediate Lamport arrays were not audited. |
| [ERC-2334: BLS12-381 Deterministic Account Hierarchy][2334] | Complete text, especially Path, Purpose, Validator keys, and Backwards Compatibility. |
| [ERC-2335: BLS12-381 Keystore][2335] | Specification, password preparation, modules, KDF table, checksum, cipher, metadata/schema, and both published test cases. |

## From seed bytes to a secret scalar

ERC-2333's `HKDF_mod_r` takes at least 32 bytes of input keying material, `IKM`.
Its default `key_info` is empty.
The scalar order is:

```text
r = 52435875175126190479447740508185965837690552500527637822603658699938581184513
```

Start with the 20 ASCII bytes `BLS-SIG-KEYGEN-SALT-`.
For each attempt, hash the previous salt with SHA-256, extract with HKDF-SHA-256 on `IKM || 00`, and expand to 48 bytes.
The expansion's information string is `key_info || 0030`, where `0030` encodes 48 in two big-endian bytes.
Interpret the output as a big-endian integer and reduce modulo `r`.
Retry if it is zero.
Although the Outputs paragraph permits zero, the procedure returns only `1 <= SK < r`.
Master derivation applies this procedure to the seed.
A length check does not establish that the seed contains sufficient unpredictable entropy.

Child derivation uses the parent's secret scalar, not its public key.
Serialize the parent into 32 big-endian bytes and the child index into four big-endian bytes, with `0 <= index < 2^32`.
Use the index bytes as HKDF-Extract's salt in two SHA-256 derivations: one from the parent bytes and one from their bitwise complement.
Each HKDF-Expand uses an empty information string and produces 255 chunks of 32 bytes.
Hash each chunk separately, then concatenate the first derivation's 255 digests followed by the second's 255 digests, preserving their order.
Hash the resulting `510 * 32 = 16320` bytes once more to obtain the compressed Lamport public key.
Pass those 32 bytes to `HKDF_mod_r` to obtain the child scalar.

The intermediate Lamport construction is the standard's proposed emergency backup, not a statement that BLS is post-quantum secure.
Its rationale says that using the backup reveals the associated BLS secret key and requires retiring that BLS key.
This note does not assess the backup's security argument.
Deterministic derivation also supplies no replacement for a lost seed or another sufficient ancestor secret.

## Published master and child

ERC-2333 Test Case 0 gives these public test values:

```text
seed = c55257c360c07c72029aebc1b53c05ed0362ada38ead3e3e9efa3708e53495531
       f09a6987599d18264c1e1c92f2cf141630c7a3c4ab7c81b2f001698e7463b04
master_SK = 6083874454709270928345386274498605044986640685124978867557563392430687146096
child_index = 0
child_SK = 20397789859736650942317412262472558107875392172444076792671091975210932703118
```

Concatenate the two seed lines before decoding their hex.
The decimal outputs are scalars, not ASCII encodings of keys.
The child is `m/0`, not an Ethereum validator signing key.
Reaching such a key requires the additional ERC-2334 indices.
These known answers come from the ERC, not EthLib output.

## Paths assign different roles

ERC-2334 requires at least four indices after `m`, with purpose `12381` and ERC-2333 as the derivation function.
For its Ethereum validator hierarchy, coin type is `3600`:

```text
withdrawal key i: m/12381/3600/i/0
signing key i:    m/12381/3600/i/0/0
```

The signing key is child zero of its withdrawal key.
The source allows affected keys to be derived otherwise when this arrangement is infeasible, including secret-shared or custodial arrangements.
These are the pinned document's key roles, not an audit of every network's withdrawal mechanisms.

There is a terminology tension: ERC-2333 describes only hardened derivation, while ERC-2334 says its paths do not support hardened keys and forbids the apostrophe.
The procedures still require the parent secret, and the notation must omit the apostrophe.
Do not substitute BIP-32 derivation or infer public-parent derivation from that wording.
A stored path string is a claim about origin, not evidence that the secret was derived there.

## Password characters become bytes first

ERC-2335 requires this order: Unicode NFKD normalization, removal of control code points, then UTF-8 encoding.
Remove U+0000 through U+001F and U+007F through U+009F, inclusive.
Keep ordinary space U+0020.
Do not filter the UTF-8 bytes themselves.

This hand-derived case makes the order and representations explicit:

```text
input code points: U+00E9 U+0000 U+007F U+0085 U+0020 U+2460
after NFKD:        U+0065 U+0301 U+0000 U+007F U+0085 U+0020 U+0031
after removal:     U+0065 U+0301 U+0020 U+0031
UTF-8 bytes:       65 cc 81 20 31
```

Both published keystore fixtures use password `𝔱𝔢𝔰𝔱𝔭𝔞𝔰𝔰𝔴𝔬𝔯𝔡🔑` and prepared bytes `7465737470617373776f7264f09f9491`.
Normalization removes the letters' compatibility styling and preserves the key emoji.
Neither password example is a recommendation for an actual secret.

## What the keystore checks

The version-4 modules specify PBKDF2-HMAC-SHA-256 or scrypt to derive an intermediate key, `DK`.
Password verification compares:

```text
SHA256(DK[16:32] || ciphertext) == checksum.message
```

Only after a match does AES-128-CTR decrypt with `DK[0:16]` and the stored IV.
The two slices have different roles.
The checksum is not a BLS signature, proof of possession, or authentication of the entire JSON document.
For fixed KDF parameters and ciphertext, changing only the IV leaves that checksum input unchanged.
Neither `path`, `pubkey`, nor `uuid` occurs in it.
An application needs separate checks when it relies on the decrypted bytes' scalar range or association with a public key.

The published PBKDF2 fixture uses 262144 iterations, 32 output bytes, and this salt:

```text
salt = d4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3
IV = 264daa3f303d7259501c93d997d84fe6
ciphertext = cee03fde2af33149775b7223e7845e4fb2c8ae1792e5f99fe9ecf474cc8c16ad
checksum = 8a9f5d9912ed7e75ea794bc5a89bca5f193721d30868ade6f73043c6ea6febf1
secret = 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
```

Its path is `m/12381/60/0/0`, not the validator hierarchy's `3600` path.
This is a generic storage fixture, not evidence of Ethereum path provenance or a checked public-key match.

## EthLib connections and review limits

At the recorded EthLib commit, [`keyGen`][keygen] rejects short inputs and tries at most 16 salts.
After 16 zero reductions it returns `keyGenNoKey`, whereas ERC-2333's loop has no finite retry limit.
[`deriveMasterSk`, `deriveChildSk`, and `derivePath`][derive] reuse this bounded operation.
[`SecretKey.fromBytes`][scalar] separately rejects a wrong byte length, zero, and an integer at least `r`.
[`withdrawalPath` and `signingPath`][paths] construct the paths above.

[`passwordBytes`][password] prepares characters using Unicode 16.0.0 tables.
[`decrypt`][decrypt] returns secret bytes without authenticating metadata or requiring a BLS scalar.
Its caller supplies KDF work and memory limits, checked before derivation.
Those are models, not proved resource bounds for the running code.
[`exists_decryptWithKey_eq_ok_iff`][acceptance] characterizes checksum-stage acceptance as a key of at least 32 bytes with a matching checksum.
It is not a cryptographic security theorem.

The cited definitions and theorem statement were read, without a Lean build or proof replay.
An independent Python calculation reproduced ERC-2333 Test Case 0's master and child, both password-byte examples, and the published PBKDF2 fixture's checksum and decrypted secret.
It used standard hash/HMAC/PBKDF2 functions and PyCryptodome 3.23.0 for AES-128-CTR.
Its Unicode 15.0.0 checks cover the displayed examples, not agreement with all of EthLib's Unicode 16.0.0 tables.
These are checks with separate tools, not executed EthLib vectors or a full standards-conformance result.
No seed-generation, constant-time, backup-security, public-key ownership, or operational key-management claim follows.
[SOUNDNESS.md A3][sidechannels] explicitly makes no constant-time claim.

[2333]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2333.md
[2334]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2334.md
[2335]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2335.md
[old2333]: https://github.com/ethereum/ERCs/blob/be307c948f28897adb8f25afcc253c2c909de35d/ERCS/erc-2333.md
[old2334]: https://github.com/ethereum/ERCs/blob/be307c948f28897adb8f25afcc253c2c909de35d/ERCS/erc-2334.md
[old2335]: https://github.com/ethereum/ERCs/blob/be307c948f28897adb8f25afcc253c2c909de35d/ERCS/erc-2335.md
[keygen]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/KeyGen.lean#L35
[derive]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/KeyDerivation.lean#L65
[scalar]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/Signatures.lean#L65
[paths]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Bls/KeyPath.lean#L113
[password]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Keystore.lean#L150
[decrypt]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Keystore.lean#L267
[acceptance]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Cryptography/Proofs/Keystore.lean#L96
[sidechannels]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/SOUNDNESS.md#a3-side-channels
