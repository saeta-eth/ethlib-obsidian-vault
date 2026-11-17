---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: S04
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
solidity_release: "v0.8.30"
solidity_commit: "73712a01b2de56d9ad91e3b6936f85c90cb7de36"
tags:
  - ethlib
  - external-sources
  - abi
---

# ABI encoding and event topics

The Contract Application Binary Interface (ABI) needs a type schema to interpret bytes.
Function arguments, event topics, and packed data follow different encoding rules.
Choosing the rule first prevents a correct byte sequence from acquiring the wrong meaning.

## Source version and scope

This note follows Solidity **v0.8.30**, at [commit `73712a01`](https://github.com/argotorg/solidity/commit/73712a01b2de56d9ad91e3b6936f85c90cb7de36), without making a claim about later releases.
The primary text is [Contract ABI Specification, `docs/abi-spec.rst`](https://github.com/argotorg/solidity/blob/73712a01b2de56d9ad91e3b6936f85c90cb7de36/docs/abi-spec.rst).
The [versioned rendering](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html) supplies section links below.
Read sections were Function Selector, Types, Formal Specification of the Encoding, Function Selector and Argument Encoding, Events, Errors, Strict Encoding Mode, Non-standard Packed Mode, and Encoding of Indexed Event Parameters.
The ERC-20 example also uses [Events/Transfer at the pinned ERC revision](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-20.md#transfer-1).

## Signatures identify the schema

A function's canonical signature contains its name and comma-separated parameter types, without spaces, parameter names, data locations, or return types.
Thus `f(uint amount, string memory label)` has signature `f(uint256,string)`.
Its selector is the first four bytes of Keccak-256 of that signature.
The argument tuple follows those four bytes.
Return values use a tuple encoding without a selector.
These rules are in [Function Selector](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#function-selector) and [Function Selector and Argument Encoding](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#function-selector-and-argument-encoding).

## Heads, tails, and relative offsets

In a tuple, a static member contributes its encoding to the head.
A dynamic member contributes a 32-byte offset to the head and its encoding to the tail.
The offset measures bytes from the start of that tuple's encoding, not from the transaction or selector.
A static member can occupy several words, so head size is not always 32 times the number of members.

For nested tuples, identify the enclosing encoding each time.
A dynamic array starts with its element count, followed by its elements encoded as a tuple.
Offsets inside that element tuple are relative to its start, after the count word.
The [formal specification](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#formal-specification-of-the-encoding) defines these rules recursively.

Dynamic `bytes` stores a byte count, contents, and enough trailing zero bytes to reach a multiple of 32.
A `string` first becomes UTF-8 bytes, so its count measures bytes rather than characters.
Unsigned integers receive leading zeros.
Fixed `bytesN` receives trailing zeros.

## Worked encoding: `(uint256,string)` with `(7,"cat")`

The head contains two words: the integer and an offset.
It occupies 64 bytes, so the string tail begins at offset `0x40`.
The UTF-8 bytes of `cat` are `63 61 74`, giving length 3 and 29 trailing zero bytes.
The complete argument encoding is four consecutive words:

```text
offset  word
0x00    0000000000000000000000000000000000000000000000000000000000000007
0x20    0000000000000000000000000000000000000000000000000000000000000040
0x40    0000000000000000000000000000000000000000000000000000000000000003
0x60    6361740000000000000000000000000000000000000000000000000000000000
```

The argument block has 128 bytes.
A call to `f(uint256,string)` prepends selector `1b a8 4e 08`, giving 132 bytes.
Consequently the length word starts at calldata offset `0x44`, although its argument-relative offset remains `0x40`.
This example was derived from the source rules, not generated from EthLib.

## Packed bytes lose boundaries

[Non-standard Packed Mode](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#non-standard-packed-mode) omits the length and trailing padding of direct `string` and `bytes` arguments.
Thus packing `("a","bc")` and `("ab","c")` produces the same bytes, `61 62 63`.
Hashing those bytes cannot restore the missing boundary.
This ambiguity already exists before Keccak runs.
Array elements have their own padding rules, so concatenating arbitrary raw values is not a general packed encoder.

## Events put some values in topics

For a non-anonymous event, topic 0 is the full Keccak-256 hash of its canonical event signature.
The signature includes every parameter type, with neither names nor `indexed` annotations.
Up to three indexed parameters occupy the remaining topics.
An anonymous event omits the signature topic and permits four indexed parameters.
The data field encodes the tuple of non-indexed arguments.

Elementary indexed values occupy one word with their normal padding or sign extension.
Arrays, structs, `string`, and dynamic `bytes` instead contribute a hash of the [special indexed encoding](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#encoding-of-indexed-event-parameters).
This includes fixed arrays.
For an indexed string `cat`, the hash preimage is `63 61 74`, without a length or padding.
Array and struct members follow additional padding rules.
Do not substitute the ordinary argument encoding for these preimages.

For ERC-20's non-anonymous `Transfer(address,address,uint256)`, the addresses occupy topics 1 and 2 and the amount occupies the data word.
The [transfer-log note](ERC-20-and-native-ETH-transfer-logs.md) separately explains emission requirements and native ETH comparisons.
The ABI layout supplies no rule about when a transfer occurs.

## Decoder and error limits

[Strict Encoding Mode](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#strict-encoding-mode) describes minimal, non-overlapping offsets with no gaps.
The pinned text says Solidity's encoder produces that layout while its decoder does not enforce strict mode.
Encoder layout and decoder acceptance therefore need separate claims.

An [ABI-encoded error](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html#errors) uses a selector followed by encoded arguments.
Matching bytes do not establish an error's origin: nested calls can propagate them, and another contract can fabricate them.

## EthLib entry points and review limits

At the recorded commit, [the `Log` structure](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Evm/Log.lean#L27) stores address, topics, and data.
[`depositRequestOfEvent`](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Blocks/Requests.lean#L113) parses one fixed deposit-event layout.
These are limited entry points for log work, not evidence of a general ABI codec.
The selected source sections and these definitions were inspected.
An independent source review found no material discrepancy in the selected passages or explanation.
Python package `eth-abi` 5.2.0 reproduced all four argument words, decoded them to `(7,"cat")`, and reproduced the packed ambiguity.
PyCryptodome 3.23.0 recomputed the selector with Keccak-256; the 128/132-byte lengths and calldata offset were checked separately.
No compiler invocation, Lean build, binding test, general decoder audit, or comparison with later ABI revisions was performed.
