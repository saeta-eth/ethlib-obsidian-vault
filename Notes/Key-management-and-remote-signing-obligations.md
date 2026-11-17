---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: I04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
keymanager_api_commit: "e5aab3e3ac4b116a2b193000996e4645c8e1bd5f"
remote_signing_api_commit: "44e9e0dcc115c91f8b739684f4789a8310476bd9"
tags:
  - ethlib
  - external-sources
  - validator-keys
  - remote-signing
---

# Key management and remote-signing obligations

Moving an encrypted key, transferring signing history, and requesting a signature are different operations.
An API response establishes only its specified outcome for the addressed actor.
Keep the key-holding binary, validator client's remote-key references, and signer enforcing history separate before describing a migration as safe.

## Sources and reading scope

[ERC-2335][2335] is Review at the recorded ERCs commit; [EIP-3076][3076] is Last Call at the recorded EIPs commit.
The official [Keymanager API snapshot][km] is commit `e5aab3e3ac4b116a2b193000996e4645c8e1bd5f`, whose OpenAPI version label is `Dev`.
The official [Remote Signing API snapshot][rs] is commit `44e9e0dcc115c91f8b739684f4789a8310476bd9`, labelled `v1.4.0`.
These repository versions and document statuses do not establish deployment support.

Reading covered local/remote keystore operations, their selected types, keymanager Import/Delete flows, and remote public-key/sign operations with selected attestation schemas.
ERC-2335 supplied selected keystore-format sections; EIP-3076 supplied its schema, conditions, domain descriptions, and security recommendations.
Other duties, implementation-specific behaviour, and linked cryptographic algorithms were not evaluated.
The EthLib commit supplies contribution context only; this note makes no implementation or proof-coverage claim.

## Identify which actor holds the key

The keymanager API is served by the binary holding validator keys, which may be a validator client or remote signer.
Its specification requires token authentication for sensitive routes and recommends a secure channel.
Those access rules authorize management requests; they do not establish a duty's signing domain. [API description][km-root]

| Operation | Actor and bounded meaning |
| --- | --- |
| `GET /eth/v1/keystores` | Lists validating public keys known to and decrypted by the addressed keymanager. |
| `POST /eth/v1/keystores` | Imports encrypted keystore strings with corresponding passwords into that keymanager. |
| `DELETE /eth/v1/keystores` | Stops local signing before deletion/history serialization, under the requirements below. |
| `/eth/v1/remotekeys` operations | Manage a validator client's remote-key references; they do not import or erase the remote signer's private key. |
| `GET /api/v1/eth2/publicKeys` | Lists public keys for private keys loaded into the remote signer. |

ERC-2335 version 4 specifies encrypted storage and password processing, not past validator activity.
The [keystore note](Validator-key-derivation-and-keystores.md) explains those bytes and checks separately.
An API-shaped public key is also not proof of a valid curve point, custody, or exclusive possession.

## Import status and local deletion order

For local import, `passwords[i]` must unlock `keystores[i]`; both fields contain arrays, and each keystore is itself a JSON-encoded string.
The flow rejects unequal array lengths with HTTP 400.
For nonempty import processing, unreadable supplied slashing data also yields 400.
An empty batch returns 200, with an option to read and reject invalid supplied history.
Users SHOULD provide associated slashing protection, whose format MUST follow EIP-3076.
A processed batch can return HTTP 200 with per-key `imported`, `duplicate`, or `error` results in request order.
Neither HTTP 200 nor `duplicate` proves every key imported or every supplied history entry merged. [Local operations][local] [Import flow][flows]

Local deletion explicitly requires a single atomic sequential operation: first guarantee the keys cannot produce further signatures, then delete them and serialize associated history.
Persistent slashing history must remain after the response, permitting repeated deletion requests to retrieve it.
Only requested keys with final `deleted` or `not_active` status may have history returned.
The endpoint should not return HTTP 404 merely because keys are absent. [Delete operation][local]

This is not an all-or-nothing batch rollback guarantee.
The explanatory flow permits per-key failures and presents an atomic database-wide deletion as one possible implementation.
It can also replace provisional deletions with `error` when history export fails; an error is therefore not a complete diagnosis of whether key material remains.
Remote-reference deletion has different response fields and supplies neither this local history-export result nor a guarantee that the remote signer stopped. [Delete flow][flows] [Remote operations][remote]

## Complete conditional deletion example

Assume four public identifiers with the following outcomes: active key removed, inactive key with retained history, unknown key, and a removal failure.
The repeated bytes below satisfy the API's 48-byte hex shape only; no valid BLS keys or real validators are asserted.
This is a constructed request body for local DELETE, never sent to a service:

```json
{
  "pubkeys": [
    "0x111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111",
    "0x222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222",
    "0x333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333",
    "0x444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444"
  ]
}
```

A conditional HTTP-200 response is:

```json
{
  "data": [
    {
      "status": "deleted"
    },
    {
      "status": "not_active"
    },
    {
      "status": "not_found"
    },
    {
      "status": "error",
      "message": "Constructed key-removal failure"
    }
  ],
  "slashing_protection": "{\"metadata\":{\"interchange_format_version\":\"5\",\"genesis_validators_root\":\"0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\"},\"data\":[{\"pubkey\":\"0x111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111\",\"signed_blocks\":[{\"slot\":\"10\"}],\"signed_attestations\":[]},{\"pubkey\":\"0x222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222\",\"signed_blocks\":[{\"slot\":\"20\"}],\"signed_attestations\":[]}]}"
}
```

The outer `slashing_protection` value is a string containing a second JSON document, not a directly nested object.
Its synthetic genesis root has 32 bytes; its two records retain block slots 10 and 20 with unknown signing roots.
The four result positions align with the four requested identifiers, while only the first two statuses qualify for returned history.

| Position | First result | Identical later delete, assuming no reimport or other changes |
| --- | --- | --- |
| 1 | `deleted` | `not_active`, with retained history. |
| 2 | `not_active` | `not_active`, with retained history. |
| 3 | `not_found` | `not_found`; no history was known. |
| 4 | `error` | No result inferred. |

These are conditional source-rule cases, not an observed migration or an implementation test.
They establish nothing about copies of the same private key in other binaries.

## History and signing-domain obligations remain

EIP-3076's format version is 5, and its conditions bind the signer after import.
After importing the first synthetic record, the signer must treat another block at slot 10 as slashable because `signing_root` is missing.
A numeric slot alone cannot establish an identical-message exception.
The [slashing-import note](Slashing-protection-imports-and-missing-roots.md) covers refusal floors, unknown roots, and complete/minimal database distinctions.
EIP-3076 recommends stopping the relevant signer/client for export and the validator client for import; these recommendations do not create distributed exclusion among key copies.

Remote `POST /api/v1/eth2/sign/{identifier}` selects a public key and request type.
For `ATTESTATION`, the schema requires the attestation and inherited `fork_info`; that object requires a fork object and `genesis_validators_root`.
The fork properties describe previous/current four-byte versions and an epoch.
Do not infer universal field requirements from this one duty: deposit requests use a different shape. [Signing path][sign] [Selected schemas][schemas]

The API spells its optional verification value `signingRoot`; the interchange spells its historical field `signing_root`.
The YAML describes optional verification, without an unconditional MUST-compare rule for every signer.
EIP-3076 separately identifies block/attestation roots using the appropriate signing domain, fork, and genesis-validators root.
Supplying these fields does not establish that a server recomputed the domain correctly or used complete signing history.
The [BLS note](BLS-ciphersuites-and-verification-preconditions.md) treats cryptographic verification conditions separately.

The signing operation documents HTTP 412 for a slashing-protection refusal, 404 for an unknown key, and 200 for a returned signature.
Those outcomes do not certify protection against every signing process sharing that key.
Python checked the synthetic identifiers' widths, JSON/string round trip, result/history alignment, and conditional slot-floor arithmetic.
No keystore decryption, key import/deletion, endpoint request, signing-root computation, BLS operation, EthLib binding, Lean build, or proof replay ran.

[2335]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2335.md
[3076]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3076.md
[km]: https://github.com/ethereum/keymanager-APIs/tree/e5aab3e3ac4b116a2b193000996e4645c8e1bd5f
[rs]: https://github.com/ethereum/remote-signing-api/tree/44e9e0dcc115c91f8b739684f4789a8310476bd9
[km-root]: https://github.com/ethereum/keymanager-APIs/blob/e5aab3e3ac4b116a2b193000996e4645c8e1bd5f/keymanager-oapi.yaml
[local]: https://github.com/ethereum/keymanager-APIs/blob/e5aab3e3ac4b116a2b193000996e4645c8e1bd5f/apis/local_keystores.yaml
[flows]: https://github.com/ethereum/keymanager-APIs/blob/e5aab3e3ac4b116a2b193000996e4645c8e1bd5f/flows/README.md
[remote]: https://github.com/ethereum/keymanager-APIs/blob/e5aab3e3ac4b116a2b193000996e4645c8e1bd5f/apis/remote_keystores.yaml
[sign]: https://github.com/ethereum/remote-signing-api/blob/44e9e0dcc115c91f8b739684f4789a8310476bd9/signing/paths/sign.yaml
[schemas]: https://github.com/ethereum/remote-signing-api/blob/44e9e0dcc115c91f8b739684f4789a8310476bd9/signing/schemas.yaml
