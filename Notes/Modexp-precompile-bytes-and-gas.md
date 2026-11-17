---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: E06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
eips_commit: "8194528a2ee9a4893a398a5871c470ab0ac3b09a"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
tags:
  - ethlib
  - external-sources
  - evm
  - precompiles
---

# MODEXP precompile bytes and gas

The MODEXP precompile at address `0x05` accepts a byte layout, charges gas, and returns an explicitly sized byte string.
Computing the same modular power as an integer covers only its arithmetic step.
The declared operand lengths also determine offsets, output width, and gas, even when leading zeros leave the integers unchanged.

## Sources and historical scope

This note composes EIP-198's interface with EIP-2565's repricing, the historical Berlin rule set selected here.
The name also identifies the `.berlin` branch in the pinned EthLib definitions.
It is not a claim about current network rules or activation dates.
EIP-2565's source still expresses activation with `FORK_BLOCK_NUMBER`.

| Primary source | Version and reading |
| --- | --- |
| [EIP-198][198] | EIPs commit above, Final: complete Specification, examples, and Rationale. |
| [EIP-2565][2565] | Same pin, Final: complete Specification, Rationale, Test Cases, and Security Considerations. |
| [Yellow Paper][yp] | Commit above, Shanghai text: Appendix E's precompile template and complete MODEXP paragraph/formulas, particularly zero-modulus and output-encoding conventions. |
| [EIP-7823][7823], [EIP-7883][7883] | Same EIPs pin, Final: selected Specifications identifying later length restrictions and repricing, outside the composition calculated here. |

EIP-198 supplies the byte interface, while EIP-2565 states that its change leaves the interface and arithmetic algorithms unchanged.
The pinned Yellow Paper supplements an edge convention that EIP-198's `% MODULUS` expression alone does not state explicitly.

## Decode lengths before operands

The first 96 bytes contain three 32-byte, left-zero-padded length integers.
Each length counts operand bytes, not bits or 32-byte words.
The operands immediately follow those headers in base, exponent, modulus order, without ABI offsets or alignment between operands:

```text
offset 0:             base length b       (32 bytes)
offset 32:            exponent length e   (32 bytes)
offset 64:            modulus length m    (32 bytes)
offset 96:            base                (b bytes)
offset 96+b:          exponent            (e bytes)
offset 96+b+e:        modulus             (m bytes)
```

Read integers big-endian.
EIP-198 assumes infinitely many zero bytes beyond the actual input and ignores excess bytes after the declared fields.
A zero base length consumes no base bytes, so the exponent starts at offset 96.
A short buffer therefore does not by itself establish malformed input. [EIP-198, Specification][198]

Right-padding matters inside a declared field.
With lengths `(1,1,2)` and body `03 05 07`, the missing final byte makes the modulus `07 00`, or 1792.
It does not become `00 07`.
The result is `3^5 mod 1792 = 243`, returned as `00 f3`.
Providing body `03 05 00 07` instead makes the modulus 7 and returns `00 05`.
The second result has two bytes because the modulus length is two, although the integer result is 5.

## Complete small input and result

Concatenate these four lines without spaces or line breaks to obtain the complete 99-byte input:

```text
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000001
030507
```

The headers occupy offsets 0–95, followed by base `03`, exponent `05`, and modulus `07` at offsets 96–98.
Since `3^5 = 243 = 34*7 + 5`, the complete successful output is the single byte `05`.
Appending `ff` to the input changes neither this result nor its precompile gas requirement.
Both are derived examples, rather than published test vectors.

For nonzero modulus, the modular result lies below the modulus and fits in its declared width.
The Yellow Paper explicitly defines `x mod 0 = 0` for this precompile and encodes the result big-endian in the modulus's format. [Yellow Paper, Appendix E][yp]
Consequently these cases remain distinct, assuming enough gas:

| Declared lengths and body | Decoded modulus | Complete output |
| --- | --- | --- |
| `(1,1,1)`, `03 05 00` | Zero, one-byte field. | `00` |
| `(1,1,1)`, `03 05` | Zero through the missing byte. | `00` |
| `(1,1,0)`, `03 05` | Zero, empty field. | Empty bytes. |
| Entire input empty. | All lengths and fields read as zero. | Empty bytes. |

An empty successful output does not imply a failed call.
A standalone modular-power function also needs an explicit modulus-zero convention before it can represent these cases.

## Calculate the scoped gas price

For the worked exponents, the declared exponent length is at most 32 bytes.
EIP-198's adjusted exponent length is zero for exponent zero and otherwise the highest set-bit index, counting from zero.
It is the index, not the bit count.
Both prices use at least one iteration, so exponents 0 and 1 still give iteration count 1.
Exponent 5 has highest set bit 2 and gives count 2. [EIP-198, Specification][198] [EIP-2565, Specification][2565]

Let `x = max(b,m)` and `I` be that iteration count.
EIP-198 charges `floor(mult_complexity(x)*I/20)`, using this piecewise complexity:

```text
x <= 64:         x^2
64 < x <= 1024:  floor(x^2/4) + 96*x - 3072
x > 1024:        floor(x^2/16) + 480*x - 199680
```

EIP-2565 replaces the complexity with `W^2`, where `W = ceil(x/8)`, and charges:

```text
max(200, floor(W^2 * I / 3))
```

For the 99-byte example, `x=1`, `I=2`, and `W=1`.
The original price is `floor(1*2/20)=0`, whereas the Berlin price is `max(200,floor(2/3))=200`.
The zero-modulus cases above also cost 200 under this composition, including entirely empty input.
They do not bypass the minimum price.

To expose division rounding beyond that minimum, take declared base and modulus lengths 64 and exponent bytes `01 00 01`, or 65537.
Then `I=16`, EIP-198 complexity is 4096, and its price is `floor(4096*16/20)=3276`.
Berlin gives `W=8`, complexity 64, and `floor(64*16/3)=341`.
Leading-zero operand bytes still count toward these declared lengths.
These figures price the precompile, not an entire transaction or the caller's other EVM work.
The Yellow Paper's precompile template fails when available gas is below the requirement and subtracts the requirement otherwise. [Yellow Paper, Appendix E][yp]

## Source boundary and the selected Lean definitions

For exponents longer than 32 bytes, EIP-198 explicitly uses the first 32 exponent bytes and separately handles an all-zero prefix.
EIP-2565's pseudocode instead masks the supplied integer with `2^256-1` and applies `bit_length()-1` in its long-exponent branch.
That expression selects low bits of its integer argument and gives `-1` for zero.
Do not silently equate it with EIP-198's prefix rule.
The pinned [`Gas.lean`][gas] documents this difference and follows EIP-198's prefix convention.
The calculations here avoid that unresolved interpretation by using exponent lengths at most 32 bytes.

[`Core.lean`][core] reads headers, derives operand starts, checks the selected fork's length policy, computes the price from the exponent head, and checks gas before computing output.
Its `compute` explicitly returns zero for a zero modulus and writes exactly the declared modulus length.
[`Encoding.readNat`][encoding] combines a zero-padded buffer read with big-endian integer decoding.
[`powMod`][pow] returns an integer and supplies neither headers nor gas nor fixed-width output.
These are observations from definitions, not proof-coverage claims.

EIP-7823 introduces a 1024-byte bound per declared operand, and EIP-7883 changes the price again.
The pinned Lean definitions contain separate `.fusaka` branches for these rules.
Those later branches are outside this Berlin calculation, and this note does not infer activation from their presence in the code.

## Review limits and contribution use

A standalone Python integer/byte calculation reproduced nine short-exponent cases, including the examples, zero-length distinctions, and both gas prices.
Its zero-modulus branch applies the separately cited Yellow Paper convention.
The source passages, explanation, and selected definitions received a separate review.
An independent byte/integer calculation reproduced all nine bounded cases and the original/Berlin gas results, including the floor giving 341.
No EthLib import, EVM call, Lean build, binding test, or proof replay ran.
The source reading does not resolve long-exponent ambiguity or assess later amendment interactions.
A bounded contribution can state byte parsing, fixed output width, and a chosen fork's gas relation separately, while preserving these source questions.

[198]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-198.md#specification
[2565]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-2565.md#specification
[7823]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7823.md#specification
[7883]: https://github.com/ethereum/EIPs/blob/8194528a2ee9a4893a398a5871c470ab0ac3b09a/EIPS/eip-7883.md#specification
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex#L1731-L1823
[gas]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Precompiles/Modexp/Gas.lean
[core]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Precompiles/Modexp/Core.lean
[encoding]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Precompiles/Modexp/Encoding.lean
[pow]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/PowMod.lean
