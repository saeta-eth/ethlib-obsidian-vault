---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: W04
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
snappy_commit: "7406111ac4ae539ea0db8b7ea2dc76730cd957f4"
consensus_specs_commit: "aa16bb4c156184e9548a997d53efaf2a228a6304"
rfc3720_version: "April 2004"
tags:
  - ethlib
  - external-sources
  - snappy
  - req-resp
---

# Snappy frames and response chunk boundaries

A Snappy raw block, a Snappy framed stream and a consensus response chunk have different boundaries and length fields.
One decoded message can cross several Snappy chunks, while a response can contain several separately encoded messages.
This note derives small byte examples before connecting those layers.

## Sources and their roles

The primary format descriptions are Google's [raw format][raw] and [framing format][framing] at commit `7406111ac4ae539ea0db8b7ea2dc76730cd957f4`.
Both were read completely, including their stated revisions of 2011-10-05 and 2013-10-25.
The raw document calls itself an explanatory format description rather than a formal specification.
[RFC 3720 §12.1][crc] supplies the CRC-32C polynomial and mapping, with byte numbering from §2.3.3 and four 32-byte controls from [Appendix B.4][controls].
Its iSCSI packet layout is not imported into Snappy.

The supporting profile is consensus-specs commit `aa16bb4c156184e9548a997d53efaf2a228a6304`.
Reading covered Phase 0 networking's payload-size constants, maximum-size helpers, gossip encoding, Req/Resp interaction and SSZ-snappy strategy, plus Altair's response context and block-method v2 sections.
Under EthLib's source policy, this peer specification supplies profile/provenance context, not authority above the primary formats or EthLib's own definitions.
The selected files describe fork-specific profiles at that revision, not every later networking amendment or deployed client.

## A five-byte raw block

Let the decoded bytes be ASCII `abc`, or `61 62 63`.
The [raw preamble][raw] declares decoded length 3 as varint `03`.
For a literal of at most 60 bytes, the tag's low two bits are `00` and its high six bits hold `length - 1`.
Thus the tag is `(3 - 1) << 2 = 08`:

```text
03 08 61 62 63
│  │  └──────── three literal bytes
│  └─────────── literal tag, decoded length 3
└────────────── raw-block decoded-length preamble
```

These five encoded bytes contain no framed-stream identifier, checksum or response code.
The preamble permits decoded lengths up to `2^32 - 1`, using at most five varint bytes.
That limit is neither a recommendation to allocate it nor the networking profile's payload bound.
Copies can overlap already decoded data, but offset zero and offsets reaching before its start are not legal.

## A complete uncompressed framed stream

Framing uses `type:1 || body-length:3 || body`, with the length little-endian and excluding the four-byte header.
The [required initial identifier][framing] is ten bytes: `ff 06 00 00` followed by ASCII `sNaPpY`.
An uncompressed data chunk has type `01`, four checksum bytes and the data itself.
It does not embed the raw block's `03 08` preamble and literal tag.

The checksum is CRC-32C, not an interchangeable ordinary CRC-32.
The RFC's generator is `0x11edc6f41`, including its degree-32 term.
A reflected calculation uses `0x82f63b78`, starts with `0xffffffff` and complements the result.
For `abc`, that calculation gives `0x364b3fb7`.
These three-byte-input results are derived calculations, not published RFC test vectors.

Snappy [masks the checksum][framing] by rotating right 15 bits, then adding `0xa282ead8` modulo `2^32`:

```text
CRC32C(61 62 63)                     = 364b3fb7
ROR32(CRC32C,15) + a282ead8 mod 2^32 = 21f1576e
stored little-endian checksum        = 6e 57 f1 21
```

The checksum covers the decoded `abc`, not the frame header, encoded literal tag or outer protocol header.
The complete 21-byte stream is:

```text
ff060000734e61507059 01070000 6e57f121 616263
└──── identifier ──┘ └header┘ └─ CRC ┘ └data┘
```

Its data-chunk length is 7, counting checksum 4 plus data 3.
A compressed-data chunk instead uses type `00` and places the five-byte raw block after that same checksum:

```text
ff060000734e61507059 00090000 6e57f121 0308616263
```

This alternative occupies 23 bytes, with body length 9.
Using a compressed chunk does not guarantee smaller output for a tiny literal.
The examples are complete author-derived streams, without a claim that a particular compressor emits them.

## The SSZ prefix counts decoded bytes

The selected [Req/Resp profile][reqresp] sends:

```text
request payload = SSZ-byte-length varint || Snappy framed stream
response chunk  = result byte || context bytes || SSZ-byte-length varint
                  || Snappy framed stream
```

For illustrative SSZ type `Vector[uint8,3]`, the serialization can be `61 62 63`.
Its length prefix is `03`, followed by the complete 21-byte uncompressed stream above, making 22 bytes.
The prefix counts neither 21 framed bytes nor 7 chunk-body bytes.
With compressed framing, the outer SSZ prefix remains `03` even though the stream also contains its own raw-block preamble `03`.
This generic type illustration is not a valid `SignedBeaconBlock` response.

The [Altair profile][altair] adds context between the result and length prefix.
Phase 0 methods have empty context by default, as do nonzero-result error chunks.
Successful block-method v2 chunks use a four-byte `ForkDigest` to select the payload's fork namespace.
Do not consume those four bytes as a varint or include them in the Snappy checksum.
Conversely, do not assume every successful method has four context bytes.

## Response boundaries and resource limits

Snappy framing has no explicit end marker.
Its generic stream can contain repeated valid identifiers and skippable chunks.
A response chunk therefore needs the enclosing profile's decoded-length and method context, not a search for a special terminal Snappy byte.
The profile permits multiple response chunks and emits SSZ-list response elements individually.
Its trailing-byte/EOF rules must be considered alongside those outer response boundaries.
They do not mean that the next valid response chunk is an extra part of the previous payload.

At this profile revision, `MAX_PAYLOAD_SIZE` is 10 MiB, with the applicable SSZ-type bound also enforced.
The length prefix MUST be an unsigned varint of at most ten bytes, decoded with full `Uint64` range.
Minimal encoding is SHOULD, not MUST.
After a declared length `n`, the reader MUST NOT read more than `32 + n + n // 6` encoded bytes.
For `n = 3`, that bound is 35, covering either example frame.
Early EOF or inconsistent trailing payload bytes are invalid.
The profile requires an InvalidRequest response for invalid requests and rejection of invalid responses as bad server behavior.

The [framing description][framing] separately limits a compressed chunk to at most 65,536 decoded bytes.
For uncompressed chunks it uses “should”, then describes 65,540 as the maximum legal body length including checksum.
Reserved unskippable types `02`–`7f` should cause an error, while `80`–`fd` must be skipped.
Padding type `fe` must not have its contents interpreted or verified.
CRC-32C provides error detection, not peer authentication or SSZ semantic validity.

## Selected Lean boundaries and checks

At the recorded EthLib commit, [`decodeChunk`][lean-frame] validates decoded chunk sizes and checksum equality, rejecting oversize uncompressed chunks as well as compressed ones.
The selected consensus encoding passages do not independently state a universal CRC-validation requirement.
Keep the format's checksum definition, EthLib's rejection behavior and a particular network profile's obligations distinct.

[`readFrames`][lean-read] stops at the chunk completing the requested decoded length and returns trailing bytes.
[`decodePayload` and `decodeResponseChunk`][lean-reqresp] operate on one supplied payload/chunk, reject trailing bytes within that input and leave SSZ deserialization to the caller.
They are not a complete streaming response parser.
The [varint decoder][lean-varint] additionally enforces minimal encoding, a stricter choice than the profile's SHOULD.

A separate source review checked the cited passages and the limits of the explanation.
An independent calculation also reproduced the displayed example values.
A standalone bitwise CRC calculation reproduced all four RFC 3720 Appendix B.4 controls before deriving the `abc` checksum and masking.
Byte arithmetic checked both complete framed streams, the raw literal block and the 22-byte prefixed payload.
No Snappy decoder, consensus client, EthLib binding, Lean build or proof replay ran.
Malformed-input cases above identify source or observed-code conditions, not an executed exhaustive failure suite.

[raw]: https://github.com/google/snappy/blob/7406111ac4ae539ea0db8b7ea2dc76730cd957f4/format_description.txt
[framing]: https://github.com/google/snappy/blob/7406111ac4ae539ea0db8b7ea2dc76730cd957f4/framing_format.txt
[crc]: https://www.rfc-editor.org/rfc/rfc3720.html#section-12.1
[controls]: https://www.rfc-editor.org/rfc/rfc3720.html#appendix-B.4
[reqresp]: https://github.com/ethereum/consensus-specs/blob/aa16bb4c156184e9548a997d53efaf2a228a6304/specs/phase0/p2p-interface.md#ssz-snappy-encoding-strategy
[altair]: https://github.com/ethereum/consensus-specs/blob/aa16bb4c156184e9548a997d53efaf2a228a6304/specs/altair/p2p-interface.md#req-resp-interaction
[lean-frame]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Snappy/Frame.lean#L112
[lean-read]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Snappy/Frame.lean#L218
[lean-reqresp]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/ReqResp/Encoding.lean#L142
[lean-varint]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Encoding/Varint.lean#L179
