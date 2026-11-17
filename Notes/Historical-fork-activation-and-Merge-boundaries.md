---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 3
delivery_batch: "follow-up-11"
catalogue_family: E11
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellow_paper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - historical-forks
  - merge
---

# Historical fork activation and Merge boundaries

The block that reaches terminal total difficulty is still a proof-of-work block.
Its first canonical proof-of-stake child is a different boundary, with different header rules and an external fork-choice input.
Keep those conditions separate from historical block-number forks, chain-ID signing domains and later timestamp schedules.

## Sources and reading scope

[EIP-2][2], [EIP-155][155], [EIP-161][161], [EIP-3675][3675] and [EIP-4399][4399] are Final at EIPs commit `8194528a2ee9a4893a398a5871c470ab0ac3b09a`.
Reading covered their selected activation, signature, account-clearing, Merge configuration, block-structure and validity rules.
The Shanghai [Yellow Paper][yp] at `efc5f9a1f356cba376c978eedb63cb0363c2aa85` supplies its transition-to-PoS and blocktree discussion, plus selected header definitions.
The linked beacon-chain algorithms, full historical difficulty formulas and Ethash internals were not audited here.
These version identities do not establish a live network's configuration or validate its history.

## Activation conditions qualify the rule

EIP-2 applies its Homestead changes when the configured fork block is reached; its table gives mainnet block 1,150,000.
Among its changes, transaction signatures with `s > n/2` become invalid while ECRECOVER retains high-s acceptance.
That signature restriction is separate from a chain-ID signing domain, as the [curve note](Curve-validation-ECDSA-and-recovery-boundaries.md) explains.

EIP-155 gives mainnet block 2,675,000 and makes its chain-protected signing form conditional on that activation and an available chain ID.
EIP-161 names the same historical mainnet fork block for its account-clearing changes.
It defines empty accounts by no code, zero nonce and zero balance, with touched empty accounts deleted at transaction end.
Its addendum requires those deletions to revert when state reverts.
Applying either change solely because a document is Final would omit the chain and activation conditions.

Paris instead uses terminal total difficulty, not a predetermined transition block number.
Post-Paris execution upgrades can use timestamps, as the Yellow Paper's Post-Paris Updates discussion explains.
Each upgrade's own source supplies its comparison and configuration; a timestamp is not interchangeable with a block number or a consensus slot.

## Legacy signatures survive the chain-protected form

After its activation, EIP-155 says a signer SHOULD hash nine RLP fields instead of the six legacy signing fields:

```text
legacy:    [nonce, gasprice, startgas, to, value, data]
protected: [nonce, gasprice, startgas, to, value, data, chainId, 0, 0]
```

If the protected form is chosen, v MUST be `35 + 2*chainId + parity`.
The six-field form instead keeps `27 + parity`, and EIP-155 explicitly preserves v=27/28 signatures.
Their survival still requires other applicable rules, including Homestead's low-s bound.

For parity 1, the arithmetic is:

| Signing form | v |
| --- | --- |
| Legacy six fields | `27+1 = 28` |
| Protected, chain ID 1 | `35+2*1+1 = 38` |
| Protected, chain ID 2 | `35+2*2+1 = 40` |

Changing chain ID changes the protected signing input as well as v.
Legacy v=28 signs no chain ID, so EIP-155 alone does not distinguish its use on two chains whose other admission conditions permit it.
Two networks reusing the same chain ID also do not gain distinct signing domains from this mechanism alone.
These are domain observations, not executed replay tests.
The [typed-transaction note](Typed-transaction-envelopes-and-signing.md) covers additional envelope formats and complete signing-byte examples.

## A conditional terminal-difficulty trace

EIP-3675 defines a terminal PoW block B by both inequalities:

```text
TD(parent(B)) < TTD
TD(B) >= TTD
```

The Yellow Paper defines PoW total difficulty recursively as parent total difficulty plus the block's difficulty.
Thus equality is sufficient for the second condition, and a block can overshoot the threshold.
A total already above TTD does not make every descendant another terminal PoW block: its parent condition matters too.

For a small arithmetic exercise, choose unit `U = 131072`, configured `TTD = 100*U = 13107200`, and no terminal-hash override.
The scale does not replace checks of the minimum difficulty, the difficulty formula and the Ethash seal.
These numbers are not a historical network configuration.

| Item | Parent total difficulty | Own difficulty | Result |
| --- | --- | --- | --- |
| Parent P | Not needed for this arithmetic. | Not supplied. | `TD(P)=90*U=11796480`. |
| PoW candidate B, child of P | `90*U` | `10*U=1310720` | `TD(B)=100*U`; it meets the terminal inequalities if otherwise a valid PoW block. |
| PoW candidate D, child of B | `100*U` | `U` | Its parent has reached TTD; EIP-3675 says PoW descendants of a terminal PoW block MUST NOT be imported. |
| PoS candidate C, child of B | B satisfies the terminal condition above. | `0` | The transition-parent condition is satisfied conditionally; complete PoS validity and canonical selection are not established. |

B retains PoW validation; C does not become a valid PoS block merely by setting difficulty to zero.
EIP-3675 requires `TRANSITION_BLOCK`, the earliest canonical PoS block, to be a child of a terminal PoW block.
It also permits multiple terminal PoW blocks, so the inequalities alone do not select a unique parent chain.

If configured, `TERMINAL_BLOCK_HASH` and `TERMINAL_BLOCK_NUMBER` constrain the canonical chain as specified by EIP-3675.
When the hash is the all-zero stub, those two parameters MUST NOT take effect.
This trace assumes that stubbed case rather than silently substituting a block-number trigger for TTD.

## Header rules across the boundary

The terminal block remains on the PoW side of this table.
The right column describes EIP-3675 together with EIP-4399, beginning at `TRANSITION_BLOCK`:

| Check or field | Before transition | From transition |
| --- | --- | --- |
| `difficulty` | Validate the applicable PoW difficulty formula. | MUST be zero. |
| `nonce` | Participates in Ethash verification; zero is not categorically excluded. | MUST be eight zero bytes. |
| `mixHash` | PoW mix digest checked through Ethash. | EIP-4399's previous-block post-beacon-state latest RANDAO mix. |
| Ommers | Historical ommer rules apply. | Empty list, with the prescribed empty-list hash. |
| Ethash seal checks | Verify nonce and mix digest under the PoW rules. | Remove those Ethash checks. |
| Block/ommer issuance | Existing reward rules apply. | Remove block and ommer rewards; transaction-fee mechanics remain unchanged. |

The fixed ommer hash is `Keccak256(c0)`:

```text
1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347
```

EIP-3675's minimal table originally sets mixHash to zero and explicitly allows later EIPs to override constants.
EIP-4399 supplies that override: the RANDAO value is mandatory, while renaming the field to `prevRandao` and opcode to `PREVRANDAO` is SHOULD.
It reuses header field 13 rather than appending a randomness field.
The [opcode note](EVM-word-arithmetic-and-shift-boundaries.md) treats the changed opcode meaning separately.

The Merge rules retain other unspecified validity checks and the overall block format; extraData must be at most 32 bytes.
Removing Ethash verification does not remove transaction, execution-state or header validation.
Canonical-head selection also requires the specified PoS fork-choice update, rather than automatically promoting every processed child.
This note does not execute that consensus mechanism.

## Selected EthLib observations and checks

At EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`, [`activeFork`][active] reads configured block/timestamp activations, and mainnet's table records a resolved historical Paris block.
That representation is not the live TTD transition test described above.
[`headerChecks` and `validateHeader`][header] select fixed post-Paris PoW-field checks and run Ethash only before Paris.
[`legacyRecovery`][legacy] preserves 27/28 separately from the chain-protected v form, while its signing helpers choose the corresponding six- or nine-field input.
These are selected definition observations, not claims of transition-proof coverage or full historical replay.

Python integer arithmetic checked the scaled terminal inequalities, threshold countercases and v decoding; PyCryptodome 3.23.0 reproduced the empty-ommer hash.
No PoW seal, PoS block, fork-choice event, transaction signature, replay, EthLib binding, Lean build or proof replay ran.

[2]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2.md
[155]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-155.md#specification
[161]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md
[3675]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3675.md
[4399]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4399.md#specification
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[active]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Base/ChainConfig.lean#L120
[header]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Blocks/Validate.lean#L422
[legacy]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Transactions/Signature.lean#L108
