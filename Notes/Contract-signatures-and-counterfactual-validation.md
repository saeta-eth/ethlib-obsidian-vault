---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - contract-signatures
---

# Contract signatures and counterfactual validation

A contract signature is a statement accepted by a contract's validation policy at the queried state.
Recovering an ECDSA address does not replace that policy.
ERC-6492 adds deployment or preparation data so the policy can run when the expected account is not yet deployed or ready.
Its wrapper identifies a validation procedure, not an additional cryptographic signature scheme.

## Sources and scope

The primary sources are [ERC-1271, Standard Signature Validation Method for Contracts][1271], and [ERC-6492, Signature Validation for Predeploy Contracts][6492], both Final at ERCs commit `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.
Reading covered their complete Specifications, Rationales, Reference Implementations and Security Considerations, including ERC-6492's on-chain/off-chain helper descriptions and compatibility discussion.
The reference Solidity is examined as published example code, not as an audited implementation or evidence of deployed behavior.

For a separate account-model boundary, reading also covered the Abstract and Delegation indicator subsection of [EIP-7702][7702], Final at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Document status does not establish network activation or a contract's conformance.
This is external-source preparation for EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, without implementation or proof-coverage claims.

## A magic return value answers a policy question

ERC-1271 defines:

```text
isValidSignature(bytes32 hash, bytes signature) returns (bytes4)
success = 0x1626ba7e
```

The four-byte function selector is also `0x1626ba7e`, the first four bytes of Keccak-256 over ASCII `isValidSignature(bytes32,bytes)`.
An application calls the expected signing contract with a 32-byte digest and signature bytes.
The function MUST NOT modify state and MUST allow external calls.
When validation passes, it MUST return that magic value.
Read-only validation can consult other contracts or depend on time, account state, authorized signers or a multisignature policy.
It need not interpret the signature as a 65-byte ECDSA value.

Only a successfully decoded return value equal to the magic indicates success.
A different value or a reverted call does not establish validity.
ERC-1271's reference returns `0xffffffff` on one rejection path, but the specification does not require that particular invalid value.
Its example owner's ECDSA recovery and low-`s` checks are an example policy, not mandatory algorithms for every signing contract.

The same digest and bytes can pass before a policy change and fail afterwards.
Record the queried chain and state instead of treating a prior acceptance as permanently valid.
ERC-1271 supplies no universal gas cap.
Its security discussion cautions against hardcoding one that excludes valid policies.
The application's digest construction and replay conditions remain separate, as in the [signed-data note](Signed-data-preimages-and-domains.md).

## Recognize the wrapper before choosing a verifier

ERC-6492 detects a signature ending in this 32-byte marker:

```text
6492649264926492649264926492649264926492649264926492649264926492
```

Remove that suffix and ABI-decode the preceding bytes as `(address, bytes, bytes)`.
For an undeployed account these are `(create2Factory, factoryCalldata, originalERC1271Signature)`.
For an already deployed but unready account they are `(prepareTo, prepareData, originalERC1271Signature)`.
The same field positions have different operational roles.
The factory interface is deliberately unrestricted.
CREATE2 is RECOMMENDED, not required.

The signer address and digest are separate verification inputs, absent from this wrapper.
The Signer side section explicitly assumes the expected account address is already known.
Although a compatibility paragraph suggests address recovery, arbitrary factory calldata has no standardized address-derivation algorithm here.
Do not infer a general address-recovery function from the wrapper alone.

The verifier MUST preserve this order:

1. Detect the suffix before ordinary ERC-1271 verification or `ecrecover`.
   For wrapped data, simulate the factory/deployment call when code is absent, then validate the unwrapped signature at the expected account.
2. When code already exists, try its current ERC-1271 policy.
   A wrapper created before deployment remains wrapped after deployment.
3. For wrapped data, if validation fails after skipping the factory call because code already existed, execute the supplied preparation call and retry ERC-1271.
4. Only the no-code, ordinary-signature path reaches ECDSA recovery.
   A contract-policy rejection is not permission to bypass that policy through `ecrecover`.

The introduction broadly says deployment MUST precede validation on wrapper detection.
The detailed Verifier side rules explicitly skip deployment for existing code and try the current policy first.
This note retains that qualification rather than requiring an already deployed wallet to be recreated.

## A complete synthetic 256-byte wrapper

Choose factory/prepare address `0x1111111111111111111111111111111111111111`, calldata `0x1234` and inner signature `0xaabbcc`.
These are invented byte examples, not a real factory operation or valid signature.
The [ABI note](ABI-encoding-and-event-topics.md) explains tuple-relative dynamic offsets and right padding.
Concatenate the following eight 32-byte words in order:

```text
offset  bytes
0000    0000000000000000000000001111111111111111111111111111111111111111
0020    0000000000000000000000000000000000000000000000000000000000000060
0040    00000000000000000000000000000000000000000000000000000000000000a0
0060    0000000000000000000000000000000000000000000000000000000000000002
0080    1234000000000000000000000000000000000000000000000000000000000000
00a0    0000000000000000000000000000000000000000000000000000000000000003
00c0    aabbcc0000000000000000000000000000000000000000000000000000000000
00e0    6492649264926492649264926492649264926492649264926492649264926492
```

Offsets are hexadecimal byte positions from the start of the ABI tuple.
The first dynamic tail starts at `0x60` and occupies 64 bytes, so the second starts at `0xa0`.
The tuple occupies 224 bytes and the appended marker brings the wrapper to 256 bytes.
The marker is neither a function selector nor part of the inner signature passed to ERC-1271.
This wrapper is not itself the digest or signed-message preimage.

Conditionally, suppose the expected wallet already has code and the first ERC-1271 check rejects `0xaabbcc` for the application's digest.
The deployment call was skipped, so the verifier tries `0x1234` at the supplied prepare address and retries the wallet policy in the resulting state.
Acceptance requires the retried policy to return `0x1626ba7e`.
If code was initially absent instead, the deployment path must make that expected wallet callable before its policy can validate anything.
No deployment, preparation or policy outcome is asserted for these synthetic bytes.

## Simulation does not make every helper side-effect-free

ERC-6492's verifier procedure uses an `eth_call` to a multicall/helper so deployment and validation can occur in one simulated execution.
Those transient state changes enable the policy check without publishing a deployment transaction.
Its reference also exposes methods for actual on-chain calls, where deployment/preparation uses `CALL`, unlike ERC-1271's read-only policy check.
The source explicitly identifies reentrancy concerns and says underlying calls can revert.

There is a limit in the displayed reference's rollback logic.
Its explicit revert-with-result branch tests `contractCodeLen == 0 && isCounterfactual && !allowSideEffects`, with code length measured before deployment.
An existing-code preparation retry has nonzero initial code length and does not satisfy that guard.
Thus the broader prose claim that side effects always trigger rollback does not follow for every preparation path shown.
Treat that discrepancy as an unresolved source/code boundary, not a guarantee that the named `isValidSig` helper is universally side-effect-free or reentrancy-safe.
This observation comes from reading the branches, without executing or auditing a deployed helper.

## Account classification and remaining limits

Separately, EIP-7702 allows an EOA to hold a 23-byte delegation indicator, with `CALL` and `STATICCALL` following its code pointer.
Code presence therefore does not mean an address has no associated private key.
This context does not amend the pinned ERC-6492 validation order or claim that every delegated account implements ERC-1271.
The [authorization note](Authorization-order-and-rollback.md) covers that distinct mechanism.

ERC-6492 leaves correct deployment permissions to the account implementation and notes cross-network replay possibilities after key rotation.
Wrapper detection does not establish those permissions or application-specific replay protection.
Local byte construction checked the complete tuple offsets, lengths, padding and marker.
PyCryptodome 3.23.0 reproduced the ERC-1271 selector.
The source passages and explanation received a separate review, including decoding the displayed wrapper.
An independent manual construction and ABI encode/decode comparison reproduced the complete wrapper, offsets, lengths, padding, and suffix.
No signature verification, contract deployment, RPC simulation, EVM execution, EthLib build or Lean proof replay ran.

[1271]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1271.md
[6492]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6492.md
[7702]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md#delegation-indicator
