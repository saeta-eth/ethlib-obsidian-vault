---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 3
delivery_batch: "follow-up-18"
catalogue_family: E12
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
yellowpaper_commit: "efc5f9a1f356cba376c978eedb63cb0363c2aa85"
kevm_source_version: "CSF 2018 author PDF at the catalogue URL"
kevm_sha256: "c07487d3c3c2ea085c74759a7d2141040c8c2c25aa1cb94f363c009272b7650c"
tags:
  - ethlib
  - external-sources
  - evm
  - operational-semantics
  - verification
---

# KEVM semantics and validation evidence boundaries

An executable semantics, a passing test suite and a program proof provide different evidence.
KEVM's paper explains how one operational definition can support both concrete execution and deductive verification.
This note separates those claims and traces a small program under a separately specified historical schedule.
No KEVM–EthLib refinement or general EVM conformance result follows from that trace.

## Exact sources and historical scope

The comparison source is [KEVM: A Complete Formal Semantics of the Ethereum Virtual Machine][kevm], the CSF 2018 author PDF at the catalogue URL.
The retrieved file has 14 pages, 303178 bytes and the SHA-256 recorded in frontmatter.
Its PDF metadata reports creation in 2021, which does not establish a separate 2021 protocol revision.
Reading covered selected introductory scope, §III's configuration/execution/gas rules, §IV-A's evaluation, §V-A's logic, selected contract-proof qualifications, and the relevant §VI/§VIII boundaries.
The underlying K metatheory, complete semantics implementation and proof artifacts were not audited.

Section III-E makes gas constants depend on a selected fee schedule and shows `DEFAULT` and `EIP150` examples.
This mechanism does not establish that the historical paper covers every later opcode or fork rule.
Section VIII explicitly anticipates further changes as the EVM evolves.

For numeric instruction rules, this note instead uses the [Yellow Paper's pinned Shanghai text][yp], specifically Execution Model, Message Call, Appendix G's fee schedule and Appendix H's instruction definitions.
Shanghai is this calculation's explicit baseline, not a claimed capability of the 2018 KEVM artifact.
The paper supplies comparison methods and research evidence, while published protocol sources justify EthLib rules under its contribution policy.

## A semantic definition and three meanings of completeness

Section III represents execution using cells such as program counter, word stack, memory, gas and accounts.
Rewrite rules change the cells they mention and retain the remaining configuration.
Its execution cycle separates exception checks, opcode computation, program-counter movement and exception handling, using saved state and explicit control operators.
This gives a concrete object to interpret, symbolically execute or reason about.

The paper reports passing its publication-era test suites.
Table II lists 40683 cases in `VMAll` and 22955 in `GSAll`, as separate test families.
These historical counts are reported evidence, not tests reproduced here or an exhaustive enumeration of all machine states.
Section VI-B also notes that its `VMTESTS` mode does not execute CALL and CREATE, so a comparison must identify the harness as well as the inputs.

Section V-A describes Reachability Logic as sound and relatively complete, with the language's execution semantics as trusted axioms in Figure 3.
That proof-system description differs from covering all opcodes or passing all tests.
It does not make every desired claim automatic, nor independently establish that the trusted semantics implements Ethereum's intended rules.
The paper's program-proof discussion also retains input bounds and sufficient-gas conditions.
A proof must preserve those hypotheses when someone reuses its result.

## A complete small instruction trace

Choose an already-entered frame with empty initial stack and the following six bytes:

```text
byte offset:  0  1  2  3  4  5
code:        60 02 60 03 01 00
instruction: PUSH1 2; PUSH1 3; ADD; STOP
```

Use nine units of frame gas, with no intrinsic transaction or entry cost included.
The program accesses no memory and executes no persistent writes.
Under the pinned Shanghai schedule, each PUSH1 and ADD costs three gas, while STOP costs zero.
Stack notation places the top on the left, and arithmetic words are modulo `2^256`.

| PC before | Operation | Stack before → after | Gas before → after | PC after |
| --- | --- | --- | --- | --- |
| 0 | PUSH1 2. | `[] → [2]`. | `9 → 6`. | 2. |
| 2 | PUSH1 3. | `[2] → [3,2]`. | `6 → 3`. | 4. |
| 4 | ADD. | `[3,2] → [5]`. | `3 → 0`. | 5. |
| 5 | STOP. | `[5] → [5]`. | `0 → 0`. | 6. |

PUSH1 skips its immediate byte, rather than interpreting that byte as another opcode.
At ADD, the first popped operand is three and the second is two.
The final five is an internal stack value, not return data: this program executes no RETURN.
STOP gives a normal halt with an empty output sequence.
Zero remaining gas is compatible with success because the next instruction costs zero.

Addition alone cannot detect reversed operands.
Replace only byte four with `03`, yielding `600260030300` and SUB instead of ADD.
The same initial gas and stack give `3-2 = 1`, while reversing the operands would give `2^256-1`.
This is an operand-order control, not another arithmetic security theorem.

## Exceptional termination is a different outcome

With the original code and eight initial gas, the two pushes leave PC four, stack `[3,2]` and two gas.
ADD requires three gas, so the exceptional-halting predicate holds before its arithmetic result is committed.
There is no successful ADD step producing five in that run.
The Message Call rule gives zero gas back on this exceptional outcome and discards the call's state effects.
The diagnostic stack before failure is not a returned stack specification.

A separate `0100` program with an empty stack and nine gas fails the stack-size condition for ADD despite having enough gas for its price.
This isolates underflow from insufficient gas, without choosing precedence for an input that fails both checks.
The Yellow Paper distinguishes its exceptional-result marker from the empty sequence of a normal STOP.
Thus neither empty returned bytes nor zero remaining gas alone identifies success.
These cases say nothing about a transaction's intrinsic charge, fee settlement or any value transfer outside the traced instructions.

## Reachability and selected EthLib observations

Section V-A uses all-path reachability: a claim `phi => psi` says that executions starting in phi reach psi or do not terminate under the given semantics.
That partial-correctness reading must not be silently replaced with an unconditional termination guarantee.
Gas bounds and outcome conditions still belong in the property being proved.
For this fixed trace, the desired outcomes are explicit, but computing them does not prove an arbitrary bytecode program satisfies its postcondition.

At the recorded EthLib commit, [`binaryOp`][lean-binary] pops its first operand before its second and then applies the supplied function.
[`pushN`][lean-push] advances past the immediate bytes, while [`stopStep`][lean-stop] clears the running flag without charging gas.
[`runFrameWith`][lean-run] separates frame-halting errors from errors that abort the whole run, using [`exceptionalHalt`][lean-exception] for the former.
These selected definitions were read, without claiming a proof that their complete executions match KEVM or the Yellow Paper at every fork.

Standalone Python checked the displayed PC, stack and gas arithmetic and the isolated SUB, out-of-gas and underflow controls.
No KEVM executable, EVM client, contract, EthLib binding, Lean build, proof replay, historical test suite or benchmark ran.
A useful contribution must pin both semantics and test revisions, relate their states and outcomes, and state whether its evidence is a concrete check or a universally quantified correspondence proof.

[kevm]: https://fsl.cs.illinois.edu/publications/hildenbrandt-saxena-zhu-rodrigues-daian-guth-moore-zhang-park-rosu-2018-csf.pdf
[yp]: https://github.com/ethereum/yellowpaper/blob/efc5f9a1f356cba376c978eedb63cb0363c2aa85/Paper.tex
[lean-binary]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Step.lean#L64-L74
[lean-push]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/StackOnly.lean#L47-L61
[lean-stop]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Instructions/ControlFlow.lean#L32-L34
[lean-run]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Interpreter.lean#L176-L223
[lean-exception]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Execution/Evm/Step.lean#L86-L93
