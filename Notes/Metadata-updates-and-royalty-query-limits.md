---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A08
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
solidity_version: "v0.8.30"
solidity_commit: "73712a01b2de56d9ad91e3b6936f85c90cb7de36"
tags:
  - ethlib
  - external-sources
  - metadata
  - royalties
---

# Metadata updates and royalty query limits

A metadata URI identifies a resource, an update event tells observers to refresh information, and a royalty query returns a receiver and amount.
These are separate interfaces with separate obligations.
None of the three operations, by itself, authenticates the resource's claims or transfers a royalty payment.

## Sources and scope

The ERCs below have status Final at the recorded ERCs commit.
Their status does not establish adoption by a wallet, marketplace, or deployed token.

| Source | Selected reading |
| --- | --- |
| [ERC-1046][1046] | Complete Specification, Rationale, Backwards Compatibility, and Security Considerations. |
| [ERC-4906][4906] | Complete document, including update triggers, explicit interface ID, and permission-control advice. |
| [ERC-2981][2981] | Complete Specification and Rationale, plus Abstract, compatibility, and security text. |
| [ERC-721][721], [ERC-1155][1155] | Metadata interfaces/schemas and ERC-1155's Metadata Extensions rules. |
| [Solidity ABI specification][abi] | v0.8.30 at the recorded commit: Function Selector, integer/tuple encoding, and Events. |

It makes no claim about EthLib token-module implementation or proof coverage.
The [receiver-interface note](Token-receiver-interfaces-and-callbacks.md) explains ERC-165 discovery, and the [ABI note](ABI-encoding-and-event-topics.md) explains selectors and logs.

## Separate URI signatures and metadata claims

ERC-1046's ERC-20 extension requires `tokenURI()` with no argument, returning a string whose resolved JSON MUST conform to its ERC-20 metadata interface.
That signature differs from ERC-721's `tokenURI(uint256)` and ERC-1155's `uri(uint256)`.
ERC-721's metadata extension is optional.
ERC-1155's optional metadata extension requires its URI to point to conforming JSON and explicitly forbids using `uri` to determine whether a token exists. [ERC-1046, ERC-20 extension][1046] [ERC-721, metadata extension][721] [ERC-1155, Metadata Extensions][1155]

In ERC-1046's ERC-20 JSON, `interop` is required while fields such as `name`, `symbol`, and `decimals` are optional.
Within `interop`, the listed standard flags are optional properties, but each applicable flag MUST be true and an inapplicable flag MUST be omitted.
Thus this is a constructed shape example for ERC-1046 metadata:

```json
{
  "interop": {"erc1046": true},
  "name": "Example token",
  "symbol": "EXAMPLE",
  "decimals": 6
}
```

If corresponding ERC-20 functions exist, nonempty `name()` and `symbol()` results MUST match the metadata values, as must an existing `decimals()` result.
The decimals default is 18 only when neither the metadata field nor that function is present.
The example does not establish those conditions for any real contract.
ERC-1046 recommends its interoperability metadata for ERC-721 and ERC-1155, with additional MUST requirements when its ERC-1155 interoperability interface is implemented.
That interface's `tokenURI(tokenId)` MUST equal `uri(tokenId)` if present. [ERC-1046, Specification][1046]

These schemas describe how to interpret supplied information, rather than verifying that a token represents a genuine asset or that an image's claims are true.
ERC-1046 separately warns about server-side request forgery and recommends restricting URI schemes and ports when wallets make requests.
No URI or remote metadata is fetched for this note.

## Distinguish a JSON change from a URI change

ERC-4906 is an optional ERC-721 extension.
It requires `MetadataUpdate(tokenId)` or `BatchMetadataUpdate(fromTokenId,toTokenId)` when JSON metadata changes for a token or consecutive range.
It RECOMMENDS not emitting `MetadataUpdate` on mint, on burn, or when the URI changes but the JSON metadata does not.
Those recommendations are not prohibitions. [ERC-4906, Specification][4906]

The event carries identifiers, not the replacement metadata or its hash.
An observer can call ERC-721's `tokenURI` and refresh the resource after observing the notification.
The same URI can serve changed JSON, so comparing URI strings alone does not implement the update trigger.
Off-chain resource changes do not automatically execute a contract event.
ERC-4906's security text suggests a trigger method for such changes and calls for correct permission controls. [ERC-4906, Rationale and Security Considerations][4906]

ERC-1155 has its own `URI` event rules.
Its optional metadata extension MUST emit on URI changes expressible as events, excluding dynamic/programmatic changes.
When an event exists for an ID, `uri(id)` MUST match its latest emitted value.
Do not import ERC-4906's JSON-change event rule into every ERC-1155 implementation. [ERC-1155, Metadata Extensions][1155]

## Calculate identifiers and one complete log layout

ERC-4906 explicitly requires `supportsInterface(0x49064906)` to return true.
Its new declarations are events, so the specified identifier cannot be derived by naively XORing new function selectors or event topics.
ERC-2981 instead specifies the one-function royalty interface ID from this exact canonical signature:

```text
royaltyInfo(uint256,uint256)
Keccak-256 = 2a55205a9d4720a297dfcf892a8ff23e6f95ca084ce993231bd653f83741ac42
selector   = 2a55205a
```

The return types do not enter the selector.
Neither interface identifier is the metadata-update event topic. [ERC-4906, Specification][4906] [ERC-2981, interface][2981] [Solidity ABI, Function Selector][abi]

For `MetadataUpdate(42)`, the ERC-4906 declaration has no indexed arguments and is not anonymous.
The ABI therefore gives one topic and one 32-byte data word:

```text
topic[0] = Keccak-256("MetadataUpdate(uint256)")
         = f8e1a15aba9398e019f0b49df1a4fde98ee17ae345cb5f6b5e2c27f5033e8ce7
data     = 000000000000000000000000000000000000000000000000000000000000002a
```

The emitting contract's address is a separate log field.
There is no indexed token-ID topic in this declaration, and these bytes are a derived layout rather than an observed log. [Solidity ABI, Events][abi]

## Calculate a royalty under an explicit policy

Assume token 42's royalty policy returns receiver `0x1111111111111111111111111111111111111111` and charges a fixed 750/10000 of the sale price, rounding down.
That is a chosen 7.5% policy, not a percentage or denominator required by ERC-2981.
Let `U` denote the monetary unit in which the caller expresses the sale price.
For `royaltyInfo(42,1001)`:

```text
1001 * 750 = 750750
750750 = 75 * 10000 + 750
royaltyAmount = floor(750750/10000) = 75 U
receiver = 0x1111111111111111111111111111111111111111
```

ERC-2981 permits rounding up or down when a remainder exists, so 76 would be the permitted alternative under a rounding-up policy.
Under the selected policy, price 10000 U gives 750 U.
The percentage stays independent of sale price.
The function does not discover the currency.
If the input is wei, the output is wei.
If the input uses another token's smallest units, the output uses those same units. [ERC-2981, Specification][2981]

Implementers MUST calculate a percentage and MUST NOT ignore price by returning a constant amount or selecting amounts by comparison with fixed price thresholds.
Subsequent calls MAY return different amounts.
The percentage MAY change using predictable variables independent of currency, such as token ID or transfer count.
The source says it SHOULD NOT depend on unpredictable variables such as `block.timestamp`.
Consequently one query does not establish an immutable royalty quote.

Payment remains a separate action.
ERC-2981 says participating marketplaces SHOULD implement a transfer method.
They MUST pay in the sale's unit of exchange and MUST pay royalties regardless of where the sale occurred.
Participating marketplaces SHOULD NOT send a zero-value transaction when the returned amount is zero.
These obligations do not make every token transfer a sale or cause the view query to move funds. [ERC-2981, Specification and Optional royalty payments][2981]

## Checks and limits

An independent calculation also reproduced the displayed example values.
A standalone PyCryptodome calculation reproduced the selector and event topic, and Python integer arithmetic reproduced the quotient, remainder, and encoded token ID.
No deployed contract, metadata server, royalty payment, EthLib binding, or Lean proof was executed.
The reading does not assess marketplace adoption, content authenticity, or payment enforcement in any implementation.

[1046]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1046.md#specification
[4906]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4906.md#specification
[2981]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2981.md#specification
[721]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-721.md#specification
[1155]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1155.md#metadata
[abi]: https://github.com/argotorg/solidity/blob/73712a01b2de56d9ad91e3b6936f85c90cb7de36/docs/abi-spec.rst
