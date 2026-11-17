---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: A15
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
tags:
  - ethlib
  - external-sources
  - governance
  - historical-votes
---

# Governance clocks, vote history, and DAO metadata

A historical voting weight needs a timepoint in the voting contract's clock domain.
A DAO metadata resource can describe governance, but does not supply that weight or authorize execution.
Keep the clock, delegated weight, and published description separate before connecting a voting token to a consumer.

## Sources and scope

[ERC-5805][5805] defines voting with delegation, [ERC-6372][6372] defines a contract clock, and [ERC-4824][4824] defines DAO metadata interfaces.
All three have status Review at ERCs commit `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.
Reading covered their complete texts, including methods, expected properties, examples, rationale, compatibility, and security sections.
Linked assets, implementations, and remote metadata were not examined.
No implementation or theorem-coverage claim is made at the recorded EthLib revision.
A specific clock/lookup consumer is not assumed.

## The clock determines the meaning of a timepoint

An ERC-6372 contract MUST implement `clock()` returning `uint48` and `CLOCK_MODE()` returning a string.
The clock MUST be non-decreasing; equal readings are permitted.
The descriptor MUST use URL-query-string form, specifically `application/x-www-form-urlencoded`, decodable with JavaScript `URLSearchParams`.
It is a descriptor rather than a complete resource URI. [Methods][clock-methods]

For the `NUMBER` opcode's block number, the descriptor MUST be `mode=blocknumber&from=default`.
Another block-number source MUST identify its chain with `mode=blocknumber&from=<CAIP-2-ID>`; timestamp mode MUST return `mode=timestamp`.
For another mode, the encoded `mode` field SHOULD use a unique identifier.
An integer alone does not identify its unit or clock source.
Neither this interface nor a non-decreasing clock establishes chain finality or a reorganization policy.

ERC-5805 contracts SHOULD implement ERC-6372.
Without it, they MUST use the default block-number clock.
Thus a consumer must not interpret a missing clock interface as permission to choose timestamps. [ERC-5805 clock methods][vote-clock]

The largest `uint48` value is `2^48-1 = 281474976710655`.
For compatibility, `getPastVotes(address,uint256)` accepts a wider timepoint argument.
A value fitting `uint256` is not necessarily a past timepoint in a `uint48` clock.
The following cases assume the default block-number mode and current clock 100:

| Lookup timepoint | Relation to current clock | ERC-5805 recommendation |
| --- | --- | --- |
| 99 | Strictly past. | SHOULD NOT revert. |
| 100 | Current, not past. | SHOULD revert. |
| 101 | Future. | SHOULD revert. |
| `2^48 = 281474976710656` | Above every `uint48` clock reading. | SHOULD revert. |

The timepoint MUST match the operating mode, but these lookup/rejection clauses are SHOULD and SHOULD NOT, not MUST guarantees.
The Methods section also describes immutable past checkpoints with SHOULD; Expected properties separately states that `getPastVotes(a,t)` MUST be constant after `t < clock`.
The rationale identifies values at least `2^48` as future lookups; narrowing or wrapping them first would change the question. [Historical lookup][past]

## Delegation changes voting weight without moving assets

ERC-5805 distinguishes an owner's balance from weight delegated to an account.
`delegate(address)` changes the caller's delegate; `getVotes(address)` reads current delegated weight.
`getPastVotes` reads historical weight, not historical ownership balances.
Delegation can target the owner, another account, or address zero; undelegated assets cannot express voting power.
The source says zero-address votes SHOULD not be counted or checkpointed. [Specification and methods][5805]

Assume a fixed chain history and a token with one vote per asset unit.
Let A hold 8 units, B hold 3, and C hold 5, with no transfers, minting, or burning during this trace.
At the end of block 99, A delegates to B, B delegates to itself, and C delegates to zero.
In block 100, A changes its delegate to C; later in the same block, C delegates to itself.
Assume the checkpoint implementation follows the source's weighted-sum expectations.

| State | A/B/C asset balances | B's current weight | C's current weight |
| --- | --- | --- | --- |
| End of clock 99. | 8 / 3 / 5 | 11 | 0 |
| Clock 100, after A delegates to C. | 8 / 3 / 5 | 3 | 8 |
| Clock 100, after C self-delegates. | 8 / 3 / 5 | 3 | 13 |

The asset total remains 16 throughout.
Counted voting weight grows from 11 to 16 only because C activates its own five units, not because assets are created.
The same-clock intermediate weight 8 is a current observation, not a separately guaranteed historical timepoint.
Once the clock reaches 101, this assumed history gives B/C weights 11/0 at timepoint 99 and 3/13 at timepoint 100.
The source describes past weight using the delegations in place when the clock overtook the requested timepoint. [Expected properties][properties]

This does not execute a governor or establish a complete double-voting prevention argument.
The consumer still needs a chosen proposal snapshot, ballot rules, and execution policy.
ERC-5805's rationale explains snapshot-based voting, but its interface does not choose those application rules.

The signed-delegation text contains unresolved discrepancies.
ERC-5805's prose compares the current timestamp with `expiry` using `timestamp <= expiry`, while its displayed code reverses that inequality.
Its typed-message example also prints `primaryType: "Permit"` beside `Delegation` fields.
This note preserves those discrepancies and supplies no signed-delegation vector or recovery claim. [delegateBySig][signed]

## DAO metadata describes rather than executes governance

ERC-4824 requires its `IERC4824` interface, including `daoURI()` pointing to DAO JSON-LD metadata.
A DAO MAY expose it directly or use an external registration contract; that registration MUST store the DAO's primary address.
The DAO SHOULD provide update access control, but the security section leaves URI-setting rules and data preparation unspecified.
It asks implementers to align updates with governance and maintain freshness. [Specification and security][4824]

For a constructed `daoURI` of `https://example.invalid/dao/dao.json`, a metadata layout using the printed top-level fields is:

```json
{
  "@context": "http://www.daostar.org/schemas",
  "type": "DAO",
  "name": "Clock demonstration",
  "membersURI": "https://example.invalid/dao/members.json",
  "proposalsURI": "https://example.invalid/dao/proposals.json",
  "activityLogURI": "https://example.invalid/dao/activity.json",
  "governanceURI": "https://example.invalid/dao/governance.md",
  "contractsURI": "https://example.invalid/dao/contracts.json"
}
```

This is illustrative JSON, not a fetched resource or a validated JSON-LD document.
The subsidiary member, proposal, activity, and contract resources have SHOULD schema requirements; `governanceURI` SHOULD point to a flatfile, normatively Markdown.
Subsidiary JSON MAY instead be embedded, with the URI suffix removed and the relevant schema followed as MUST requirements.
Both forms SHOULD NOT coexist for one field; if they do, the unsuffixed field MUST take precedence. [URI fields][uri-fields]

Member listings do not determine delegated voting weight, and proposal `status` is descriptive text rather than a standardized executable state machine.
Metadata can describe proposed calls without granting their execution authority.
ERC-4824's indexing preferences are also recommendations: registration-factory publication SHOULD take precedence over direct publication, and the latest registration SHOULD take precedence among registrations.
Those preferences do not establish that a resource is authentic, fresh, or consistent with a particular voting contract.

## Consumer conditions and checks

A consumer's inputs include the voting contract, chosen clock mode, snapshot convention, metadata publisher, and trust policy.
The open questions are consumer-specific: how to handle unsupported modes, nonconforming lookups, same-clock updates, reorganizations, and conflicting registrations.
No DAO execution model is selected here.

Independent calculations reproduced the displayed example values.
Python integer calculations checked the `uint48` boundary and conditional delegation sums/history; JSON parsing checked only the constructed document's syntax.
No token, governor, signature, clock query, remote resource, JSON-LD processor, EthLib binding, Lean build, or proof replay ran.

[5805]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md
[6372]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6372.md
[4824]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4824.md
[clock-methods]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6372.md#methods
[vote-clock]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md#erc-6372-clock-and-clock_mode
[past]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md#getpastvotes
[properties]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md#expected-properties
[signed]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md#delegatebysig
[uri-fields]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4824.md#uri-fields
