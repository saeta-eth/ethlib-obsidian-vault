---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - account-abstraction
  - modules
---

# Account abstraction validation and module authority

A UserOperation can pass account validation and still fail during execution.
Likewise, installing a module to validate operations does not automatically authorize it to execute calls.
Track validation, execution authority, fee sponsorship, and protocol delegation as distinct decisions.

## Sources and version boundary

| Source | Pin, status, and selected reading |
| --- | --- |
| [ERC-4337][4337] | ERCs commit above, Final: definitions, both operation structures, account interface, nonce rules, required EntryPoint functionality, EIP-712/7702 support, paymaster extension, simulation, error codes, and caller checks. |
| [ERC-7579][7579] | Same ERCs pin, Draft: complete account Validation, Execution Behavior, module configuration, Hooks, module-type interfaces, and selected security text. |
| [ERC-6900][6900] | Same pin, Draft: Terms, Overview, Interfaces, execution manifest, Validation Call Flow, Checking Validation Applicability, and Execution Call Flow. |
| [EIP-7702][7702] | EIPs commit above, Final: authorization tuple processing, delegation execution, and initialization/delegate security passages. |

These pins do not establish deployment, wallet support, or fork activation.
ERC-7579's optional `executeUserOp` comment explicitly references ERC-4337 v0.7, while the pinned ERC-4337 text includes EIP-712 hashing and optional paymaster-signature suffixes.
Do not assume every historical EntryPoint version implements every rule in this pinned text.
This note prepares source-based contributions without asserting EthLib account-abstraction coverage.

## Assign each actor its check

The account is the UserOperation's `sender`, while a bundler packages operations into an ordinary transaction calling EntryPoint's `handleOps`.
The bundler is not thereby the account's authorizer.
EntryPoint performs verification and execution loops.
During verification, the account MUST check its caller is a trusted EntryPoint and validate the operation's signature.
On signature mismatch it SHOULD return `SIG_VALIDATION_FAILED=1` without reverting, while other account-validation errors MUST revert. [ERC-4337, Smart Contract Account Interface][4337]

Account validation returns packed authorizer and validity information.
An authorizer value of zero denotes a valid signature, one denotes signature failure, and another value names an aggregator/authorizer contract.
`validUntil=0` denotes an unbounded end time.
ValidationData zero therefore represents ordinary signature success with zero start and unbounded end, subject to the other checks.
It does not report the success of the requested application call.

When a paymaster is present, EntryPoint also checks its deposit and asks `validatePaymasterUserOp` whether it will pay.
The account receives `missingAccountFunds=0` in this flow.
The paymaster's willingness to fund gas does not substitute for account authorization.
A nonempty returned context requires `postOp` after the main execution call, and the source's `opReverted` mode expressly retains the paymaster's gas obligation. [ERC-4337, Extension: paymasters][4337]

Bundler simulation validates account and paymaster behaviour against current state.
The source does not require simulation of execution, and validation can change as state changes.
A successful validation simulation is consequently not an application-success guarantee. [ERC-4337, UserOperation Simulation][4337]

## Exact nonce and execution bytes for a conditional trace

ERC-4337 divides its 256-bit nonce into a 192-bit key and 64-bit sequence, maintained by EntryPoint for the sender.
Each key has its own sequence, beginning at zero for a newly introduced key.
For key 3 and sequence 7:

```text
nonce = (3 << 64) | 7 = 55340232221128654855
32-byte integer:
0000000000000000000000000000000000000000000000030000000000000007
```

Shifting this integer right by 64 recovers 3, and masking its low 64 bits recovers 7.
Assume that sequence is currently expected.
The source increments sequences monotonically, but this note does not derive rollback behaviour from nonce packing alone. [ERC-4337, Semi-abstracted Nonce Support][4337]

For ERC-7579, choose the supported mode consisting of 32 zero bytes.
Its first byte selects a single `CALL`, and its second selects execution that reverts on failure.
The remaining 30 bytes comprise four unused bytes, four mode-selector bytes, and 22 payload bytes.
For target `0x2222222222222222222222222222222222222222`, zero value, and call data `12345678`, the complete inner `executionCalldata` is these lines concatenated:

```text
2222222222222222222222222222222222222222
0000000000000000000000000000000000000000000000000000000000000000
12345678
```

That is `20+32+4=56` bytes, using the proposal's packed single-call encoding.
It is not the outer ABI encoding of `execute(mode,executionCalldata)` or a complete UserOperation. [ERC-7579, Execution Behavior][7579]

Now assume deployed account `A`, trusted EntryPoint `E`, validator `V` installed only as type 1, funded paymaster `P`, and sufficient gas.
Assume `A` supports that mode, uses the expected nonce, and its actual signature/authorization checks succeed.
This trace assigns validation results, rather than computing a signature:

1. `E` calls `A.validateUserOp` and receives validationData zero after the account's checks.
2. `P.validatePaymasterUserOp` also returns validationData zero and context `beef`.
3. In execution, `E` calls `A` with the requested `execute` encoding, and `A` calls the chosen target.
4. Assume that target reverts, so the selected revert-on-failure mode makes account execution revert.
5. If enclosing EntryPoint processing continues, `P.postOp` receives the nonempty context with `opReverted`, and sponsorship still covers gas.

This is not a claim that every possible post-operation failure leaves the whole bundle successful.
It separates a successful validation result from a failed execution result.

## Module roles are authority boundaries

ERC-7579 assigns validator type 1, executor type 2, fallback type 3, and hook type 4.
One module MAY implement multiple types, but the account MUST distinguish installed types.
`execute` MUST enforce adequate authorization, and `executeFromExecutor` MUST enforce executor-module authorization.
Installing `V` only as a validator therefore supplies no automatic executor permission.
Accounts MUST also authorize module installation and removal, and reject unsupported execution modes. [ERC-7579, Execution Behavior and Module configurations][7579]

Validator selection encoding is implementation-defined.
If selection data shares a field passed to a validator, the account MUST sanitize the affected values first.
The account SHOULD return the validator's result.
Hooks are optional, but accounts adopting that extension MUST invoke its pre/post checks around `execute` and `executeFromExecutor` calls or batches.
None of those interfaces proves that arbitrary installed modules are trustworthy.

ERC-6900 defines a different module system, including execution manifests and validation applicability to selectors.
Its account MUST check that a selected validator applies to the intended function, including checking or rejecting self-calls through `execute`/`executeBatch`.
Its validation hooks precede the validator, and resulting time bounds MUST be their intersection.
Its required account interfaces include `IAccountExecute`, which ERC-7579 makes optional.
These are contrasts between proposals, not interchangeable module permissions. [ERC-6900, Interfaces and Validation Call Flow][6900]

## Hashing and EIP-7702 do not merge these decisions

ERC-4337 requires signatures to depend on chain ID and EntryPoint address and specifies an EIP-712 UserOperation hash.
The account signature is excluded, and its optional paymaster-signature suffix also does not affect that hash.
Use the [signed-data note](Signed-data-preimages-and-domains.md) before constructing a version-specific preimage.

EIP-7702 instead authorizes a delegation address with an authority account nonce.
That protocol nonce is distinct from EntryPoint's key/sequence nonce.
The authorization's signed fields do not by themselves authorize a UserOperation's application call or install a module.
Successful delegation changes precede transaction execution and survive its failure. [EIP-7702, Behavior][7702]

When `eip7702Auth` is supplied on a network supporting EIP-7702, ERC-4337 requires a valid sender-signed authorization tuple alongside the UserOperation.
The bundler then includes the required tuples in a set-code transaction.
Its special `initCode` prefix is `7702` followed by 18 zero bytes.
For an EIP-7702-delegated account using that prefix, hash calculation replaces the first 20 bytes with the delegate address.
This path does not call a factory.
The source warns that omitting this form can leave the current delegation uncommitted by the hash. [ERC-4337, Support for EIP-7702 authorizations][4337]

## Review limits and unresolved source text

ERC-4337's required-functionality passage permits skipping a failed `validateUserOp` and possibly reverting, while its Error codes section requires reverting on validation failures.
This note preserves that tension and uses successful validation in its trace.
A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
A standalone Python calculation checked nonce packing, mode width, and the 56-byte inner call.
No account, EntryPoint, paymaster, signature, EVM trace, EthLib binding, or Lean proof was executed.
Full bundler rules, aggregation, hash construction, and post-operation failure recovery require separate version-specific work.

[4337]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4337.md#specification
[7579]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7579.md#specification
[6900]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6900.md#validation-call-flow
[7702]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md#behavior
