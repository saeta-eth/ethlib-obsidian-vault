---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E01
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - transactions
  - signatures
---

# Transaction envelopes and sender signing preimages

A transaction's transmitted bytes, sender signing preimage, and inclusion checks serve different purposes.
Identify each before changing an encoder or signature rule.
The [RLP note](RLP-canonical-encoding.md) supplies the encoding rules; the [signed-data note](Signed-data-preimages-and-domains.md) explains separate off-chain signing domains.

## Sources and scope

The following EIPs were read at commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
All six are Final there; that status is not a network activation audit.

| Source | Sections examined |
| --- | --- |
| [EIP-155][e155] | Specification and Example. |
| [EIP-2718][e2718] | Transactions, Receipts, and the signing-type and opaque-payload rationales. |
| [EIP-2930][e2930] | Specification, including access-list validation and duplicate handling. |
| [EIP-1559][e1559] | Specification, including its fee-validation and pricing code. |
| [EIP-4844][e4844] | Blob Transaction, Signature, Execution Layer Validation, and Networking. |
| [EIP-7702][e7702] | Set Code Transaction and Behavior, including the separate authorization signature. |

The [Shanghai Yellow Paper][yp], commit `efc5f9a1f356cba376c978eedb63cb0363c2aa85`, section 4.2 and Appendix F supply the legacy/type-1/type-2 destination and signature context.
This note describes those base documents, not every later amendment.

## Separate the envelope from the signing input

EIP-2718 gives typed transactions the form `typeByte || payload`, where `||` concatenates bytes.
The payload is opaque to the envelope standard: RLP is a choice made by the individual types below.
Do not RLP-encode the type byte together with the payload as an extra list.

EIP-2718 says future types **SHOULD** sign the type byte; it deliberately does not say **MUST**.
The exact signature formulas in EIP-2930, EIP-1559, and EIP-4844 include their type byte.
Their signed data therefore cannot be changed to another type merely by replacing the transmitted prefix.

EIP-2718 also requires a typed receipt's type to match its transaction's type at the same block index.
The four base type specifications below define receipt payload `RLP([status, cumulativeGasUsed, logsBloom, logs])`.
Prefixing that payload with the matching type produces receipt bytes, not the transaction's signing input.

For legacy transactions, define:

```text
L = [nonce, gasPrice, gasLimit, to, value, data]
```

The signed wire form is `RLP(L ++ [v, r, s])`, with `++` denoting list concatenation.
An unprotected legacy signature hashes `RLP(L)` and uses `v = 27 + yParity`.
Under EIP-155's stated fork/chain conditions, signing the protected form hashes `RLP(L ++ [chainId, 0, 0])` and requires `v = 35 + 2 * chainId + yParity`.
The two zeros are signing placeholders, not the signature fields of the transmitted transaction.
EIP-155 retains the unprotected form; its existence is not proof of cross-chain replay protection.

## Compare the typed schemas

Use these lists to display the common fields:

```text
A = [chainId, nonce, gasPrice, gasLimit, to, value, data, accessList]
F = [chainId, nonce, maxPriorityFeePerGas, maxFeePerGas,
     gasLimit, to, value, data, accessList]
S = [yParity, r, s]
```

| Type | Signed payload after the type byte | Total fields | Destination |
| --- | --- | ---: | --- |
| `01`, EIP-2930 | `RLP(A ++ S)` | 11 | Address or creation. |
| `02`, EIP-1559 | `RLP(F ++ S)` | 12 | Address or creation. |
| `03`, EIP-4844 | `RLP(F ++ [maxFeePerBlobGas, blobVersionedHashes] ++ S)` | 14 | A 20-byte address. |
| `04`, EIP-7702 | `RLP(F ++ [authorizationList] ++ S)` | 13 | A 20-byte address. |

Creation uses an empty byte string for `to`, not the twenty-byte zero address.
EIP-4844 forbids a null destination; EIP-7702 inherits that restriction.
Type 3 requires nonempty versioned hashes; type 4 requires a nonempty authorization list.
Access lists are not access whitelists, and duplicate entries are permitted but charged again.

For types 1–3, hash the type byte followed by RLP of the displayed fields before `S`.
Typed `yParity` is 0 or 1; do not replace it with legacy EIP-155 `v` arithmetic.
An authorization tuple inside type 4 still carries its own six fields, including its signature.
Its separate preimage is `05 || RLP([chainId, address, nonce])`.

The pinned EIP-7702 text has an unresolved naming problem: it defines `TransactionPayload` with the outer signature fields, then describes signing `SET_CODE_TX_TYPE || TransactionPayload`.
Read literally, that includes the signature being calculated.
EthLib's inspected `signingHash7702` instead omits the final outer three fields.
This records an implementation reading, not a resolution of the source wording.

## Blob propagation is another representation

EIP-4844's pooled-transaction representation is:

```text
03 || RLP([tx_payload_body, blobs, commitments, proofs])
```

Here `tx_payload_body` is the signed fourteen-field list, not another type-prefixed byte string.
Block-body retrieval uses the execution envelope `03 || RLP(tx_payload_body)`.
The accompanying blobs, commitments, and proofs are not additional signed fields.
Their counts, commitment hashes, and KZG checks are separate validation obligations.
This representation does not cover later network-wrapper amendments.

## Hand-derived unsigned type-2 preimage

Choose chain ID 1, nonce 0, priority cap 2 wei/gas, fee cap 10 wei/gas, gas limit 21000, zero value, and empty data/access list.
Let `to` be twenty bytes of `11`.

| Field | RLP bytes |
| --- | --- |
| `chainId = 1` | `01` |
| `nonce = 0` | `80` |
| Priority cap 2 | `02` |
| Fee cap 10 | `0a` |
| `gasLimit = 21000 = 0x5208` | `82 52 08` |
| Twenty-byte destination | `94`, followed by twenty `11` bytes. |
| `value = 0` | `80` |
| Empty data | `80` |
| Empty access list | `c0` |

The list payload has `1 + 1 + 1 + 1 + 3 + 21 + 1 + 1 + 1 = 31` bytes.
Its prefix is `0xc0 + 31 = 0xdf`.
Prepending type `02` gives this complete 33-byte signing preimage:

```text
02df0180020a8252089411111111111111111111111111111111111111118080c0
```

Keccak-256 of these bytes is the sender's signing digest under EIP-1559.
No personal-sign or EIP-712 prefix belongs here.
This hand-derived unsigned example supplies neither a signature nor evidence of transaction admissibility.

## Fee calculation needs validity conditions

EIP-1559 requires `maxFeePerGas >= baseFee` and `maxFeePerGas >= maxPriorityFeePerGas` before pricing a valid transaction.
With the example's caps and a base fee of 7 wei/gas:

```text
priority = min(2, 10 - 7) = 2 wei/gas
effective price = 7 + 2 = 9 wei/gas
```

If the settled charged gas is 21000, the execution fee is 189000 wei: 147000 burned and 42000 paid as the priority fee.
The affordability check uses `gasLimit * maxFeePerGas + value`, here 210000 wei, rather than that eventual charge.
At base fee 11, the fee cap fails validity; subtracting or clamping cannot make the transaction includable.
Nonce, signature, chain, intrinsic gas, state, and block-capacity conditions remain necessary.
Blob transactions also have a separate blob fee cap and blob-gas affordability requirement.

## EthLib connections and review limits

At the recorded commit, [`encodeTransaction`][envelope-code] constructs wire bytes, while [`signingHash1559` and `signingHash7702`][signature-code] select signing fields.
`transactionHash` hashes the signed wire bytes, rather than the unsigned signing input.
[`validateTransaction`][validity-code] checks fields without account state; [`checkInclusion`][inclusion-code] adds base-fee and account checks.
These inspected entry points do not establish complete implementation agreement or identify a missing proof.

The cited passages and declarations were read; the bytes and fee arithmetic were derived by hand.
Independent calculations with `rlp` 4.1.0 and PyCryptodome 3.23.0 reproduced the 33-byte preimage and EIP-155's published signing preimage/hash as a control.
The displayed conditional fee arithmetic was also checked separately.
No signature, transaction execution, Lean build, proof replay, or complete amendment audit was performed.
This note does not assess the versions or forks of later rules, including EIP-8037's gas changes.

[e155]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-155.md
[e2718]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2718.md
[e2930]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2930.md
[e1559]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1559.md
[e4844]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4844.md
[e7702]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[envelope-code]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Envelope.lean#L193
[signature-code]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Signature.lean#L68
[validity-code]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Validity.lean#L337
[inclusion-code]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Transactions/Inclusion.lean#L223
