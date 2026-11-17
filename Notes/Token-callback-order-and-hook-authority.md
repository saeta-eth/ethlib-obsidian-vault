---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A09
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - token-hooks
  - reentrancy
---

# Token callback order and hook authority

A token callback exposes a particular point in a transfer's state changes.
Hook discovery also decides which external contract can observe or reject that transfer.
Registration authority, token-spending authority, and callback acceptance therefore need separate checks.
The [receiver-interface note](Token-receiver-interfaces-and-callbacks.md) and [allowance note](Permit-nonces-and-allowance-authority.md) supply the prerequisites.

## Sources and versions

All five ERCs have status Final at the recorded ERCs commit.
That status does not establish deployment or uniform behaviour among tokens bearing their names.

| Source | Reading used |
| --- | --- |
| [ERC-223][223] | Transfer and receiver specifications, plus selected embedded reference code, including contract detection and transfer functions. |
| [ERC-777][777] | Operators, Sending Tokens, sender/recipient hook sections, selected Hooks rationale, and ERC-20 compatibility requirements. |
| [ERC-1363][1363] | Complete Specification, Rationale, Backwards Compatibility, and Security Considerations. |
| [ERC-1820][1820] | Supersession notice, interface-name rules, lookup/registration/manager prose, and corresponding embedded registry functions. |
| [ERC-820][820] | Header and supersession notice, as historical context only. |

At this pin, ERC-820 is a full document, not a moved-file redirect.
Its notice says ERC-1820 supersedes it and MUST be used instead, identifying an ERC-165 compatibility fix.
The lookup examples below consequently use ERC-1820.
This comparison makes no claim about EthLib token implementation or proof coverage.

## Compare the callback boundaries

| Interface | Discovery and order | Acceptance or rejection |
| --- | --- | --- |
| ERC-223 `tokenReceived` | Token calls the receiving contract after its other operations, according to the transfer prose. | Missing handling must revert; return-value wording has the qualifications below. |
| ERC-777 `tokensToSend` | ERC-1820 lookup for the holder, before state updates. | Registered hook can reject by reverting. |
| ERC-777 `tokensReceived` | ERC-1820 lookup for the recipient, after state updates. | Registered hook can reject by reverting; no acceptance magic is returned. |
| ERC-1363 `onTransferReceived` | Token calls the recipient after `transferAndCall` or `transferFromAndCall` updates the transfer. | Callback MAY reject by throwing; wrong magic MUST revert the transaction. |
| ERC-1363 `onApprovalReceived` | Token calls the spender after `approveAndCall` updates approval. | Callback MAY reject by throwing; wrong magic MUST revert the transaction. |

ERC-1363 adds named operations rather than changing every ordinary ERC-20 transfer into a callback operation.
Its approval callback concerns spending permission, not proof that a payment happened.
The token contract is callback `msg.sender`; `operator` identifies the transfer caller, while `from` identifies the token owner.
The approval hook instead receives the approving owner. [ERC-1363, Specification][1363]

ERC-777 differs: when its token also implements ERC-20, registered sender/recipient hooks MUST run even for `transfer` and `transferFrom`.
For a contract recipient without a registered recipient hook, ERC-777 send/mint MUST revert.
Its ERC-20 compatibility path SHOULD continue instead, with an explicit MAY-revert qualification to avoid locked tokens.
Those rules do not add hooks to an arbitrary ERC-20 implementation. [ERC-777, ERC-20 compatibility][777]

## Registry control does not grant operator permission

ERC-1820 lets anyone query an implementer, but only an address's manager can change its registration.
Each address initially manages itself, and the manager can appoint another manager or reset management to the address itself.
The registered implementation can be another contract, including for an externally owned holder.
ERC-777 queries the holder for the sender hook and the recipient for the receiving hook.
The hook need not reside at the address whose balance changes. [ERC-1820, registration and Manager][1820] [ERC-777, hook interfaces][777]

Registering a helper hook is distinct from authorizing it as an ERC-777 operator.
Operator permission allows sending/burning on a holder's behalf.
A holder is always its own operator, while other operator authority follows the token's separate rules.
A helper's callback cannot assume that its own `msg.sender` is the holder: it receives the call from the token contract.
ERC-777 uses one registered implementation per address and hook interface across its tokens.
The hook must therefore expect calls from different token contracts, whose identities are separate from the holder and registry manager.

The registry's prose requires a separate implementer to acknowledge the interface for the represented address with `ERC1820_ACCEPT_MAGIC`.
Its embedded `setInterfaceImplementer` code instead skips that handshake when the implementer equals the calling manager, or is zero.
These exceptions are not the same when a manager differs from the represented address.
This note does not silently equate them.
A third-party implementation distinct from both address and manager satisfies both passages when it returns the required magic. [ERC-1820, Set An Interface For An Address and embedded registry][1820]

## Calculate a complete lookup key and callback selectors

For recipient-hook discovery, hash the exact ASCII interface name, not a function prototype or an ABI-encoded string:

```text
text: ERC777TokensRecipient
ASCII hex: 455243373737546f6b656e73526563697069656e74
Keccak-256:
b281fc8c12954d22544db45de3159a39272895b169a852b314f9cc762e44c53b
```

The token passes that entire 32-byte hash to `getInterfaceImplementer(recipient,hash)`.
This is not an ERC-165 XOR identifier or the first four bytes of a function hash.
The source publishes the same name hash. [ERC-777, ERC777TokensRecipient][777] [ERC-1820, Interface Name][1820]

ERC-1363's acceptance selectors use canonical function signatures instead:

```text
first4(Keccak256("onTransferReceived(address,address,uint256,bytes)"))
    = 88a7ca5c
first4(Keccak256("onApprovalReceived(address,uint256,bytes)"))
    = 7b04a2d0
```

These values acknowledge particular callbacks.
The registry's separate `ERC1820_ACCEPT_MAGIC` acknowledges implementation registration, not token receipt or spending authority.

## Conditional ERC-777 send with a nested observation

Assume conforming token `T`, granularity 1, holder `F`, recipient contract `R`, and distinct regular address `D`.
Initial balances are `F=100`, `R=10`, `D=0`.
Assume stable registrations: `F` has sender hook `H`, `R` registers itself as recipient hook, `R` has no sender hook, and `D` has no hooks.
Both addresses manage their own registrations.
All addresses are nonzero, gas is sufficient, and all hook checks initially accept.

`F` calls `T.send(R,30,0x1234)`.
The hook arguments identify operator `F`, from `F`, to `R`, amount 30, data `1234`, and empty operatorData.
The following observations follow ERC-777's required ordering:

1. `T` calls `H.tokensToSend` before the debit, so a balance query observes `F=100`, `R=10`.
2. `T` updates the balances to `F=70`, `R=40` and invokes `R.tokensReceived` with those outer-send arguments.
3. Assume `R`'s hook calls `T.send(D,5,emptyData)` before returning.
4. This nested call acts for `R` itself, so operator and holder are `R`, not `F` or helper `H`.
5. If the nested call and hooks complete, balances become `F=70`, `R=35`, `D=5`.

Thus a recipient hook can observe its credited balance and initiate another movement before the outer call returns.
The proposal expressly permits chained movements, applying the send rules to each.
If `R` instead reverts after that nested call, the outer send MUST revert, including the nested balance changes.
The final balances return to the initial values if the enclosing caller handles that failed token call and continues.
This conditional arithmetic is not a deployed-token trace or a universal reentrancy proof. [ERC-777, Sending Tokens][777]

## Preserve ERC-223's textual limits

The pinned ERC-223 receiver declaration returns `bytes4` and requires `0x8943ec02`, while its descriptive paragraph also says it returns nothing.
The text permits fallback handling without that magic, and the embedded token calls the receiver without checking the returned value.
Do not import ERC-1363's explicit wrong-magic rejection rule into this unresolved boundary.
Likewise, the transfer prose says the hook follows all other operations, while the embedded reference emits its Transfer event after the hook.
The ordering comparison above describes the prose and does not claim that reference matches it completely.

ERC-223's embedded contract-detection helper also warns that code-size checks have false negatives during construction.
A zero code size therefore does not establish every recipient's future behaviour.
These source limitations reinforce the need to pin the exact standard and function being modelled.

## Checks and limits

Independent calculations reproduced the displayed example values.
A standalone PyCryptodome calculation reproduced lookup hashes and callback selectors, and integer arithmetic traced the stated balances.
No registry query, hook, token contract, EthLib binding, EVM, or Lean proof was executed.

[223]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-223.md#specification
[777]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-777.md
[820]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-820.md
[1363]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1363.md#specification
[1820]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1820.md#specification
