---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 6
catalogue_family: A07
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - tokenized-vaults
---

# ERC-4626 rounding and vault limits

A conversion estimate, a preview, a limit, and a successful vault operation establish different facts.
Treating them as interchangeable can reverse a rounding inequality or mistake an exchange-rate estimate for permission to transact.
This note separates those contracts and works one small integer model.

## Sources and units

All ERC links pin revision `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.
The three documents have status Final at that revision.

| Source | Reading and scope |
| --- | --- |
| [ERC-4626: Tokenized Vaults][4626] | Specification definitions and all Methods, plus complete Security Considerations. Main interface and rounding rules. |
| [ERC-7540: Asynchronous ERC-4626 Tokenized Vaults][7540] | Definitions, Request Flows, and Request Lifecycle. A bounded contrast with synchronous previews. |
| [ERC-7575: Multi-Asset ERC-4626 Vaults][7575] | Definitions, Methods including `share`, and Multi-Asset Vaults. A bounded contrast with the share-token location. |

An **asset** is the underlying ERC-20 token; a **share** is the vault token representing a claim on its holdings.
Both amounts use their respective token's smallest units.
Equal integer amounts do not imply equal economic value or equal decimal scaling.
ERC-4626 recommends matching the underlying token's decimals where possible, without making that equality mandatory.

`totalAssets` reports the amount managed by the vault, SHOULD include compounding yield, and MUST include fees charged against assets in the vault.
It MUST NOT revert.
The term managed does not specify that every implementation counts every unsolicited transfer identically.
Identify the implementation's accounting before using its token balance as `totalAssets`.

## Four different kinds of answer

`convertToShares(assets)` and `convertToAssets(shares)` describe an ideal exchange estimate.
They MUST exclude fees, be independent of the caller, omit slippage and actual-exchange conditions, and round down toward zero.
Their only permitted revert is integer overflow from an unreasonably large input.
They need not equal a particular user's executable price.
The Security Considerations even permit an inexact conversion based on a time-weighted average.
Thus ERC-4626 does not require one `totalAssets / totalSupply` formula for every implementation.

The four previews instead approximate actual operations under current on-chain conditions.
The source's same-transaction comparisons are:

| Preview | Required comparison with the corresponding successful operation | Fees included |
| --- | --- | --- |
| `previewDeposit(a)` | Preview shares are no more than `deposit(a, receiver)` returns. | Deposit fees. |
| `previewMint(s)` | Preview assets are no fewer than `mint(s, receiver)` consumes/returns. | Deposit fees. |
| `previewWithdraw(a)` | Preview shares are no fewer than `withdraw(a, receiver, owner)` burns/returns. | Withdrawal fees. |
| `previewRedeem(s)` | Preview assets are no more than `redeem(s, receiver, owner)` sends/returns. | Withdrawal fees. |

Each preview MUST be as close as possible to the operation's exact amount while respecting its inequality.
A preview MUST ignore vault-specific user/global limits and cannot revert because of those limits.
It MAY revert for other conditions that would also make the corresponding operation revert.
An off-chain quote across intervening state changes does not inherit a same-transaction guarantee.
The numerical examples below hold the relevant state and conditions fixed.

`maxDeposit(receiver)` and `maxWithdraw(owner)` return asset amounts; `maxMint(receiver)` and `maxRedeem(owner)` return share amounts.
Their reported maximum MUST NOT exceed the actual amount accepted under the applicable limits, and may underestimate if necessary.
They MUST account for user/global restrictions, return zero when the operation is disabled, and MUST NOT revert.
For deposit/mint, the calculation assumes infinite user assets and MUST NOT depend on the user's asset balance.
If there is no deposit/mint limit, the corresponding maximum MUST be `2^256 - 1`.
Deposit/mint capacity does not establish the caller's asset balance or allowance.

The mutable calls perform transfers and minting or burning.
`deposit` fixes assets supplied; `mint` fixes shares received.
`withdraw` fixes assets sent to the receiver; `redeem` fixes shares burned from the owner.
Each operation MUST revert if the whole specified amount cannot be processed, and MUST emit its corresponding Deposit or Withdraw event.
A preview's deliberate omission of limits does not override these requirements.

## A proportional model with ten assets and six shares

Assume positive managed assets `A = 10`, share supply `S = 6`, zero fees, no limits, sufficient balances/allowances, and proportional pricing.
The proportional formula is an assumption for this example, not a universal ERC requirement.

The Security Considerations recommend favouring the vault when rounding: down for amounts given to a user, up for amounts the user must provide.
Together with the preview inequalities, that gives this conventional model:

| Independent operation | Exact rational amount | Rounded amount |
| --- | --- | --- |
| Deposit 3 assets | `3 * S / A = 18/10` shares | Floor: 1 share. |
| Mint 2 shares | `2 * A / S = 20/6` assets | Ceiling: 4 assets. |
| Withdraw 3 assets | `3 * S / A = 18/10` shares | Ceiling: 2 shares. |
| Redeem 2 shares | `2 * A / S = 20/6` assets | Floor: 3 assets. |

Every row starts from the same `A = 10, S = 6` snapshot.
Executing them sequentially would change the ratio and require recalculation.
For example, after the deposit alone, this model has `A = 13, S = 7`.

The model's `convertToShares(3)` is 1 and `convertToAssets(2)` is 3 because both conversions round down.
Using `convertToAssets(2)` to price a mint would supply only 3 assets where the model requires 4.
Fees would require additional rules, not merely exchanging floor for ceiling.
The standard requires previews to include their relevant fees but does not give one universal fee formula.

## Limits and rounding are separate checks

Suppose this same snapshot has `maxDeposit(receiver) = 0` because deposits are disabled.
The preview must still model the deposit without that limit, while the actual deposit must revert.
Conversely, a nonzero preview does not promise that a later transaction's state, limits, or price will be unchanged.
ERC-4626's Security Considerations discuss additional methods for users to enforce acceptable outputs or limits.
They also warn that previews can be manipulated through on-chain conditions, so a preview is not automatically a safe price oracle.

A separate conditional example shows why truncation matters near an empty vault.
Start with `A = 1, S = 1`.
Assume a 100-asset donation increases managed assets without minting shares, making `A = 101, S = 1`.
The proportional calculation for a 100-asset deposit is then `floor(100/101) = 0` shares.
An implementation that also accepts that zero-output deposit would transfer assets for no shares.
This arithmetic does not establish that any particular vault counts the donation that way or accepts the call.
It is neither a universal exploit nor a security proof for alternative accounting.

## Extensions change the interpretation

Under ERC-7540's asynchronous deposit flow, `previewDeposit` and `previewMint` MUST revert for all callers and inputs.
Under its asynchronous redemption flow, `previewWithdraw` and `previewRedeem` MUST do so.
A side not implemented asynchronously retains ERC-4626's synchronous pattern.
The lifecycle distinguishes Pending, Claimable, and Claimed.
A Request may become Claimable immediately, but the user must still perform a separate claim.
Existing entry/exit methods claim previously requested amounts.
An integrator cannot substitute the synchronous preview table for this lifecycle.

ERC-7575 externalizes the share-token interface through `share()`.
That function MAY return the vault itself, so an external share contract is not mandatory.
When it is external, entry and exit functions must update that token's receiver/owner balances as specified.
Do not assume every compatible entry point is itself the ERC-20 share contract.
These contrasts do not review either extension's full interface or state machine.

## Arithmetic connection and review limits

At the pinned EthLib commit, [`U256`][u256] is `BitVec 256`; addition, subtraction, and multiplication wrap modulo `2^256`.
A rational formula proved over natural numbers does not automatically survive that representation.
An intermediate product can overflow even when the final quotient fits.
An arithmetic contribution needs explicit bounds or a suitable wider calculation, plus the chosen rounding rule.
The ERC's limited overflow exception for conversions is not permission to silently substitute a wrapped ratio.
Empty-vault initialization, fee accounting, and division-by-zero handling also need implementation-specific definitions; the positive-state example does not determine them.

The cited source sections and the `U256` definition were read.
The arithmetic was derived by hand from the stated model, not taken from EthLib output or a deployed contract.
The sources and explanation received a separate review.
Independent exact-rational calculations reproduced the four rounded amounts, the deposit post-state, and the conditional zero-share example.
No Lean build, proof replay, EVM execution, live vault query, full extension review, or economic-security audit was performed.
This note supplies background for an interface or arithmetic contribution without proposing a vault module or making coverage claims.

[4626]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4626.md
[7540]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7540.md#request-flows
[7575]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7575.md#share
[u256]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Base/U256.lean#L28
