---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
delivery_batch: "follow-up-3"
catalogue_family: W02
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
devp2p_commit: "88f1c80800d622cc3a542a2125bbe33cc18c9f9c"
tags:
  - ethlib
  - external-sources
  - discovery-v5
---

# Discovery v5 challenge and key binding

Discovery v5 binds a handshake to a challenge, an initiator's identity key and the intended recipient.
Its signature and key derivation use different labels and different inputs.
Matching one hash or deriving two keys does not establish that a caller matched the expected peer, retained the right challenge or enforced nonce freshness.

## Sources and scope

This note uses the official devp2p [Theory][theory], [Wire Protocol][wire] and [Wire Test Vectors][vectors] at commit `88f1c80800d622cc3a542a2125bbe33cc18c9f9c`.
The theory and wire documents identify themselves as protocol version v5.1, while the packet header's version field is `0x0001`.
Reading covered theory's Nodes, Records and Distances and Sessions sections, wire's Notation, UDP Communication and Packet Encoding sections, and the cryptographic test vectors.
Topic discovery, routing-table behavior and the full handshake state machine are outside this note.

The [ENR note](ENR-record-bytes-and-identity.md) explains the signed record and identity-key representation.
The [HKDF note](HKDF-extract-expand-and-context.md) explains extraction, expansion and context arguments.
Code observations below pin EthLib `4f5f4111d17b040fe562bc0f2c362ebf625ed78d` and cover only its `v4` identity-scheme path.

## A conditional five-step trace

Suppose A wants to send B a FINDNODE request, has B's validated record, and associates its public key with the expected B identity.
Suppose B has no usable session with A, and both actors follow the challenge and endpoint rules below.
The following is an explanatory trace, not a captured exchange or one combined published fixture.

1. A sends an ordinary packet with a fresh 12-byte message nonce `q`.
   Without session keys, its message content is random.
2. B cannot authenticate that message and sends WHOAREYOU with header nonce `q` and a unique random 16-byte `id-nonce`.
   The challenge also carries the sequence number of A's record that B knows.
   B uses sequence zero if it has no record, and must retain the challenge and any known record briefly for the response.
3. A matches `q` to its pending request and generates a fresh ephemeral key.
   It derives session keys with B's static public key and signs the identity-proof input below with A's static private key.
   A sends a handshake packet with a new message nonce, containing the repeated request encrypted under `initiator-key`.
   In this trace B advertised sequence zero, so A must include its ENR.
4. B loads its stored challenge and validates the included ENR signature.
   It verifies A's identity signature under that record's key, derives the keys using B's static private key, and authenticates the request.
   On success B responds under `recipient-key`.
5. A authenticates that response under `recipient-key` before treating B's identity and the new keys as verified.

WHOAREYOU itself carries no encrypted/authenticated message.
Header masking uses AES-CTR with the first 16 bytes of the destination node ID, a public value.
Recognizing that masked header therefore does not authenticate B.

## Sign the complete unmasked challenge

The [theory's step 3][step3] defines:

```text
challenge-data = masking-iv || static-header || authdata
id-signature-input = "discovery v5 identity proof"
                     || challenge-data || ephemeral-pubkey || node-id-B
id-signature = ECDSA_sign_A(SHA256(id-signature-input))
```

The label is 27 ASCII bytes, without a terminator or length prefix.
For `v4`, the ephemeral public key is 33 compressed secp256k1 bytes and B's node ID is 32 bytes.
The signature is 64 bytes `r || s` without a recovery byte.
This uses SHA-256, whereas an ENR's own `v4` signature hashes its content with Keccak-256.

A WHOAREYOU challenge has 63 unmasked bytes: 16-byte masking IV, 23-byte static header and 24-byte authdata.
The static header includes the echoed 12-byte request nonce.
The authdata includes the distinct 16-byte `id-nonce` and an 8-byte ENR sequence number.
Signing only `id-nonce` loses fields that the specified preimage binds.
Signing the masked packet instead also selects different bytes.

The published [ID Nonce Signing vector][sign-vector] supplies this complete challenge:

```text
000000000000000000000000000000006469736376350001010102030405060708090a0b0c00180102030405060708090a0b0c0d0e0f100000000000000000
```

It has zero masking IV, protocol ID `discv5`, version `0001`, flag `01`, request nonce `0102030405060708090a0b0c`, authdata length `0018`, ID nonce `0102030405060708090a0b0c0d0e0f10` and sequence zero.
Its remaining signing inputs and published signature are:

```text
ephemeral-pubkey = 039961e4c2356d61bedb83052c115d311acb3a96f5777296dcf297351130266231
node-id-B = bbbb9d047f0488c0b5a93c1c3f2d8bafc7c8ff337024a55434a0d0555de64db9
static-key = fb757dc581730490a1d7a00deea65e9b1936924caaea8f44d476014856b68736
id-signature = 94852a1e2318c4e5e9d422c98eaf19d1d90d876b29cd06ca7cb7546d0fff7b484fe86c09a064fe72bdbef73ba8e9c34df0cd2b53e9d65528c2c7f336d5dfc6e6
```

The private scalar is public test material, not a usable secret.
The concatenated signing input has `27 + 63 + 33 + 32 = 155` bytes.
An author-derived SHA-256 intermediate, not printed in the vector, is:

```text
f8b18af81856bc494b09db6930f1feebc8bbebc5c1738c58ed5dc1281618ddc0
```

## ECDH and directional keys

For `v4`, [identity-specific cryptography][crypto] encodes the ECDH result as the **33-byte compressed point** `ephemeral-key * B_static_pubkey`.
It does not use only the point's 32-byte `x` coordinate.
B computes the same point as `B_static_key * ephemeral-pubkey`.
A's static identity key signs the handshake but does not supply this ECDH scalar.

```text
secret = compressed_ECDH_point
info = "discovery v5 key agreement" || node-id-A || node-id-B
PRK = HKDF-Extract(challenge-data, secret)
key-data = HKDF-Expand(PRK, info, 32)
initiator-key = key-data[0:16]   -- A sends, B receives
recipient-key = key-data[16:32] -- B sends, A receives
```

HKDF uses HMAC-SHA-256.
Its salt is the complete challenge and its input keying material is `secret`.
Its 90-byte info consists of a 26-byte label followed by the ordered node IDs.
The requested 32-byte output occupies one HKDF expansion block.
Swapping the node-ID order or key halves changes the directional agreement.

The separate published [Key Derivation vector][kdf-vector] uses the same challenge above with:

```text
ephemeral-key = fb757dc581730490a1d7a00deea65e9b1936924caaea8f44d476014856b68736
dest-pubkey = 0317931e6e0840220642f230037d285d122bc59063221ef3226b1f403ddc69ca91
node-id-A = aaaa8419e9f49d0083561b48287df592939a8d19947d8c0ef88f2a4856a69fbb
node-id-B = bbbb9d047f0488c0b5a93c1c3f2d8bafc7c8ff337024a55434a0d0555de64db9
initiator-key = dccc82d81bd610f4f76d3ebe97a40571
recipient-key = ac74bb8773749920b0d3a8881c173ec5
```

An isolated integer ECDH calculation derived shared secret `022c82f214eb37159111712add00040fcdf73fd4d7d0b7c0f980da4d099aa59ba4`.
Standard-library HMAC/SHA-256 then reproduced both published keys.
This primitive fixture does not establish that its node-ID inputs were derived from its supplied keys.
It also does not combine with the separate signature fixture into a complete handshake.

## AEAD and stateful obligations

Packet messages use AES-128-GCM with a 12-byte message nonce and additional authenticated data `masking-iv || unmasked-header`.
The identity signature authenticates A's ephemeral key and challenge context, while GCM checks possession of the derived directional key and protects the packet's message/header association.
These are different checks.

The [session-cache section][cache] requires tying session secrets and handshakes to a specific UDP endpoint, rather than node ID alone.
It also requires unique message nonces and strongly recommends a 32-bit outgoing counter followed by 64 cryptographically random bits.
The [implementation considerations][considerations] say WHOAREYOU is valid only as a response to a previously sent request.
They recommend retaining pending requests and re-sending an existing challenge when another ordinary packet arrives during that handshake.
Replay resistance depends on those challenge, nonce and cache rules alongside the signature equation.

## EthLib boundary and review limits

[`ecdh`, `deriveKeys` and `idSignatureHash`][lean-identity] implement the selected point encoding, ordered HKDF context and signing preimage.
[`acceptHandshake`][lean-accept] verifies the identity signature, decodes the compressed ephemeral point and derives keys.
[`openHandshake`][lean-open] additionally authenticates the encrypted message, but takes A's trusted public key and challenge data as inputs.
It explicitly does not validate the carried ENR, bind that record's key to A, or compare the source ID against the record.
Those caller obligations remain even when the operation succeeds.

[`makeWhoareyou` and `readWhoareyou`][lean-challenge] construct or recover challenge bytes without a freshness cache.
[`openMessage`][lean-message] takes a node-ID-to-key lookup, leaving the source's endpoint binding to its caller.
These observations do not establish a complete protocol implementation or a missing-proof task.

Checks covered input widths, the signature-preimage hash and published KDF outputs using isolated Python integer arithmetic, `hashlib` and `hmac`.
The source passages, explanation, and selected definitions received a separate review.
Independent integer-curve and HMAC calculations reproduced the published directional keys, and a separate byte/hash check reproduced the 155-byte identity input and its SHA-256 digest.
No ECDSA verification, packet-vector decryption, EthLib execution, Lean proof replay or live handshake ran.
Neither these byte checks nor the ephemeral-static construction establish a general protocol-security theorem.

[theory]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-theory.md
[wire]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-wire.md#packet-encoding
[vectors]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-wire-test-vectors.md
[step3]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-theory.md#step-3-node-a-processes-the-challenge
[crypto]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-theory.md#identity-specific-cryptography-in-the-handshake
[sign-vector]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-wire-test-vectors.md#id-nonce-signing
[kdf-vector]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-wire-test-vectors.md#key-derivation
[cache]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-theory.md#session-cache
[considerations]: https://github.com/ethereum/devp2p/blob/88f1c80800d622cc3a542a2125bbe33cc18c9f9c/discv5/discv5-theory.md#handshake-implementation-considerations
[lean-identity]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Discv5/Identity.lean#L36
[lean-accept]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Discv5/Handshake.lean#L117
[lean-open]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Discv5/Handshake.lean#L174
[lean-challenge]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Discv5/Session.lean#L83
[lean-message]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Discv5/Session.lean#L111
