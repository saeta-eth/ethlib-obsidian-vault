---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A11
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - wallet-rpc
---

# Wallet RPC requests, permissions and chain changes

Wallet interfaces distinguish granting permission, adding a chain, switching the active chain, and submitting batches of calls.
These operations have different actors, validation conditions and outcome guarantees.

## Sources and scope

All five EIPs below were read at `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Status is source metadata, separate from wallet adoption and EthLib support.

| Source and status | Selected passages read | Scope and limit |
| --- | --- | --- |
| [EIP-1193][e1193], Final. | Definitions, Connectivity, `request`, Supported RPC Methods, RPC Errors, and `chainChanged`. | Application, Provider, wallet, and client responsibilities. No wallet implementation inspected. |
| [EIP-2255][e2255], Final. | Specification, Test Cases, and Security Considerations. | Permission inspection and requests. Example method names differ from the specified names. |
| [EIP-3085][e3085], Stagnant. | Specification, Rationale, and Security Considerations. | Chain-addition boundaries. The combined required-field wording remains unresolved. |
| [EIP-3326][e3326], Stagnant. | Specification, Examples, Rationale, and Security Considerations. | Conditional switch-chain case. It does not mandate one error code for every refusal. |
| [EIP-5792][e5792], Final. | Selected Specification sections for send/status/capabilities, the atomic capability, Error Codes, Call Execution Atomicity, Backwards Compatibility, and Security Considerations. | Batch-submission and capability rules. This is not a full schema-consistency or extension audit. |

EIP-1193 specifies a JavaScript request interface whose Promise resolves with the method's result or rejects with a `ProviderRpcError`.
It does not require support for every RPC method.
Its transport-independent result is distinct from an entire JSON-RPC response envelope.
Those distinctions keep a wallet action from becoming an assumed execution-node operation.

## Method boundaries and unresolved conditions

| Source | Boundary | Conditions and unresolved questions |
| --- | --- | --- |
| EIP-2255 | Permission inspection and permission requests retain the permitted method and caveats as distinct information. | How does an integration identify the invoker, retain restrictions, and associate permission with the intended application? |
| EIP-3085 | Chain configuration is distinct from chain selection and execution RPC. | Which metadata does a wallet ignore or require, which endpoints does it trust, and how do the overlapping field requirements combine? |
| EIP-3326 | Requesting a switch, wallet refusal, actual switching, and Provider notifications are separate events. | Which active-chain concept, supported chain set, pending-request policy, and Provider connection state apply? |
| EIP-5792 | One request does not imply one transaction, atomic execution, or finality. | Which capability overrides, optional capabilities, per-chain atomic support, identifiers, and receipt/status interpretation apply? |

EIP-2255's normative method names are `wallet_getPermissions` and `wallet_requestPermissions`, while its Test Cases omit `wallet_`.
EIP-3085 first describes only `chainId` as required, then explicitly requires rejection for absent or empty `rpcUrls`, alongside its field-ignore rules.
Those passages leave their combined meaning unresolved.

EIP-5792 requires ordered calls on the requested chain, subject to explicitly defined capability overrides.
Its `atomicRequired` flag and per-chain `atomic` capability control guarantees.
Apps must not assume every batch uses one transaction, even with atomicity.
Its prose requires a chain ID without leading zeroes, while example requests use `0x01`.
Final status does not resolve these textual inconsistencies.

This comparison establishes neither a backend execution implementation nor wallet support or EthLib coverage.

## A conditional switch request

An application requests a switch through its Provider:

```json
{
  "method": "wallet_switchEthereumChain",
  "params": [{ "chainId": "0x1" }]
}
```

Assume the wallet implements the method and has a single active chain.
The target chain is known to it, and it can switch to and service requests on that chain.
Those conditions make the request eligible, not compulsory: the wallet can still refuse it.

| Actor or condition | Observable consequence |
| --- | --- |
| Wallet accepts and completes the switch. | The method succeeds with `null`; Provider `request` resolves with that result. |
| Wallet cannot switch, lacks an active-chain concept, or rejects the request. | The method fails; the Provider Promise rejects with a compatible error. Do not invent an EIP-3326-mandated `4902` code. |
| Provider's connected chain changes. | EIP-1193 requires `chainChanged` with the chain ID string. This condition is about the Provider, not merely a displayed wallet label. |

This request neither adds an unknown chain nor grants a permission nor submits a transaction.
EIP-3326 recommends identifying the requester in confirmation and cancelling pending RPC requests and chain-specific confirmations on switching.
The conditional case establishes no ordering guarantee for concurrent requests.
No wallet or provider was called for this example.

No Lean build, proof replay, wallet test, or execution test was performed.

[e1193]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1193.md
[e2255]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2255.md
[e3085]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3085.md
[e3326]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3326.md
[e5792]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5792.md
