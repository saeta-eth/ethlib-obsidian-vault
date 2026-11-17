---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
catalogue_family: W05
ethlib_commit: "3df5b24641b4a251ed094af02c02d28a621b31c7"
noise_revision: "34, 2018-07-11"
noise_pdf_sha256: "7562d34c04a661f5679918e56b139b066bd9fa6b821080dfe59eb8f80c7dccb7"
libp2p_specs_commit: "c750236de43cd8b246a824b377dfdb26bae686db"
tags:
  - ethlib
  - external-sources
  - networking
  - noise
---

# Noise handshakes and libp2p peer identity

An encrypted connection can terminate at a peer other than the one the caller intended.
Noise establishes keys for a channel; libp2p adds an identity signature binding a Noise static key to a libp2p identity.
When the caller selected a particular Peer ID, it also needs to compare that identity's derived ID with its expectation.
These checks answer different questions.

## Sources and applicability

| Source | Passages examined and role |
| --- | --- |
| [Noise revision 34][noise], dated 2018-07-11 | Sections 5.3, 7.1, 7.5, 7.7, 12.1, 12.3, 12.5, and 14. Handshake processing, XX order, payload properties, primitives, and application authentication policy. |
| [Pinned `noise/README.md`][libp2p] | Complete document, especially Static Key Authentication, libp2p Data in Handshake Messages, The libp2p Handshake Payload, XX, and Wire Format. |
| [Pinned `peer-ids/peer-ids.md`][peerids] | Keys, deterministic protobuf requirements, Ed25519, Secp256k1, Peer IDs, and text representations. Its inline schema specifies identity-key encoding. |
| [RFC 8032 §7.1][edvector] and [RFC 7748 §6.1][dhvector] | Public example inputs, combined below into an original case. |
| [Protobuf encoding guide][protobuf] | Message Structure, Length-Delimited Records, and Last One Wins; accessed 2026-11-17. Wire-format explanation. |

The libp2p links pin commit `c750236de43cd8b246a824b377dfdb26bae686db`.
These are version-specific rules, not a claim about every deployed implementation or later protocol revision.
EthLib observations below pin commit `3df5b24641b4a251ed094af02c02d28a621b31c7`.

## Three messages, two kinds of long-lived key

The required concrete protocol name is `Noise_XX_25519_ChaChaPoly_SHA256`.
Its DH operation is X25519; ChaCha20-Poly1305 encrypts and authenticates keyed messages, and SHA-256 supplies Noise's hash.
An **ephemeral** DH key is generated for a handshake.
A **static Noise key** supplies the other DH role; libp2p permits replacing it for each session.
The separate **identity key** signs that static public key and determines the Peer ID.
An Ed25519 identity key is not an X25519 DH key merely because both have 32-byte public encodings.

Let `I` be the initiator and `R` the responder.
Noise's tokens describe public-key transmission and local DH calculations:

```text
I -> R: e
R -> I: e, ee, s, es
I -> R: s, se
```

| Message | What happens |
| --- | --- |
| 1, initiator to responder | Send the initiator's ephemeral public key. No peer static key has yet been authenticated. |
| 2, responder to initiator | Send the responder's ephemeral key; mix ephemeral–ephemeral DH (`ee`); encrypt the responder's static key; mix initiator-ephemeral/responder-static DH (`es`); encrypt the responder's payload. |
| 3, initiator to responder | Encrypt the initiator's static key; mix initiator-static/responder-ephemeral DH (`se`); encrypt the initiator's payload. |

The initiator checks the responder's libp2p identity after receiving message 2; the responder checks the initiator's after receiving message 3.
Message 2 can therefore carry encrypted data to a recipient whose identity the responder has not yet authenticated.
Noise §7.7 distinguishes this property from encryption to a known recipient.
The pinned libp2p paragraph mistakenly names the unauthenticated party in message 2 as the responder; the XX order above makes the actors explicit.

## What the identity signature authenticates

The decrypted handshake payload is a `NoiseHandshakePayload` protobuf.
Field 1, `identity_key`, contains a serialized libp2p `PublicKey` message.
Field 2, `identity_sig`, contains the identity key's signature.
Optional field 4 contains extensions.
The signed message is exactly:

```text
UTF8("noise-libp2p-static-key:") || raw_X25519_static_public_key
```

There is no terminating zero byte, Peer ID, protobuf wrapper, or handshake transcript appended to this signature message.
The [signed-data note](Signed-data-preimages-and-domains.md) discusses why these distinctions matter for other protocols.
The libp2p source requires a receiver to decode the identity key into a usable form.
The receiver must then validate its signature against the static Noise key received in the handshake.
An invalid signature requires immediate connection termination.
The signature establishes possession of the identity secret when the payload was generated; it need not be generated freshly for this handshake.
Noise's DH exchange provides the separate connection to the static private key used by the channel participant.

To establish a connection to a previously selected Peer ID, derive the ID from the authenticated identity public key.
Compare binary IDs, since the same ID can have different text representations.
A valid binding for another ID does not satisfy that caller expectation.
This comparison expresses the caller's acceptability policy under Noise §14; it is not an invented expected-ID parameter in the pinned Noise API.
An incoming connection without a prior expected ID can instead learn the authenticated ID and apply its own policy.

## A public identity and exact bytes

This constructed case combines RFC 8032's Ed25519 Test 1 public key with RFC 7748's Alice X25519 public key, chosen here as a Noise static key:

```text
identity public key:
d75a980182b10ab7d54bfed3c964073a0ee172f3daa62325af021a68f707511a

Noise static public key:
8520f0098930a754748b7ddcb43ef75a0dbf3a0d26381af4eba4a98eaa9b4e6a
```

The ASCII prefix has 24 bytes, so the signature message has 56 bytes.
Concatenate the following hex lines and decode them as bytes:

```text
6e6f6973652d6c69627032702d7374617469632d6b65793a
8520f0098930a754748b7ddcb43ef75a0dbf3a0d26381af4eba4a98eaa9b4e6a
```

The Ed25519 profile signs those bytes using ordinary Ed25519.
Do not import the secp256k1 profile's external SHA-256 prehash into this example.
The RFC's published empty-message signature is not a signature of this constructed message.
No signature value or completed handshake is asserted here.

The deterministic identity protobuf uses field 1 with enum value `Ed25519 = 1`, then field 2 containing 32 public-key bytes:

```text
08 01 12 20
d75a980182b10ab7d54bfed3c964073a0ee172f3daa62325af021a68f707511a
```

`08` tags a field-1 varint; `12` tags field-2 bytes; `20` is their length, hexadecimal 32.
The result has 36 bytes.
Because it is at most 42 bytes, the Peer ID rules select the identity multihash rather than SHA-256:

```text
00 24 08 01 12 20
d75a980182b10ab7d54bfed3c964073a0ee172f3daa62325af021a68f707511a
```

This 38-byte binary Peer ID has identity code `00` and length `24`, hexadecimal 36.
It names the Ed25519 identity, not the X25519 static key.
Replacing the static key changes the signed message while leaving this identity's Peer ID unchanged.
This is a byte derivation from primary rules, not a published Noise test vector or an EthLib acceptance test.

## Early data and EthLib boundaries

The libp2p specification permits early data only for internal libp2p APIs and forbids user or application data there.
Processing must be idempotent because some early payloads may be replayed.
The initiator MUST NOT send extensions in message 1.
The Noise layer MUST NOT process early data during the handshake beyond the specified signature work; failed handshakes require immediate discard.
The source says early data **should** reach other libp2p components only after completion and signature validation.
That wording must not silently become a stronger uppercase requirement.
The identity signature covers the static key; the Noise payload encryption supplies the surrounding payload's channel protection.

At the pinned EthLib commit, [`signedStatic`, `readLibp2p`, and `complete`][identity] implement this separation for a **secp256k1 identity profile**, not the Ed25519 example above.
[`Libp2p.verify`][peer-code] accepts that profile's canonical key protobuf and low-`s` DER signature on SHA-256 of the message.
Its key protobuf has 37 bytes, and its binary identity Peer ID begins `00 25`.
The active identity decoder does not accept the example's Ed25519 protobuf.

`readLibp2p` validates the remote payload against the decrypted static token in message 2 or 3.
`complete` requires handshake step 3 and a stored remote with the same static key; it does not rerun the signature check or compare an expected Peer ID.
The theorem [`Produced.complete_authenticates`][proof] requires a state produced from initialization through successful reads and writes, plus successful completion.
Its conclusion retains the static-key binding, successful signature verification, and successful identity-key decoding.
It is not a computational handshake-security theorem for arbitrary constructed states.

This review read the cited passages and selected definitions and theorem statements, and recomputed the displayed byte lengths.
The sources and explanation received a separate review.
Independent calculations reproduced the displayed bytes and the RFC 8032 Test 1 signature as a control.
PyCryptodome 3.23.0 signed the constructed 56-byte message using the public test seed, verified it, and rejected a changed static key or omitted prefix.
These checks exercised the Ed25519 example with separate tools, not EthLib's secp256k1 profile.
No Noise handshake, Lean build, proof replay, or network interoperability test was run.
Protocol security games, resumption, randomness quality, and application acceptance policy remain outside this note's checked scope.

[noise]: https://noiseprotocol.org/noise_rev34.pdf
[libp2p]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/noise/README.md
[peerids]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/peer-ids/peer-ids.md
[edvector]: https://www.rfc-editor.org/rfc/rfc8032#section-7.1
[dhvector]: https://www.rfc-editor.org/rfc/rfc7748#section-6.1
[protobuf]: https://protobuf.dev/programming-guides/encoding/
[identity]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Networking/Noise/Identity.lean#L40-L168
[peer-code]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Networking/Libp2p/PeerId.lean#L57-L144
[proof]: https://github.com/ethereum/EthLib/blob/3df5b24641b4a251ed094af02c02d28a621b31c7/EthLib/Networking/Proofs/Noise/Identity.lean#L171-L248
