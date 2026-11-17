---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 6
delivery_batch: "follow-up-8"
catalogue_family: A13
assessment_priority: Unresolved
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
rfc3986_version: "January 2005"
tags:
  - ethlib
  - external-sources
  - payment-uris
  - assessment
---

# Payment URI revisions and request boundaries

An Ethereum payment URI describes a request for a consumer to interpret and present.
It is not a signed transaction, approval, or payment receipt.
This assessment keeps A13's priority **Unresolved** because the target revision and consumer workflow are still undecided.
It supplies a concrete comparison before selecting a parser, transaction builder, or wallet-facing contribution.

## Source readiness and revision roles

The complete short texts were read at ERCs commit `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`, including headers, grammar, semantics, examples, compatibility, and security text.
The EthLib revision records this assessment's context, without presuming a URI module or implementation need.

| Source | Status and role at the pin |
| --- | --- |
| [ERC-67][67] | Withdrawn, with explicit withdrawal reason “Superseded by EIP-681.” Historical raw-data and human-readable function requests. |
| [ERC-681][681] | Final. Payment and ABI-specified transaction requests, with optional chain ID, function name, and typed parameters. |
| [ERC-831][831] | Stagnant. A broader scheme/prefix envelope whose payload semantics belong to another ERC. |
| [RFC 3986][3986] | January 2005, selected §§2.1, 2.2, and 2.4: generic percent encoding and separation of components before decoding. |

Final status does not establish support in a particular wallet.
ERC-831's description of an envelope also does not silently replace every ERC-681 production.
RFC 3986 supplies generic URI syntax, not wallet authorization or payment semantics.

## Compare the grammars before normalizing

ERC-67 uses `ethereum:<address>` with optional `value`, `gas`, and raw `data`, or a human-readable `function=...` query.
Its printed sketch repeats `?` for optional query fields, while the published multi-field examples use `&`.
Its Abstract mentions ethers, but the short `value` production supplies no explicit atomic-unit conversion rule.
Do not assign ERC-681's more precise amount semantics to this withdrawn format without an explicit compatibility decision.

ERC-681 instead specifies:

```text
ethereum:[pay-]target_address[@chain_id][/function_name][?parameters]
parameters = key=value[&key=value...]
```

This display abbreviates the source grammar, rather than replacing it.
The target is a hexadecimal address or ENS name, the optional chain ID uses decimal digits, and a function name appears after `/`.
Reserved query names include `value`, `gas`, `gasLimit`, and `gasPrice`; other keys are ABI type names.
`gas` and `gasLimit` are synonymous suggestions.

ERC-831 permits both `eth:` and `ethereum:` and an optional `prefix-` before its payload.
With no prefix, it assumes `pay-`, but requires the payload to begin with `0x`.
That differs from ERC-681's allowance for an ENS target without requiring a prefix.
ERC-831 delegates payload structure to the corresponding use-case ERC, with ERC-681 as its payment example.
A consumer must select a compatibility profile before claiming to accept both grammars completely.

## Parse the complete published native-payment example

ERC-681 publishes this exact request:

```text
ethereum:0xfb6916095ca1df60bb79Ce92ce3ea74c37c5d359?value=2.014e18
```

| Field | Interpretation under ERC-681 |
| --- | --- |
| Scheme | `ethereum`. |
| Target | `0xfb6916095ca1df60bb79Ce92ce3ea74c37c5d359`. |
| Chain ID | Absent, so the client's current network setting remains effective. |
| Function | Absent, so this requests native-token payment. |
| Amount | `2.014e18` atomic units, here wei. |

The amount has an exact integer interpretation:

```text
2.014 * 10^18 = (2014/1000) * 10^18
              = 2014 * 10^15
              = 2014000000000000000 wei
```

The exponent 18 exceeds the three digits after the decimal point, satisfying the source's stated integer constraint.
The expression does not request 2.014 wei or provide an approximate floating-point amount.
Parsing these fields also does not establish address checksum validity, recipient identity, available balance, or a successful transaction.
No explicit chain ID means “use the current setting,” not “assume chain 1.” [ERC-681, Semantics][681]

The number grammar allows scientific notation and a leading sign, but semantic/type checks remain necessary.
Its optional digit groups are not a complete numeric-validation algorithm.
In particular, the source requires integer results and says the exponent MUST be at least the number of fractional digits.
A generic floating-point parser does not establish those conditions or an ABI integer's range.

## The token contract is not the beneficiary parameter

The complete published ERC-20 request is:

```text
ethereum:0x89205a3a3b2a69de6dbf7f01ed13b2108b2c43e7/transfer?address=0x8e23ee67d1332ad560396262c48ffbb01f93d052&uint256=1
```

Here the target is the token contract, `/transfer` names the requested function, `address` supplies the beneficiary, and `uint256=1` supplies one atomic token unit.
The URI does not supply the token's decimals or establish that one atomic unit is one nominal token.
The typed amount is also distinct from the reserved `value` parameter used for native value.
This example requests `transfer`, not `approve`, `transferFrom`, or a permit.
The [allowance note](Permit-nonces-and-allowance-authority.md) explains why those actions carry different authority.

ERC-681 permits broader ABI-specified requests.
A type name is therefore a parameter type, not itself proof of safe application behaviour.
Repeated keys, duplicate reserved parameters, overload resolution, and error handling need a defined consumer profile and conformance cases.
Do not silently flatten a parameter sequence into a map that discards repeated entries.

For ENS targets, ERC-681 leaves resolution to the payer between receipt and sending.
It explicitly gives a `0x` address followed by 40 hexadecimal digits precedence over ENS interpretation.
The [ENS note](ENS-namehash-and-resolver-boundaries.md) supplies the separate resolution boundary.
No name is resolved here.

## Keep delimiter parsing separate from decoding

ERC-681 describes STRING as URL-encoded Unicode with delimiters and `%` hex-encoded.
RFC 3986 §2.1 defines a percent-encoded octet as `%` followed by two hexadecimal digits.
Sections 2.2 and 2.4 require separating significant components before decoding reserved delimiters and prohibit decoding the same string repeatedly.

For the illustrative query fragment `string=A%26B`, `%26` represents the data character `&`, not another parameter boundary.
For `string=A%2526B`, one decoding yields `A%26B`; another decoding would change its meaning again.
These ASCII fragments illustrate generic delimiter handling, not a complete Unicode, normalization, or wallet-form-decoding policy.
The selected ERCs do not settle every character-conversion choice for a production consumer.

## Assessment and remaining choice

| Judgment | Bounded result |
| --- | --- |
| Connection | A URI may supply inputs to a transaction-request consumer; no present EthLib implementation is assumed. |
| Contribution value | A revision-specific parsing specification could separate syntax, typed values, chain selection, and consumer approval. |
| Additional knowledge | The examples distinguish native value from token arguments and compare legacy, payment-specific, and envelope formats. |
| Coverage and effort | Three complete short ERCs and selected RFC sections support this first assessment; full consumer conformance remains a separate task. |

ERC-681 makes all supplied arguments user-editable suggestions and asks for an amount when none is supplied.
Its security text calls for authentic sources and integrity protection because modified requests can redirect irreversible transactions.
The formats define no intrinsic signature authenticating the request, and parsing does not authorize a consumer to send funds.

Choose a revision/profile and consumer workflow before promoting this family from Unresolved.
A wallet import review, QR producer, and generic request validator need different outputs and rejection policies.
A first follow-up can record acceptance/rejection cases against that chosen profile while preserving the grammar discrepancies above.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Python exact rational arithmetic reproduced the native amount, and the two published ASCII examples were checked for the stated field boundaries.
The field checks exercised only the displayed ASCII cases.
No wallet, browser, ENS resolver, signed transaction, approval, payment, EthLib binding, or Lean proof was executed.
The examples establish neither wallet interoperability nor end-to-end payment correctness.

[67]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-67.md
[681]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-681.md
[831]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-831.md
[3986]: https://www.rfc-editor.org/rfc/rfc3986.html
