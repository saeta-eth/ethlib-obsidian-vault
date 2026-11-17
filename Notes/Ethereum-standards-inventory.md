---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: working
tags:
  - ethlib
  - standards-inventory
---

# Ethereum standards inventory

This metadata inventory contains 1232 distinct numbered documents: 607 EIPs and 625 ERCs.
It resolves 365 Moved records in EIPs to the actual ERC document, so a redirect and its destination are not counted twice.
Every committed document matching EIPS/eip-N.md or ERCS/erc-N.md in these snapshots is retained, regardless of category or document status.
Unmerged proposals, documents outside those paths, RFCs, interface repositories, and papers are outside this metadata census.

## Sources and method

| Repository | Exact revision | Number of retained documents |
| --- | --- | --- |
| [ethereum/EIPs](https://github.com/ethereum/EIPs/tree/8194528a2ee9a4893a398a5871c470ab0ac3b09a) | 8194528a2ee9a4893a398a5871c470ab0ac3b09a | 607 |
| [ethereum/ERCs](https://github.com/ethereum/ERCs/tree/dbba0f4479dd817d19d7d7771dea0dcd76c68d17) | dbba0f4479dd817d19d7d7771dea0dcd76c68d17 | 625 |

The generator read document front matter from the GitHub source archives without executing or extracting repository code.
It checked that each declared number matches its filename, retained title/status/type/category/requires metadata, and resolved Moved records.
Official titles are preserved as source metadata.
The source titles and metadata are attributed to the repositories above.
Both snapshots dedicate their material under CC0 1.0: [EIPs licence](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/LICENSE.md) and [ERCs licence](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/LICENSE.md).
The grouping uses category when present and type otherwise; it does not infer a protocol layer from an ERC label.

| Archive at the revision above | SHA-256 |
| --- | --- |
| eips.tar.gz | f0ceba9a7f013cd04edbc15e2ba221837b11c1bb4ebe7c6af1fcdeaf2ddfa4d6 |
| ercs.tar.gz | 8b7132d1de57bea9fc495ca58efed6a3230f4e75d659b72d2d2ecb92b6f95d8f |

Archive names describe the locally downloaded inputs; archive bytes are not copied into the vault.
To refresh, resolve new repository commits, enumerate the same paths, read the same fields, resolve redirects, and compare added/removed/changed records.
Review changed requirements and applicability before updating a source note.

## Counts

| Category, or type when no category is declared | Documents |
| --- | --- |
| Core | 449 |
| ERC | 625 |
| Informational | 24 |
| Interface | 60 |
| Meta | 43 |
| Networking | 31 |

| Document status | Documents |
| --- | --- |
| Draft | 382 |
| Final | 289 |
| Last Call | 44 |
| Living | 3 |
| Review | 73 |
| Stagnant | 396 |
| Withdrawn | 45 |

## Assessment meaning

129 documents are linked to at least one candidate family in the [source catalogue](Ethereum-source-catalogue.md).
The remaining 1103 are Unassessed for note relevance.
A family link is a proposed knowledge connection, not evidence that the full document has been read or that EthLib implements it.
Unassessed does not mean irrelevant, rejected, or lower priority.
Requires lists the source metadata dependencies; it is not a complete cross-specification dependency graph.
A status such as Final, Draft, Stagnant, or Withdrawn is not a note-writing priority or proof of network activation.
Read the linked exact revision before using a requirement.

## Core

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [EIP-2](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2.md) Homestead Hard-fork Changes | Final | None recorded | E11, C06 |
| [EIP-3](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3.md) Addition of CALLDEPTH opcode | Withdrawn | None recorded | Unassessed |
| [EIP-5](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5.md) Gas Usage for `RETURN` and `CALL*` | Final | None recorded | Unassessed |
| [EIP-7](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7.md) DELEGATECALL | Final | None recorded | Unassessed |
| [EIP-86](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-86.md) Abstraction of transaction origin and signature | Stagnant | None recorded | Unassessed |
| [EIP-100](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-100.md) Change difficulty adjustment to target mean block time including uncles | Final | None recorded | Unassessed |
| [EIP-101](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-101.md) Serenity Currency and Crypto Abstraction | Stagnant | None recorded | Unassessed |
| [EIP-140](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-140.md) REVERT instruction | Final | None recorded | S04, E04, A01 |
| [EIP-141](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-141.md) Designated invalid EVM instruction | Final | None recorded | Unassessed |
| [EIP-145](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-145.md) Bitwise shifting instructions in EVM | Final | None recorded | E05 |
| [EIP-150](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-150.md) Gas cost changes for IO-heavy operations | Final | None recorded | E04 |
| [EIP-152](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-152.md) Add BLAKE2 compression function `F` precompile | Final | None recorded | E06, C02 |
| [EIP-155](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-155.md) Simple replay attack protection | Final | None recorded | E01, E11, C06 |
| [EIP-158](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-158.md) State clearing | Final | None recorded | Unassessed |
| [EIP-160](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-160.md) EXP cost increase | Final | None recorded | Unassessed |
| [EIP-161](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md) State trie clearing (invariant-preserving alternative) | Final | None recorded | E04, E09, E11 |
| [EIP-170](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-170.md) Contract code size limit | Final | None recorded | E04 |
| [EIP-196](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-196.md) Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128 | Final | None recorded | E06 |
| [EIP-197](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-197.md) Precompiled contracts for optimal ate pairing check on the elliptic curve alt_bn128 | Final | None recorded | E06 |
| [EIP-198](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-198.md) Big integer modular exponentiation | Final | None recorded | E06 |
| [EIP-210](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-210.md) Blockhash refactoring | Stagnant | None recorded | Unassessed |
| [EIP-211](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-211.md) New opcodes: RETURNDATASIZE and RETURNDATACOPY | Final | None recorded | S04 |
| [EIP-214](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-214.md) New opcode STATICCALL | Final | None recorded | Unassessed |
| [EIP-225](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-225.md) Clique proof-of-authority consensus protocol | Final | None recorded | Unassessed |
| [EIP-615](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-615.md) Subroutines and Static Jumps for the EVM | Stagnant | None recorded | Unassessed |
| [EIP-616](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-616.md) SIMD Operations for the EVM | Stagnant | None recorded | Unassessed |
| [EIP-649](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-649.md) Metropolis Difficulty Bomb Delay and Block Reward Reduction | Final | None recorded | Unassessed |
| [EIP-658](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-658.md) Embedding transaction status code in receipts | Final | 140 | E08 |
| [EIP-663](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-663.md) SWAPN, DUPN and EXCHANGE instructions | Stagnant | 3540, 5450 | Unassessed |
| [EIP-665](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-665.md) Add precompiled contract for Ed25519 signature verification | Stagnant | None recorded | Unassessed |
| [EIP-684](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-684.md) Revert creation in case of collision | Final | None recorded | E04 |
| [EIP-689](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-689.md) Address Collision of Contract Address Causes Exceptional Halt | Stagnant | None recorded | Unassessed |
| [EIP-698](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-698.md) OPCODE 0x46 BLOCKREWARD | Stagnant | None recorded | Unassessed |
| [EIP-858](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-858.md) Reduce block reward and delay difficulty bomb | Stagnant | None recorded | Unassessed |
| [EIP-908](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-908.md) Reward clients for a sustainable network | Withdrawn | None recorded | Unassessed |
| [EIP-969](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-969.md) Modifications to ethash to invalidate existing dedicated hardware implementations | Stagnant | None recorded | Unassessed |
| [EIP-999](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-999.md) Restore Contract Code at 0x863DF6BFa4469f3ead0bE8f9F2AAE51c91A907b4 | Withdrawn | None recorded | Unassessed |
| [EIP-1010](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1010.md) Uniformity Between 0xAb5801a7D398351b8bE11C439e05C5B3259aeC9B and 0x15E55EF43efA8348dDaeAa455F16C43B64917e3c | Stagnant | None recorded | Unassessed |
| [EIP-1011](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1011.md) Hybrid Casper FFG | Stagnant | None recorded | Unassessed |
| [EIP-1014](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1014.md) Skinny CREATE2 | Final | None recorded | E04 |
| [EIP-1015](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1015.md) Configurable On Chain Issuance | Stagnant | None recorded | Unassessed |
| [EIP-1051](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1051.md) Overflow checking for the EVM | Stagnant | None recorded | Unassessed |
| [EIP-1052](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1052.md) EXTCODEHASH opcode | Final | 161 | E05 |
| [EIP-1057](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1057.md) ProgPoW, a Programmatic Proof-of-Work | Stagnant | None recorded | Unassessed |
| [EIP-1087](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1087.md) Net gas metering for SSTORE operations | Stagnant | None recorded | Unassessed |
| [EIP-1108](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1108.md) Reduce alt_bn128 precompile gas costs | Final | 196, 197 | E06 |
| [EIP-1109](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1109.md) PRECOMPILEDCALL opcode (Remove CALL costs for precompiled contracts) | Stagnant | None recorded | Unassessed |
| [EIP-1153](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1153.md) Transient storage opcodes | Final | 2200, 3529 | E03 |
| [EIP-1227](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1227.md) Defuse Difficulty Bomb and Reset Block Reward | Stagnant | 649 | Unassessed |
| [EIP-1234](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1234.md) Constantinople Difficulty Bomb Delay and Block Reward Adjustment | Final | None recorded | Unassessed |
| [EIP-1240](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1240.md) Remove Difficulty Bomb | Withdrawn | None recorded | Unassessed |
| [EIP-1276](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1276.md) Eliminate Difficulty Bomb and Adjust Block Reward on Constantinople Shift | Stagnant | None recorded | Unassessed |
| [EIP-1283](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1283.md) Net gas metering for SSTORE without dirty maps | Final | None recorded | Unassessed |
| [EIP-1285](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1285.md) Increase Gcallstipend gas in the CALL opcode | Stagnant | None recorded | Unassessed |
| [EIP-1295](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1295.md) Modify Ethereum PoW Incentive Structure and Delay Difficulty Bomb | Stagnant | None recorded | Unassessed |
| [EIP-1344](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1344.md) ChainID opcode | Final | 155 | E05 |
| [EIP-1352](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1352.md) Specify restricted address range for precompiles/system contracts | Stagnant | None recorded | Unassessed |
| [EIP-1355](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1355.md) Ethash 1a | Withdrawn | None recorded | Unassessed |
| [EIP-1380](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1380.md) Reduced gas cost for call to self | Stagnant | 150 | Unassessed |
| [EIP-1418](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1418.md) Blockchain Storage Rent Payment | Stagnant | 1559 | Unassessed |
| [EIP-1482](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1482.md) Define a maximum block timestamp drift | Stagnant | None recorded | Unassessed |
| [EIP-1485](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1485.md) TEthashV1 | Stagnant | None recorded | Unassessed |
| [EIP-1559](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1559.md) Fee market change for ETH 1.0 chain | Final | 2718, 2930 | E01, E08 |
| [EIP-1681](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1681.md) Temporal Replay Protection | Stagnant | None recorded | Unassessed |
| [EIP-1682](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1682.md) Storage Rent | Withdrawn | None recorded | Unassessed |
| [EIP-1702](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1702.md) Generalized Account Versioning Scheme | Stagnant | None recorded | Unassessed |
| [EIP-1706](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1706.md) Disable SSTORE with gasleft lower than call stipend | Withdrawn | 1283 | Unassessed |
| [EIP-1829](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1829.md) Precompile for Elliptic Curve Linear Combinations | Stagnant | None recorded | Unassessed |
| [EIP-1884](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1884.md) Repricing for trie-size-dependent opcodes | Final | 150, 1052 | Unassessed |
| [EIP-1890](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1890.md) Commitment to Sustainable Ecosystem Funding | Withdrawn | None recorded | Unassessed |
| [EIP-1895](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1895.md) Support for an Elliptic Curve Cycle | Stagnant | None recorded | Unassessed |
| [EIP-1930](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1930.md) CALLs with strict gas semantic. Revert if not enough gas available. | Stagnant | None recorded | Unassessed |
| [EIP-1959](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1959.md) New Opcode to check if a chainID is part of the history of chainIDs | Stagnant | 155 | Unassessed |
| [EIP-1962](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1962.md) EC arithmetic and pairings with runtime definitions | Stagnant | 1109 | Unassessed |
| [EIP-1965](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1965.md) Method to check if a chainID is valid at a specific block Number | Stagnant | 155 | Unassessed |
| [EIP-1985](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1985.md) Sane limits for certain EVM parameters | Stagnant | None recorded | Unassessed |
| [EIP-2014](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2014.md) Extended State Oracle | Stagnant | 140 | Unassessed |
| [EIP-2025](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2025.md) Block Rewards Proposal for funding Eth1.x | Withdrawn | 1890 | Unassessed |
| [EIP-2026](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2026.md) State Rent H - Fixed Prepayment for accounts | Stagnant | None recorded | Unassessed |
| [EIP-2027](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2027.md) State Rent C - Net contract size accounting | Stagnant | None recorded | Unassessed |
| [EIP-2028](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2028.md) Transaction data gas cost reduction | Final | None recorded | Unassessed |
| [EIP-2029](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2029.md) State Rent A - State counters contract | Stagnant | None recorded | Unassessed |
| [EIP-2031](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2031.md) State Rent B - Net transaction counter | Stagnant | 2029 | Unassessed |
| [EIP-2035](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2035.md) Stateless Clients - Repricing SLOAD and SSTORE to pay for block proofs | Stagnant | None recorded | Unassessed |
| [EIP-2045](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2045.md) Particle gas costs for EVM opcodes | Stagnant | None recorded | Unassessed |
| [EIP-2046](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2046.md) Reduced gas cost for static calls made to precompiles | Stagnant | 214, 1352 | Unassessed |
| [EIP-2200](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2200.md) Structured Definitions for Net Gas Metering | Final | None recorded | E03 |
| [EIP-2242](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2242.md) Transaction Postdata | Stagnant | None recorded | Unassessed |
| [EIP-2315](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2315.md) Simple Subroutines for the EVM | Withdrawn | 3540, 3670, 4200 | Unassessed |
| [EIP-2327](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2327.md) BEGINDATA opcode | Stagnant | None recorded | Unassessed |
| [EIP-2330](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2330.md) EXTSLOAD opcode | Stagnant | 2929 | Unassessed |
| [EIP-2384](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2384.md) Muir Glacier Difficulty Bomb Delay | Final | None recorded | Unassessed |
| [EIP-2474](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2474.md) Coinbase calls | Stagnant | None recorded | Unassessed |
| [EIP-2488](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2488.md) Deprecate the CALLCODE opcode | Stagnant | 7 | Unassessed |
| [EIP-2515](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2515.md) Implement Difficulty Freeze | Stagnant | None recorded | Unassessed |
| [EIP-2537](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2537.md) Precompile for BLS12-381 curve operations | Final | None recorded | E06 |
| [EIP-2539](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2539.md) BLS12-377 curve operations | Stagnant | 1109, 2046 | Unassessed |
| [EIP-2542](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2542.md) New opcodes TXGASLIMIT and CALLGASLIMIT | Withdrawn | None recorded | Unassessed |
| [EIP-2565](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2565.md) ModExp Gas Cost | Final | 198 | E06 |
| [EIP-2583](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2583.md) Penalty for account trie misses | Stagnant | None recorded | Unassessed |
| [EIP-2584](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2584.md) Trie format transition with overlay trees | Stagnant | None recorded | Unassessed |
| [EIP-2593](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2593.md) Escalator fee market change for ETH 1.0 chain | Stagnant | None recorded | Unassessed |
| [EIP-2666](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2666.md) Repricing of precompiles and Keccak256 function | Stagnant | 1352, 2046, 2565 | Unassessed |
| [EIP-2677](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2677.md) Limit size of `initcode` | Withdrawn | None recorded | Unassessed |
| [EIP-2681](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2681.md) Limit account nonce to 2^64-1 | Final | None recorded | Unassessed |
| [EIP-2711](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2711.md) Sponsored, expiring and batch transactions. | Withdrawn | 2718 | Unassessed |
| [EIP-2718](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2718.md) Typed Transaction Envelope | Final | None recorded | E01, E08 |
| [EIP-2733](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2733.md) Transaction Package | Withdrawn | 2718 | Unassessed |
| [EIP-2780](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2780.md) Resource-Based Intrinsic Transaction Gas | Last Call | 2718, 2929, 2930, 6780, 7623, 7702, 7708, 7928, 8037, 8038 | Unassessed |
| [EIP-2803](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2803.md) Rich Transactions | Stagnant | None recorded | Unassessed |
| [EIP-2926](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2926.md) Chunk-Based Code Merkleization | Draft | 161, 170 | Unassessed |
| [EIP-2929](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2929.md) Gas cost increases for state access opcodes | Final | None recorded | E03 |
| [EIP-2930](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2930.md) Optional access lists | Final | 2718, 2929 | E01, E03 |
| [EIP-2935](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2935.md) Serve historical block hashes from state | Final | None recorded | E07 |
| [EIP-2936](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2936.md) EXTCLEAR Opcode For SELFDESTRUCTed contracts | Stagnant | None recorded | Unassessed |
| [EIP-2937](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2937.md) SET_INDESTRUCTIBLE opcode | Stagnant | None recorded | Unassessed |
| [EIP-2938](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2938.md) Account Abstraction | Withdrawn | 2718 | Unassessed |
| [EIP-2970](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2970.md) IS_STATIC opcode | Stagnant | None recorded | Unassessed |
| [EIP-2972](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2972.md) Wrapped Legacy Transactions | Withdrawn | 155, 2718 | Unassessed |
| [EIP-2997](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2997.md) IMPERSONATECALL Opcode | Stagnant | None recorded | Unassessed |
| [EIP-3026](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3026.md) BW6-761 curve operations | Stagnant | 2539 | Unassessed |
| [EIP-3068](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3068.md) Precompile for BN256 HashToCurve Algorithms | Stagnant | 198, 1108 | Unassessed |
| [EIP-3074](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3074.md) AUTH and AUTHCALL opcodes | Withdrawn | 155 | Unassessed |
| [EIP-3102](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3102.md) Binary trie structure | Stagnant | None recorded | Unassessed |
| [EIP-3143](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3143.md) Increase block rewards to 5 ETH | Stagnant | None recorded | Unassessed |
| [EIP-3198](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3198.md) BASEFEE opcode | Final | 1559 | E05 |
| [EIP-3220](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3220.md) Crosschain Identifier Specification | Stagnant | None recorded | Unassessed |
| [EIP-3238](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3238.md) Difficulty Bomb Delay to Q2/2022 | Stagnant | None recorded | Unassessed |
| [EIP-3267](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3267.md) Giving Ethereum fees to Future Salaries | Stagnant | None recorded | Unassessed |
| [EIP-3298](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3298.md) Remove storage-clear refund and refund cap | Draft | 2780, 3529, 7778, 8037, 8038 | Unassessed |
| [EIP-3300](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3300.md) Phase out refunds | Stagnant | None recorded | Unassessed |
| [EIP-3322](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3322.md) Account gas storage opcodes | Stagnant | None recorded | Unassessed |
| [EIP-3332](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3332.md) MEDGASPRICE Opcode | Withdrawn | None recorded | Unassessed |
| [EIP-3336](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3336.md) Paged memory allocation for the EVM | Stagnant | None recorded | Unassessed |
| [EIP-3337](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3337.md) Frame pointer support for memory load and store operations | Stagnant | 3336 | Unassessed |
| [EIP-3338](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3338.md) Limit account nonce to 2^52 | Withdrawn | None recorded | Unassessed |
| [EIP-3368](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3368.md) Increase block rewards to 3 ETH, with 2 Year Decay to 1 ETH Scheduled | Stagnant | None recorded | Unassessed |
| [EIP-3372](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3372.md) 5 FNV primes for ethash | Stagnant | None recorded | Unassessed |
| [EIP-3374](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3374.md) Predictable Proof-of-Work (POW) Sunsetting | Withdrawn | None recorded | Unassessed |
| [EIP-3382](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3382.md) Hardcoded Block Gas Limit | Withdrawn | None recorded | Unassessed |
| [EIP-3403](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3403.md) Partial removal of refunds | Stagnant | None recorded | Unassessed |
| [EIP-3416](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3416.md) Median Gas Premium | Stagnant | None recorded | Unassessed |
| [EIP-3436](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3436.md) Expanded Clique Block Choice Rule | Stagnant | 225 | Unassessed |
| [EIP-3455](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3455.md) SUDO Opcode | Stagnant | None recorded | Unassessed |
| [EIP-3508](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3508.md) Transaction Data Opcodes | Stagnant | None recorded | Unassessed |
| [EIP-3520](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3520.md) Transaction Destination Opcode | Stagnant | 3508 | Unassessed |
| [EIP-3521](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3521.md) Reduce access list cost | Stagnant | 2028, 2930 | Unassessed |
| [EIP-3529](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3529.md) Reduction in refunds | Final | 2200, 2929, 2930 | E03 |
| [EIP-3534](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3534.md) Restricted Chain Context Type Transactions | Stagnant | 2718, 2930 | Unassessed |
| [EIP-3540](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3540.md) EOF - EVM Object Format v1 | Stagnant | 3541, 3860 | Unassessed |
| [EIP-3541](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3541.md) Reject new contract code starting with the 0xEF byte | Final | None recorded | Unassessed |
| [EIP-3554](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3554.md) Difficulty Bomb Delay to December 2021 | Final | None recorded | Unassessed |
| [EIP-3584](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3584.md) Block Access List | Stagnant | 2929, 2930 | Unassessed |
| [EIP-3607](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3607.md) Reject transactions from senders with deployed code | Final | None recorded | E02 |
| [EIP-3651](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3651.md) Warm COINBASE | Final | 2929 | Unassessed |
| [EIP-3670](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3670.md) EOF - Code Validation | Stagnant | 3540 | Unassessed |
| [EIP-3675](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3675.md) Upgrade consensus to Proof-of-Stake | Final | 2124 | E11 |
| [EIP-3690](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3690.md) EOF - JUMPDEST Table | Stagnant | 3540, 3670 | Unassessed |
| [EIP-3756](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3756.md) Gas Limit Cap | Stagnant | None recorded | Unassessed |
| [EIP-3779](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3779.md) Safer Control Flow for the EVM | Withdrawn | None recorded | Unassessed |
| [EIP-3788](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3788.md) Strict enforcement of chainId | Stagnant | 155 | Unassessed |
| [EIP-3855](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3855.md) PUSH0 instruction | Final | None recorded | E05 |
| [EIP-3860](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3860.md) Limit and meter initcode | Final | 170 | E04 |
| [EIP-3978](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3978.md) Gas refunds on reverts | Stagnant | 2929 | Unassessed |
| [EIP-4200](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4200.md) EOF - Static relative jumps | Stagnant | 3540, 3670 | Unassessed |
| [EIP-4345](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4345.md) Difficulty Bomb Delay to June 2022 | Final | None recorded | Unassessed |
| [EIP-4396](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4396.md) Time-Aware Base Fee Calculation | Stagnant | None recorded | Unassessed |
| [EIP-4399](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4399.md) Supplant DIFFICULTY opcode with PREVRANDAO | Final | 3675 | E05, E11 |
| [EIP-4488](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4488.md) Transaction calldata gas cost reduction with total calldata limit | Stagnant | None recorded | Unassessed |
| [EIP-4520](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4520.md) Multi-byte opcodes prefixed by EB and EC. | Stagnant | None recorded | Unassessed |
| [EIP-4573](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4573.md) Procedures for the EVM | Stagnant | 2315, 3540, 3670, 3779, 4200 | Unassessed |
| [EIP-4747](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4747.md) Simplify EIP-161 | Stagnant | 161 | Unassessed |
| [EIP-4750](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4750.md) EOF - Functions | Stagnant | 3540, 3670, 5450 | Unassessed |
| [EIP-4758](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4758.md) Deactivate SELFDESTRUCT | Stagnant | None recorded | Unassessed |
| [EIP-4760](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4760.md) SELFDESTRUCT bomb | Stagnant | None recorded | Unassessed |
| [EIP-4762](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4762.md) Statelessness gas cost changes | Draft | None recorded | Unassessed |
| [EIP-4788](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4788.md) Beacon block root in the EVM | Final | 1559 | E07 |
| [EIP-4803](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4803.md) Limit transaction gas to a maximum of 2^63-1 | Stagnant | None recorded | Unassessed |
| [EIP-4844](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4844.md) Shard Blob Transactions | Final | 1559, 2718, 2930, 4895 | E01, E06, E08, C09, N06 |
| [EIP-4863](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4863.md) Beacon chain push withdrawals | Stagnant | None recorded | Unassessed |
| [EIP-4895](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4895.md) Beacon chain push withdrawals as operations | Final | None recorded | E08 |
| [EIP-5000](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5000.md) MULDIV instruction | Stagnant | None recorded | Unassessed |
| [EIP-5003](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5003.md) Insert Code into EOAs with AUTHUSURP | Withdrawn | 3074, 3607 | Unassessed |
| [EIP-5022](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5022.md) Increase price of SSTORE from zero to non-zero to 40k gas | Stagnant | None recorded | Unassessed |
| [EIP-5027](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5027.md) Remove the limit on contract code size | Stagnant | 170, 2929, 2930 | Unassessed |
| [EIP-5065](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5065.md) Instruction for transferring ether | Stagnant | 2929 | Unassessed |
| [EIP-5081](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5081.md) Expirable Transaction | Stagnant | 155, 1559, 2718, 2929, 2930 | Unassessed |
| [EIP-5133](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5133.md) Delaying Difficulty Bomb to mid-September 2022 | Final | None recorded | Unassessed |
| [EIP-5283](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5283.md) Semaphore for Reentrancy Protection | Stagnant | 20, 1283, 1352 | Unassessed |
| [EIP-5450](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5450.md) EOF - Stack Validation | Stagnant | 3540, 3670, 4200, 4750 | Unassessed |
| [EIP-5478](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5478.md) CREATE2COPY Opcode | Stagnant | 1014, 2929 | Unassessed |
| [EIP-5656](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5656.md) MCOPY - Memory copying instruction | Final | None recorded | E05 |
| [EIP-5806](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5806.md) Delegate transaction | Stagnant | 2718, 2930 | Unassessed |
| [EIP-5920](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5920.md) PAY opcode | Draft | 214, 2929, 7523 | Unassessed |
| [EIP-5988](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5988.md) Add Poseidon hash function precompile | Stagnant | None recorded | Unassessed |
| [EIP-6046](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6046.md) Replace SELFDESTRUCT with DEACTIVATE | Stagnant | 2681, 2929, 3529 | Unassessed |
| [EIP-6110](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6110.md) Supply validator deposits on chain | Final | 7685 | E07 |
| [EIP-6188](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6188.md) Nonce Cap | Stagnant | 2929 | Unassessed |
| [EIP-6189](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6189.md) Alias Contracts | Stagnant | 2929, 6188 | Unassessed |
| [EIP-6190](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6190.md) Verkle-compatible SELFDESTRUCT | Stagnant | 2929, 6188, 6189 | Unassessed |
| [EIP-6206](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6206.md) EOF - JUMPF and non-returning functions | Stagnant | 4750, 5450 | Unassessed |
| [EIP-6404](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6404.md) SSZ transactions | Draft | 155, 1559, 2718, 2930, 4844, 7495, 7702, 7916, 7932, 8016 | Unassessed |
| [EIP-6465](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6465.md) SSZ withdrawals root | Draft | 2718, 4895, 6404, 7495, 7916 | Unassessed |
| [EIP-6466](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6466.md) SSZ receipts | Draft | 658, 2718, 6404, 7495, 7668, 7702, 7916, 8016, 8116 | Unassessed |
| [EIP-6475](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6475.md) SSZ Optional | Stagnant | None recorded | Unassessed |
| [EIP-6493](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6493.md) SSZ transaction signature scheme | Draft | 6404, 6466 | Unassessed |
| [EIP-6690](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6690.md) EVM Modular Arithmetic Extensions | Stagnant | None recorded | Unassessed |
| [EIP-6780](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6780.md) SELFDESTRUCT only in same transaction | Final | 2681, 2929, 3529 | E04 |
| [EIP-6800](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6800.md) Ethereum state using a unified verkle tree | Stagnant | 6780 | E10 |
| [EIP-6810](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6810.md) Ex Post Facto Cascading Revert | Stagnant | 2718, 2929 | Unassessed |
| [EIP-6811](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6811.md) To The Moon—10 Minute Blocks | Stagnant | None recorded | Unassessed |
| [EIP-6873](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6873.md) Preimage retention | Stagnant | None recorded | Unassessed |
| [EIP-6888](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6888.md) Arithmetic verification at EVM level | Stagnant | None recorded | Unassessed |
| [EIP-6913](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6913.md) SETCODE instruction | Withdrawn | None recorded | Unassessed |
| [EIP-6914](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6914.md) Reuse Withdrawn Validator Indices | Stagnant | None recorded | Unassessed |
| [EIP-6916](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6916.md) Automatically Reset Testnet | Final | None recorded | Unassessed |
| [EIP-6968](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6968.md) Contract Secured Revenue on an EVM based L2 | Stagnant | None recorded | Unassessed |
| [EIP-6988](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6988.md) Elected block proposer has not been slashed | Stagnant | None recorded | Unassessed |
| [EIP-7002](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7002.md) Execution layer triggerable withdrawals | Final | 7685 | E07 |
| [EIP-7044](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7044.md) Perpetually Valid Signed Voluntary Exits | Final | None recorded | Unassessed |
| [EIP-7045](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7045.md) Increase max attestation inclusion slot | Final | None recorded | Unassessed |
| [EIP-7069](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7069.md) Revamped CALL instructions | Stagnant | 150, 211, 214, 2929, 3540 | Unassessed |
| [EIP-7251](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7251.md) Increase the MAX_EFFECTIVE_BALANCE | Final | 7002, 7685 | E07, N01, N02, N03 |
| [EIP-7266](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7266.md) Remove BLAKE2 compression precompile | Stagnant | None recorded | Unassessed |
| [EIP-7377](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7377.md) Migration Transaction | Stagnant | 170, 1559, 2200, 2718 | Unassessed |
| [EIP-7378](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7378.md) Add time-weighted averaging to the base fee | Stagnant | None recorded | Unassessed |
| [EIP-7441](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7441.md) Upgrade block proposer election to Whisk | Stagnant | None recorded | Unassessed |
| [EIP-7480](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7480.md) EOF - Data section access instructions | Stagnant | 3540, 3670 | Unassessed |
| [EIP-7495](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7495.md) SSZ ProgressiveContainer | Review | 7916 | S03 |
| [EIP-7503](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7503.md) Zero-Knowledge Wormholes | Stagnant | 2718, 4844, 7708 | Unassessed |
| [EIP-7514](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7514.md) Add Max Epoch Churn Limit | Final | None recorded | N02 |
| [EIP-7516](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7516.md) BLOBBASEFEE instruction | Final | 3198, 4844 | Unassessed |
| [EIP-7519](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7519.md) Atomic Storage Operations SCREDIT and SDEBIT | Stagnant | 2200, 2929 | Unassessed |
| [EIP-7523](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7523.md) Empty accounts deprecation | Last Call | 161 | E10 |
| [EIP-7543](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7543.md) EVM arbitrary precision decimal math | Stagnant | None recorded | Unassessed |
| [EIP-7545](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7545.md) Verkle proof verification precompile | Stagnant | None recorded | Unassessed |
| [EIP-7547](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7547.md) Inclusion lists | Stagnant | None recorded | Unassessed |
| [EIP-7549](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7549.md) Move committee index outside Attestation | Final | None recorded | N02 |
| [EIP-7557](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7557.md) Block-level Warming with fair cost savings | Stagnant | None recorded | Unassessed |
| [EIP-7591](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7591.md) BLS signed transactions | Stagnant | None recorded | Unassessed |
| [EIP-7594](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7594.md) PeerDAS - Peer Data Availability Sampling | Final | 4844 | C09, C10, C11, N06 |
| [EIP-7609](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7609.md) Decrease base cost of TLOAD/TSTORE | Stagnant | 1153 | Unassessed |
| [EIP-7610](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7610.md) Revert creation in case of non-empty storage | Last Call | None recorded | Unassessed |
| [EIP-7612](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7612.md) Verkle state transition via an overlay tree | Stagnant | 4762, 6800, 7545 | Unassessed |
| [EIP-7619](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7619.md) Precompile Falcon512 generic verifier | Draft | None recorded | Unassessed |
| [EIP-7620](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7620.md) EOF Contract Creation | Stagnant | 170, 684, 2929, 3540, 3541, 3670 | Unassessed |
| [EIP-7623](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7623.md) Increase calldata cost | Final | None recorded | E03 |
| [EIP-7637](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7637.md) Optimize EOA EXTCODEHASH | Stagnant | 1052 | Unassessed |
| [EIP-7643](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7643.md) History accumulator for pre-PoS data | Stagnant | None recorded | I06 |
| [EIP-7645](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7645.md) Alias ORIGIN to SENDER | Stagnant | None recorded | Unassessed |
| [EIP-7650](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7650.md) Programmable access lists | Stagnant | 2929, 2930 | Unassessed |
| [EIP-7657](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7657.md) Sync committee slashings | Stagnant | None recorded | N05 |
| [EIP-7658](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7658.md) Light client data backfill | Stagnant | None recorded | N05 |
| [EIP-7664](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7664.md) Access-Key opcode | Withdrawn | 1153, 2930, 4844 | Unassessed |
| [EIP-7666](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7666.md) EVM-ify the identity precompile | Draft | 3855 | Unassessed |
| [EIP-7667](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7667.md) Raise gas costs of hash functions | Stagnant | None recorded | Unassessed |
| [EIP-7668](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7668.md) Remove bloom filters | Stagnant | None recorded | Unassessed |
| [EIP-7676](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7676.md) EOF - Prepare for Address Space Extension | Stagnant | 3540, 3670, 7069 | Unassessed |
| [EIP-7684](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7684.md) Return deposits for distinct credentials | Stagnant | None recorded | Unassessed |
| [EIP-7685](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7685.md) General purpose execution layer requests | Final | None recorded | E07, E08 |
| [EIP-7686](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7686.md) Linear EVM memory limits | Stagnant | None recorded | Unassessed |
| [EIP-7688](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7688.md) Forward Compatible Consensus Data Structures | Review | 7495, 7607, 7916 | Unassessed |
| [EIP-7691](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7691.md) Blob throughput increase | Final | None recorded | Unassessed |
| [EIP-7698](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7698.md) EOF - Creation transaction | Stagnant | 3540 | Unassessed |
| [EIP-7701](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7701.md) Native Account Abstraction | Withdrawn | None recorded | Unassessed |
| [EIP-7702](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7702.md) Set Code for EOAs | Final | 2, 161, 1052, 2718, 2929, 2930, 3541, 3607, 4844 | E01, E02, E04, A05, A06 |
| [EIP-7703](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7703.md) Increase calldata cost | Stagnant | None recorded | Unassessed |
| [EIP-7705](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7705.md) NONREENTRANT and REENTRANT opcodes | Stagnant | None recorded | Unassessed |
| [EIP-7706](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7706.md) Separate gas type for calldata | Stagnant | 1559, 4844 | Unassessed |
| [EIP-7707](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7707.md) Incentivize Access List Provisioning | Stagnant | 2930 | Unassessed |
| [EIP-7708](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7708.md) ETH Transfers Emit a Log | Last Call | 1559, 4788, 6780, 8246 | E08, A01 |
| [EIP-7709](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7709.md) Read BLOCKHASH from Storage and Update Cost | Draft | 2935 | Unassessed |
| [EIP-7716](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7716.md) Anti-Correlation Attestation Penalties | Draft | None recorded | Unassessed |
| [EIP-7727](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7727.md) EVM Transaction Bundles | Stagnant | 2718 | Unassessed |
| [EIP-7732](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7732.md) Enshrined Proposer-Builder Separation | Last Call | 7685 | N07 |
| [EIP-7736](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7736.md) Leaf-level state expiry in verkle trees | Stagnant | 6800 | Unassessed |
| [EIP-7742](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7742.md) Uncouple blob count between CL and EL | Stagnant | 4844 | Unassessed |
| [EIP-7745](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7745.md) Trustless log and transaction index | Draft | 7916 | Unassessed |
| [EIP-7748](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7748.md) State conversion to Verkle Tree | Draft | 7612 | Unassessed |
| [EIP-7761](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7761.md) EXTCODETYPE instruction | Stagnant | 3540, 7692 | Unassessed |
| [EIP-7762](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7762.md) Increase MIN_BASE_FEE_PER_BLOB_GAS | Stagnant | 4844 | Unassessed |
| [EIP-7775](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7775.md) BURN opcode | Stagnant | None recorded | Unassessed |
| [EIP-7778](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7778.md) Block Gas Accounting without Refunds | Last Call | None recorded | E03 |
| [EIP-7782](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7782.md) Reduce Block Latency | Draft | 7623, 7778 | Unassessed |
| [EIP-7784](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7784.md) GETCONTRACT opcode | Review | None recorded | Unassessed |
| [EIP-7788](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7788.md) Dynamic target blob count | Withdrawn | 7742 | Unassessed |
| [EIP-7791](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7791.md) GAS2ETH opcode | Draft | 2929 | Unassessed |
| [EIP-7792](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7792.md) Verifiable logs | Stagnant | 6466 | Unassessed |
| [EIP-7793](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7793.md) Conditional Transactions | Stagnant | 7843 | Unassessed |
| [EIP-7797](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7797.md) Double speed for hash_tree_root | Stagnant | None recorded | Unassessed |
| [EIP-7799](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7799.md) System logs | Draft | 4895, 6466, 7708, 7916, 8115 | E08, A01 |
| [EIP-7804](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7804.md) Withdrawal Credential Update Request | Draft | None recorded | Unassessed |
| [EIP-7805](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7805.md) Fork-choice enforced Inclusion Lists (FOCIL) | Draft | None recorded | N07 |
| [EIP-7807](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7807.md) SSZ execution blocks | Draft | 7495, 7773, 7916 | Unassessed |
| [EIP-7819](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7819.md) SETDELEGATE instruction | Draft | 7702 | Unassessed |
| [EIP-7823](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7823.md) Set upper bounds for MODEXP | Final | 198 | Unassessed |
| [EIP-7825](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7825.md) Transaction Gas Limit Cap | Final | None recorded | Unassessed |
| [EIP-7830](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7830.md) Contract size limit increase for EOF | Stagnant | 170, 3540, 3860 | Unassessed |
| [EIP-7833](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7833.md) Scheduled function calls | Stagnant | None recorded | Unassessed |
| [EIP-7834](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7834.md) Separate Metadata Section for EOF | Review | 3540 | Unassessed |
| [EIP-7843](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7843.md) SLOTNUM Opcode | Last Call | None recorded | Unassessed |
| [EIP-7848](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7848.md) On-chain upgrade signaling | Draft | None recorded | Unassessed |
| [EIP-7851](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7851.md) Code-Controlled EOA Delegation | Draft | 7702 | Unassessed |
| [EIP-7862](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7862.md) Delayed State Root | Draft | None recorded | Unassessed |
| [EIP-7863](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7863.md) Block-level Warming | Draft | None recorded | Unassessed |
| [EIP-7864](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7864.md) Ethereum state using a unified binary tree | Draft | None recorded | E10 |
| [EIP-7873](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7873.md) EOF - TXCREATE and InitcodeTransaction type | Stagnant | 3540, 3670, 3860, 7620 | Unassessed |
| [EIP-7877](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7877.md) Enhanced RETURN opcodes | Stagnant | 6, 1153, 3855 | Unassessed |
| [EIP-7880](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7880.md) EOF - EXTCODEADDRESS instruction | Review | 7692, 7702, 7761 | Unassessed |
| [EIP-7883](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7883.md) ModExp Gas Cost Increase | Final | 2565 | Unassessed |
| [EIP-7885](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7885.md) Precompile for NTT operations | Draft | None recorded | Unassessed |
| [EIP-7886](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7886.md) Delayed execution | Stagnant | 1559, 2930, 4844, 7623, 7702 | Unassessed |
| [EIP-7889](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7889.md) Emit log on revert | Stagnant | None recorded | Unassessed |
| [EIP-7898](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7898.md) Uncouple execution payload from beacon block | Stagnant | None recorded | Unassessed |
| [EIP-7903](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7903.md) Remove Initcode Size Limit | Stagnant | 170, 3860 | Unassessed |
| [EIP-7906](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7906.md) Transaction Assertions via State Diff Opcode | Draft | 2929, 8141 | Unassessed |
| [EIP-7907](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7907.md) Meter Contract Code Size | Draft | 170, 2929, 7702, 7954 | Unassessed |
| [EIP-7911](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7911.md) Scaling Ethereum with a Perceptron Tree ZKP | Draft | None recorded | Unassessed |
| [EIP-7912](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7912.md) Pragmatic stack manipulation tools | Stagnant | None recorded | Unassessed |
| [EIP-7915](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7915.md) Adaptive mean reversion blob pricing | Stagnant | 4844 | Unassessed |
| [EIP-7916](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7916.md) SSZ ProgressiveList | Review | None recorded | S03 |
| [EIP-7917](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7917.md) Deterministic proposer lookahead | Final | None recorded | Unassessed |
| [EIP-7918](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7918.md) Blob base fee bounded by execution cost | Final | 4844, 7840 | Unassessed |
| [EIP-7921](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7921.md) Skip `JUMPDEST` immediate argument check | Stagnant | None recorded | Unassessed |
| [EIP-7922](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7922.md) Dynamic exit queue rate limit | Stagnant | None recorded | Unassessed |
| [EIP-7923](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7923.md) Linear, Page-Based Memory Costing | Draft | None recorded | Unassessed |
| [EIP-7928](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7928.md) Block-Level Access Lists | Last Call | None recorded | Unassessed |
| [EIP-7932](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7932.md) Secondary Signature Algorithms | Draft | None recorded | Unassessed |
| [EIP-7934](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7934.md) RLP Execution Block Size Limit | Final | None recorded | Unassessed |
| [EIP-7937](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7937.md) EVM64 - 64-bit mode EVM opcodes | Draft | None recorded | Unassessed |
| [EIP-7939](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7939.md) Count leading zeros (CLZ) opcode | Final | None recorded | Unassessed |
| [EIP-7942](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7942.md) Available Attestation | Draft | None recorded | Unassessed |
| [EIP-7951](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7951.md) Precompile for secp256r1 Curve Support | Final | None recorded | E06 |
| [EIP-7954](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7954.md) Increase Maximum Contract Size | Last Call | 170, 3860 | Unassessed |
| [EIP-7956](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7956.md) Tx Ordering via Block-level Randomness | Stagnant | 7998 | Unassessed |
| [EIP-7957](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7957.md) EVM64 - EOF support | Stagnant | 3540, 3670, 4200, 7937 | Unassessed |
| [EIP-7958](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7958.md) EVM64 - Little endian opcodes | Stagnant | 7937 | Unassessed |
| [EIP-7960](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7960.md) EOF - Extended types section | Draft | 3540 | Unassessed |
| [EIP-7961](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7961.md) EVM64 - EOF code section | Draft | 3540, 4750, 7960 | Unassessed |
| [EIP-7971](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7971.md) Hard Limits for Transient Storage | Draft | 1153 | Unassessed |
| [EIP-7973](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7973.md) Warm Account Write Metering | Draft | 2200 | Unassessed |
| [EIP-7976](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7976.md) Increase Calldata Floor Cost | Last Call | 7623 | Unassessed |
| [EIP-7979](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7979.md) Call and Return Opcodes for the EVM | Review | None recorded | Unassessed |
| [EIP-7980](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7980.md) Ed25519 transaction support | Withdrawn | 7932 | Unassessed |
| [EIP-7981](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7981.md) Increase Access List Cost | Last Call | 2930, 7623, 7976 | Unassessed |
| [EIP-7997](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7997.md) Deterministic Factory Contract | Last Call | 1014 | Unassessed |
| [EIP-7998](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7998.md) Turn `randao_reveal` into a VRF | Draft | None recorded | Unassessed |
| [EIP-7999](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7999.md) Unified multidimensional fee market | Draft | 1559, 2718, 4844, 7516, 7691, 7840, 7918 | Unassessed |
| [EIP-8011](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8011.md) Multidimensional Gas Metering | Draft | 1559 | Unassessed |
| [EIP-8012](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8012.md) Generalized consolidation requests | Stagnant | None recorded | Unassessed |
| [EIP-8013](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8013.md) Static relative jumps and calls for the EVM | Draft | 7979 | Unassessed |
| [EIP-8015](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8015.md) Remove `deposit` and `eth1data` fields | Draft | 6110, 7688, 7773 | Unassessed |
| [EIP-8016](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8016.md) SSZ CompatibleUnion | Draft | 7495, 7916 | S03 |
| [EIP-8024](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8024.md) Backward Compatible SWAPN, DUPN, EXCHANGE | Last Call | None recorded | Unassessed |
| [EIP-8025](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8025.md) Optional Execution Proofs | Draft | 4844, 6110, 7002, 7251, 7688, 7732, 7928, 8282 | Unassessed |
| [EIP-8030](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8030.md) P256 algorithm support | Draft | 7932, 7951 | Unassessed |
| [EIP-8032](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8032.md) Size-Based Storage Gas Pricing | Draft | None recorded | Unassessed |
| [EIP-8037](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8037.md) State Creation Gas Cost Increase | Last Call | 161, 684, 2780, 6780, 7623, 7702, 7825, 7928, 7976, 8038 | E02, E03, E04 |
| [EIP-8038](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8038.md) State-Access Gas Cost Update | Last Call | 8037 | E03 |
| [EIP-8045](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8045.md) Exclude Slashed Validators from Proposing | Last Call | 7917 | Unassessed |
| [EIP-8046](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8046.md) Uniform price auction over inclusion lists | Draft | 7805, 8037, 8141, 8369 | Unassessed |
| [EIP-8051](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8051.md) Precompile for ML-DSA signature verification | Draft | 7932 | Unassessed |
| [EIP-8052](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8052.md) Precompile for Falcon support | Draft | 7932 | Unassessed |
| [EIP-8053](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8053.md) Milli-gas for High-precision Gas Metering | Draft | 150, 2200, 2780, 8037 | Unassessed |
| [EIP-8057](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8057.md) Inter-Block Temporal Locality Gas Discounts | Draft | 2929, 7928 | Unassessed |
| [EIP-8058](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8058.md) Contract Bytecode Deduplication Discount | Draft | 2930 | Unassessed |
| [EIP-8059](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8059.md) Gas Units Rebase for High-precision Metering | Draft | None recorded | Unassessed |
| [EIP-8061](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8061.md) Increase Exit and Consolidation Churn | Last Call | 7251 | N03 |
| [EIP-8062](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8062.md) Add sweep withdrawal fee for 0x01 validators | Draft | 4895, 7251 | Unassessed |
| [EIP-8068](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8068.md) Neutral effective balance design | Draft | 7251 | Unassessed |
| [EIP-8071](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8071.md) Prevent using consolidations as withdrawals | Draft | None recorded | Unassessed |
| [EIP-8075](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8075.md) Adaptive state cost to cap growth & scale L1 | Draft | 1559, 4844, 8037 | Unassessed |
| [EIP-8079](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8079.md) Native rollups | Draft | None recorded | Unassessed |
| [EIP-8080](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8080.md) Let exits use the consolidation queue | Draft | None recorded | Unassessed |
| [EIP-8096](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8096.md) Increase Gas Cost of Point Evaluation | Draft | 4844 | Unassessed |
| [EIP-8099](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8099.md) MEVless Protocol | Draft | None recorded | Unassessed |
| [EIP-8101](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8101.md) Payload Chunking with Chunk Access Lists | Draft | 4844, 4895, 7732, 7825, 7928 | Unassessed |
| [EIP-8105](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8105.md) Universal Enshrined Encrypted Mempool | Draft | 7732 | Unassessed |
| [EIP-8115](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8115.md) Batch priority fees at end of block | Draft | 1559, 4895 | Unassessed |
| [EIP-8116](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8116.md) Replace cumulative receipt fields | Draft | None recorded | Unassessed |
| [EIP-8120](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8120.md) MLOAD8 and CALLDATALOAD8 Opcodes | Draft | None recorded | Unassessed |
| [EIP-8125](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8125.md) Temporary Contract Storage | Draft | 2200, 2929 | Unassessed |
| [EIP-8130](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8130.md) Keystore Accounts | Draft | 170, 1014 | Unassessed |
| [EIP-8131](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8131.md) Unified Transaction Content Floor | Draft | 2028, 4844, 7623, 7702, 7976, 7981 | Unassessed |
| [EIP-8141](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8141.md) Frame Transaction | Draft | 1559, 2718, 2780, 3529, 3607, 4844, 7594, 7702, 7708, 7778, 7825, 7976, 8037 | Unassessed |
| [EIP-8142](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8142.md) Block-in-Blobs (BiB) | Draft | 4844, 7594, 7732, 7892, 7928 | Unassessed |
| [EIP-8146](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8146.md) Block Access List Sidecars | Draft | 7732, 7928 | Unassessed |
| [EIP-8148](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8148.md) Custom sweep threshold for validators | Draft | 7251, 7685 | Unassessed |
| [EIP-8149](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8149.md) Multi KZG Point Evaluation Precompile | Draft | 4844 | Unassessed |
| [EIP-8151](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8151.md) Account Code Restricted ecRecover | Draft | 3607, 7702, 8038 | Unassessed |
| [EIP-8163](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8163.md) Reserve `EXTENSION (0xae)` opcode | Review | 141 | Unassessed |
| [EIP-8164](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8164.md) Native Key Delegation for EOAs | Draft | 2, 2718, 2929, 3541, 3607, 4844, 7702 | Unassessed |
| [EIP-8175](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8175.md) Composable Transaction | Draft | 2, 1559, 2718 | Unassessed |
| [EIP-8178](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8178.md) Binary SSZ Transport for the Engine API | Draft | None recorded | Unassessed |
| [EIP-8182](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8182.md) Private ETH and ERC-20 Transfers | Review | 20 | Unassessed |
| [EIP-8184](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8184.md) LUCID encrypted mempool | Draft | 2718, 7805 | Unassessed |
| [EIP-8188](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8188.md) Last-Written Block for Accounts and Slots | Draft | 8037 | Unassessed |
| [EIP-8197](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8197.md) Cryptographically Agile Transactions | Draft | 1559, 2718, 2930, 4844, 7702 | Unassessed |
| [EIP-8198](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8198.md) Quick Slots | Draft | 7843, 7892 | Unassessed |
| [EIP-8200](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8200.md) EVMification | Draft | 152, 7666, 7823, 7883 | Unassessed |
| [EIP-8202](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8202.md) Scheme-Agile Transactions | Draft | 2, 155, 1559, 2718, 2780, 4844, 7702, 7976 | Unassessed |
| [EIP-8205](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8205.md) Withdrawal credentials preregistration | Draft | 6110, 7685, 7732 | Unassessed |
| [EIP-8209](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8209.md) Commit-Reveal Transaction Frames | Draft | 8141 | Unassessed |
| [EIP-8219](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8219.md) Checked Arithmetic Opcodes | Draft | None recorded | Unassessed |
| [EIP-8222](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8222.md) Lean Staking | Draft | 6110, 7251 | Unassessed |
| [EIP-8237](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8237.md) Independent CL/EL Sync | Draft | 7732 | Unassessed |
| [EIP-8243](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8243.md) Batching Attestations at Source | Draft | None recorded | Unassessed |
| [EIP-8246](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8246.md) Remove SELFDESTRUCT Burn | Last Call | 161, 6780 | A01 |
| [EIP-8247](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8247.md) Ancestor-Indexed BALs | Draft | 7928 | Unassessed |
| [EIP-8250](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8250.md) Keyed Nonces for Frame Transactions | Draft | 7928, 7976, 8037, 8038, 8141 | Unassessed |
| [EIP-8253](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8253.md) Bump nonce of zero-nonce storage accounts | Draft | 161, 684 | Unassessed |
| [EIP-8254](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8254.md) Cap Deposit Requests Per Block | Draft | 6110 | Unassessed |
| [EIP-8256](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8256.md) Blob Streaming | Draft | 2935, 4788, 4844, 7002, 7594, 7732, 7918 | Unassessed |
| [EIP-8266](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8266.md) Expiring Nonces for Frame Transactions | Draft | 8141 | Unassessed |
| [EIP-8268](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8268.md) Storage Roots in Block Access Lists | Draft | 7928 | Unassessed |
| [EIP-8272](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8272.md) Recent Roots for Frame Transactions | Draft | 7843, 8141 | Unassessed |
| [EIP-8279](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8279.md) Block Access List Byte Floor | Draft | 7623, 7702, 7825, 7928, 7976, 7981, 8131 | Unassessed |
| [EIP-8282](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8282.md) Builder Execution Requests | Last Call | 1559, 7685, 7732, 7997 | Unassessed |
| [EIP-8288](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8288.md) In-mempool signature and proof aggregation | Draft | 2718, 8141 | Unassessed |
| [EIP-8289](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8289.md) Multi-block Access List Warming | Draft | 2929, 2930, 2935, 3651, 7702, 7928 | Unassessed |
| [EIP-8292](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8292.md) Post-Quantum Attestation Aggregators | Draft | None recorded | Unassessed |
| [EIP-8295](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8295.md) State Tiering by Periods | Draft | 2929, 2930, 7702, 7825, 8037, 8038, 8188 | Unassessed |
| [EIP-8296](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8296.md) Fixed-Cutoff State Tiering | Draft | 2929, 7702, 7825, 8037, 8038, 8188 | Unassessed |
| [EIP-8297](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8297.md) Partitioned Binary Tree | Draft | None recorded | Unassessed |
| [EIP-8298](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8298.md) SETCODEFROM Code Reuse Instruction | Draft | 3607, 6780, 7702, 7928, 8037, 8038 | Unassessed |
| [EIP-8304](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8304.md) Trustless log and transaction index | Draft | 4788 | Unassessed |
| [EIP-8311](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8311.md) Increase Calldata Floor Cost to 96 | Draft | 3860, 7623, 7976 | Unassessed |
| [EIP-8321](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8321.md) Hash-Chain RANDAO | Draft | 7916 | Unassessed |
| [EIP-8333](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8333.md) Align Checkpoint with Epoch Boundary Block | Withdrawn | None recorded | Unassessed |
| [EIP-8337](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8337.md) Validated EVM Code | Draft | 3541, 7979 | Unassessed |
| [EIP-8347](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8347.md) Offline State Migration to the PBT | Draft | 7523, 7928, 8159, 8297 | Unassessed |
| [EIP-8355](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8355.md) Precompiles for ML-DSA Verification | Draft | None recorded | Unassessed |
| [EIP-8357](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8357.md) EVM Verification Key Registry | Draft | 161, 1014, 1559, 7910, 7928, 7997, 8025, 8037, 8288 | Unassessed |
| [EIP-8358](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8358.md) Net Gas Metering for Account Changes | Draft | 161, 2200, 2780, 2929, 3529, 7702, 7708, 8037, 8038 | Unassessed |
| [EIP-8360](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8360.md) TCREATE Opcode | Draft | 161, 684, 1014, 1153, 2681, 3529, 6780, 7708, 7928, 8037, 8038 | Unassessed |
| [EIP-8363](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8363.md) Tapered Issuance Burn | Draft | None recorded | Unassessed |
| [EIP-8364](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8364.md) SSZ Compact Multiproofs | Draft | None recorded | Unassessed |
| [EIP-8365](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8365.md) Disallow new 0x00 validators | Draft | 6110, 7251, 7732 | Unassessed |
| [EIP-8368](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8368.md) CPSB Recalibration for New Gas Limit | Draft | 8037 | Unassessed |
| [EIP-8372](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8372.md) Normalized state gas limit | Draft | 8037 | Unassessed |
| [EIP-8374](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8374.md) Persist Warm Access Sets Across Reverts | Draft | 2929 | Unassessed |
| [EIP-8379](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8379.md) Top-up Sync | Draft | None recorded | Unassessed |
| [EIP-8390](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8390.md) Remove the Sync Committee | Draft | None recorded | N05 |
| [EIP-8397](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8397.md) Frame Authenticator Signatures | Draft | 2929, 7702, 7928, 8141 | Unassessed |
| [EIP-8425](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8425.md) Quantum Freeze and Account Recovery | Draft | 2929, 7702 | Unassessed |
| [EIP-8433](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8433.md) Retire 0x00 Validators | Draft | 7251, 8365 | Unassessed |
| [EIP-8435](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8435.md) Last-Written Block in PBT Leaves | Draft | 7954, 8037, 8297 | Unassessed |
| [EIP-8449](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8449.md) Engine API Proposal Lookahead | Draft | 7917 | Unassessed |
| [EIP-8450](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8450.md) Post-Quantum Consensus Key Registry | Draft | 7495, 7732, 7916, 8282 | Unassessed |

## Networking

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [EIP-8](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8.md) devp2p Forward Compatibility Requirements for Homestead | Final | None recorded | W01, W03 |
| [EIP-627](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-627.md) Whisper Specification | Final | None recorded | Unassessed |
| [EIP-706](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-706.md) DEVp2p snappy compression | Final | None recorded | W03 |
| [EIP-778](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-778.md) Ethereum Node Records (ENR) | Final | None recorded | W01 |
| [EIP-868](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-868.md) Node Discovery v4 ENR Extension | Final | 8, 778 | W01 |
| [EIP-1459](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1459.md) Node Discovery via DNS | Stagnant | 778 | Unassessed |
| [EIP-2124](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2124.md) Fork identifier for chain compatibility checks | Final | None recorded | W03 |
| [EIP-2364](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2364.md) eth/64: forkid-extended protocol handshake | Final | 2124 | Unassessed |
| [EIP-2464](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2464.md) eth/65: transaction announcements and retrievals | Final | 2364 | Unassessed |
| [EIP-2481](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2481.md) eth/66 request identifier | Final | 2464 | Unassessed |
| [EIP-2976](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2976.md) Typed Transactions over Gossip | Final | 2718 | Unassessed |
| [EIP-4444](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4444.md) Bound Historical Data in Execution Clients | Draft | None recorded | I06 |
| [EIP-4938](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4938.md) eth/67 - Removal of GetNodeData | Final | 2464, 2481 | Unassessed |
| [EIP-5793](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5793.md) eth/68 - Add tx type to tx announcement | Final | 2464, 2481, 4938 | Unassessed |
| [EIP-6122](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6122.md) Forkid checks based on timestamps | Final | 2124 | W03 |
| [EIP-7542](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7542.md) eth/70 - available-blocks-extended protocol | Withdrawn | 7642 | Unassessed |
| [EIP-7636](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7636.md) Extension of EIP-778 for "client" ENR Entry | Withdrawn | 778 | Unassessed |
| [EIP-7639](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7639.md) eth/70 - Cease serving history before PoS | Stagnant | None recorded | Unassessed |
| [EIP-7642](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7642.md) eth/69 - history expiry and simpler receipts | Final | 5793 | Unassessed |
| [EIP-7801](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7801.md) etha - Sharded Blocks Subprotocol | Draft | 7642 | Unassessed |
| [EIP-7975](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7975.md) eth/70 - Partial Block Receipt Lists | Last Call | 7642, 7825 | Unassessed |
| [EIP-8070](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8070.md) eth/72 - Sparse Blobpool | Last Call | 4844, 7594, 7870, 8159 | N06 |
| [EIP-8077](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8077.md) eth/XX - announce transactions with nonce | Draft | 7642 | Unassessed |
| [EIP-8094](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8094.md) eth/vhash - Blob-Aware Mempool | Draft | 7642 | Unassessed |
| [EIP-8136](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8136.md) Cell-Level Deltas for Data Column Broadcast | Last Call | 7594 | Unassessed |
| [EIP-8159](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8159.md) eth/71 - Block Access List Exchange | Last Call | 7928, 7975 | Unassessed |
| [EIP-8189](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8189.md) snap/2 - BAL-Based State Healing | Last Call | 7928 | Unassessed |
| [EIP-8371](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8371.md) RowDAS - Distributed Blob Reconstruction | Draft | 7594, 8136 | Unassessed |
| [EIP-8383](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8383.md) Reduce CL Block Retention Window | Draft | None recorded | Unassessed |
| [EIP-8408](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8408.md) eth/73 - Indexed Cell Requests | Draft | 8070 | Unassessed |
| [EIP-8411](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8411.md) Fast Execution Payload Broadcast | Draft | 7732 | Unassessed |

## Interface

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [EIP-6](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6.md) Renaming SUICIDE opcode | Final | None recorded | Unassessed |
| [EIP-107](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-107.md) safe "eth_sendTransaction" authorization via html popup | Stagnant | None recorded | Unassessed |
| [EIP-234](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-234.md) Add `blockHash` to JSON-RPC filter options. | Final | 1474 | Unassessed |
| [EIP-695](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-695.md) Create `eth_chainId` method for JSON-RPC | Final | 155 | Unassessed |
| [EIP-712](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-712.md) Typed structured data hashing and signing | Final | 155, 191 | A03, A04 |
| [EIP-747](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-747.md) wallet_watchAsset RPC Method | Final | 20, 1046, 1193 | A11 |
| [EIP-758](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-758.md) Subscriptions and filters for completed transactions | Stagnant | 1474 | Unassessed |
| [EIP-1102](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1102.md) Opt-in account exposure | Stagnant | 1474 | Unassessed |
| [EIP-1186](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1186.md) RPC-Method to get Merkle Proofs - eth_getProof | Stagnant | 1474 | S05, E09 |
| [EIP-1193](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1193.md) Ethereum Provider JavaScript API | Final | 155, 695 | A11 |
| [EIP-1474](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1474.md) Remote procedure call specification | Stagnant | None recorded | S05 |
| [EIP-1571](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1571.md) EthereumStratum/2.0.0 | Stagnant | None recorded | Unassessed |
| [EIP-1767](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1767.md) GraphQL interface to Ethereum node data | Stagnant | None recorded | Unassessed |
| [EIP-1803](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1803.md) Rename opcodes for clarity | Stagnant | 141 | Unassessed |
| [EIP-1898](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1898.md) Add `blockHash` to defaultBlock methods | Final | 234 | S05 |
| [EIP-1901](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1901.md) Add OpenRPC Service Discovery To JSON-RPC Services | Stagnant | None recorded | Unassessed |
| [EIP-2003](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2003.md) EVMC modules for implementations of precompiled contracts | Stagnant | 1352 | Unassessed |
| [EIP-2015](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2015.md) wallet_updateEthereumChain RPC Method | Stagnant | 155 | Unassessed |
| [EIP-2159](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2159.md) Common Prometheus Metrics Names for Clients | Final | None recorded | Unassessed |
| [EIP-2255](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2255.md) Wallet Permissions System | Final | 1193 | Unassessed |
| [EIP-2256](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2256.md) wallet_getOwnedAssets JSON-RPC Method | Stagnant | 55, 155, 1474 | Unassessed |
| [EIP-2566](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2566.md) Human Readable Parameters for Contract Function Execution | Stagnant | None recorded | Unassessed |
| [EIP-2696](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2696.md) JavaScript `request` method RPC transport | Final | None recorded | Unassessed |
| [EIP-2700](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2700.md) JavaScript Provider Event Emitter | Final | None recorded | Unassessed |
| [EIP-2786](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2786.md) Ethereum Provider Connect/Disconnect Events | Withdrawn | 2700 | Unassessed |
| [EIP-2831](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2831.md) Transaction Replacement Message Type | Stagnant | 1193 | Unassessed |
| [EIP-2844](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2844.md) Add DID related methods to the JSON-RPC | Stagnant | None recorded | Unassessed |
| [EIP-3014](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3014.md) eth_symbol JSON-RPC method | Stagnant | None recorded | Unassessed |
| [EIP-3030](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3030.md) BLS Remote Signer HTTP API | Stagnant | None recorded | Unassessed |
| [EIP-3041](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3041.md) Adds `baseFee` to `eth_getBlockByHash` | Stagnant | 1474, 1559 | Unassessed |
| [EIP-3044](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3044.md) Adds `baseFee` to `eth_getBlockByNumber` | Stagnant | 1474, 1559 | Unassessed |
| [EIP-3045](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3045.md) Adds `baseFee` to `eth_getUncleByBlockHashAndIndex` | Stagnant | 1474, 1559 | Unassessed |
| [EIP-3046](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3046.md) Adds `baseFee` to `eth_getUncleByBlockNumberAndIndex` | Stagnant | 1474, 1559 | Unassessed |
| [EIP-3076](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3076.md) Slashing Protection Interchange Format | Last Call | None recorded | I04, I05 |
| [EIP-3085](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3085.md) wallet_addEthereumChain RPC Method | Stagnant | 155 | Unassessed |
| [EIP-3091](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3091.md) Block Explorer API Routes | Stagnant | None recorded | Unassessed |
| [EIP-3155](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3155.md) EVM trace specification | Last Call | None recorded | Unassessed |
| [EIP-3326](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3326.md) Wallet Switch Ethereum Chain RPC Method (`wallet_switchEthereumChain`) | Stagnant | 155, 695 | Unassessed |
| [EIP-3709](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3709.md) Remove Support for Type 1 Transactions | Stagnant | 1559 | Unassessed |
| [EIP-4736](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4736.md) Consensus Layer Withdrawal Protection | Final | None recorded | Unassessed |
| [EIP-4881](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4881.md) Deposit Contract Snapshot Interface | Final | None recorded | Unassessed |
| [EIP-5345](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5345.md) Silent Signing Extension for JSON-RPC | Stagnant | None recorded | Unassessed |
| [EIP-5593](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5593.md) Restrict Ethereum Provider API Injection | Stagnant | 1193 | Unassessed |
| [EIP-5749](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5749.md) The 'window.evmproviders' object | Final | 1193 | Unassessed |
| [EIP-5792](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5792.md) Wallet Call API | Final | 1193 | Unassessed |
| [EIP-6051](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6051.md) Private Key Encapsulation | Stagnant | None recorded | Unassessed |
| [EIP-6789](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6789.md) Rename gas to mana | Stagnant | None recorded | Unassessed |
| [EIP-6963](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6963.md) Multi Injected Provider Discovery | Final | 1193 | A11 |
| [EIP-7039](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7039.md) Scheme-Handler Discovery Option for Wallets | Stagnant | 1193 | Unassessed |
| [EIP-7713](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7713.md) Box type for EIP-712 messages | Stagnant | 712 | Unassessed |
| [EIP-7749](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7749.md) Add wallet_signIntendedValidatorData method | Draft | 191, 712 | Unassessed |
| [EIP-7756](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7756.md) EOF/EVM Trace Specification | Stagnant | 3155, 4750 | Unassessed |
| [EIP-7867](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7867.md) Flow Control Wallet Call Capability | Stagnant | 5792 | Unassessed |
| [EIP-7896](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7896.md) ABI attachment in `wallet_sendCalls` | Stagnant | 5792 | Unassessed |
| [EIP-7910](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7910.md) eth_config JSON-RPC Method | Final | None recorded | Unassessed |
| [EIP-7966](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7966.md) eth_sendRawTransactionSync Method | Draft | None recorded | Unassessed |
| [EIP-8072](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8072.md) Transaction Inclusion Subscription | Draft | None recorded | Unassessed |
| [EIP-8123](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8123.md) RPC Method for Transaction Gas Limit Cap | Draft | 7825 | Unassessed |
| [EIP-8310](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8310.md) Post-Quantum Keystore for Stateful Keys | Draft | 2335, 3076 | Unassessed |
| [EIP-8317](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8317.md) Contract Storage Layout Descriptor Format | Draft | 7201, 7730, 8042 | Unassessed |

## ERC

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [ERC-20](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-20.md) Token Standard | Final | None recorded | A01, A04 |
| [ERC-55](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-55.md) Mixed-case checksum address encoding | Final | None recorded | Unassessed |
| [ERC-67](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-67.md) URI Scheme with Metadata, Value and Bytecode | Withdrawn | None recorded | A13 |
| [ERC-137](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-137.md) Ethereum Domain Name Service - Specification | Final | None recorded | A12 |
| [ERC-162](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-162.md) Initial ENS Hash Registrar | Final | None recorded | Unassessed |
| [ERC-165](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-165.md) Standard Interface Detection | Final | 214 | A02 |
| [ERC-173](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-173.md) Contract Ownership Standard | Final | None recorded | Unassessed |
| [ERC-181](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-181.md) ENS support for reverse resolution of Ethereum addresses | Final | None recorded | A12 |
| [ERC-190](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-190.md) Ethereum Smart Contract Packaging Standard | Final | None recorded | Unassessed |
| [ERC-191](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-191.md) Signed Data Standard | Final | None recorded | A03 |
| [ERC-205](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-205.md) ENS support for contract ABIs | Stagnant | 137, 181 | Unassessed |
| [ERC-223](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-223.md) Token with transaction handling model | Final | None recorded | A09 |
| [ERC-600](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-600.md) Ethereum purpose allocation for Deterministic Wallets | Final | None recorded | Unassessed |
| [ERC-601](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-601.md) Ethereum hierarchy for deterministic wallets | Final | None recorded | Unassessed |
| [ERC-634](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-634.md) Storage of text records in ENS | Stagnant | 137, 165 | A12 |
| [ERC-681](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-681.md) URL Format for Transaction Requests | Final | 20, 137 | A13 |
| [ERC-721](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-721.md) Non-Fungible Token Standard | Final | 165 | A02, A08 |
| [ERC-725](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-725.md) General data key/value store and execution | Draft | 165, 173 | Unassessed |
| [ERC-777](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-777.md) Token Standard | Final | 1820 | A09 |
| [ERC-801](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-801.md) Canary Standard | Stagnant | None recorded | Unassessed |
| [ERC-820](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-820.md) Pseudo-introspection Registry Contract | Final | 165, 214 | A09 |
| [ERC-823](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-823.md) Token Exchange Standard | Stagnant | 20 | Unassessed |
| [ERC-831](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-831.md) URI Format for Ethereum | Stagnant | 67, 681 | A13 |
| [ERC-838](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-838.md) ABI specification for REVERT reason string | Draft | None recorded | Unassessed |
| [ERC-875](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-875.md) Simpler NFT standard with batching and native atomic swaps | Withdrawn | None recorded | Unassessed |
| [ERC-884](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-884.md) DGCL Token | Stagnant | None recorded | Unassessed |
| [ERC-897](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-897.md) DelegateProxy | Stagnant | None recorded | Unassessed |
| [ERC-900](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-900.md) Simple Staking Interface | Stagnant | None recorded | Unassessed |
| [ERC-902](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-902.md) Token Validation | Stagnant | 1066 | Unassessed |
| [ERC-918](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-918.md) Mineable Token Standard | Stagnant | None recorded | Unassessed |
| [ERC-926](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-926.md) Address metadata registry | Stagnant | 165 | Unassessed |
| [ERC-927](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-927.md) Generalised authorisations | Stagnant | 926 | Unassessed |
| [ERC-998](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-998.md) Composable Non-Fungible Token | Draft | 20, 165, 721 | Unassessed |
| [ERC-1046](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1046.md) tokenURI Interoperability | Final | 20, 721, 1155 | A08 |
| [ERC-1056](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1056.md) Ethereum Lightweight Identity | Stagnant | None recorded | Unassessed |
| [ERC-1062](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1062.md) Formalize IPFS hash into ENS(Ethereum Name Service) resolver | Stagnant | None recorded | Unassessed |
| [ERC-1066](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1066.md) Status Codes | Stagnant | None recorded | Unassessed |
| [ERC-1077](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1077.md) Gas relay for contract calls | Stagnant | 20, 191, 1271, 1344 | Unassessed |
| [ERC-1078](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1078.md) Universal login / signup using ENS subdomains | Stagnant | 191, 681, 725, 1077 | Unassessed |
| [ERC-1080](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1080.md) Recoverable Token | Stagnant | None recorded | Unassessed |
| [ERC-1081](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1081.md) Standard Bounties | Stagnant | 20 | Unassessed |
| [ERC-1123](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1123.md) Revised Ethereum Smart Contract Packaging Standard | Withdrawn | None recorded | Unassessed |
| [ERC-1129](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1129.md) Standardised DAPP announcements | Stagnant | None recorded | Unassessed |
| [ERC-1132](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1132.md) Extending ERC20 with token locking capability | Stagnant | None recorded | Unassessed |
| [ERC-1154](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1154.md) Oracle Interface | Withdrawn | None recorded | Unassessed |
| [ERC-1155](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1155.md) Multi Token Standard | Final | 165 | A02, A08 |
| [ERC-1167](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1167.md) Minimal Proxy Contract | Final | 211 | A10 |
| [ERC-1175](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1175.md) Wallet & shop standard for all tokens (erc20) | Stagnant | 20 | Unassessed |
| [ERC-1178](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1178.md) Multi-class Token Standard | Stagnant | None recorded | Unassessed |
| [ERC-1185](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1185.md) Storage of DNS Records in ENS | Review | 137 | Unassessed |
| [ERC-1191](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1191.md) Add chain id to mixed-case checksum address encoding | Last Call | 55, 155 | Unassessed |
| [ERC-1202](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1202.md) Voting Interface | Review | 5269 | Unassessed |
| [ERC-1203](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1203.md) ERC-1203 Multi-Class Token Standard (ERC-20 Extension) | Stagnant | None recorded | Unassessed |
| [ERC-1207](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1207.md) DAuth Access Delegation Standard | Stagnant | None recorded | Unassessed |
| [ERC-1261](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1261.md) Membership Verification Token (MVT) | Stagnant | 165, 173 | Unassessed |
| [ERC-1271](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1271.md) Standard Signature Validation Method for Contracts | Final | None recorded | A05 |
| [ERC-1319](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1319.md) Smart Contract Package Registry Interface | Stagnant | None recorded | Unassessed |
| [ERC-1328](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1328.md) WalletConnect URI Format | Final | None recorded | Unassessed |
| [ERC-1337](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1337.md) Subscriptions on the blockchain | Stagnant | 20, 165 | Unassessed |
| [ERC-1363](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1363.md) Payable Token | Final | 20, 165 | A09 |
| [ERC-1386](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1386.md) Attestation management contract | Stagnant | None recorded | Unassessed |
| [ERC-1387](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1387.md) Merkle Tree Attestations with Privacy enabled | Stagnant | None recorded | Unassessed |
| [ERC-1388](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1388.md) Attestation Issuers Management List | Stagnant | None recorded | Unassessed |
| [ERC-1417](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1417.md) Poll Standard | Stagnant | 165, 1261 | Unassessed |
| [ERC-1438](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1438.md) dApp Components (avatar) & Universal Wallet | Stagnant | None recorded | Unassessed |
| [ERC-1444](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1444.md) Localized Messaging with Signal-to-Text | Stagnant | None recorded | Unassessed |
| [ERC-1450](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1450.md) RTA-Controlled Security Token | Final | 20, 165, 6093 | Unassessed |
| [ERC-1462](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1462.md) Base Security Token | Stagnant | 20, 1066 | Unassessed |
| [ERC-1484](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1484.md) Digital Identity Aggregator | Stagnant | 191 | Unassessed |
| [ERC-1491](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1491.md) Human Cost Accounting Standard (Like Gas but for humans) | Stagnant | None recorded | Unassessed |
| [ERC-1504](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1504.md) Upgradable Smart Contract | Stagnant | None recorded | Unassessed |
| [ERC-1523](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1523.md) Standard for Insurance Policies as ERC-721 Non Fungible Tokens | Stagnant | 721 | Unassessed |
| [ERC-1538](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1538.md) Transparent Contract Standard | Withdrawn | None recorded | Unassessed |
| [ERC-1577](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1577.md) contenthash field for ENS | Stagnant | None recorded | Unassessed |
| [ERC-1581](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1581.md) Non-wallet usage of keys derived from BIP-32 trees | Stagnant | None recorded | Unassessed |
| [ERC-1592](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1592.md) Address and ERC20-compliant transfer rules | Stagnant | None recorded | Unassessed |
| [ERC-1613](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1613.md) Gas Stations Network | Draft | 712, 2771 | Unassessed |
| [ERC-1616](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1616.md) Attribute Registry Standard | Stagnant | 165 | Unassessed |
| [ERC-1620](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1620.md) Money Streaming | Stagnant | None recorded | Unassessed |
| [ERC-1633](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1633.md) Re-Fungible Token Standard (RFT) | Stagnant | 20, 165, 721 | Unassessed |
| [ERC-1710](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1710.md) URL Format for Web3 Browsers | Stagnant | 155 | Unassessed |
| [ERC-1753](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1753.md) Smart Contract Interface for Licences | Stagnant | None recorded | Unassessed |
| [ERC-1761](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1761.md) Scoped Approval Interface | Stagnant | 165 | Unassessed |
| [ERC-1775](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1775.md) App Keys, application specific wallet accounts | Stagnant | None recorded | Unassessed |
| [ERC-1812](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1812.md) Ethereum Verifiable Claims | Stagnant | 712 | Unassessed |
| [ERC-1820](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1820.md) Pseudo-introspection Registry Contract | Final | 165, 214 | A09 |
| [ERC-1822](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1822.md) Universal Upgradeable Proxy Standard (UUPS) | Stagnant | None recorded | Unassessed |
| [ERC-1844](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1844.md) ENS Interface Discovery | Stagnant | 137, 165 | Unassessed |
| [ERC-1900](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1900.md) dType - Decentralized Type System for EVM | Stagnant | None recorded | Unassessed |
| [ERC-1921](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1921.md) dType Functions Extension | Stagnant | 1900 | Unassessed |
| [ERC-1922](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1922.md) zk-SNARK Verifier Standard | Stagnant | 165, 196, 197 | Unassessed |
| [ERC-1923](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1923.md) zk-SNARK Verifier Registry Standard | Stagnant | 165, 196, 197 | Unassessed |
| [ERC-1948](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1948.md) Non-fungible Data Token | Stagnant | 721 | Unassessed |
| [ERC-1967](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1967.md) Proxy Storage Slots | Final | None recorded | A10 |
| [ERC-1973](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1973.md) Scalable Rewards | Stagnant | None recorded | Unassessed |
| [ERC-1996](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-1996.md) Holdable Token | Stagnant | 20 | Unassessed |
| [ERC-2009](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2009.md) Compliance Service | Stagnant | 1066 | Unassessed |
| [ERC-2018](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2018.md) Clearable Token | Stagnant | 1996 | Unassessed |
| [ERC-2019](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2019.md) Fundable Token | Stagnant | 20 | Unassessed |
| [ERC-2020](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2020.md) E-Money Standard Token | Stagnant | 20, 1066, 1996, 2009, 2018, 2019, 2021 | Unassessed |
| [ERC-2021](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2021.md) Payoutable Token | Stagnant | 20, 1066, 1996 | Unassessed |
| [ERC-2098](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2098.md) Compact Signature Representation | Final | 2 | A03 |
| [ERC-2135](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2135.md) Consumable Interface (Tickets, etc) | Final | 165, 721, 1155 | Unassessed |
| [ERC-2157](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2157.md) dType Storage Extension - Decentralized Type System for EVM | Stagnant | 1900 | Unassessed |
| [ERC-2193](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2193.md) dType Alias Extension - Decentralized Type System | Stagnant | 155, 1900, 2157 | Unassessed |
| [ERC-2266](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2266.md) Atomic Swap-based American Call Option Contract Standard | Last Call | None recorded | Unassessed |
| [ERC-2304](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2304.md) Multichain address resolution for ENS | Stagnant | 137 | Unassessed |
| [ERC-2309](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2309.md) ERC-721 Consecutive Transfer Extension | Final | 721 | Unassessed |
| [ERC-2333](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2333.md) BLS12-381 Key Generation | Review | None recorded | C16 |
| [ERC-2334](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2334.md) BLS12-381 Deterministic Account Hierarchy | Review | 2333 | C16 |
| [ERC-2335](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2335.md) BLS12-381 Keystore | Review | 2333, 2334 | C05, I04, C16 |
| [ERC-2386](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2386.md) Ethereum 2 Hierarchical Deterministic Walletstore | Stagnant | 2334, 2335 | Unassessed |
| [ERC-2390](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2390.md) Geo-ENS | Stagnant | 137, 165, 1062, 1185 | Unassessed |
| [ERC-2400](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2400.md) Transaction Receipt URI | Stagnant | 155, 681 | Unassessed |
| [ERC-2470](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2470.md) Singleton Factory | Stagnant | 1014 | Unassessed |
| [ERC-2477](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2477.md) Token Metadata Integrity | Stagnant | 165, 721, 1155 | Unassessed |
| [ERC-2494](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2494.md) Baby Jubjub Elliptic Curve | Stagnant | None recorded | Unassessed |
| [ERC-2520](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2520.md) Multiple contenthash records for ENS | Stagnant | 1577 | Unassessed |
| [ERC-2525](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2525.md) ENSLogin | Stagnant | 137, 634, 1193, 2304 | Unassessed |
| [ERC-2535](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2535.md) Diamonds, Multi-Facet Proxy | Final | None recorded | A10 |
| [ERC-2544](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2544.md) ENS Wildcard Resolution | Stagnant | 137 | Unassessed |
| [ERC-2569](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2569.md) Saving and Displaying Image Onchain for Universal Tokens | Stagnant | None recorded | Unassessed |
| [ERC-2612](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2612.md) Permit Extension for EIP-20 Signed Approvals | Final | 20, 712 | A04 |
| [ERC-2615](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2615.md) Non-Fungible Token with mortgage and rental functions | Stagnant | 165, 721 | Unassessed |
| [ERC-2645](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2645.md) Hierarchical Deterministic Wallet for Layer-2 | Stagnant | None recorded | Unassessed |
| [ERC-2678](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2678.md) Revised Ethereum Smart Contract Packaging Standard (EthPM v3) | Final | None recorded | Unassessed |
| [ERC-2680](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2680.md) Ethereum 2 wallet layout | Stagnant | None recorded | Unassessed |
| [ERC-2746](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2746.md) Rules Engine Standard | Stagnant | None recorded | Unassessed |
| [ERC-2767](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2767.md) Contract Ownership Governance | Stagnant | 20, 165, 173 | Unassessed |
| [ERC-2770](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2770.md) Meta-Transactions Forwarder Contract | Withdrawn | 712, 2771 | Unassessed |
| [ERC-2771](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2771.md) Secure Protocol for Native Meta Transactions | Final | None recorded | Unassessed |
| [ERC-2848](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2848.md) My Own Messages (MOM) | Stagnant | None recorded | Unassessed |
| [ERC-2876](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2876.md) Deposit contract and address standard | Stagnant | None recorded | Unassessed |
| [ERC-2917](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2917.md) Staking Reward Calculation | Stagnant | None recorded | Unassessed |
| [ERC-2942](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2942.md) EthPM URI Specification | Stagnant | 2678 | Unassessed |
| [ERC-2980](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2980.md) Swiss Compliant Asset Token | Stagnant | 20 | Unassessed |
| [ERC-2981](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2981.md) NFT Royalty Standard | Final | 165 | A08 |
| [ERC-3000](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3000.md) Optimistic enactment governance standard | Stagnant | None recorded | Unassessed |
| [ERC-3005](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3005.md) Batched meta transactions | Stagnant | None recorded | Unassessed |
| [ERC-3009](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3009.md) Transfer With Authorization | Draft | 20, 712 | Unassessed |
| [ERC-3135](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3135.md) Exclusive Claimable Token | Stagnant | 20 | Unassessed |
| [ERC-3156](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3156.md) Flash Loans | Final | None recorded | Unassessed |
| [ERC-3224](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3224.md) Described Data | Stagnant | 191 | Unassessed |
| [ERC-3234](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3234.md) Batch Flash Loans | Stagnant | None recorded | Unassessed |
| [ERC-3386](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3386.md) ERC-721 and ERC-1155 to ERC-20 Wrapper | Stagnant | 165 | Unassessed |
| [ERC-3440](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3440.md) ERC-721 Editions Standard | Stagnant | 712, 721 | Unassessed |
| [ERC-3448](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3448.md) MetaProxy Standard | Final | None recorded | Unassessed |
| [ERC-3450](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3450.md) Standardized Shamir Secret Sharing Scheme for BIP-39 Mnemonics | Stagnant | None recorded | Unassessed |
| [ERC-3475](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3475.md) Abstract Storage Bonds | Final | 20, 721, 1155 | Unassessed |
| [ERC-3525](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3525.md) Semi-Fungible Token | Final | 20, 165, 721 | Unassessed |
| [ERC-3561](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3561.md) Trust Minimized Upgradeability Proxy | Stagnant | None recorded | Unassessed |
| [ERC-3569](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3569.md) Sealed NFT Metadata Standard | Stagnant | None recorded | Unassessed |
| [ERC-3589](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3589.md) Assemble assets into NFTs | Stagnant | 721 | Unassessed |
| [ERC-3643](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3643.md) T-REX - Token for Regulated EXchanges | Final | 20, 173 | Unassessed |
| [ERC-3668](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3668.md) CCIP Read—Secure offchain data retrieval | Final | None recorded | Unassessed |
| [ERC-3722](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3722.md) Poster | Stagnant | None recorded | Unassessed |
| [ERC-3754](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3754.md) A Vanilla Non-Fungible Token Standard | Stagnant | None recorded | Unassessed |
| [ERC-3770](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3770.md) Chain-specific addresses | Draft | None recorded | Unassessed |
| [ERC-3772](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-3772.md) Compressed Integers | Stagnant | None recorded | Unassessed |
| [ERC-4337](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4337.md) Account Abstraction Using Alt Mempool | Final | 712, 7702 | A06 |
| [ERC-4341](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4341.md) Ordered NFT Batch Standard | Stagnant | None recorded | Unassessed |
| [ERC-4353](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4353.md) Interface for Staked Tokens in NFTs | Stagnant | 165 | Unassessed |
| [ERC-4361](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4361.md) Sign-In with Ethereum | Final | 55, 137, 155, 191, 1271, 1328 | Unassessed |
| [ERC-4393](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4393.md) Micropayments for NFTs and Multi Tokens | Stagnant | 165, 721, 1155 | Unassessed |
| [ERC-4400](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4400.md) EIP-721 Consumable Extension | Final | 165, 721 | Unassessed |
| [ERC-4430](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4430.md) Described Transactions | Stagnant | None recorded | Unassessed |
| [ERC-4494](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4494.md) Permit for ERC-721 NFTs | Stagnant | 165, 712, 721 | A04 |
| [ERC-4519](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4519.md) Non-Fungible Tokens Tied to Physical Assets | Final | 165, 721 | Unassessed |
| [ERC-4521](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4521.md) 721/20-compatible transfer | Stagnant | 721 | Unassessed |
| [ERC-4524](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4524.md) Safer ERC-20 | Stagnant | 20, 165 | Unassessed |
| [ERC-4527](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4527.md) QR Code transmission protocol for wallets | Stagnant | None recorded | Unassessed |
| [ERC-4546](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4546.md) Wrapped Deposits | Stagnant | None recorded | Unassessed |
| [ERC-4626](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4626.md) Tokenized Vaults | Final | 20, 2612 | A07 |
| [ERC-4671](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4671.md) Non-Tradable Tokens Standard | Stagnant | 165 | Unassessed |
| [ERC-4675](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4675.md) Multi-Fractional Non-Fungible Tokens | Stagnant | 165, 721 | Unassessed |
| [ERC-4799](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4799.md) Non-Fungible Token Ownership Designation Standard | Stagnant | 165 | Unassessed |
| [ERC-4804](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4804.md) Web3 URL to EVM Call Message Translation | Final | 137 | Unassessed |
| [ERC-4824](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4824.md) Common Interfaces for DAOs | Review | None recorded | A15 |
| [ERC-4834](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4834.md) Hierarchical Domains | Final | None recorded | Unassessed |
| [ERC-4883](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4883.md) Composable SVG NFT | Draft | 721 | Unassessed |
| [ERC-4885](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4885.md) Subscription NFTs and Multi Tokens | Stagnant | 165, 721, 1155 | Unassessed |
| [ERC-4886](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4886.md) Proxy Ownership Register | Stagnant | None recorded | Unassessed |
| [ERC-4906](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4906.md) EIP-721 Metadata Update Extension | Final | 165, 721 | A08 |
| [ERC-4907](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4907.md) Rental NFT, an Extension of EIP-721 | Final | 165, 721 | Unassessed |
| [ERC-4910](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4910.md) Royalty Bearing NFTs | Final | 165, 721 | Unassessed |
| [ERC-4931](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4931.md) Generic Token Upgrade Standard | Stagnant | 20 | Unassessed |
| [ERC-4944](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4944.md) Contract with Exactly One Non-fungible Token | Stagnant | 721 | Unassessed |
| [ERC-4950](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4950.md) Entangled Tokens | Stagnant | 20, 721, 1155 | Unassessed |
| [ERC-4955](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4955.md) Vendor Metadata Extension for NFTs | Final | 721, 1155 | Unassessed |
| [ERC-4972](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4972.md) Name-Owned Account | Draft | 137 | Unassessed |
| [ERC-4973](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4973.md) Account-bound Tokens | Review | 165, 712, 721, 1271 | Unassessed |
| [ERC-4974](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4974.md) Ratings | Stagnant | 165 | Unassessed |
| [ERC-4987](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-4987.md) Held token interface | Stagnant | 20, 165, 721, 1155 | Unassessed |
| [ERC-5005](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5005.md) Zodiac Modular Accounts | Stagnant | 165 | Unassessed |
| [ERC-5006](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5006.md) Rental NFT, NFT User Extension | Final | 165, 1155 | Unassessed |
| [ERC-5007](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5007.md) Time NFT, ERC-721 Time Extension | Final | 165, 721 | Unassessed |
| [ERC-5008](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5008.md) ERC-721 Nonce Extension | Last Call | 165, 721 | Unassessed |
| [ERC-5018](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5018.md) Filesystem-like Interface for Contracts | Stagnant | None recorded | Unassessed |
| [ERC-5023](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5023.md) Shareable Non-Fungible Token | Final | 165 | Unassessed |
| [ERC-5050](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5050.md) Interactive NFTs with Modular Environments | Stagnant | 165, 173, 721, 1155, 1820, 4906 | Unassessed |
| [ERC-5058](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5058.md) Lockable Non-Fungible Tokens | Stagnant | 20, 165, 721 | Unassessed |
| [ERC-5094](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5094.md) URL Format for Ethereum Network Switching | Stagnant | 681, 831 | Unassessed |
| [ERC-5095](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5095.md) Principal Token | Stagnant | 20, 2612 | Unassessed |
| [ERC-5114](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5114.md) Soulbound Badge | Last Call | None recorded | Unassessed |
| [ERC-5115](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5115.md) SY Token | Draft | 20 | Unassessed |
| [ERC-5131](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5131.md) SAFE Authentication For ENS | Stagnant | 137, 181, 634 | Unassessed |
| [ERC-5139](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5139.md) Remote Procedure Call Provider Lists | Stagnant | 155, 1577 | Unassessed |
| [ERC-5143](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5143.md) Slippage Protection for Tokenized Vault | Stagnant | 20, 4626 | Unassessed |
| [ERC-5164](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5164.md) Cross-Chain Execution | Last Call | None recorded | A14 |
| [ERC-5169](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5169.md) Client Script URI for Token Contracts | Final | 20, 165, 721, 777, 1155 | Unassessed |
| [ERC-5173](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5173.md) NFT Future Rewards (nFR) | Draft | 165, 721 | Unassessed |
| [ERC-5185](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5185.md) NFT Updatable Metadata Extension | Stagnant | 721, 1155 | Unassessed |
| [ERC-5187](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5187.md) Extend EIP-1155 with rentable usage rights | Stagnant | 165, 1155 | Unassessed |
| [ERC-5189](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5189.md) Account Abstraction via Endorsed Operations | Draft | None recorded | Unassessed |
| [ERC-5192](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5192.md) Minimal Soulbound NFTs | Final | 165, 721 | Unassessed |
| [ERC-5202](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5202.md) Blueprint contract format | Final | 170 | Unassessed |
| [ERC-5216](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5216.md) ERC-1155 Allowance Extension | Last Call | 20, 165, 1155 | Unassessed |
| [ERC-5218](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5218.md) NFT Rights Management | Stagnant | 721 | Unassessed |
| [ERC-5219](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5219.md) Contract Resource Requests | Final | None recorded | Unassessed |
| [ERC-5247](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5247.md) Smart Contract Executable Proposal Interface | Review | None recorded | Unassessed |
| [ERC-5252](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5252.md) Account-bound Finance | Stagnant | 20, 721, 1155, 5114 | Unassessed |
| [ERC-5267](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5267.md) Retrieval of EIP-712 domain | Final | 155, 712, 2612 | A03 |
| [ERC-5269](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5269.md) ERC Detection and Discovery | Review | 5750 | Unassessed |
| [ERC-5289](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5289.md) Ethereum Notary Interface | Review | 165, 5568 | Unassessed |
| [ERC-5298](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5298.md) ENS Trust to hold NFTs under ENS name | Stagnant | 137, 721, 1155 | Unassessed |
| [ERC-5313](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5313.md) Light Contract Ownership | Final | 165, 173 | Unassessed |
| [ERC-5334](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5334.md) EIP-721 User And Expires And Level Extension | Stagnant | 165, 721 | Unassessed |
| [ERC-5375](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5375.md) NFT Author Information and Consent | Final | 55, 155, 712, 721, 1155 | Unassessed |
| [ERC-5380](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5380.md) ERC-721 Entitlement Extension | Final | 165, 721, 1046 | Unassessed |
| [ERC-5409](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5409.md) EIP-1155 Non-Fungible Token extension | Stagnant | 165, 721, 1155 | Unassessed |
| [ERC-5437](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5437.md) Security Contact Interface | Stagnant | 165 | Unassessed |
| [ERC-5453](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5453.md) Endorsement - Permit for Any Functions | Last Call | 165, 712, 1271, 5750 | Unassessed |
| [ERC-5484](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5484.md) Consensual Soulbound Tokens | Final | 165, 721 | Unassessed |
| [ERC-5485](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5485.md) Jurisdiction, Accreditation, and Enforcement | Review | 5247 | Unassessed |
| [ERC-5489](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5489.md) NFT Hyperlink Extension | Final | 165, 721 | Unassessed |
| [ERC-5496](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5496.md) Multi-privilege Management NFT Extension | Last Call | 721 | Unassessed |
| [ERC-5501](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5501.md) Rental & Delegation NFT - EIP-721 Extension | Stagnant | 165, 721, 4400, 4907 | Unassessed |
| [ERC-5505](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5505.md) EIP-1155 asset backed NFT extension | Stagnant | 1155 | Unassessed |
| [ERC-5507](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5507.md) Refundable Tokens | Final | 20, 165, 721, 1155 | Unassessed |
| [ERC-5516](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5516.md) Soulbound Multi-owner Tokens | Final | 165 | Unassessed |
| [ERC-5521](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5521.md) Referable NFT | Final | 165, 721 | Unassessed |
| [ERC-5528](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5528.md) Refundable Fungible Token | Final | 20 | Unassessed |
| [ERC-5539](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5539.md) Revocation List Registry | Stagnant | 712 | Unassessed |
| [ERC-5553](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5553.md) Representing IP and its Royalty Structure | Stagnant | 20, 721 | Unassessed |
| [ERC-5554](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5554.md) NFT Legal Use, Repurposing, and Remixing | Stagnant | 5218 | Unassessed |
| [ERC-5559](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5559.md) Cross Chain Write Deferral Protocol | Stagnant | 712 | Unassessed |
| [ERC-5560](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5560.md) Redeemable NFTs | Stagnant | 165, 721 | Unassessed |
| [ERC-5564](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5564.md) Stealth Addresses | Final | None recorded | Unassessed |
| [ERC-5568](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5568.md) Well-Known Format for Required Actions | Review | 140 | Unassessed |
| [ERC-5570](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5570.md) Digital Receipt Non-Fungible Tokens | Final | 721 | Unassessed |
| [ERC-5573](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5573.md) Sign-In with Ethereum Capabilities, ReCaps | Draft | 4361 | Unassessed |
| [ERC-5585](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5585.md) ERC-721 NFT Authorization | Final | 721 | Unassessed |
| [ERC-5604](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5604.md) NFT Lien | Draft | 165, 721 | Unassessed |
| [ERC-5606](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5606.md) Multiverse NFTs | Final | 721, 1155 | Unassessed |
| [ERC-5615](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5615.md) ERC-1155 Supply Extension | Final | 1155 | Unassessed |
| [ERC-5625](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5625.md) NFT Metadata JSON Schema dStorage Extension | Final | 721, 1155 | Unassessed |
| [ERC-5630](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5630.md) New approach for encryption / decryption | Draft | None recorded | Unassessed |
| [ERC-5633](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5633.md) Composable Soulbound NFT, EIP-1155 Extension | Stagnant | 165, 1155 | Unassessed |
| [ERC-5635](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5635.md) NFT Licensing Agreements | Stagnant | 165, 721, 1155, 2981 | Unassessed |
| [ERC-5639](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5639.md) Delegation Registry | Review | None recorded | Unassessed |
| [ERC-5643](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5643.md) Subscription NFTs | Stagnant | 721 | Unassessed |
| [ERC-5646](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5646.md) Token State Fingerprint | Final | 165 | Unassessed |
| [ERC-5679](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5679.md) Token Minting and Burning | Final | 20, 165, 721, 1155 | Unassessed |
| [ERC-5700](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5700.md) Bindable Token Interface | Draft | 165, 721, 1155 | Unassessed |
| [ERC-5719](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5719.md) Signature replacement interface | Stagnant | 1271 | Unassessed |
| [ERC-5725](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5725.md) Transferable Vesting NFT | Final | 721 | Unassessed |
| [ERC-5727](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5727.md) Semi-Fungible Soulbound Token | Draft | 165, 712, 721, 3525, 4906, 5192, 5484 | Unassessed |
| [ERC-5732](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5732.md) Commit Interface | Final | 165, 1271 | Unassessed |
| [ERC-5744](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5744.md) Latent Fungible Token | Stagnant | 20, 2612 | Unassessed |
| [ERC-5750](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5750.md) General Extensibility for Method Behaviors | Final | 165 | Unassessed |
| [ERC-5753](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5753.md) Lockable Extension for EIP-721 | Stagnant | 165, 721 | Unassessed |
| [ERC-5773](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5773.md) Context-Dependent Multi-Asset Tokens | Final | 165, 721 | Unassessed |
| [ERC-5791](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5791.md) Physical Backed Tokens | Draft | 191, 721 | Unassessed |
| [ERC-5805](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5805.md) Voting with delegation | Review | 712, 6372 | A15 |
| [ERC-5827](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5827.md) Auto-renewable allowance extension | Stagnant | 20, 165 | Unassessed |
| [ERC-5850](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5850.md) Complex Numbers stored in `bytes32` types | Stagnant | None recorded | Unassessed |
| [ERC-5851](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5851.md) On-Chain Verifiable Credentials | Stagnant | 721, 1155, 1167, 1967, 3475 | Unassessed |
| [ERC-5883](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5883.md) Token Transfer by Social Recovery | Stagnant | None recorded | Unassessed |
| [ERC-5902](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5902.md) Smart Contract Event Hooks | Stagnant | 712 | Unassessed |
| [ERC-5982](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-5982.md) Role-based Access Control | Review | 165, 5750 | Unassessed |
| [ERC-6047](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6047.md) ERC-721 Balance indexing via Transfer event | Stagnant | 721 | Unassessed |
| [ERC-6059](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6059.md) Parent-Governed Nestable Non-Fungible Tokens | Final | 165, 721 | Unassessed |
| [ERC-6065](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6065.md) Real Estate Token | Review | 721 | Unassessed |
| [ERC-6066](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6066.md) Signature Validation Method for NFTs | Final | 165, 721, 1155, 1271, 5750 | Unassessed |
| [ERC-6093](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6093.md) Custom errors for commonly-used tokens | Final | 20, 721, 1155 | S04 |
| [ERC-6105](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6105.md) No Intermediary NFT Trading Protocol | Final | 20, 165, 721, 2981 | Unassessed |
| [ERC-6120](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6120.md) Universal Token Router | Review | 20, 165, 721, 1014, 1155 | Unassessed |
| [ERC-6123](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6123.md) Smart Derivative Contract | Draft | None recorded | Unassessed |
| [ERC-6147](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6147.md) Guard of NFT/SBT, an Extension of ERC-721 | Final | 165, 721 | Unassessed |
| [ERC-6150](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6150.md) Hierarchical NFTs | Final | 165, 721 | Unassessed |
| [ERC-6170](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6170.md) Cross-Chain Messaging Interface | Draft | None recorded | Unassessed |
| [ERC-6220](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6220.md) Composable NFTs utilizing Equippable Parts | Final | 165, 721, 5773, 6059 | Unassessed |
| [ERC-6224](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6224.md) Contracts Dependencies Registry | Last Call | 1967, 5750 | Unassessed |
| [ERC-6229](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6229.md) Tokenized Vaults with Lock-in Period | Draft | 4626 | Unassessed |
| [ERC-6239](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6239.md) Semantic Soulbound Tokens | Final | 165, 721, 5192 | Unassessed |
| [ERC-6268](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6268.md) Untransferability Indicator for EIP-1155 | Stagnant | 165, 1155 | Unassessed |
| [ERC-6315](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6315.md) ERC-2771 Namespaced Account Abstraction | Review | 165, 2771 | Unassessed |
| [ERC-6327](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6327.md) Elastic Signature | Draft | None recorded | Unassessed |
| [ERC-6353](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6353.md) Charity token | Stagnant | 20 | Unassessed |
| [ERC-6357](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6357.md) Single-contract Multi-delegatecall | Last Call | None recorded | Unassessed |
| [ERC-6358](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6358.md) Cross-Chain Token States Synchronization | Review | None recorded | Unassessed |
| [ERC-6366](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6366.md) Permission Token | Review | 6617 | Unassessed |
| [ERC-6372](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6372.md) Contract clock | Review | None recorded | A15 |
| [ERC-6381](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6381.md) Public Non-Fungible Token Emote Repository | Final | 165 | Unassessed |
| [ERC-6384](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6384.md) Human-readable offline signatures | Stagnant | 712 | Unassessed |
| [ERC-6454](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6454.md) Minimal Transferable NFT detection interface | Final | 165, 721 | Unassessed |
| [ERC-6464](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6464.md) Multi-operator, per-token ERC-721 approvals. | Stagnant | 165, 721 | Unassessed |
| [ERC-6492](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6492.md) Signature Validation for Predeploy Contracts | Final | 1271 | A05 |
| [ERC-6506](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6506.md) P2P Escrowed Governance Incentives | Stagnant | None recorded | Unassessed |
| [ERC-6538](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6538.md) Stealth Meta-Address Registry | Final | 712, 1271, 5564 | Unassessed |
| [ERC-6551](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6551.md) Non-fungible Token Bound Accounts | Review | 165, 721, 1167, 1271 | Unassessed |
| [ERC-6596](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6596.md) Cultural and Historical Asset Token | Review | 721, 1155 | Unassessed |
| [ERC-6604](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6604.md) Abstract Token | Draft | 20, 165, 721, 1155 | Unassessed |
| [ERC-6617](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6617.md) Bit Based Permission | Review | None recorded | Unassessed |
| [ERC-6662](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6662.md) AA Account Metadata For Authentication | Draft | 4337, 4804 | Unassessed |
| [ERC-6672](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6672.md) Multi-redeemable NFTs | Final | 165, 721 | Unassessed |
| [ERC-6682](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6682.md) NFT Flashloans | Draft | 20, 721, 3156 | Unassessed |
| [ERC-6734](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6734.md) L2 Token List | Review | 155 | Unassessed |
| [ERC-6735](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6735.md) L2 Aliasing of EVM-based Addresses | Review | 55 | Unassessed |
| [ERC-6785](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6785.md) ERC-721 Utilities Information Extension | Draft | 165, 721 | Unassessed |
| [ERC-6786](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6786.md) Registry for royalties payment for NFTs | Draft | 165, 2981 | Unassessed |
| [ERC-6787](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6787.md) Order Book DEX with Two Phase Withdrawal | Draft | None recorded | Unassessed |
| [ERC-6806](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6806.md) ERC-721 Holding Time Tracking | Draft | 721 | Unassessed |
| [ERC-6808](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6808.md) Fungible Key Bound Token | Final | 20 | Unassessed |
| [ERC-6809](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6809.md) Non-Fungible Key Bound Token | Final | 721 | Unassessed |
| [ERC-6821](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6821.md) Support ENS Name for Web3 URL | Draft | 137, 634, 3770, 4804 | Unassessed |
| [ERC-6823](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6823.md) Token Mapping Slot Retrieval Extension | Draft | 20, 721, 1155 | Unassessed |
| [ERC-6860](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6860.md) Web3 URL to EVM Call Message Translation | Draft | 137 | Unassessed |
| [ERC-6864](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6864.md) Upgradable Fungible Token | Draft | 20 | Unassessed |
| [ERC-6865](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6865.md) On-Chain EIP-712 Visualization | Draft | 712 | Unassessed |
| [ERC-6900](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6900.md) Modular Smart Contract Accounts | Draft | 165, 1271, 4337 | A06 |
| [ERC-6909](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6909.md) Minimal Multi-Token Interface | Final | 165 | Unassessed |
| [ERC-6932](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6932.md) Subscription-Based Token | Draft | 20 | Unassessed |
| [ERC-6944](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6944.md) ERC-5219 Resolve Mode | Draft | 4804, 5219 | Unassessed |
| [ERC-6956](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6956.md) Asset-bound Non-Fungible Tokens | Review | 165, 721 | Unassessed |
| [ERC-6960](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6960.md) Dual Layer Token | Draft | None recorded | Unassessed |
| [ERC-6981](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6981.md) Reserved Ownership Accounts | Draft | 1167, 1271, 6492 | Unassessed |
| [ERC-6982](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6982.md) Efficient Default Lockable Tokens | Final | 165, 721 | Unassessed |
| [ERC-6997](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-6997.md) ERC-721 with transaction validation step. | Review | 721 | Unassessed |
| [ERC-7007](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7007.md) Verifiable AI-Generated Content Token | Final | 165, 721 | Unassessed |
| [ERC-7015](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7015.md) NFT Creator Attribution | Review | 55, 155, 712, 721, 1155 | Unassessed |
| [ERC-7053](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7053.md) Interoperable Digital Media Indexing | Final | None recorded | Unassessed |
| [ERC-7066](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7066.md) Lockable Extension for ERC-721 | Final | 165, 721 | Unassessed |
| [ERC-7085](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7085.md) NFT Relationship Enhancement | Draft | 721, 1155 | Unassessed |
| [ERC-7087](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7087.md) MIME type for Web3 URL in Auto Mode | Draft | 6860 | Unassessed |
| [ERC-7092](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7092.md) Financial Bonds | Final | 165 | Unassessed |
| [ERC-7093](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7093.md) Social Recovery Interface | Draft | 1271 | Unassessed |
| [ERC-7144](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7144.md) ERC-20 with transaction validation step. | Review | 20 | Unassessed |
| [ERC-7160](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7160.md) ERC-721 Multi-Metadata Extension | Final | 165, 721 | Unassessed |
| [ERC-7196](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7196.md) Simple token, Simplified ERC-20 | Draft | 20 | Unassessed |
| [ERC-7201](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7201.md) Namespaced Storage Layout | Final | None recorded | Unassessed |
| [ERC-7204](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7204.md) Contract wallet management token | Draft | 165 | Unassessed |
| [ERC-7208](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7208.md) On-Chain Data Containers | Final | 165 | Unassessed |
| [ERC-7231](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7231.md) Identity-aggregated NFT | Final | 165, 721, 1271 | Unassessed |
| [ERC-7246](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7246.md) Encumber - Splitting Ownership & Guarantees | Review | 20 | Unassessed |
| [ERC-7254](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7254.md) Token Revenue Sharing | Draft | None recorded | Unassessed |
| [ERC-7272](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7272.md) Ethereum Access Token | Draft | 712 | Unassessed |
| [ERC-7280](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7280.md) NFT Metadata Extension like JSON-LD | Draft | 721, 1155, 3525 | Unassessed |
| [ERC-7291](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7291.md) Purpose bound money | Final | 165, 173, 1155 | Unassessed |
| [ERC-7303](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7303.md) Token-Controlled Token Circulation | Draft | 165, 721, 1155, 5679 | Unassessed |
| [ERC-7390](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7390.md) Vanilla Options for ERC-20 Tokens | Draft | 20, 1155 | Unassessed |
| [ERC-7399](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7399.md) ⚡ Flash Loans ⚡ | Review | 20 | Unassessed |
| [ERC-7401](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7401.md) Parent-Governed Non-Fungible Tokens Nesting | Final | 165, 721 | Unassessed |
| [ERC-7405](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7405.md) Portable Smart Contract Accounts | Draft | 191, 1967 | Unassessed |
| [ERC-7406](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7406.md) Multi-Namespace Onchain Registry | Draft | 137 | Unassessed |
| [ERC-7409](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7409.md) Public Non-Fungible Tokens Emote Repository | Final | 165 | Unassessed |
| [ERC-7410](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7410.md) ERC-20 Update Allowance By Spender | Draft | 20, 165 | Unassessed |
| [ERC-7412](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7412.md) On-Demand Off-Chain Data Retrieval | Draft | None recorded | Unassessed |
| [ERC-7417](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7417.md) Token Converter | Review | 20, 165, 223 | Unassessed |
| [ERC-7425](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7425.md) Tokenized Reserve | Draft | 20, 4626 | Unassessed |
| [ERC-7432](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7432.md) Non-Fungible Token Roles | Final | 165, 721 | Unassessed |
| [ERC-7439](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7439.md) Prevent ticket touting | Final | 165, 721 | Unassessed |
| [ERC-7444](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7444.md) Time Locks Maturity | Draft | 165 | Unassessed |
| [ERC-7484](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7484.md) Registry Extension for ERC-7579 | Draft | 7579 | Unassessed |
| [ERC-7496](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7496.md) NFT Dynamic Traits | Draft | 165, 721, 1155 | Unassessed |
| [ERC-7498](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7498.md) NFT Redeemables | Draft | 165, 712, 721, 1155, 1271 | Unassessed |
| [ERC-7506](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7506.md) Trusted Hint Registry | Draft | 712 | Unassessed |
| [ERC-7507](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7507.md) Multi-User NFT Extension | Draft | 721 | Unassessed |
| [ERC-7508](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7508.md) Dynamic On-Chain Token Attributes Repository | Draft | 165 | Unassessed |
| [ERC-7509](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7509.md) Entity Component System | Draft | None recorded | Unassessed |
| [ERC-7510](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7510.md) Cross-Contract Hierarchical NFT | Draft | 721 | Unassessed |
| [ERC-7511](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7511.md) Minimal Proxy Contract with PUSH0 | Draft | 7, 211, 1167, 3855 | Unassessed |
| [ERC-7512](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7512.md) Onchain Representation for Audits | Draft | 712 | Unassessed |
| [ERC-7513](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7513.md) Smart NFT - A Component for Intent-Centric | Draft | 165, 1155 | Unassessed |
| [ERC-7517](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7517.md) Content Consent for AI/ML Data Mining | Draft | 721, 7053 | Unassessed |
| [ERC-7518](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7518.md) Dynamic Compliant Interop Security Token | Review | 165, 712, 1155, 1271, 5267 | Unassessed |
| [ERC-7521](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7521.md) General Intents for Smart Contract Wallets | Draft | None recorded | Unassessed |
| [ERC-7522](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7522.md) OIDC ZK Verifier for AA Account | Draft | 4337 | Unassessed |
| [ERC-7524](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7524.md) PLUME Signature in Wallets | Draft | None recorded | Unassessed |
| [ERC-7527](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7527.md) Token Bound Function Oracle AMM | Draft | 165, 721 | Unassessed |
| [ERC-7528](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7528.md) ETH (Native Asset) Address Convention | Final | 20, 55, 4626 | Unassessed |
| [ERC-7529](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7529.md) Contract Discovery and eTLD+1 Association | Draft | 1191 | Unassessed |
| [ERC-7531](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7531.md) Staked ERC-721 Ownership Recognition | Review | 165, 721 | Unassessed |
| [ERC-7533](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7533.md) Public Cross Port | Draft | None recorded | Unassessed |
| [ERC-7535](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7535.md) Native Asset ERC-4626 Tokenized Vault | Final | 20, 4626, 7528 | Unassessed |
| [ERC-7538](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7538.md) Multiplicative Tokens | Draft | 20, 1046, 1155 | Unassessed |
| [ERC-7540](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7540.md) Asynchronous ERC-4626 Tokenized Vaults | Final | 20, 165, 4626, 7575 | A07 |
| [ERC-7546](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7546.md) Upgradeable Clone for Scalable Contracts | Draft | 165, 1967, 7201 | Unassessed |
| [ERC-7548](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7548.md) Open IP Protocol built on NFTs | Draft | 165, 721 | Unassessed |
| [ERC-7555](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7555.md) Single Sign-On for Account Discovery | Draft | 4337 | Unassessed |
| [ERC-7561](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7561.md) Simple NFT, Simplified ERC-721 | Draft | 721 | Unassessed |
| [ERC-7562](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7562.md) Account Abstraction Validation Scope Rules | Review | None recorded | Unassessed |
| [ERC-7564](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7564.md) Contract wallet management NFT | Draft | 165 | Unassessed |
| [ERC-7565](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7565.md) Perpetual Contract NFTs as Collateral | Draft | 721, 4907 | Unassessed |
| [ERC-7566](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7566.md) Multiplayer Game Communication | Draft | None recorded | Unassessed |
| [ERC-7572](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7572.md) Contract-level metadata via `contractURI()` | Draft | None recorded | Unassessed |
| [ERC-7573](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7573.md) Conditional-upon-Transfer-Decryption for DvP | Draft | None recorded | Unassessed |
| [ERC-7574](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7574.md) Authentication SBT using Credential | Draft | 165, 721, 5192 | Unassessed |
| [ERC-7575](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7575.md) Multi-Asset ERC-4626 Vaults | Final | 20, 165, 2771, 4626 | A07 |
| [ERC-7578](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7578.md) Physical Asset Redemption | Final | 165, 721 | Unassessed |
| [ERC-7579](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7579.md) Minimal Modular Smart Accounts | Draft | 165, 1271, 2771, 4337 | A06 |
| [ERC-7580](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7580.md) Advertisement Tracking Interface | Draft | None recorded | Unassessed |
| [ERC-7582](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7582.md) Modular Accounts with Delegated Validation | Draft | 4337 | Unassessed |
| [ERC-7585](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7585.md) MixHash and Public Data Storage Proofs | Draft | 165, 721, 1155 | Unassessed |
| [ERC-7586](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7586.md) Interest Rate Swaps | Review | 20, 165 | Unassessed |
| [ERC-7588](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7588.md) Blob Transactions Metadata JSON Schema | Final | 4844 | Unassessed |
| [ERC-7589](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7589.md) Semi-Fungible Token Roles | Draft | 165, 1155 | Unassessed |
| [ERC-7590](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7590.md) ERC-20 Holder Extension for NFTs | Review | 20, 165, 721 | Unassessed |
| [ERC-7595](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7595.md) Collateralized NFT | Draft | 20, 721 | Unassessed |
| [ERC-7597](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7597.md) Signature Validation Extension for Permit | Draft | 1271, 2612 | Unassessed |
| [ERC-7598](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7598.md) Use contract signature for signed transfer | Draft | 1271, 3009 | Unassessed |
| [ERC-7603](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7603.md) ERC-1155 Multi-Asset extension | Draft | 165, 1155 | Unassessed |
| [ERC-7604](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7604.md) ERC-1155 Permit Approvals | Draft | 165, 712, 1155, 1271, 2098, 5216 | Unassessed |
| [ERC-7613](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7613.md) Puppet Proxy Contract | Draft | None recorded | Unassessed |
| [ERC-7615](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7615.md) Atomic Push-based Data Feed Among Contracts | Draft | None recorded | Unassessed |
| [ERC-7617](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7617.md) Chunk support for ERC-5219 mode in Web3 URL | Draft | 5219, 6944 | Unassessed |
| [ERC-7618](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7618.md) Content encoding in ERC-5219 mode Web3 URL | Draft | 5219, 6944 | Unassessed |
| [ERC-7621](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7621.md) Basket Token | Draft | 20, 165, 173 | Unassessed |
| [ERC-7627](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7627.md) Secure Messaging Protocol | Final | None recorded | Unassessed |
| [ERC-7628](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7628.md) ERC-721 Ownership Shares Extension | Review | 721 | Unassessed |
| [ERC-7629](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7629.md) ERC-20/ERC-721 Unified Token Interface | Draft | 20, 165, 721 | Unassessed |
| [ERC-7631](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7631.md) Dual Nature Token Pair | Final | 20, 721 | Unassessed |
| [ERC-7632](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7632.md) Interfaces for Named Token | Draft | 165 | Unassessed |
| [ERC-7634](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7634.md) Limited Transfer Count NFT | Final | 165, 721 | Unassessed |
| [ERC-7638](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7638.md) Batch Calls Encoding in SCA | Draft | None recorded | Unassessed |
| [ERC-7641](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7641.md) Intrinsic RevShare Token | Draft | 20 | Unassessed |
| [ERC-7644](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7644.md) ERC-721 Name Registry Extension | Draft | 721 | Unassessed |
| [ERC-7649](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7649.md) Bonding curve-embedded liquidity for NFTs | Draft | 721 | Unassessed |
| [ERC-7651](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7651.md) Fractionally Represented Non-Fungible Token | Draft | 20, 165, 721 | Unassessed |
| [ERC-7652](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7652.md) ERC-721 Guarantee Extension | Draft | 165, 721 | Unassessed |
| [ERC-7654](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7654.md) Request Method Types | Draft | None recorded | Unassessed |
| [ERC-7656](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7656.md) Generalized Contract-Linked Services | Final | 20, 165, 721, 1155, 1167, 4337 | Unassessed |
| [ERC-7662](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7662.md) AI Agent NFTs | Draft | 721 | Unassessed |
| [ERC-7673](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7673.md) Distinguishable base256emoji Addresses | Review | None recorded | Unassessed |
| [ERC-7674](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7674.md) Temporary Approval Extension for ERC-20 | Review | 20, 1153 | Unassessed |
| [ERC-7677](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7677.md) Paymaster Web Service Capability | Review | 4337, 5792 | Unassessed |
| [ERC-7679](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7679.md) UserOperation Builder | Draft | 4337 | Unassessed |
| [ERC-7681](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7681.md) Dual Nature Multi Token Protocol | Draft | 20, 1155 | Unassessed |
| [ERC-7682](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7682.md) Auxiliary Funds Capability | Draft | 5792 | Unassessed |
| [ERC-7683](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7683.md) Cross Chain Intents | Draft | 7930 | Unassessed |
| [ERC-7694](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7694.md) Solana Storage Router | Draft | 3668, 7700 | Unassessed |
| [ERC-7695](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7695.md) Ownership Delegation and Context for ERC-721 | Draft | 165, 721 | Unassessed |
| [ERC-7699](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7699.md) ERC-20 Transfer Reference Extension | Draft | 20, 165 | Unassessed |
| [ERC-7700](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7700.md) Cross-chain Storage Router Protocol | Draft | 155 | Unassessed |
| [ERC-7710](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7710.md) Smart Contract Delegation | Draft | 1271, 7579 | Unassessed |
| [ERC-7715](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7715.md) Request Permissions from Wallets | Draft | 4337, 7710 | Unassessed |
| [ERC-7720](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7720.md) Deferred Token Transfer | Draft | 20 | Unassessed |
| [ERC-7721](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7721.md) Lockable Extension for ERC-1155 | Draft | 165, 1155 | Unassessed |
| [ERC-7722](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7722.md) Opaque Token | Draft | None recorded | Unassessed |
| [ERC-7726](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7726.md) Common Quote Oracle | Draft | 7528 | Unassessed |
| [ERC-7729](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7729.md) Token with Metadata | Draft | 20 | Unassessed |
| [ERC-7730](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7730.md) Structured Data Clear Signing Format | Draft | 155, 712 | Unassessed |
| [ERC-7734](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7734.md) Decentralized Identity Verification (DID) | Final | None recorded | Unassessed |
| [ERC-7738](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7738.md) Permissionless Script Registry | Draft | 173 | Unassessed |
| [ERC-7739](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7739.md) Readable Typed Signatures for Smart Accounts | Draft | 191, 712, 1271, 5267 | Unassessed |
| [ERC-7741](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7741.md) Authorize Operator | Draft | 712, 1271 | Unassessed |
| [ERC-7743](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7743.md) Multi-Owner Non-Fungible Tokens (MO-NFT) | Final | None recorded | Unassessed |
| [ERC-7744](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7744.md) Code Index | Last Call | None recorded | Unassessed |
| [ERC-7746](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7746.md) Composable Security Middleware Hooks | Last Call | None recorded | Unassessed |
| [ERC-7750](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7750.md) Decentralized Employment System | Review | None recorded | Unassessed |
| [ERC-7751](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7751.md) Wrapping of bubbled up reverts | Final | None recorded | Unassessed |
| [ERC-7754](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7754.md) Tamperproof Extension Wallets API (TWIST) | Draft | 1193 | Unassessed |
| [ERC-7758](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7758.md) Transfer With Authorization | Review | 20, 712 | Unassessed |
| [ERC-7760](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7760.md) Minimal Upgradeable Proxies | Draft | 1967 | Unassessed |
| [ERC-7765](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7765.md) Privileged Non-Fungible Tokens Tied To RWA | Draft | 165, 721 | Unassessed |
| [ERC-7766](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7766.md) Signature Aggregation for ERC-4337 | Withdrawn | 4337, 7562 | Unassessed |
| [ERC-7769](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7769.md) JSON-RPC API for ERC-4337 | Draft | 155, 4337, 7562 | Unassessed |
| [ERC-7770](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7770.md) Fractional Reserve Token | Draft | 20 | Unassessed |
| [ERC-7774](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7774.md) Cache invalidation in ERC-5219 mode Web3 URL | Draft | 5219, 6944 | Unassessed |
| [ERC-7776](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7776.md) Transparent Financial Statements | Review | 20 | Unassessed |
| [ERC-7777](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7777.md) Governance for Human Robot Societies | Review | None recorded | Unassessed |
| [ERC-7779](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7779.md) Interoperable Delegated Accounts | Draft | 7201, 7702 | Unassessed |
| [ERC-7780](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7780.md) Validation Module Extension for ERC-7579 | Draft | 7579 | Unassessed |
| [ERC-7785](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7785.md) Onchain registration of chain identifiers | Draft | 2304 | Unassessed |
| [ERC-7786](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7786.md) Cross-Chain Messaging Gateway | Final | 7930 | Unassessed |
| [ERC-7787](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7787.md) Soulbound Degradable Governance | Draft | 20 | Unassessed |
| [ERC-7794](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7794.md) Grant Registry | Draft | None recorded | Unassessed |
| [ERC-7795](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7795.md) Wallet Call Token Capabilities | Draft | 20, 721, 1155, 5792 | Unassessed |
| [ERC-7796](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7796.md) Conditional send transaction RPC | Review | None recorded | Unassessed |
| [ERC-7802](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7802.md) Token With Mint/Burn Access Across Chains | Draft | 165, 5679 | A14 |
| [ERC-7803](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7803.md) EIP-712 Extensions for Account Abstraction | Draft | 712 | Unassessed |
| [ERC-7806](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7806.md) Minimal intent-centric EOA smart account | Draft | 7702 | Unassessed |
| [ERC-7811](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7811.md) Wallet Asset Discovery | Draft | 20, 155, 721, 5792 | Unassessed |
| [ERC-7812](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7812.md) ZK Identity Registry | Review | None recorded | Unassessed |
| [ERC-7813](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7813.md) Store, Table-Based Introspectable Storage | Final | None recorded | Unassessed |
| [ERC-7818](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7818.md) Expirable ERC-20 | Final | 20 | Unassessed |
| [ERC-7820](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7820.md) Access Control Registry | Final | None recorded | Unassessed |
| [ERC-7821](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7821.md) Minimal Batch Executor Interface | Draft | 7579 | Unassessed |
| [ERC-7827](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7827.md) JSON Contract with Value Version Control | Draft | None recorded | Unassessed |
| [ERC-7828](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7828.md) Interoperable Names | Review | 155, 7930 | Unassessed |
| [ERC-7829](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7829.md) Data Asset NFT | Review | 165, 721 | Unassessed |
| [ERC-7831](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7831.md) Multi-Chain Addressing | Draft | 55, 137, 155, 165, 2304 | Unassessed |
| [ERC-7832](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7832.md) Sustainable collaborative NFT collections | Review | 721 | Unassessed |
| [ERC-7836](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7836.md) Wallet Call Preparation API | Draft | 1193, 5792 | Unassessed |
| [ERC-7837](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7837.md) Diffusive Tokens | Final | None recorded | Unassessed |
| [ERC-7841](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7841.md) Cross-chain Message Format and Mailbox | Draft | None recorded | Unassessed |
| [ERC-7845](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7845.md) Universal Orchestrator RPC | Draft | None recorded | Unassessed |
| [ERC-7846](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7846.md) Wallet Connection API | Draft | None recorded | Unassessed |
| [ERC-7847](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7847.md) Social Media NFTs | Draft | 721, 1155 | Unassessed |
| [ERC-7856](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7856.md) Chain-Specific Payment Requests | Draft | None recorded | Unassessed |
| [ERC-7857](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7857.md) AI Agents NFT with Private Metadata | Final | None recorded | Unassessed |
| [ERC-7858](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7858.md) Expirable NFTs and SBTs | Final | 165, 721 | Unassessed |
| [ERC-7861](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7861.md) ERC-721 Verifiable Credential Extension | Draft | 165, 721 | Unassessed |
| [ERC-7866](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7866.md) Decentralised User Profiles | Review | 165 | Unassessed |
| [ERC-7871](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7871.md) Wallet Signing API | Draft | 191, 712, 5792 | Unassessed |
| [ERC-7876](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7876.md) Ethereum Network Configuration for DApps | Draft | 55 | Unassessed |
| [ERC-7878](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7878.md) Bequeathable Contracts | Final | 20, 165, 721, 1155 | Unassessed |
| [ERC-7884](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7884.md) Operation Router | Draft | None recorded | Unassessed |
| [ERC-7887](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7887.md) Cancelation for ERC-7540 Tokenized Vaults | Draft | 165, 7540 | Unassessed |
| [ERC-7888](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7888.md) Crosschain Broadcaster | Draft | None recorded | Unassessed |
| [ERC-7891](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7891.md) Splitting and Merging of NFTs | Draft | 721, 6150 | Unassessed |
| [ERC-7893](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7893.md) DeFi Protocol Solvency Proof Mechanism | Final | 20, 165 | Unassessed |
| [ERC-7895](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7895.md) API for Hierarchical Accounts | Draft | None recorded | Unassessed |
| [ERC-7897](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7897.md) Wallet-Linked Services for Smart Accounts | Withdrawn | 165, 1167, 4337, 6551, 7656 | Unassessed |
| [ERC-7902](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7902.md) Wallet Capabilities for Account Abstraction | Draft | 5792, 7702, 7906, 8141, 8250, 8272 | Unassessed |
| [ERC-7908](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7908.md) HD wallet In Treasury Management | Final | None recorded | Unassessed |
| [ERC-7913](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7913.md) Signature Verifiers | Final | None recorded | Unassessed |
| [ERC-7920](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7920.md) Composite EIP-712 Signatures | Draft | 20, 712 | Unassessed |
| [ERC-7929](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7929.md) PermaLink Asset Bound Token | Draft | 721 | Unassessed |
| [ERC-7930](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7930.md) Interoperable Addresses | Review | None recorded | Unassessed |
| [ERC-7936](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7936.md) Versioned Proxy Contract Interface | Review | None recorded | Unassessed |
| [ERC-7943](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7943.md) uRWA - Universal Real World Asset Interface | Final | 165 | Unassessed |
| [ERC-7945](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7945.md) Confidential Transactions Supported Token | Final | 20 | Unassessed |
| [ERC-7946](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7946.md) Unidirectional Wallet Uplink aka UWULink | Draft | 155, 7702 | Unassessed |
| [ERC-7947](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7947.md) Account Abstraction Recovery Interface | Draft | None recorded | Unassessed |
| [ERC-7950](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7950.md) Encode chain id with transaction hash | Final | 155 | Unassessed |
| [ERC-7955](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7955.md) Permissionless CREATE2 Factory | Draft | 1014, 7702 | Unassessed |
| [ERC-7962](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7962.md) Key Hash Based Tokens | Review | 20, 712, 721, 5267 | Unassessed |
| [ERC-7963](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7963.md) Oracle-Permissioned ERC-20 with ZK Proofs | Last Call | 20, 1967 | Unassessed |
| [ERC-7964](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7964.md) Crosschain EIP-712 Signatures | Draft | 712, 1271, 5267 | Unassessed |
| [ERC-7965](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7965.md) Proof-based Broadcast in ERC-7786 Gateways | Draft | 7786, 7930 | Unassessed |
| [ERC-7968](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7968.md) Owner-Authorized Token Transfer Protocol | Draft | 20 | Unassessed |
| [ERC-7969](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7969.md) DomainKeys Identified Mail (DKIM) Registry | Draft | None recorded | Unassessed |
| [ERC-7984](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7984.md) Confidential Fungible Token | Draft | 165 | Unassessed |
| [ERC-7985](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7985.md) Gateway Attributes for Message Control | Draft | 7786 | Unassessed |
| [ERC-7988](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7988.md) Minimal Avatar Smart Wallet (MASW) | Draft | 7702 | Unassessed |
| [ERC-7992](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7992.md) Verifiable ML Model Inference (ZKML) | Draft | 165 | Unassessed |
| [ERC-7994](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7994.md) Purpose-Bound ERC-20 with Conditional Unlock | Final | 20, 7291 | Unassessed |
| [ERC-7996](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-7996.md) Contract Feature Detection | Draft | 165 | Unassessed |
| [ERC-8000](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8000.md) Operator contract for non delegated EOAs | Draft | 1153, 2771 | Unassessed |
| [ERC-8001](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8001.md) Agent Coordination Framework | Final | 712, 1271, 2098, 5267 | Unassessed |
| [ERC-8002](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8002.md) Simplified Payment Verification Gateway | Draft | None recorded | Unassessed |
| [ERC-8004](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8004.md) Trustless Agents | Draft | 155, 712, 721, 1271 | Unassessed |
| [ERC-8017](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8017.md) Payout Race | Draft | 20 | Unassessed |
| [ERC-8019](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8019.md) Minimal Wallet-Managed Auto-Login for SIWE | Review | 4361 | Unassessed |
| [ERC-8023](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8023.md) Multi-step Contract Ownership | Draft | None recorded | Unassessed |
| [ERC-8033](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8033.md) Agent Council Oracles | Draft | 20 | Unassessed |
| [ERC-8034](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8034.md) Referable NFT Royalties | Final | 165, 712, 5521 | Unassessed |
| [ERC-8040](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8040.md) ESG Tokenization Protocol | Draft | 20, 721, 1155 | Unassessed |
| [ERC-8041](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8041.md) Fixed-Supply Agent NFT Collections | Draft | 7572, 8004, 8119 | Unassessed |
| [ERC-8042](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8042.md) Diamond Storage | Final | 7201 | Unassessed |
| [ERC-8047](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8047.md) Forensic Token (Forest) | Draft | 165, 1155, 5615 | Unassessed |
| [ERC-8048](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8048.md) Onchain Metadata for Token Registries | Draft | 165, 7930, 8119 | Unassessed |
| [ERC-8049](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8049.md) Contract-Level Onchain Metadata | Draft | 20, 165, 7930 | Unassessed |
| [ERC-8054](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8054.md) Forkable ERC-20 Token | Draft | 20 | Unassessed |
| [ERC-8056](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8056.md) Scaled UI Amount Extension for ERC-20 Tokens | Draft | 20, 165 | Unassessed |
| [ERC-8063](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8063.md) Groups - Membership Tokens | Final | 20, 165 | Unassessed |
| [ERC-8065](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8065.md) Zero Knowledge Token Wrapper | Draft | 20, 165, 721, 1155, 6909 | Unassessed |
| [ERC-8074](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8074.md) Self-Describing Bytes via EIP-712 Selectors | Draft | 712 | Unassessed |
| [ERC-8084](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8084.md) Zero-knowledge proof metadata | Draft | None recorded | Unassessed |
| [ERC-8085](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8085.md) Dual-Mode Fungible Tokens | Draft | 20, 8086 | Unassessed |
| [ERC-8086](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8086.md) Privacy Token | Draft | None recorded | Unassessed |
| [ERC-8092](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8092.md) Associated Accounts | Draft | 712, 1271, 6492, 7930 | Unassessed |
| [ERC-8100](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8100.md) Representable Contract State | Draft | 155 | Unassessed |
| [ERC-8106](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8106.md) RWA Event-based Compliance Framework | Draft | 20 | Unassessed |
| [ERC-8107](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8107.md) ENS Trust Registry for Agent Coordination | Review | 137, 165, 712, 1271, 5267, 8001 | Unassessed |
| [ERC-8109](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8109.md) Diamonds, Simplified | Withdrawn | None recorded | Unassessed |
| [ERC-8110](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8110.md) Domain Architecture for Diamonds | Draft | 2535, 8042 | Unassessed |
| [ERC-8111](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8111.md) Bound Signatures | Review | None recorded | Unassessed |
| [ERC-8113](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8113.md) Series Accounting for Incentivized Vaults | Draft | 4626, 7540 | Unassessed |
| [ERC-8117](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8117.md) Anti-Poisoning Compact EVM Address Format | Draft | 55, 1191 | Unassessed |
| [ERC-8119](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8119.md) Parameterized Storage Keys | Draft | None recorded | Unassessed |
| [ERC-8121](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8121.md) Cross-Chain Function Calls via Hooks | Draft | 3668, 7930 | Unassessed |
| [ERC-8122](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8122.md) Minimal Agent Registry | Draft | 6909, 7930, 8048, 8049, 8127 | Unassessed |
| [ERC-8126](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8126.md) AI Agent Verification | Final | 155, 191, 712, 721, 3009, 8004 | Unassessed |
| [ERC-8127](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8127.md) Human Readable Token Identifiers | Draft | 7930 | Unassessed |
| [ERC-8143](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8143.md) Smart Credential Resolution Interface | Draft | 165, 3668 | Unassessed |
| [ERC-8152](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8152.md) Content-Addressable Logic Modules (CALM) | Review | None recorded | Unassessed |
| [ERC-8153](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8153.md) Facet-Based Diamonds | Final | 2535 | Unassessed |
| [ERC-8161](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8161.md) Transferable Tokenized Vault Requests | Final | 165, 7540 | Unassessed |
| [ERC-8167](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8167.md) Modular Dispatch Proxies | Final | None recorded | Unassessed |
| [ERC-8176](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8176.md) Descriptor Attestations for ERC-7730 | Draft | 712, 1271, 7730 | Unassessed |
| [ERC-8179](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8179.md) Blob Space Segments | Draft | 4844 | Unassessed |
| [ERC-8180](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8180.md) Blob Authenticated Messaging | Draft | 4844, 8179 | Unassessed |
| [ERC-8183](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8183.md) Agentic Commerce | Draft | 20 | Unassessed |
| [ERC-8187](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8187.md) Token Puller | Draft | 20, 712, 1271, 2612, 5267, 6492 | Unassessed |
| [ERC-8196](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8196.md) AI Agent Authenticated Wallet | Final | 155, 191, 712, 4337, 8126 | Unassessed |
| [ERC-8199](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8199.md) Sandboxed Smart Wallet | Draft | 165, 173, 8023 | Unassessed |
| [ERC-8217](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8217.md) Agent NFT Identity Bindings | Draft | 7930, 8004 | Unassessed |
| [ERC-8226](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8226.md) Regulated Agent Mandate | Draft | 165, 712, 1271 | Unassessed |
| [ERC-8255](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8255.md) Expiring Token Approvals | Draft | 20, 2612 | Unassessed |
| [ERC-8257](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8257.md) Agent Tool Registry | Draft | 150, 165, 712, 1967 | Unassessed |
| [ERC-8259](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8259.md) AI Agent Identity & Threat Registry | Draft | None recorded | Unassessed |
| [ERC-8262](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8262.md) Zero-Knowledge Compliance Oracle | Review | 165 | Unassessed |
| [ERC-8273](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8273.md) Attestation-Gated Agentic Actions | Draft | 165, 1153 | Unassessed |
| [ERC-8286](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8286.md) Modular Accounts for Frame Transactions | Draft | 7579, 8141 | Unassessed |
| [ERC-8290](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8290.md) Shielded Note Teleportation | Draft | None recorded | Unassessed |
| [ERC-8320](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8320.md) Regulated Asset Claim | Draft | 165, 712, 1271 | Unassessed |
| [ERC-8325](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8325.md) Asset Anchor Registry | Final | 165 | Unassessed |
| [ERC-8326](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8326.md) Canonical Document Bundle Anchor | Final | 165 | Unassessed |
| [ERC-8327](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8327.md) Directional Transfer Domain Registry | Final | 165 | Unassessed |
| [ERC-8328](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8328.md) Subject-Linked Compliance Event Log | Final | 165 | Unassessed |
| [ERC-8329](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8329.md) Subject-Linked Impact Snapshot Log | Final | 165 | Unassessed |
| [ERC-8330](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8330.md) Subject-Linked NAV Snapshot Oracle | Last Call | 165 | Unassessed |
| [ERC-8348](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8348.md) Financial Lease | Draft | 20, 165, 721, 4907, 8325 | Unassessed |
| [ERC-8349](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8349.md) Index-Based Multi-Facet Proxy | Draft | 165 | Unassessed |
| [ERC-8350](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8350.md) Agent Memory State Registry | Draft | 712, 1271, 7702 | Unassessed |
| [ERC-8354](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8354.md) Confidential Agent Policy Verdicts | Draft | 165, 712, 721, 1271, 7812, 8004 | Unassessed |
| [ERC-8376](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8376.md) Token Launch Abuse Detection and Remediation | Draft | 20, 165, 214 | Unassessed |
| [ERC-8377](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8377.md) Reference-Relative Slippage Bounds | Draft | 165, 7726 | Unassessed |
| [ERC-8410](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8410.md) Portable Execution Plan Artifact | Draft | 155, 712 | Unassessed |
| [ERC-8426](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8426.md) Wallet Pass Extension for NFTs | Draft | 165, 721, 1271, 4361, 6492 | Unassessed |
| [ERC-8436](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8436.md) Agent Collective Decision Framework | Draft | 165, 712, 1271, 6372 | Unassessed |
| [ERC-8441](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8441.md) Hybrid post-quantum stealth address scheme | Draft | 5564, 6538 | Unassessed |
| [ERC-8444](https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-8444.md) Agentic Resource Discovery (ARD) Onchain | Draft | 165, 712, 1271 | Unassessed |

## Meta

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [EIP-1](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1.md) EIP Purpose and Guidelines | Living | None recorded | P01 |
| [EIP-4](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4.md) EIP Classification | Final | None recorded | Unassessed |
| [EIP-233](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-233.md) Formal process of hard forks | Stagnant | None recorded | Unassessed |
| [EIP-606](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-606.md) Hardfork Meta: Homestead | Final | 2, 7, 8 | Unassessed |
| [EIP-607](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-607.md) Hardfork Meta: Spurious Dragon | Final | 155, 160, 161, 170, 608 | Unassessed |
| [EIP-608](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-608.md) Hardfork Meta: Tangerine Whistle | Final | 150, 779 | Unassessed |
| [EIP-609](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-609.md) Hardfork Meta: Byzantium | Final | 100, 140, 196, 197, 198, 211, 214, 607, 649, 658 | Unassessed |
| [EIP-779](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-779.md) Hardfork Meta: DAO Fork | Final | 606 | Unassessed |
| [EIP-867](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-867.md) Standardized Ethereum Recovery Proposals | Stagnant | None recorded | Unassessed |
| [EIP-1013](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1013.md) Hardfork Meta: Constantinople | Final | 145, 609, 1014, 1052, 1234, 1283 | Unassessed |
| [EIP-1588](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1588.md) Hardfork Meta: Ethereum ProgPoW | Stagnant | 1057 | Unassessed |
| [EIP-1679](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1679.md) Hardfork Meta: Istanbul | Final | 152, 1108, 1344, 1716, 1884, 2028, 2200 | Unassessed |
| [EIP-1716](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1716.md) Hardfork Meta: Petersburg | Final | 1013, 1283 | Unassessed |
| [EIP-1872](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1872.md) Ethereum Network Upgrade Windows | Stagnant | None recorded | Unassessed |
| [EIP-2070](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2070.md) Hardfork Meta: Berlin | Withdrawn | 1679 | Unassessed |
| [EIP-2378](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2378.md) EIPs Eligible for Inclusion | Stagnant | None recorded | Unassessed |
| [EIP-2387](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2387.md) Hardfork Meta: Muir Glacier | Final | 1679, 2384 | Unassessed |
| [EIP-2657](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2657.md) Ephemeral Testnet Yolo | Stagnant | None recorded | Unassessed |
| [EIP-5069](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5069.md) EIP Editor Handbook | Living | 1 | Unassessed |
| [EIP-5757](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5757.md) Process for Approving External Resources | Final | 1 | Unassessed |
| [EIP-6049](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6049.md) Deprecate SELFDESTRUCT | Final | None recorded | Unassessed |
| [EIP-7199](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7199.md) Linter Scope | Withdrawn | None recorded | Unassessed |
| [EIP-7329](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7329.md) ERC/EIP Repository split | Final | 1 | Unassessed |
| [EIP-7568](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7568.md) Hardfork Meta Backfill - Berlin to Shapella | Final | 2070, 2387, 2982, 6122, 6953 | Unassessed |
| [EIP-7569](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7569.md) Hardfork Meta - Dencun | Final | 1153, 4788, 4844, 5656, 6780, 7044, 7045, 7514, 7516, 7568 | Unassessed |
| [EIP-7577](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7577.md) Versioning Scheme for EIPs | Stagnant | None recorded | Unassessed |
| [EIP-7587](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7587.md) Reserve Precompile Address Range for RIPs | Final | None recorded | Unassessed |
| [EIP-7600](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7600.md) Hardfork Meta - Pectra | Final | 2537, 2935, 6110, 7002, 7251, 7549, 7569, 7623, 7685, 7691, 7702 | Unassessed |
| [EIP-7607](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7607.md) Hardfork Meta - Fusaka | Final | 7594, 7600, 7642, 7823, 7825, 7883, 7892, 7910, 7917, 7918, 7934, 7935, 7939, 7951 | Unassessed |
| [EIP-7675](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7675.md) Retroactively Included EIPs | Withdrawn | 2681, 3607, 4803, 7523, 7610 | Unassessed |
| [EIP-7692](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7692.md) EVM Object Format (EOFv1) Meta | Stagnant | 663, 3540, 3670, 4200, 4750, 5450, 6206, 7069, 7480, 7620, 7698 | Unassessed |
| [EIP-7723](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7723.md) Network Upgrade Inclusion Stages | Last Call | None recorded | P01 |
| [EIP-7768](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7768.md) No-Ether transactions with free-for-all tips | Stagnant | None recorded | Unassessed |
| [EIP-7773](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7773.md) Hardfork Meta - Glamsterdam | Review | 7607, 7723 | Unassessed |
| [EIP-7808](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7808.md) Reserve Tx-Type Range for RIPs | Stagnant | 2718 | Unassessed |
| [EIP-7872](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7872.md) Max blob flag for local builders | Review | None recorded | Unassessed |
| [EIP-7919](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7919.md) Pureth Meta | Stagnant | 6404, 6465, 6466, 7495, 7668, 7708, 7745, 7799, 7807, 7916 | Unassessed |
| [EIP-7927](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7927.md) History Expiry Meta | Stagnant | 4444 | Unassessed |
| [EIP-8007](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8007.md) Glamsterdam Gas Repricings | Draft | 2780, 2926, 7686, 7778, 7904, 7923, 7971, 7973, 7976, 7981, 8011, 8032, 8037, 8038, 8053, 8057, 8058, 8059 | Unassessed |
| [EIP-8081](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8081.md) Hardfork Meta - Hegotá | Draft | 7723, 7773 | Unassessed |
| [EIP-8134](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8134.md) Hardfork Meta - BPO1 | Final | 7892 | Unassessed |
| [EIP-8135](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8135.md) Hardfork Meta - BPO2 | Final | 7892, 8134 | Unassessed |
| [EIP-8138](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8138.md) Hardfork Meta - BPO3 | Draft | 7892, 8135 | Unassessed |

## Informational

| Document and official title | Document status | Requires | Candidate families or assessment |
| --- | --- | --- | --- |
| [EIP-1470](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1470.md) Smart Contract Weakness Classification (SWC) | Stagnant | None recorded | Unassessed |
| [EIP-2069](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2069.md) Recommendation for using YAML ABI in ERCs/EIPs | Stagnant | None recorded | Unassessed |
| [EIP-2228](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2228.md) Canonicalize the name of network ID 1 and chain ID 1 | Final | None recorded | Unassessed |
| [EIP-2294](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2294.md) Explicit bound to Chain ID size | Stagnant | 155 | Unassessed |
| [EIP-2458](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2458.md) Updates and Updated-by Header | Withdrawn | None recorded | Unassessed |
| [EIP-2982](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2982.md) Serenity Phase 0 | Final | None recorded | Unassessed |
| [EIP-6953](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-6953.md) Network Upgrade Activation Triggers | Final | 2982, 3675, 6122 | Unassessed |
| [EIP-7783](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7783.md) Add Controlled Gas Limit Increase Strategy | Stagnant | None recorded | Unassessed |
| [EIP-7790](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7790.md) Controlled Gas Limit Increase Guidelines | Stagnant | 7783 | Unassessed |
| [EIP-7840](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7840.md) Add blob schedule to EL config files | Final | None recorded | Unassessed |
| [EIP-7870](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7870.md) Hardware and Bandwidth Recommendations | Living | None recorded | Unassessed |
| [EIP-7892](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7892.md) Blob Parameter Only Hardforks | Final | 7840 | Unassessed |
| [EIP-7904](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7904.md) Compute Gas Cost Analysis | Last Call | None recorded | Unassessed |
| [EIP-7935](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7935.md) Set default gas limit to 60M | Final | None recorded | Unassessed |
| [EIP-7938](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7938.md) Exponential Gas Limit Increase | Stagnant | None recorded | Unassessed |
| [EIP-7940](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7940.md) Ethereum Shah | Draft | None recorded | Unassessed |
| [EIP-7949](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7949.md) Genesis File Format | Draft | None recorded | Unassessed |
| [EIP-8066](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8066.md) Upgrade Mascots | Last Call | None recorded | Unassessed |
| [EIP-8133](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8133.md) Network Upgrade Naming | Review | 150, 606, 607, 608, 609, 779, 1013, 1283, 1679, 1716, 2387, 4345, 5133, 7569, 7600, 7607, 7892, 8134, 8135 | Unassessed |
| [EIP-8173](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8173.md) Foundations of EVM Control Flow | Review | None recorded | Unassessed |
| [EIP-8252](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8252.md) Execution-Layer Reorg State Retention Window | Draft | None recorded | Unassessed |
| [EIP-8261](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8261.md) Gas Limit Schedule | Last Call | 1559, 7732, 7892, 7935 | Unassessed |
| [EIP-8369](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8369.md) VOPS Profiles for FOCIL Eligibility | Draft | 1559, 2718, 2930, 3607, 4844, 7702, 7732, 7805, 7843, 7928, 8141, 8250, 8272 | Unassessed |
| [EIP-8457](https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-8457.md) Batched KZG Point Evaluation via EIP-2537 | Draft | 2537, 4844 | Unassessed |
