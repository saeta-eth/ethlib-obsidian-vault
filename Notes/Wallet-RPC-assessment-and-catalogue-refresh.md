---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 6
catalogue_family: A11
assessment_kind: "bounded Unassessed cluster and source refresh"
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
ercs_baseline_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
ercs_refresh_commit: "50b3a3cdd2c12e2b81288b56f073c0ee812f95a8"
paper_version: "arXiv:2101.08778v6, 2022-09-15"
paper_sha256: "9079375c121a8033457a952b58b5dc2fb6f2ad9d94ba9ba247b1994b127c5b2c"
tags:
  - ethlib
  - external-sources
  - wallet-rpc
  - catalogue-assessment
---

# Wallet RPC assessment and catalogue refresh

Four previously Unassessed wallet proposals belong in A11's provider and wallet boundary research.
They concern different actions: granting permission, adding a chain, switching the active chain, and submitting batches of calls.
This assessment recommends connecting all four to A11 while retaining its **When needed** priority.
It also records a fresh source comparison and one literature addition, without replacing the pinned baseline.

## Reading and readiness

All five EIPs below were read at `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Status is source metadata, separate from wallet adoption, EthLib support, and note priority.
**Ready** means enough reading for the stated assessment, not an implementation-ready protocol design.

| Source and status | Selected passages read | Readiness and limit |
| --- | --- | --- |
| [EIP-1193][e1193], Final, prerequisite already in A11. | Definitions, Connectivity, `request`, Supported RPC Methods, RPC Errors, and `chainChanged`. | Ready to distinguish application, Provider, wallet, and client responsibilities. No wallet implementation inspected. |
| [EIP-2255][e2255], Final. | Specification, Test Cases, and Security Considerations. | Ready for a permissions assessment. Example method names differ from the specified names. |
| [EIP-3085][e3085], Stagnant. | Specification, Rationale, and Security Considerations. | Ready for add-chain boundaries. Required-field wording needs clarification before a minimal-request implementation. |
| [EIP-3326][e3326], Stagnant. | Specification, Examples, Rationale, and Security Considerations. | Ready for a conditional switch-chain case. It does not mandate one error code for every refusal. |
| [EIP-5792][e5792], Final. | Selected Specification sections for send/status/capabilities, the atomic capability, Error Codes, Call Execution Atomicity, Backwards Compatibility, and Security Considerations. | Ready for batch/capability assessment. This is not a full schema-consistency or extension audit. |

EIP-1193 specifies a JavaScript request interface whose Promise resolves with the method's result or rejects with a `ProviderRpcError`.
It does not require support for every RPC method.
Its transport-independent result is distinct from an entire JSON-RPC response envelope.
Those distinctions keep a wallet action from becoming an assumed execution-node operation.

## Four separate judgments per source

The following judgments concern useful knowledge for a future contribution, not missing implementation findings.
Effort estimates describe follow-up research scope rather than engineering time.

| Source | Connection | Contribution value | Additional knowledge needed | Effort |
| --- | --- | --- | --- | --- |
| EIP-2255 | A11's permission boundary between requester and wallet. | Explain permission inspection versus requesting permission, with the permitted method and caveats kept distinct. | How an integration identifies the invoker, retains restrictions, and associates permission with the intended application. | Small conceptual note, then a focused integration review. Resolve the example discrepancy first. |
| EIP-3085 | A11's chain-configuration request, distinct from execution RPC. | Trace chain ID, endpoint metadata validation, rejection, and successful addition without assuming chain selection. | Wallet policy for metadata it ignores or requires, endpoint trust, and the source's overlapping field requirements. | Moderate source clarification and state/validation modelling. No live endpoint access is necessary for the initial note. |
| EIP-3326 | A11's active-chain transition. | Separate requesting a switch, wallet refusal, actual switching, and Provider notifications. | The wallet's active-chain concept, supported chain set, pending-request policy, and Provider connection state. | Small conditional transition case. A complete concurrent-request model is a separate task. |
| EIP-5792 | A11's batch submission, capability discovery, and status reporting. | Prevent assumptions that one request means one transaction, atomic execution, or finality. | Capability-specific overrides, optional capabilities, per-chain atomic support, identifiers, and receipt/status interpretation. | Moderate initial state diagram. Extension semantics and a full consistency review need further work. |

EIP-2255's normative method names are `wallet_getPermissions` and `wallet_requestPermissions`, while its Test Cases omit `wallet_`.
Record that discrepancy rather than copying the examples silently.
EIP-3085 first describes only `chainId` as required, then explicitly requires rejection for absent or empty `rpcUrls`, alongside its field-ignore rules.
This assessment preserves those passages without deciding their combined meaning.

EIP-5792 requires ordered calls on the requested chain, subject to explicitly defined capability overrides.
Its `atomicRequired` flag and per-chain `atomic` capability control guarantees.
Apps must not assume every batch uses one transaction, even with atomicity.
Its prose requires a chain ID without leading zeroes, while example requests use `0x01`.
These are reasons to retain exact pins and further questions, rather than promote Final status into evidence of complete consistency.

All four connections have concrete questions, so leaving them Unassessed would lose information gained by this reading.
Retaining **When needed** keeps full follow-up notes tied to an actual provider, wallet, or interface contribution.
No backend execution implementation, wallet support survey, or EthLib coverage claim follows from this mapping.

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

| Actor or condition | Observable consequence to review |
| --- | --- |
| Wallet accepts and completes the switch. | The method succeeds with `null`; Provider `request` resolves with that result. |
| Wallet cannot switch, lacks an active-chain concept, or rejects the request. | The method fails; the Provider Promise rejects with a compatible error. Do not invent an EIP-3326-mandated `4902` code. |
| Provider's connected chain changes. | EIP-1193 requires `chainChanged` with the chain ID string. This condition is about the Provider, not merely a displayed wallet label. |

This request neither adds an unknown chain nor grants a permission nor submits a transaction.
EIP-3326 recommends identifying the requester in confirmation and cancelling pending RPC requests and chain-specific confirmations on switching.
Those recommendations need a separate concurrency case before asserting an ordering guarantee.
No wallet or provider was called for this example.

## Refresh the census without rewriting its history

The official GitHub API resolved both repositories' default branches as `master` on 17 November 2026.
The observed heads were:

| Repository | Baseline | Fresh head |
| --- | --- | --- |
| ethereum/EIPs | `8194528a2ee9a4893a398a5871c470ab0ac3b09a` | Same commit. |
| ethereum/ERCs | `dbba0f4479dd817d19d7d7771dea0dcd76c68d17` | [Fresh snapshot `50b3a3cd`](https://github.com/ethereum/ERCs/tree/50b3a3cdd2c12e2b81288b56f073c0ee812f95a8). |

The comparison enumerated the same numbered Markdown paths as the baseline, checked filename/declared-number agreement, retained the same metadata fields, and resolved EIP Moved records to ERC destinations.
Before using the parser for the refresh, it reproduced all 1,232 baseline records and 365 redirects exactly.
Compared fields were identifier, title, status, type, category, requires, created, owning repository, and path.
Revision and URL change mechanically with the source pin and were excluded from semantic metadata differences.

There are **no added or removed IDs, no changed selected metadata, and no changed redirects**.
The fresh census still contains 607 EIPs and 625 ERCs, with 365 EIP redirects resolved.
The new ERC archive's SHA-256 is `9640e9ce2c0d0b43e65128ba41d2e4422350909d3e0f780279e3649a06bd85b3`.

Equal metadata does not mean equal bodies.
A separate SHA-256 comparison of all 625 numbered ERC Markdown files found changed bytes only in [ERC-8048](https://github.com/ethereum/ERCs/blob/50b3a3cdd2c12e2b81288b56f073c0ee812f95a8/ERCS/erc-8048.md) and [ERC-8049](https://github.com/ethereum/ERCs/blob/50b3a3cdd2c12e2b81288b56f073c0ee812f95a8/ERCS/erc-8049.md).
Their titles are *Onchain Metadata for Token Registries* and *Contract-Level Onchain Metadata* respectively, both Draft at these pins.
Their changed text was not semantically reviewed, and both remain candidates for later assessment.
The selected ERCs 137, 181, 634, 4626, 7540, and 7575 have identical bytes across these revisions.
The pinned inventory remains a baseline, with this comparison recorded separately.

## A literature addition for A07

**R17: [*SoK: Decentralized Finance (DeFi)*, arXiv:2101.08778v6][paper]**, revised 15 September 2022, is by Sam M. Werner, Daniel Perez, Lewis Gudgeon, Ariah Klages-Mundt, Dominik Harz, and William J. Knottenbelt.
Reading covered the abstract, §1's scope, and selected opening definitions/discussion in §5, Technical Security, and §6, Economic Security.
The arguments, empirical cases, and full model assumptions were not reviewed.

The authors distinguish atomic technical exploits from non-atomic economic exploits with costs and incentive risks.
That taxonomy is supporting context for A07's tokenized-vault questions: interface compliance and correct share arithmetic alone do not establish economic security under market interaction.
The paper does not specify ERC-4626, establish wallet support, or identify an EthLib implementation gap.
Its market figures describe April 2022, not current conditions.
Further reading should identify the particular economic model and assumptions before carrying a security claim into a contribution.

This note records source reading, metadata reproduction/comparison, and document-byte hashing.
The sources, assessment, and literature scope received a separate review.
An independent comparison checked the complete old/fresh metadata records on all nine tracked fields, repository counts, and the refreshed archive digest.
No Lean build, proof replay, wallet test, execution test, or economic-security analysis was performed.

[e1193]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1193.md
[e2255]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2255.md
[e3085]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3085.md
[e3326]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3326.md
[e5792]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5792.md
[paper]: https://arxiv.org/pdf/2101.08778v6
