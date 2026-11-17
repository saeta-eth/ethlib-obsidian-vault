---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
tags:
  - ethlib
  - external-sources
  - permit
  - signatures
---

# Permit nonces and allowance authority

A permit authorizes an approval change, rather than the transfer that may later use that approval.
Identify its signed fields, nonce key, deadline, and submitting caller separately.
ERC-2612's owner nonce and ERC-4494's NFT nonce do not describe the same state transition.
The [signed-data note](Signed-data-preimages-and-domains.md) supplies the EIP-712 encoding prerequisite.

## Sources and scope

| Source | Pin, status, and selected reading |
| --- | --- |
| [ERC-20][20] | ERCs commit above, Final: Methods notes, `approve`, `allowance`, `transferFrom`, and Approval/Transfer events. |
| [ERC-2612][2612] | Same ERCs pin, Final: complete Specification, Rationale, Backwards Compatibility, and Security Considerations. |
| [ERC-4494][4494] | Same ERCs pin, Stagnant: complete Specification, Rationale, Backwards Compatibility, Test Cases, and Security Considerations. |
| [ERC-721][721] | Same ERCs pin, Final: approval/transfer interface passages as ERC-4494's prerequisite. |
| [EIP-712][712] | EIPs commit above, Final: typed encoding, domain separator, and replay/front-running considerations. |

These are pinned interface rules and historical proposal text, not a wallet-support survey or deployed-token audit.
ERC-4494's Stagnant status neither makes its proposal an implemented feature nor removes its value as a comparison.

## Compare the authority and nonce

| Question | ERC-2612 | ERC-4494 |
| --- | --- | --- |
| What changes? | `allowance[owner][spender]` becomes `value`, replacing the old amount. | The spender becomes the approved address for one `tokenId`. |
| Where is the nonce? | `nonces(owner)`. | `nonces(tokenId)`. |
| Required nonce update? | Successful permit increments the owner's nonce by one. | Any transfer of that NFT increments its nonce. |
| Who authorizes? | The explicitly named nonzero owner, through its valid signature. | The current nonzero owner of that NFT, through its valid signature. |
| Who submits? | Any address. | Any address. |
| Deadline boundary? | Current blocktime must be at most deadline. | Same comparison. |

Neither signature binds `msg.sender` as the submitter.
Submitting a permit therefore does not make a relayer the approved spender unless the signed spender field names it.
The ERC-4494 approval is per NFT, rather than the collection-wide operator authority of ERC-721's `setApprovalForAll`.
ERC-721 resets the NFT's approved address on transfer.

For ERC-2612, a successful call sets the allowance, increments the nonce, and emits `Approval` if and only if the deadline, nonzero owner, matching pre-update nonce, and valid owner signature conditions hold.
If any fails, its Specification requires a revert.
Balances and a transfer recipient are not signed fields or listed permit-success conditions.
ERC-20 still governs a later `transferFrom`, including its separate authorization and balance behaviour.
Callers MUST handle `false` from ERC-20 methods returning a success Boolean.

ERC-4494 conditions likewise gate approval and its Approval event.
Its text does not repeat ERC-2612's blanket invalid-permit revert requirement, so this note does not import that requirement.
Its transfer-triggered nonce rule also does not state the ERC-2612 permit-success increment.
Do not assume submission consumes the token nonce without resolving the applicable implementation or specification rule.

## Exact ERC-2612 bytes for public example values

Use owner `0x1111111111111111111111111111111111111111` and spender `0x2222222222222222222222222222222222222222`.
Let value be 100 token units, nonce 7, and deadline 1000.
The exact ASCII type string is:

```text
Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)
```

ERC-2612 hashes that string, then ABI-encodes its hash followed by the five fields in that order.
The integer fields use `uint256`, and addresses occupy left-zero-padded 32-byte words.
The complete 192-byte struct-hash input is these six words, concatenated without separators:

```text
6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9
0000000000000000000000001111111111111111111111111111111111111111
0000000000000000000000002222222222222222222222222222222222222222
0000000000000000000000000000000000000000000000000000000000000064
0000000000000000000000000000000000000000000000000000000000000007
00000000000000000000000000000000000000000000000000000000000003e8
```

Their Keccak-256 hash is:

```text
2f5d4721ea950ebb6944ab70ec7a8402f75183adaf4104f6f0b26f25baf181b4
```

For byte arithmetic only, assume a 32-byte separator `D` consisting of repeated `aa` bytes.
This is a synthetic separator input, not a derived domain schema or a value queried from a contract.
The complete 66-byte EIP-712 signing preimage `1901 || D || structHash` is:

```text
1901aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa2f5d4721ea950ebb6944ab70ec7a8402f75183adaf4104f6f0b26f25baf181b4
```

Its Keccak-256 digest is `e1cadfe1ec407478c8cf6008515977cd3971146d2445c15710c4eb602da27973`.
These are source-derived synthetic calculations, not published token vectors, signatures, or recovered-owner evidence.

Both permit proposals require an EIP-712 domain separator and say it should be unique to the contract and chain.
Their displayed name/version/chainId/verifyingContract construction is a common choice, not a universally mandated field list.
EIP-712 supplies domain-separated encoding without supplying application replay protection itself.
Nonce and deadline checks remain application rules.

## Trace approval without inventing a signature

Consider the token at `0x3333333333333333333333333333333333333333` and relayer `0x4444444444444444444444444444444444444444`.
Assume the token follows ERC-2612, the owner's current nonce is 7, its allowance to the spender is 9, and its balance is 500.
Condition the following case on a valid owner signature for the example fields under that token's applicable domain.
No such signature or real domain is constructed here.

| Case | Required consequence |
| --- | --- |
| Relayer submits at blocktime 1000 with the matching nonce and valid signature. | Allowance becomes 100, nonce becomes 8, and `Approval(owner, spender, 100)` emits. |
| The same signed permit is submitted again after success. | Its signed nonce 7 no longer matches 8, so the call reverts. |
| From the original snapshot, submission occurs at blocktime 1001. | The expired deadline makes the call revert. |

The successful permit leaves the balance at 500 and does not transfer tokens.
Its deadline limits when the permit may be submitted, rather than making the resulting allowance expire at blocktime 1001.
Two ERC-2612 permits for the same owner and nonce compete even when they name different spenders.

For a separate ERC-4494 case, let token 42 have nonce 3.
Its required transfer increment invalidates the old nonce on a later transfer.
Even if a subsequent transfer returns the NFT to the original signer, that old authorization must still match the now-advanced token nonce.
This differs from inferring that an owner can consume all outstanding NFT permits by submitting one permit.

## Relayers, races, and source limits

A relayer may submit or withhold a signed permit, and another party may submit it first.
Neither standard defines an obligation to relay it or a universal relayer fee.
A short deadline bounds the submission window but does not force completion.
Front-running the same ERC-2612 permit can leave the intended allowance set while making a later identical permit call revert.
A helper combining permit and transfer therefore needs an explicit policy for an already-consumed nonce and still must validate the transfer it performs.
The signed fields do not name a transfer recipient, so the helper must define who can choose one.
ERC-2612 also retains ERC-20's allowance-replacement race.

ERC-4494's typed JSON example incorrectly places `value` in its message where its function and type-hash formula use `tokenId`.
Its copied security discussion also mentions self-submission invalidating prior permits without specifying a corresponding nonce-on-permit rule.
These passages remain source questions, not permission to mix incompatible fields or nonce behaviour.
Cached domains need separate chain-split analysis, as both proposals warn.

The selected primary passages were read at the recorded pins.
PyCryptodome Keccak-256 calculated the synthetic type hash, six-word struct hash, and 66-byte digest above.
The source passages and explanation received a separate review, including decoding the displayed word layout.
Independent Keccak and integer calculations reproduced the hashes, input lengths, and conditional nonce/deadline trace.
No signature, recovery, token execution, live-contract query, Lean build, proof replay, implementation-coverage review, or contract-wallet compatibility audit was performed.

[20]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-20.md
[2612]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2612.md#specification
[4494]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4494.md#specification
[721]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-721.md#specification
[712]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-712.md#specification
