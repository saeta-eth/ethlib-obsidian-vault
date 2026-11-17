---
created: "2026-11-17"
last_reviewed: "2026-11-17"
status: reviewed
catalogue_family: W07
ethlib_commit: "4f5f4111d17b040fe562bc0f2c362ebf625ed78d"
libp2p_commit: "c750236de43cd8b246a824b377dfdb26bae686db"
tags:
  - ethlib
  - external-sources
  - gossipsub
  - networking
---

# Gossipsub validation, mesh and cache boundaries

A message can remain available for an `IWANT` response after its ID stops appearing in gossip.
Later, its payload can disappear while the seen cache still suppresses duplicates.
Neither cache state establishes application validity, and accepting a message does not promise its delivery to every subscriber.

## Sources and the three decisions

All libp2p sources here are fixed at the recorded commit.
The [v1.0 specification][v10] identifies itself as 3A, Recommendation, Active, revision r2 (2020-03-12).
The [v1.1 extension][v11] identifies itself as 2A, Candidate Recommendation, Active, revision r8 (2021-12-14), and extends v1.0 with backward-compatible changes.
The [general PubSub interface][pubsub] is 3A, Recommendation, Active, revision r3 (2020-09-25).
These are the documents' lifecycle labels, not evidence of deployment or Ethereum adoption.
Reading covered v1.0 parameters, router state, message processing and heartbeat; v1.1 explicit peering, backoff, scoring thresholds, heartbeat maintenance, invalid-message scoring and extended validators; and PubSub message identification.

Separate three questions: does the application accept these contents, has this message ID been processed, and should this peer belong to this topic's mesh?
The [extended-validator interface][validators] must support at least these outcomes:

| Outcome | Delivery and forwarding | Invalid-message penalty P₄ |
| --- | --- | --- |
| Accept | Message considered valid; should be delivered and forwarded | No invalid-message penalty from this result |
| Ignore | Neither delivered nor forwarded | Not triggered |
| Reject | Message considered invalid; should be rejected | Should be triggered |

Ignore is useful when an application cannot establish validity, such as during synchronization.
It is not an assertion that the message is valid or invalid.
Reject has a scoring consequence that Ignore deliberately avoids.
The source's P₄ component squares an invalid-message counter, but its total scoring function also has weights, other components and decay parameters; this table does not calculate a peer's score.

## An ID identifies a message under a shared policy

The [PubSub identification section][ids] defaults to concatenating the author `from` and `seqno` fields.
It also permits a user-defined function, for example hashing message data, and says all peers participating in a topic must use identical calculation logic or the topic will malfunction.
Available fields depend on the topic's signature policy.
A content-based policy can treat equal contents as duplicates even when an application wanted repeated publications.
An ID collision can similarly cause distinct messages to be treated as one.
An ID therefore is neither a validity certificate nor necessarily a globally unique identifier.

The v1.0 router checks application validation, self-publication and prior-seen status before forwarding a newly received payload.
Its seen cache also avoids requesting already-seen IDs advertised in `IHAVE` gossip.
This note does not prescribe whether Ignore or Reject inserts an ID into that cache: the selected generic clauses do not fix that ordering.
A duplicate suppressed by the router is not automatically an application-validator Ignore result.

## One accepted message across five heartbeats

The [v1.0 cache][cache] holds whole messages in heartbeat history windows.
`put` inserts into the current window; `get` searches retained windows; `get_gossip_ids` selects IDs in the most recent gossip windows.
Heartbeat emits gossip before shifting the windows.
The separately described seen cache is a timed LRU cache of IDs.

Use the source's configurable reasonable defaults: five retained windows, three gossip windows, a one-second heartbeat and a two-minute seen lifetime.
These are not Ethereum profile constants.
Construct one message A on joined topic T, accepted just before heartbeat 1, with A already inserted into both caches.
Assume A is not self-published, there are no further insertions or reinsertion, and no other cache eviction during this five-second trace.
Window 0 is newest.
The table describes cache eligibility, independently of whether any particular peer is selected to receive an `IHAVE`.

| State after event | A offered by this heartbeat before shifting? | A's retained window | `get(A)` succeeds? | A eligible in the next gossip selection? | Seen A? |
| --- | --- | --- | --- | --- | --- |
| Initial insertion | — | 0 | Yes | Yes | Yes |
| Heartbeat 1 | Yes | 1 | Yes | Yes | Yes |
| Heartbeat 2 | Yes | 2 | Yes | Yes | Yes |
| Heartbeat 3 | Yes | 3 | Yes | No | Yes |
| Heartbeat 4 | No | 4 | Yes | No | Yes |
| Heartbeat 5 | No | Evicted | No | No | Yes |

Heartbeat 3 still offers A because it inspects window 2 before shifting A to window 3.
After heartbeat 4, an `IWANT` for A can still obtain its payload from this cache although A is no longer advertised.
After heartbeat 5, this cache cannot answer with that payload; its seen entry still avoids a fresh request for an `IHAVE` containing A.
Retention, advertisement and duplicate suppression thus have different lifetimes.
The sequence is hand-derived, not a published vector or a network trace.

## Mesh membership has additional conditions

Under [v1.1 backoff][backoff], both peers avoid regrafting each other during the backoff period.
An early `GRAFT` is immediately pruned and extends the backoff; applying a behavioral P₇ penalty is optional.
One minute is recommended, not a universal duration.
The peer must wait the full period plus slack for heartbeat-based clearing.
The source does not prescribe one numerical extension rule here.

During [heartbeat maintenance][maintenance], a negative-scoring peer is pruned from all meshes and ignored when selecting peers to graft for undersubscription.
For example, an existing mesh peer with score −1 is removed at that check.
A score-0 peer still under backoff is not eligible for an outgoing regraft; a score-0 peer whose backoff has ended is only a candidate.
Topic participation, selection and mesh degree still matter, as do v1.1's outbound-connection quotas.
Explicit peers sit outside the mesh and receive new valid messages; attempting to graft one is an error that should result in `PRUNE`.
Accepting A therefore does not establish that any particular connection should join the mesh.

## Limited EthLib connections and checks

At the recorded EthLib commit, [ValidationResult][lean-validation] represents the three outcomes and short-circuits composed checks after Ignore or Reject.
[MessageCache][lean-cache] implements newest-first windows, topic-filtered gossip IDs and shifting.
Its `MessageId` is specifically 20 bytes, a narrower profile than generic PubSub's user-defined ID policy.
[receiveMessage][lean-receive] takes the ID function and validator from its caller, inserts an unseen joined-topic ID before validation, and caches/delivers/forwards accepted messages.
That insertion order is an observed local choice, not a conclusion imposed on all Gossipsub implementations.
[heartbeat][lean-heartbeat] constructs gossip before shifting its message cache and expiring seen entries.
EthLib's selected [parameter definition][lean-parameters] uses six retained windows and a 700-ms interval, so the five-window example above deliberately follows the generic source defaults instead.

A separate Python list-window calculation reproduced all six table rows, including three pre-shift advertisement opportunities and payload eviction at heartbeat 5.
It did not execute EthLib or model a network.
Source review and this example do not validate topic-specific Ethereum rules, score tuning, adversarial resilience, eventual delivery, or a complete implementation.
No client, binding, Lean build or proof replay ran for this note.

[v10]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.0.md
[v11]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.1.md
[pubsub]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/README.md
[validators]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.1.md#extended-validators
[ids]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/README.md#message-identification
[cache]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.0.md#message-cache
[backoff]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.1.md#prune-backoff-and-peer-exchange
[maintenance]: https://github.com/libp2p/specs/blob/c750236de43cd8b246a824b377dfdb26bae686db/pubsub/gossipsub/gossipsub-v1.1.md#heartbeat-maintenance
[lean-validation]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Gossip/Validation.lean#L29-L61
[lean-cache]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Gossip/Cache.lean#L37-L91
[lean-receive]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Gossip/Router.lean#L217-L231
[lean-heartbeat]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Gossip/Router.lean#L624-L646
[lean-parameters]: https://github.com/ethereum/EthLib/blob/4f5f4111d17b040fe562bc0f2c362ebf625ed78d/EthLib/Networking/Gossip/Parameters.lean#L62-L71
