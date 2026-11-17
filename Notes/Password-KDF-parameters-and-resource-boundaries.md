---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: C05
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
ercs_commit: "dbba0f4479dd817d19d7d7771dea0dcd76c68d17"
rfc8018_version: "January 2017, PKCS #5 v2.1"
rfc7914_version: "August 2016"
tags:
  - ethlib
  - external-sources
  - password-kdf
  - resource-limits
---

# Password KDF parameters and resource boundaries

A password KDF can accept mathematically valid parameters that exceed a caller's memory or work budget.
A small published control specifies one byte mapping without recommending its cost for password protection.
Keep algorithm validity, password preparation, resource admission and security policy separate.

## Sources and scope

The primary texts are [RFC 8018, PKCS #5: Password-Based Cryptography Specification Version 2.1][8018], January 2017, and [RFC 7914, The scrypt Password-Based Key Derivation Function][7914], August 2016.
Both are Informational RFCs rather than Internet Standards Track documents.
Reading covered RFC 8018 §5.2, selected salt/iteration discussion in §§4.1–4.2 and §8, and RFC 7914 §§2, 4–6, the selected controls in §§11–12 and §14.
This is not a full review of scrypt's memory-hardness argument or contemporary password-cost guidance.

[ERC-2335][2335] has status Review at commit `dbba0f4479dd817d19d7d7771dea0dcd76c68d17`.
Reading here covers Password requirements, Decryption key and the published scrypt fixture's parameters.
The [validator-key and keystore note](Validator-key-derivation-and-keystores.md) already traces its Unicode preparation, checksum and AES roles.
This note extends that explanation into algorithm bounds and caller-managed resources at EthLib commit `4f5f4111d17b040fe562bc0f2c362ebf625ed78d`.

## Algorithm inputs are octets

Both KDF algorithms take password `P` and salt `S` as octet strings.
They do not choose a Unicode normalization rule for an application's password field.
ERC-2335 supplies that rule: NFKD normalization, control-code removal, then UTF-8 encoding.
Removing control code points is a character operation before encoding, not a filter over the UTF-8 bytes.
Its JSON salt is hex-decoded into bytes rather than passed as the ASCII characters of that hex representation.

The examples below use exactly the ASCII octets specified by the RFC, or empty octet strings.
They therefore test the KDFs without testing general Unicode agreement or the keystore's decryption pipeline.
The KDF name alone also does not identify PBKDF2's pseudorandom function.
Here it is explicitly HMAC-SHA-256, with output size `hLen = 32` bytes, as required by ERC-2335's PBKDF2 module.

## PBKDF2 bounds and a complete published control

[RFC 8018 §5.2][pbkdf2] requires positive iteration count `c` and positive output length `dkLen`.
The output bound is `(2^32 - 1) * hLen`, imposed by its four-byte block index.
For HMAC-SHA-256 this is `137438953440` bytes, or `128 GiB - 32 bytes`.
That algorithmic maximum is not a practical allocation limit.

For each output block `i`, PBKDF2 computes:

```text
U_1 = HMAC-SHA256(P, S || INT32_BE(i))
U_j = HMAC-SHA256(P, U_(j-1))
T_i = U_1 XOR ... XOR U_c
DK  = first dkLen bytes of T_1 || T_2 || ...
```

There are `ceil(dkLen/32)` output blocks and `c` HMAC evaluations per block.
The block index is big-endian and starts at 1.
It is not an iteration count appended to the password.

[RFC 7914 §11][pbkdf2-vector] publishes this complete 64-byte control:

```text
P = "passwd" = 706173737764
S = "salt"   = 73616c74
c = 1
dkLen = 64
DK = 55ac046e56e3089fec1691c22544b605f94185216dde0465e68b9d57c20dacbc
     49ca9cccf179b645991664b39d77ef317c71b845b1e30bd509112041d3a19783
```

Concatenate the two DK lines after decoding hex.
Because `c = 1`, each displayed 32-byte line is its block's `U_1`.
Their respective HMAC message suffixes are `00000001` and `00000002`.
The case makes two HMAC evaluations and needs no later XOR iterate.
At any fixed positive `c`, increasing output length from 32 to 33 bytes changes the count from `c` to `2*c` HMAC evaluations.
These counts do not measure processor instructions or elapsed time.

## scrypt has different cost parameters

[RFC 7914 §§2 and 6][scrypt-params] require `N > 1`, with N a power of two and strictly below `2^(16*r)`.
With positive block-size parameter `r`, the parallelization parameter must satisfy:

```text
p >= 1
p * 128 * r <= (2^32 - 1) * 32
1 <= dkLen <= (2^32 - 1) * 32
```

The strict N bound matters: with `r = 1`, `N = 65536` is excluded even though it is a power of two.
Zero `p`, zero output length and non-power-of-two N also violate the input requirements.
These are source conditions, not executed EthLib rejection tests.

scrypt first uses PBKDF2-HMAC-SHA-256 with `c = 1` to produce `p` blocks of `128*r` bytes.
It applies ROMix to each block, then uses another `c = 1` PBKDF2 call to produce DK.
N is the ROMix cost parameter, not a replacement PBKDF2 iteration count.
The independent ROMix blocks can be processed sequentially or concurrently; `p` does not require spawning that many operating-system threads.

[RFC 7914 §12][scrypt-vector] publishes this complete small case:

```text
P = empty octets
S = empty octets
N = 16, r = 1, p = 1, dkLen = 64
DK = 77d6576238657b203b19ca42c18a0497f16b4844e3074ae8dfdffa3fede21442
     fcd0069ded0948f8326a753a0fc81f17e8d3e0fb2e0d3628cf35e20c38d18906
```

Neither its empty password/salt nor its small N is a password-protection recommendation.
The example is deliberately bounded so its entire output can be checked without a large derivation.

## Count an algorithm, then identify the model

For one ROMix block, the specified V table holds `N` entries of `128*r` bytes.
The small control therefore has `128*1*16 = 2048` table-payload bytes, plus a 128-byte initial B buffer and other storage.
ROMix makes `2*N = 32` BlockMix calls, each with `2*r` Salsa20/8 core calls, giving 64 core calls.
The initial and final PBKDF2 steps make four and two HMAC calls respectively.
These are derived counts for the specified algorithm, not a lower bound for every possible implementation strategy.

The larger ERC-2335 fixture uses `N = 262144, r = 8, p = 1, dklen = 32`.
Its V-table payload alone is `128*8*262144 = 268435456` bytes, or 256 MiB.
This is arithmetic over published parameters; that expensive fixture was not derived here.

EthLib's [`Kdf.memory`][models] uses a separate buffer-payload model.
For the small control it counts `2048 + 4*128 + 64 + 64 = 2688` bytes.
It excludes object headers, pointers and ROMix temporary blocks, and its comment states that no theorem relates it to actual memory use.
Its [`Kdf.work`][models] counts SHA-256 compressions and Salsa20/8 cores as named work units.
The small case gives `64 + 16 + 12 = 92` mixed units: 64 cores and 28 SHA-256 compressions.
Those operations need not cost equal time, and the count excludes other work.

## Keep resource admission before derivation

The pinned [`pbkdf2`][lean-pbkdf2] and [`scrypt`][lean-scrypt] entry points check algorithm bounds.
Those checks do not choose an application's affordable maximum.
[`decryptStore`][decrypt] prepares the password, checks the caller's `KdfLimits`, then derives and decrypts.
The separately callable `deriveKey` does not apply those limits itself.
The model check also does not bound earlier JSON parsing or password normalization, nor prove a runtime or resident-memory cap.

RFC 8018's salt and iteration discussion explains costs of password search, while RFC 7914 warns about unreasonable parameters supplied by an attacker.
Choosing current protection costs requires a separate policy decision and measurements for the intended environment.
No historical iteration example here is promoted into current guidance.

Python's `hashlib` reproduced both published controls, and `hmac` reproduced the two `c = 1` PBKDF2 blocks independently of the PBKDF2 helper.
Integer arithmetic checked the table payloads, block/core/HMAC counts and the two EthLib model totals.
No EthLib binding, heavy scrypt case, benchmark, Unicode-conformance suite, Lean build or proof replay ran.
The selected source reading and examples do not establish complete keystore security or a memory-hardness reduction.

[8018]: https://www.rfc-editor.org/rfc/rfc8018.html
[7914]: https://www.rfc-editor.org/rfc/rfc7914.html
[2335]: https://github.com/ethereum/ERCs/blob/dbba0f4479dd817d19d7d7771dea0dcd76c68d17/ERCS/erc-2335.md
[pbkdf2]: https://www.rfc-editor.org/rfc/rfc8018.html#section-5.2
[pbkdf2-vector]: https://www.rfc-editor.org/rfc/rfc7914.html#section-11
[scrypt-params]: https://www.rfc-editor.org/rfc/rfc7914.html#section-2
[scrypt-vector]: https://www.rfc-editor.org/rfc/rfc7914.html#section-12
[models]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Keystore.lean#L76
[lean-pbkdf2]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Pbkdf2.lean#L75
[lean-scrypt]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Hash/Scrypt.lean#L261
[decrypt]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Cryptography/Keystore.lean#L234
