---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E08
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellow_paper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - receipts
  - block-commitments
---

# Receipts: status, gas, and block commitments

A receipt records an included transaction's execution outcome, cumulative gas and surviving logs.
It does not contain arbitrary return bytes or replace the block's state, transaction, withdrawal and request commitments.
Decode each object under the applicable fork and transaction type before interpreting its fields.

## Sources and limits

At EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`, [EIP-658][658], [EIP-2718][2718], [EIP-1559][1559], [EIP-4895][4895], [EIP-4844][4844] and [EIP-7685][7685] are Final.
[EIP-7708][7708] is Last Call, and [EIP-7799][7799] is Draft.
These statuses do not establish network activation.
Reading covered their selected receipt, gas, withdrawal, header-commitment and log rules, plus [EIP-140][140] for execution rollback.
The Shanghai [Yellow Paper][yp], pinned at `efc5f9a1f356cba376c978eedb63cb0363c2aa85`, supplies §4.4.1's receipt layout and Appendix B's RLP rules.
This note does not assess every subsequent gas amendment or reconstruct a complete header across all forks.

## Status is an execution result

From its Byzantium condition, EIP-658 replaces the receipt's intermediate state root with integer status `1` for success or `0` for failure of the transaction or top-level call.
Earlier receipts require the historical state-root interpretation.
Although EIP-658's motivation uses broader language about return data, its specification introduces this status field, not a field holding EVM return bytes.
A success status also does not establish an application's separate business condition.

EIP-140 places a reverted callee's error bytes in the caller's return-data buffer and discards the reverted execution's changes, including logs.
A valid `REVERT` need not consume all remaining gas, so failure cannot be inferred from gas exhaustion alone.
A caller can handle an inner failure and finish successfully; its receipt then describes the top-level outcome and surviving logs.
An accepted transaction can still consume gas and pay fees when execution fails.
The [transfer-log note](ERC-20-and-native-ETH-transfer-logs.md) treats event semantics and reverted-frame logs separately.

## Typed envelope and four-field payload

EIP-2718 requires the receipt type to match the corresponding transaction's type at the same block index.
It leaves typed receipt payloads opaque for their defining EIPs.
EIP-1559 specifically defines type `0x02` with this payload:

```text
02 || RLP([status, cumulative_transaction_gas_used, logs_bloom, logs])
```

The type byte is outside the RLP list.
This does not make every future typed payload RLP.
The [transaction-envelope note](Typed-transaction-envelopes-and-signing.md) explains the corresponding transaction boundary, and the [RLP note](RLP-canonical-encoding.md) explains minimal integer and length encodings.

The cumulative gas field counts through the transaction, rather than storing that transaction's gas alone.
In a hypothetical London sequence, let the previous receipt contain `21000` and the current receipt contain `42000`.
The current transaction contributes `42000 - 21000 = 21000` gas.
EIP-1559's reference accounting accumulates each transaction's gas into this block total.
This example does not assume that the same counter relation survives every later gas-accounting amendment.

The logs Bloom is a 256-byte filter derived from logger addresses and topics; it does not contain the log data or an exact list of events.
An empty log list gives the all-zero Bloom used below.
Neither an empty list nor a success status establishes that a particular application event occurred.

## Complete constructed type-2 receipt

Choose status `1`, cumulative gas `42000 = 0xa410`, 256 zero Bloom bytes and an empty logs list.
These are synthetic receipt fields, not the result of a supplied transaction or a published receipt vector.
Their RLP encodings are:

| Field | Encoding | Length |
| --- | --- | --- |
| Status 1 | `01` | 1 byte |
| Cumulative gas 42000 | `82a410` | 3 bytes |
| Bloom | `b90100` followed by 256 zero bytes | 259 bytes |
| Empty logs list | `c0` | 1 byte |

The list contents occupy `1+3+259+1 = 264 = 0x0108` bytes, giving list prefix `f90108`.
The RLP payload occupies 267 bytes; adding type `02` gives 268 bytes.
Concatenate every hex line in this complete encoding:

```text
02f901080182a410b90100
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
00000000000000000000000000000000
c0
```

For a representation-only failure variant with the other fields fixed, status zero encodes as `80`, replacing `01` at byte offset 4.
Integer zero uses the empty byte string, not the byte `00`; the total size remains 268 bytes.
This variant does not claim an execution witness with the same gas consumption.
Rolled-back EVM logs are absent; later proposals for protocol-generated logs need their own rules.

## Header commitments have different domains

The following map covers selected commitments and accounting fields, with each added field conditional on its EIP's fork rules:

| Field | What it records or commits to |
| --- | --- |
| `state_root` | The resulting execution state, rather than a transaction's return bytes. |
| `transactions_root` | The indexed transaction bytes, including typed envelopes. |
| `receipts_root` | The indexed receipt bytes, including their type prefix when applicable. |
| `withdrawals_root`, EIP-4895 | A separate indexed trie of withdrawal operations. |
| `blob_gas_used`, EIP-4844 | The block's blob-gas consumption, independent of normal transaction gas. |
| `excess_blob_gas`, EIP-4844 | The prior excess used by blob-gas pricing. |
| `requests_hash`, EIP-7685 | The two-level SHA-256 commitment to nonempty typed request bodies. |

EIP-2718 requires receipt-trie keys `RLP(Index)` and receipt values in the defined legacy or typed format.
Our hypothetical second receipt would use key `RLP(1) = 01`; no trie or root is calculated here.
The Bloom is a search filter, while the receipt trie commits to the complete receipt values.

EIP-4895's withdrawals are separate operations processed after transactions, with unconditional balance credits, Gwei-to-wei conversion and no associated gas costs.
Its trie uses list positions, not the withdrawal's global index counter.
These consensus-to-execution credits differ from the execution-to-consensus requests explained in the [system-call note](System-calls-and-execution-request-commitments.md).
EIP-4844 retains the four-field receipt payload for blob transactions; it does not put blob bytes into that payload.
The selected header map is not a complete fork-specific header schema.

## Proposed logs and pinned code

Last Call EIP-7708 proposes native-ETH transfer logs within transaction execution, excluding fee and withdrawal logs from its transfer mechanism.
Draft EIP-7799 separately proposes fee logs appended for every transaction, a block-level system-logs list, and a `system_logs_root` header field.
Its block-level log objects use null transaction identifiers.
Those proposal rules must not be silently added to the London receipt example or treated as deployed facts from status alone.

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`Outcome`][lean-receipt] distinguishes historical state roots from statuses, and [`encodeReceipt`][lean-encode] selects legacy versus typed bytes.
[`receiptsRoot`][lean-roots] builds the indexed trie from those encoded receipt values.
[`applyTransaction`][lean-apply] keeps receipt outcome/logs separate from its recorded message output.
The receipt module explicitly describes later Glamsterdam sender-paid cumulative gas separately from block gas; those referenced amendments were not assessed here.
These are observations of definitions, not proof-coverage claims.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
An independent Python RLP calculation checked field prefixes, the complete 268-byte encoding, the status-zero variant and the conditional gas subtraction.
No transaction execution, receipt-trie root, network query, EthLib binding, Lean build or proof replay ran.

[658]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-658.md
[2718]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2718.md#receipts
[1559]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1559.md#specification
[4895]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4895.md#specification
[4844]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4844.md
[7685]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7685.md#block-header
[7708]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7708.md
[7799]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7799.md
[140]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-140.md#specification
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[lean-receipt]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Receipt.lean#L54
[lean-encode]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Receipt.lean#L172
[lean-roots]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Roots.lean#L57
[lean-apply]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Transition.lean#L212
