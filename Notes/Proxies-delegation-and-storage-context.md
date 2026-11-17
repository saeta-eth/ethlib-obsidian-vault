---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A10
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - execution
  - proxies
---

# Proxies, delegation, and storage context

Code read from one account can execute with another account's storage, address, caller, and value.
Keep the code source separate from that execution context when reading a proxy or EIP-7702 delegation.
This note compares their routing choices and traces one storage write.

## Sources and prerequisites

Read [ABI encoding and event topics](ABI-encoding-and-event-topics.md) for selectors, then [authorization order and rollback](Authorization-order-and-rollback.md) for EIP-7702's pre-execution changes.
The primary sources below use the exact repository revisions in this note's metadata:

| Source | Sections examined |
| --- | --- |
| [EIP-7](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7.md) | Overview, Specification, and Rationale. |
| [ERC-1167](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1167.md) | Specification, Implementation, and Standard Proxy disassembly. |
| [ERC-1967](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1967.md) | Specification, Rationale, selected proxy fallback code, and Security Considerations. |
| [ERC-2535](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2535.md) | Terms, Fallback Function, Storage, and Implementation Points. |
| [EIP-7702](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md) | Behavior and Delegation Indicator, including Precompiles and Loops. |
| [EIP-140](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-140.md) | Specification. |

Apply each comparison where its EIPs are active: the proxy runtime depends on the return-data behaviour named in ERC-1167, and the delegated-account case requires EIP-7702.
The pinned texts and EthLib's fork conditions do not establish activation on a particular network.

## Choose how to find the code

ERC-1167 embeds one implementation address in proxy runtime code.
ERC-1967 specifies storage locations for a logic address, beacon, and optional administrator.
ERC-2535 routes function selectors to facets, whose code runs through `DELEGATECALL` in the diamond's context.
These are contract-level mechanisms.

EIP-7702 instead stores `ef0100 || address` as an authorizing account's code.
Code-executing operations resolve that indicator as the EIP specifies.
An ordinary `CALL` to that account therefore needs no proxy fallback or `DELEGATECALL` instruction to execute the target's code in the authorizing account's context.

### A complete minimal-proxy runtime

Choose implementation address `0x11223344556677889900aabbccddeeff00112233`.
Substitute its 20 bytes into the standard ERC-1167 template:

```text
prefix:          363d3d373d3d3d363d73
implementation:  11223344556677889900aabbccddeeff00112233
suffix:          5af43d82803e903d91602b57fd5bf3
```

Concatenating those fragments gives the complete 45-byte runtime:

```text
363d3d373d3d3d363d7311223344556677889900aabbccddeeff001122335af43d82803e903d91602b57fd5bf3
```

Zero-based byte 9 is `73`, the `PUSH20` opcode; bytes 10 through 29 hold the address.
Byte 31 is `f4`, `DELEGATECALL`.
The remaining code copies return data and either returns it or propagates a revert.
This is runtime code derived from the published template, not deployment code or a compiled test contract.
Its embedded address is fixed; the standard runtime does not read an ERC-1967 implementation slot.

### Storage lookup is a separate convention

ERC-1967's implementation slot is calculated from a literal string:

```text
bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1)
= 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc
```

Hash the literal's bytes, interpret the digest as an unsigned integer, subtract one, then represent it as 32 bytes.
The displayed result is published by the standard.
At that slot the proxy stores the implementation address, rather than executable code.

The beacon slot uses the string `eip1967.proxy.beacon`; it stores a beacon address whose `implementation()` returns the logic address.
The standard recommends considering the beacon only if the direct logic slot is empty.
The optional admin slot uses `eip1967.proxy.admin` and identifies upgrade authority, not execution code.
Slot changes SHOULD emit their corresponding events, including initialization; do not strengthen that recommendation to MUST.

These distinguished slots avoid the ordinary compiler-layout collisions discussed by ERC-1967.
They do not prevent implementation code from deliberately overwriting them or establish that a proposed upgrade preserves application storage.
ERC-2535 adds selector-to-facet routing but explicitly leaves its particular storage layout unspecified.

## Trace the execution context

Let caller `A` call proxy `P` with value 7 wei.
`P` delegates to ordinary implementation code at `I`.
Compare that with `A` calling account `D` with value 7 wei, where `D` holds EIP-7702 delegation to the same `I`.
Assume sufficient gas and balance, a non-static context, and no further calls or delegation in `I`.

| Property while implementation code runs | `A → P → DELEGATECALL(I)` | `A → CALL(D)`, with `D` delegated to `I` |
| --- | --- | --- |
| Account supplying the executing code | `I` | `I` |
| `ADDRESS` | `P` | `D` |
| `CALLER` | `A` | `A` |
| `CALLVALUE` | 7 wei | 7 wei |
| Storage read or written by that frame | `P`'s storage | `D`'s storage |

For a synthetic action that stores value 9 at key 0 and returns, the proxy case writes `P`'s storage.
The delegated-account case writes `D`'s storage.
It does not modify `I`'s storage.
This trace follows the context rules; it is not an executed bytecode vector.
The proxy's `DELEGATECALL` inherits its parent's caller and value, and transfers no additional value to `I`.
The EIP-7702 case retains the ordinary call's destination context after resolving its code.

Code inspection must identify which account or frame it inspects.
During delegated execution, `CODESIZE` and `CODECOPY` use the executing implementation code.
For delegated account `D`, `EXTCODESIZE(D)` is 23; `EXTCODECOPY(D)` reads its indicator, and `EXTCODEHASH(D)` hashes that indicator.
They do not inspect `I` on `D`'s behalf.

EIP-7702 resolves only one indicator: if the target holds another indicator, it does not chase that second pointer.
Delegation to a precompile obtains empty code instead of invoking the native precompile.
Contract code can itself issue further calls; that is different from recursively resolving indicator bytes.

## Failure boundaries

Under EIP-140, a reverting implementation frame rolls back its state changes and returns failure with its revert data.
The standard ERC-1167 runtime propagates that failure by reverting with the returned bytes.
When the enclosing proxy message reverts, its value transfer and message-local writes roll back too.
An outer caller can handle the failure rather than reverting its own entire execution.

EIP-7702's earlier processed authorization changes are a separate boundary: the EIP explicitly keeps processed delegation indicators when transaction execution fails.
That does not preserve a storage write made by the failing implementation frame.
The [authorization note](Authorization-order-and-rollback.md) separates tuple processing from execution and its rollback.

## EthLib connection and review limits

At the recorded commit, [`delegatedAddress`](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Evm/Delegation.lean#L44) recognizes an indicator's exact shape.
[`accessDelegation` and `finishCall`](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Evm/Call/Instructions.lean#L120) separately resolve code and construct the caller, value, and target context.
[`rollbackOnError`](https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Execution/Evm/Call/MessageCall.lean#L170) restores the message's state snapshot on failure.
These definitions were inspected; they do not establish a complete proxy-standard implementation or its proof coverage.

The selected source passages were read and the runtime fragments checked against ERC-1167.
A separate source review checked the context, routing, and rollback explanation, including the source's recommendation strength.
Independent calculations checked the complete 45-byte runtime, its address/opcode positions, and the 23-byte delegation indicator.
PyCryptodome 3.23.0 reproduced the published implementation, beacon, and admin slot constants with Keccak-256 and subtraction.
No EVM execution, compiler invocation, EthLib build, or Lean proof was run.
Full diamond upgrades, initialization policy, storage-layout compatibility, and fork-specific gas accounting remain outside this note.
