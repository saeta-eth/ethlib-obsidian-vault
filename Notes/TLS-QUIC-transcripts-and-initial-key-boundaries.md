---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
phase: 5
delivery_batch: "follow-up-11"
catalogue_family: W06
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
libp2p_commit: "c750236de43cd8b246a824b377dfdb26bae686db"
rfc8446_version: "August 2018"
rfc9000_version: "May 2021"
rfc9001_version: "May 2021"
rfc7301_version: "July 2014"
tags:
  - ethlib
  - external-sources
  - tls
  - quic
---

# TLS/QUIC transcripts and Initial-key boundaries

QUIC Initial keys can be derived by anyone who sees the client's initial Destination Connection ID.
They are not peer credentials.
TLS transcript authentication, application identity checks, and transport-parameter validation remain separate obligations.

## Sources and scope

The primary RFCs are [RFC 8446, TLS 1.3][8446], August 2018; [RFC 9000, QUIC transport][9000] and [RFC 9001, Using TLS to Secure QUIC][9001], May 2021; and [RFC 7301, ALPN][7301], July 2014.
Their headers identify all four as IETF Standards Track documents.
Reading covered RFC 8446 §§4.4.1, 4.4.3–4.4.4 and the §7.1 label construction; selected RFC 9001 §§4.1–4.4, §§5.1–5.2, §7, §§8.1–8.2 and Appendix A.1; RFC 9000 §§7.4 and 12.3; and RFC 7301 §§3.1–3.2.
The RFC 8446 extension table was also checked for TLS 1.3's ALPN placement.
The worked control covers QUIC version 1 key derivation, not a complete TLS/QUIC connection or later QUIC versions.

At the recorded libp2p commit, the [TLS profile][libtls] identifies itself as 3A, Recommendation, Active, revision r0 (2019-03-23).
The [QUIC profile][libquic] has the same stage/maturity/status and revision r1 (2022-12-30).
Reading covered handshake, peer authentication, ALPN, and QUIC-version selection; certificate fixtures were not validated.
These lifecycle labels do not establish network adoption or EthLib support.

## A transcript, a signature, and Finished establish different facts

TLS's transcript hashes encoded handshake messages, including their type and length fields, while excluding TLS record headers.
QUIC carries the handshake bytes in CRYPTO frames rather than using TLS record protection; CRYPTO and QUIC packet headers are not transcript messages.
After HelloRetryRequest, RFC 8446 replaces ClientHello1 with a synthetic `message_hash` handshake message containing its hash.
Simply hashing every received packet or concatenating both ClientHellos unchanged is incorrect.

For certificate authentication, CertificateVerify signs the transcript prefix through Certificate using the certificate's private key and a role-specific context.
Finished instead derives `finished_key` from the sender's handshake traffic secret and computes HMAC over the appropriate transcript prefix before that Finished.
Recipients MUST verify Finished and terminate on an incorrect value.
Finished supplies handshake/key confirmation; it does not independently select the certificate trust policy or the intended application identity.

In QUIC, handshake completion is endpoint-relative: that endpoint has sent Finished and verified its peer's Finished.
Confirmation is a further QUIC state, with separate server/client rules in RFC 9001 §4.1.2.
Having keys available is not the same event as either state.

## Four encryption levels, three packet-number spaces

RFC 9001 distinguishes Initial, 0-RTT, Handshake, and 1-RTT encryption levels.
CRYPTO frames use Initial, Handshake, and 1-RTT; they do not use 0-RTT.
RFC 9000 assigns packet-number spaces as follows:

| Packet type | Packet-number space |
| --- | --- |
| Initial | Initial. |
| Handshake | Handshake. |
| 0-RTT and 1-RTT | Shared application-data space. |

Each sender MUST NOT reuse a packet number within the same space in one connection.
Thus switching from 0-RTT to 1-RTT does not create a fresh application-data number space.
QUIC may need multiple encryption levels concurrently because packets can be lost or reordered.
These distinctions alone do not implement replay handling or make early application actions safe.

Transport parameters are unilateral declarations, not one negotiated set of equal values.
QUIC carries them in the ClientHello and EncryptedExtensions and requires the extension.
They may become available early, but their authenticity is not established until handshake completion.
RFC 9001 §8.2 explicitly forbids depending on that authenticity before completion.

ALPN selects the application protocol; it does not select a peer identity.
RFC 7301 defines the protocol-name list, while TLS 1.3 places the server's selection in EncryptedExtensions rather than the older ServerHello location.
QUIC requires authenticated protocol negotiation and, absent another agreement mechanism, MUST use ALPN.
The pinned libp2p profiles select the string `libp2p`.

## Complete published QUIC v1 Initial-key control

RFC 9001 §5.2 fixes SHA-256 and this version-1 salt.
Appendix A.1 uses the eight decoded DCID bytes, not sixteen ASCII hexadecimal characters:

```text
initial_salt = 38762cf7f55934b34d179ae6a4c80cadccbb7f0a
DCID         = 8394c8f03e515708
initial_secret = HKDF-Extract(initial_salt, DCID)
               = 7db5df06e7a69e432496adedb00851923595221596ae2ae9fb8115c1e9ed0a44
```

`HKDF-Expand-Label(secret,label,empty,L)` expands using this serialized info:

```text
uint16_BE(L) || uint8(length("tls13 " || label)) || "tls13 " || label || 00
```

The final `00` is the empty context's length byte.
It is not a SHA-256 hash of an empty transcript.
The six-byte `tls13 ` prefix includes its trailing space, with no trailing NUL.
The published info bytes are:

```text
client in: 00200f746c73313320636c69656e7420696e00
server in: 00200f746c7331332073657276657220696e00
quic key:  00100e746c7331332071756963206b657900
quic iv:   000c0d746c733133207175696320697600
quic hp:   00100d746c733133207175696320687000
```

Expand `initial_secret` with `client in` or `server in` to 32 bytes, then expand that directional secret with the three `quic` labels.
Appendix A.1 publishes every result below:

| Value | Client direction | Server direction |
| --- | --- | --- |
| Initial traffic secret, 32 bytes | `c00cf151ca5be075ed0ebfb5c80323c42d6b7db67881289af4008f1f6c357aea` | `3c199828fd139efd216c155ad844cc81fb82fa8d7446fa7d78be803acdda951b` |
| Payload key, 16 bytes | `1f369613dd76d5467730efcbe3b1a22d` | `cf3a5331653c364c88f0f379b6067e37` |
| IV, 12 bytes | `fa044b2f42a3fd3b46fb255c` | `0ac1493ca1905853b0bba03e` |
| Header-protection key, 16 bytes | `9f50449e04a0e810283a1e9933adedd2` | `c206b8d9b9f0f37644430b490eeaa314` |

Client-direction keys protect packets sent by the client and read by the server; the other direction reverses those roles.
All derivation inputs are public, so passing Initial packet protection cannot authenticate a peer against an observer who can derive these keys.
RFC 9001 §7 warns about Initial tampering, including ACK information that is not otherwise authenticated.
After Retry, Initial secrets use the server-selected DCID as §5.2 specifies; an ordinary subsequent connection-ID change is not that rederivation event.

## libp2p's certificate key is not its host identity key

The pinned TLS profile requires mutual authentication using one self-signed certificate, with validity-time and self-signature checks and a required libp2p extension.
The extension contains the public host key and its signature over `libp2p-tls-handshake:` concatenated with the certificate public key's encoded SubjectPublicKeyInfo.
That host signature binds the certificate key to a libp2p identity; CertificateVerify binds possession of the certificate key to this handshake.
This is a different trust profile from selecting a Web PKI certificate chain for a hostname.
The profile requires peers to verify the host signature, and clients MUST check that the derived Peer ID matches the intended Peer ID.
The [Noise identity note](Noise-handshakes-and-libp2p-peer-identity.md) treats a different signed input and transport profile.
Do not interchange their labels or infer a Peer ID from the QUIC DCID.

## Selected code and check limits

At the pinned EthLib commit, [`initialSecrets` and `packetKeys`][keys] separate public Initial derivation from directional packet-key expansion.
Their types record the supported key and IV sizes.
[`Transcript.append`][transcript] preserves complete received handshake bytes and handles the SHA-256 retry transcript; its framing checks do not authenticate the peer.
[`finished`][finished] computes an expected Finished value, leaving comparison to its caller.
These are selected helper observations, not a description or audit of every handshake step or proof.

Python `hashlib`/`hmac` reproduced the five published info encodings and all nine published secret/key/IV values in the control.
No packet encryption, header protection, transcript/Finished vector, certificate validation, Peer-ID check, libp2p connection, EthLib binding, Lean build, or proof replay ran.

[8446]: https://www.rfc-editor.org/rfc/rfc8446
[9000]: https://www.rfc-editor.org/rfc/rfc9000
[9001]: https://www.rfc-editor.org/rfc/rfc9001
[7301]: https://www.rfc-editor.org/rfc/rfc7301
[libtls]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/tls/tls.md
[libquic]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/quic/README.md
[keys]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Quic/Keys.lean#L40
[transcript]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Quic/Tls/Transcript.lean#L56
[finished]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Quic/Tls.lean#L113
