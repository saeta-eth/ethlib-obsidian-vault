---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: S05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - json-rpc
  - state-proofs
---

# JSON-RPC quantities and block state

A hex string's meaning depends on its RPC field: a number, a byte sequence, and a block identifier have different rules.
Selecting a block by hash identifies the requested state, while a proof response needs a separately trusted root before it can authenticate that state.
Keep encoding, block selection, and verification as distinct boundaries.

## Sources and scope

All EIP links pin `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.

| Source | Selected passages read |
| --- | --- |
| [EIP-1474][e1474], Stagnant | Concepts: JSON-RPC, Error codes, Value encoding, Block Identifier, and the complete `eth_getBalance` method section. |
| [EIP-1898][e1898], Final | Complete text, especially affected methods, hash selectors, canonicality, error recommendations, and reorganization rationale. |
| [EIP-1186][e1186], Stagnant | Complete text, especially request/result fields, proof roles, and proofs for nonexistent values. |

Status does not establish client support.
These passages specify the selected interface behavior, not a full trie-proof algorithm, consensus trust mechanism, or survey of deployed RPC servers.
The examples below use constructed identifiers and conditional outcomes, without contacting a node.

## Quantity and Data encode different things

EIP-1474 requires both forms to be `0x`-prefixed hex.
A **Quantity** uses the shortest representation, with zero spelled `0x0`.
**Data** uses exactly two hex digits per byte and retains leading zero bytes.
An empty byte sequence is `0x`, which is not a valid Quantity.

| String | Quantity interpretation | Data interpretation |
| --- | --- | --- |
| `0x0` | Valid zero. | Invalid: an incomplete byte. |
| `0x00` | Invalid: redundant leading zero. | One zero byte. |
| `0x2a` | The integer 42. | One byte with value 42. |
| `0x002a` | Invalid: redundant leading zeros. | Two bytes, `00 2a`. |
| `0x` | Invalid: no number. | Empty bytes. |

Method-specific widths are additional requirements: arbitrary Data is not automatically a 20-byte address or 32-byte hash.
Do not remove leading zeros from a hash or pad a Quantity to a byte width.
EIP-1474's `eth_getBalance` example itself has a leading-zero result, contrary to its Quantity rules; the constructed result below follows the encoding rule.

## Select the block explicitly

EIP-1898 applies to `eth_getBalance`, `eth_getStorageAt`, `eth_getTransactionCount`, `eth_getCode`, `eth_call`, and `eth_getProof`.
Its object form distinguishes `blockNumber`, a Quantity, from `blockHash`, Data.
EIP-1474's Block Identifier table requires exactly one of those fields.
`requireCanonical` is allowed only with `blockHash` and defaults to `false`.

A number identifies the callee's canonical block at that height; a hash identifies one particular block, including a retained side-branch block.
`latest` can change between requests, and a reorganization can change which block occupies a given height.
A hash avoids silently substituting the replacement block at that height.

| Hash selector condition | EIP-1898 behavior |
| --- | --- |
| Block is not found. | The callee SHOULD raise an error; `-32001`, Resource not found, is recommended. |
| Block is found but noncanonical, with `requireCanonical: true`. | The callee SHOULD additionally raise an error; `-32000`, Invalid input, is recommended. |
| Block is noncanonical, with the flag false or absent. | The selector permits querying that block rather than requiring its canonical replacement. |

The not-found check SHOULD precede the canonicality check.
The source also says the noncanonical error should be distinguishable from not-found.
Those are recommendations and recommended codes, not an unconditional mandate for two exact numeric responses.
Successful selection does not by itself guarantee that every server retains all historical state needed to answer a request.

## Two complete constructed requests

Use a 20-byte address consisting of repeated `22`, a 32-byte block hash consisting of repeated `aa`, and storage key zero on 32 bytes.
These inputs are not claims that an account, contract, or block exists.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_getBalance",
  "params": [
    "0x2222222222222222222222222222222222222222",
    {
      "blockHash": "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "requireCanonical": true
    }
  ]
}
```

If this request succeeds and the balance is 42 wei, the result is:

```json
{"jsonrpc":"2.0","id":1,"result":"0x2a"}
```

The balance is a Quantity in wei, not byte-formatted account data.
For the same selected block, request an account proof and a proof for slot zero:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "eth_getProof",
  "params": [
    "0x2222222222222222222222222222222222222222",
    ["0x0000000000000000000000000000000000000000000000000000000000000000"],
    {
      "blockHash": "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "requireCanonical": true
    }
  ]
}
```

EIP-1186 specifies 32-byte requested storage keys; its result describes each echoed key as a Quantity.
Do not infer that a request key and its result spelling must be identical strings.
EIP-1898 explicitly extends this method's original number/tag selector with the object form used here.

Suppose the balance query succeeds while the block is canonical, then a reorganization removes it before the proof query.
The second request can produce the recommended noncanonical error instead of a proof.
Using `requireCanonical: false` would permit the same historical block if the callee can still serve it.
Neither successful response promises that the block will remain canonical or become finalized.

## A proof-bearing result still needs verification

EIP-1186 separates these response roles:

| Field | Role |
| --- | --- |
| `balance`, `nonce` | Claimed account values, encoded as Quantities. |
| `codeHash`, `storageHash` | 32-byte account fields; the latter supplies the storage-proof root. |
| `accountProof` | RLP-serialized trie nodes following the hashed-address path from the selected state root. |
| `storageProof` entries | A key, claimed Quantity value, and RLP-serialized proof nodes for that slot under the account's storage root. |

The [RLP note](RLP-canonical-encoding.md) explains the underlying byte encoding, not complete trie-proof validity.
Checking a proof requires a state root authenticated for the intended block, then checking that the proof and returned fields agree with it.
Accepting a root supplied only by the same untrusted responder does not establish that connection to the intended chain.
Likewise, `requireCanonical` asks for the callee's view; it is not a cryptographic proof of canonicality.

The source calls `storageHash` “SHA3 of the StorageRoot” while its rationale identifies the account field and storage proof's starting root.
This note uses its stated root role without introducing an additional hashing step.
A zero-valued response alone also does not prove absence: the Non existent values section requires sufficient trie evidence for non-inclusion.
No fabricated proof-node array is supplied here.

## EthLib connection and review limits

At the recorded EthLib commit, [`uint`, `bytesOf`, and `fixed`][json] distinguish shortest integers, byte-paired hex, and fixed widths.
[`BlockParam` and `blockNumberOrTagOrHash`][selector] represent and decode block selection; [`ChainView.resolve`][resolve] checks a found hash's canonical membership when requested.
[`AccountProof.verify`][verify] accepts a hash function and state root, checks the account claim, then checks the existing account's slot claims against its storage root.
It does not establish the supplied root's consensus trust.

These code paths cite other EIP revisions and execution-apis schemas, including a broader `bytesMax32` storage-key reader.
They are selected implementation observations, not an assertion that every accepted wire form is identical to the three pinned EIPs.
Reading covered the cited passages and definitions.
Local JSON parsing checked both requests and the conditional response, including the 20/32-byte inputs and `0x2a = 42`.
Independent JSON and encoding checks reproduced the requests, widths, response value, and Quantity/Data contrasts.
No live RPC, proof vector, complete trie-verifier review, Lean build, or proof replay ran.

[e1474]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1474.md
[e1898]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1898.md
[e1186]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1186.md
[json]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Serialization/JsonRpc.lean#L72-L124
[selector]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Rpc/BlockParam.lean#L91-L155
[resolve]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Rpc/View.lean#L169-L206
[verify]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/State/AccountProof.lean#L190-L249
