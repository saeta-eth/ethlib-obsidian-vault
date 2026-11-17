---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - evm
  - opcode-semantics
---

# EVM word arithmetic and shift boundaries

A 256-bit word has different signed and unsigned interpretations, while a memory copy operates on bytes and ranges.
Choosing the wrong rounding rule, reducing a shift count modulo 256, or copying overlapping bytes in the wrong order changes observable EVM results.
These small boundaries need exact semantics before an optimizer or proof can replace an instruction sequence.

## Sources and scope

All seven EIPs below are Final at the recorded EIPs revision.
Their complete short texts, including printed examples and security qualifications, were read.
Final status does not establish an opcode's availability on every fork or network.

| Primary sources | Selected question |
| --- | --- |
| [EIP-145][145] | Shift operand order, signed rounding, and counts at or above 256. |
| [EIP-5656][5656] | Overlapping MCOPY, zero-length ranges, and memory-expansion charges. |
| [EIP-1052][1052] | Absent/empty accounts versus existing accounts with no code. |
| [EIP-1344][1344], [EIP-3198][3198], [EIP-4399][4399] | Configured chain ID, block base fee, and transition-dependent meaning of opcode `0x44`. |
| [EIP-3855][3855] | PUSH0's constant value, byte length, and stack boundary. |

The [pinned Shanghai Yellow Paper][yp] supplies selected Appendix H.1 memory-cost rules, H.2 SDIV, and the exceptional-halting definition.
Its edition predates MCOPY; EIP-5656 adds that instruction while referencing the memory-pricing rules.
[EIP-161's emptiness definition][161] supplies the dependency explicitly named by EIP-1052.
This note covers a bounded instruction cluster, without a complete opcode or later-amendment catalogue.

## Keep the shift count unsigned

EIP-145 pops the shift amount first, then the value.
SHL and SHR interpret the value as unsigned; SAR interprets the same 256 bits as a two's-complement signed integer.
In all three, the count is unsigned.
For a word `w`, the signed value is `w` below `2^255`, otherwise `w - 2^256`.
A result such as `-1` returns to the stack as the all-ones 256-bit word.

For counts below 256, SHL multiplies by `2^count` modulo `2^256`, SHR divides unsigned and rounds down, and SAR divides signed and rounds toward negative infinity.
The boundary cases are explicit:

| Operation, with count written first | Result |
| --- | --- |
| `SHL(255,1)` | `2^255`. |
| `SHL(256,1)` or `SHL(257,1)` | Zero. |
| `SHR(256, all_ones)` | Zero. |
| `SAR(256, all_ones)` | All ones, representing `-1`. |
| `SAR(256,7)` | Zero. |

The count does not wrap at 256.
Even the word `2^256 - 1` is a valid unsigned count: SAR returns sign fill, while SHL and SHR return zero.
These counts are value cases, not malformed instructions or out-of-gas conditions by themselves.
EIP-145 assigns each shift the three-gas `verylow` tier.

## SAR cannot generally replace SDIV

EIP-145 explicitly gives `SAR(1,-1) = -1` but signed division `SDIV(-1,2) = 0`.
The Yellow Paper's SDIV truncates toward zero.
For another source-derived case:

```text
input word for -3 = fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffd
SAR(1,-3)         = floor(-3/2) = -2
output word       = fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe
SDIV(-3,2)        = -1
output word       = ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```

The different result is caused by rounding, not a different word width.
A proof replacing signed division by a shift needs hypotheses that justify the equality, such as the relevant divisibility or sign condition.
This example does not characterize every valid compiler rewrite.

## Copy the original overlapping range

MCOPY pops `destination`, `source`, and `length`, in that order, and pushes nothing.
EIP-5656 defines the effect as if an intermediate buffer were used.
That specifies which bytes are copied; its Security Considerations advise against physically allocating an intermediate buffer.
A suitable implementation may use a memmove-style strategy instead.

Use the EIP's published overlapping examples, each starting independently with this complete 32-byte memory:

```text
before:
0001020304050607080000000000000000000000000000000000000000000000

MCOPY(destination=1, source=0, length=8):
0000010203040506070000000000000000000000000000000000000000000000

MCOPY(destination=0, source=1, length=8):
0102030405060708080000000000000000000000000000000000000000000000
```

In the first copy, destination byte 2 receives the original source byte 1, namely `01`.
It must not receive the `00` that was just written into memory byte 1.
Naively walking forward while reading the changing destination would corrupt this case.
The reverse-overlap example uses the same snapshot semantics.

Both source and destination already fit in the active word.
The published cost is `3 + 3 * ceil(8/32) = 6` gas, excluding the surrounding operand-push instructions.
No memory expansion is required.

## Zero length and memory growth are different cases

For length zero, EIP-5656 does not extend memory, even when an offset is extremely large.
With the same 32-byte starting memory, `destination = source = 2^256 - 1` and `length = 0` leave memory unchanged and cost three gas.
The instruction still requires three stack items and enough gas, and must be available in the selected fork.
Zero length does not waive those conditions.

For a derived nonzero case, copy two bytes from source 30 to destination 31.
The source bytes are both zero, but the destination ends at exclusive offset 33, so active memory grows from 32 to 64 bytes.
The appended bytes are zero.
Under the referenced memory schedule, for `a` active words:

```text
C_mem(a) = 3*a + floor(a*a/512)
expansion = C_mem(2) - C_mem(1) = 6 - 3 = 3
MCOPY cost = 3 + 3*ceil(2/32) + 3 = 9
```

Reading beyond the previous active memory can therefore require zero-filled expansion, rather than an invalid source-range error.
Large nonzero ranges still have to pay expansion costs.
Insufficient gas is an exceptional halt; it is not another defined copy result.
The earlier [creation note](Creation-collisions-rollback-and-destruction.md) separates exceptional failure from a callee's REVERT outcome.

## Account and environment values need context

EIP-1052 returns zero for a nonexistent or empty account, but the published Keccak empty-code hash for an existing, nonempty account without code.
EIP-161 defines empty as zero nonce, zero balance, and no code.
Thus an account holding 5 wei with nonce zero and no code takes the empty-code-hash case, not the zero-result case.
Only the low 160 address bits are significant.
“No code” and “no account” are different inputs to this query; its value alone is not an application authorization policy.

CHAINID returns the configured chain identifier, including for a transaction without an EIP-155 identifier.
EIP-1344 says that configuration should match the EIP-155 identifier accepted for incoming transactions.
Where continued use requires it, users must ensure their application can handle a potential chain-ID update; the source calls for case-by-case reasoning.
BASEFEE instead reads the current block's base fee; it does not return a transaction's gas price or fee cap.
The [signed-data note](Signed-data-preimages-and-domains.md) keeps application replay domains separate.

At `TRANSITION_BLOCK`, EIP-4399 requires opcode `0x44` to return the header's `mixHash` value, sourced from the previous block's post-beacon-state RANDAO mix.
Renaming that field and instruction to `prevRandao` and PREVRANDAO is SHOULD wording.
The source's biasability and predictability qualifications remain relevant; the opcode does not establish fresh, unbiased application randomness.

PUSH0 is a one-byte instruction with no immediate data, pushing a constant zero for two gas.
It avoids treating a context-dependent value such as RETURNDATASIZE as an assumed zero.
Its published 1025-PUSH0 case aborts on stack overflow, despite each individual pushed value being valid.

## EthLib observations and checks

At the pinned EthLib commit, [`bitwiseShl`, `bitwiseShr`, and `bitwiseSar`][lean-shifts] explicitly handle the 256 threshold.
[`mcopyStep`][lean-copy] requests gas/memory growth, reads the source, then writes the result.
[`extendMemory`][lean-growth] skips zero-size ranges; [`memoryReadBytes` and `memoryWrite`][lean-memory] also handle empty ranges without indexing their starts.
[`Op.ofByte`][lean-op] applies fork availability, while [`environmentStep`][lean-env] selects the corresponding block/environment fields.
These are definition readings, not proofs that arbitrary supplied execution environments are valid.

A separate source review checked the cited passages and the conditions stated in this note.
Independent calculations reproduced the displayed example values.
Standalone Python arithmetic matched all 38 printed EIP-145 shift cases, the two displayed MCOPY overlap cases, the derived rounding case, and the zero-length/expansion calculations.
A changing-source copy served as a negative control for overlap handling.
No EVM, compiler optimization, EthLib binding, Lean build, or proof replay ran.

[145]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-145.md
[5656]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-5656.md
[1052]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1052.md
[1344]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-1344.md
[3198]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3198.md
[3855]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-3855.md
[4399]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-4399.md
[161]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-161.md
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[lean-shifts]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/Bitwise.lean#L50-L69
[lean-copy]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/MemoryData.lean#L61-L71
[lean-growth]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Gas/Memory.lean#L52-L76
[lean-memory]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Machine/Memory.lean#L31-L48
[lean-op]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Op.lean#L338-L340
[lean-env]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/EnvironmentData.lean#L95-L126
