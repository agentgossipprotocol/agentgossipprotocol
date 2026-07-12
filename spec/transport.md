---
title: Transport
weight: 4
---

# Transport and the `Sync` Operation

**Status:** Draft

This document defines the **transport-independent** anti-entropy operation
that exchanges gossip records, and the abstract contract that any concrete
transport binding MUST satisfy. Nothing here depends on a particular host
protocol; the [A2A Binding](/spec/binding-a2a/) is one binding that satisfies this
contract, and is the preferred integration for A2A-compatible agents. RFC
2119 keyword conventions are inherited from [Core](/spec/core/).

## 1. The `Sync` Operation

AGP exposes exactly **one** operation: a resumable, bidirectional
anti-entropy exchange. Earlier drafts proposed five command-style RPCs
(`OpenSession`, `ExchangeDigest`, `OfferRecords`, `TransferRecords`,
`CloseSession`); this specification replaces them with a single streaming
state machine, because the five-RPC shape exposes implementation details as
protocol surface and forces round trips that add no correctness value.

Abstractly, `Sync` is a single bidirectional stream of `GossipFrame`s in
each direction. A binding MAY realize it as a streaming RPC, a sequence of
resumable request/response exchanges, or any other transport that meets the
contract in §5; the concrete method name and framing are binding-defined
(see the [A2A Binding](/spec/binding-a2a/) for the gRPC / HTTP+JSON realizations).

A session's first frame implicitly opens it; a `done: true` frame or
session expiry closes it. There is no separate open/close call.

### 1.1 Frame Contents

A `GossipFrame` MAY carry any combination of:

| Element | Purpose |
| --- | --- |
| `HELLO` | Version, limits, and digest-algorithm negotiation. |
| `SCOPE` | The sender's interest selectors for this session. |
| `SUMMARY` | Partition digests for anti-entropy comparison (§4). |
| `WANT` | Record IDs or partition ranges requested from the peer. |
| `RECORDS` | The signed records being transferred, plus their per-hop transfer envelopes (§2). |
| `ACK` | Acknowledgement of records accepted, by `recordId`. |
| `DONE` | Signals this side has nothing further for the session. |
| `ERROR` | A normalized protocol error (see `error.schema.json`). |

### 1.2 Exchange Sequence

A conforming `Sync` session proceeds through the following steps. Steps MAY
pipeline (e.g. a peer MAY send `SCOPE` and an initial `SUMMARY` in the same
frame) but MUST NOT skip validation or authorization steps.

1. Authenticate the peer using the binding's peer-authentication mechanism
   (§5; under A2A, the security scheme the peer's Agent Card declares —
   [A2A Binding §4](/spec/binding-a2a/)).
2. Confirm the peer advertises AGP support via the binding's capability
   advertisement (under A2A, `AgentCapabilities.extensions` —
   [A2A Binding §2](/spec/binding-a2a/)).
3. Negotiate the core protocol version, session limits, and digest
   algorithms (`HELLO`).
4. Compute the directional interest intersection for each direction
   ([Security §3](/spec/security/)).
5. Exchange partition summaries (`SUMMARY`, §4).
6. Where summaries differ, exchange inventories or Merkle-tree differences
   to identify which specific records differ (§4).
7. Transfer only the records actually requested (`WANT` → `RECORDS`) —
   never a full, unrequested dump.
8. Validate, authorize, deduplicate, and store accepted records
   ([Core §5](/spec/core/)), and emit `ACK`.
9. Exchange `DONE` or a `continuationToken` to end or resume the bounded
   session.

## 2. The Transfer Envelope

The **record** itself (`record.schema.json`) is immutable and carries only
the issuer's own signature ([Core §3.1](/spec/core/)). Relay/hop metadata (which
peer forwarded it, how many hops it has traveled, an optional hop budget)
MUST travel in a separate, per-hop **transfer envelope** that wraps the
record inside a `RECORDS` frame element. Appending such metadata to the
record itself is prohibited: it would grow at every hop, leak topology, and
invalidate the issuer's signature.

## 3. Errors and Unrecognized Content

A peer receiving a record with an unrecognized `kind` MUST ignore that
record for its own processing but MUST NOT terminate the session or treat
receipt as an error; it MAY still store and forward the record verbatim if
its `scope`/selector logic decides to. A peer that cannot parse a frame at
all, or that receives content violating [Core §3.6](/spec/core/) (free-form
content in a v1 kind), MUST respond with `ERROR` using a normalized error
class from `error.schema.json` — never by forwarding raw parser exceptions
or stack traces onto the wire.

## 4. Efficient Anti-Entropy

Records are partitioned by a stable key:

```
tenant / domain / record-kind / subject-namespace
```

Each partition summary carries a root digest, a generation counter, and a
record count:

```json
{
  "partition": "bank-a/finance/route.sli.v1/reconciliation",
  "rootHash": "sha256:...",
  "generation": 712,
  "recordCount": 84
}
```

Reconciliation uses a two-level scheme:

- If both sides' `rootHash` match, transfer nothing for that partition.
- For small partitions, exchange a compact inventory of `(recordId,
  version)` pairs and diff locally.
- For large partitions, use a Merkle tree or prefix-range reconciliation to
  localize the differing subset without transferring the whole partition.

Bloom filters MAY be used as an optimization hint (e.g. to cheaply guess
"probably don't have") but MUST NOT be the sole mechanism deciding whether
a record is transferred — false positives could silently withhold records
that are actually needed.

A session MUST be bounded by limits including: maximum bytes, maximum
records, maximum partitions, maximum processing time per session, maximum
outstanding sessions per peer, and per-issuer/per-tenant storage quotas. A
peer exceeding a limit MUST resume via `continuationToken` rather than
being silently truncated without signaling that more remains.

## 5. Binding Requirements

AGP is transport-independent: the record model, interest model, and `Sync`
state machine above are defined without reference to any host protocol. A
concrete **binding** maps `Sync` and its `GossipFrame`s onto a real
transport. Any conforming binding MUST provide:

- **Bidirectional, reliable, ordered delivery** of `GossipFrame`s within a
  session, sufficient to realize the state machine in §1.
- **Confidentiality and integrity** on the wire (e.g. TLS).
- **Peer authentication** — a mechanism by which each side establishes the
  authenticated identity of the other, so that a record's `issuer` and a
  session's authorization ([Security §3](/spec/security/)) can be evaluated. A
  binding MUST NOT define an authentication path weaker than the host
  protocol's own.
- **Capability advertisement and activation** — a way for an agent to
  advertise AGP support and negotiate core version, session limits, and
  digest algorithms before exchanging records.
- **Normalized errors** — the binding MUST surface protocol errors using
  `error.schema.json` classes and MUST NOT leak raw parser exceptions or
  stack traces onto the wire (§3).

A binding MUST NOT weaken the security requirements in
[Security §6](/spec/security/), and MUST NOT repurpose the host protocol's
ordinary task/message channels to carry routine gossip
([A2A Binding §6](/spec/binding-a2a/) states this concretely for A2A).

### 5.1 The A2A Binding

The normative binding defined by this specification is the
[**A2A Binding**](/spec/binding-a2a/): AGP as a standard A2A extension identified
by `https://agentgossipprotocol.org/a2a/extensions/gossip/v1`, with gRPC and
HTTP+JSON realizations of `Sync`. A2A-compatible agents SHOULD use it.

> **Other bindings (non-normative).** Because the core is
> transport-independent, the same signed records and the same acceptance,
> expiry, and anti-entropy rules could also be carried over a direct
> HTTP/gRPC binding, a message broker, or local IPC for non-A2A
> deployments. Such bindings are not specified here; any future binding
> MUST satisfy the requirements in this section and reuse the version 1
> schemas unchanged.
